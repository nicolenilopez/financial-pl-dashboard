# DAX Measures & Calculation Logic

This document highlights selected DAX measures used across the Financial P&L Dashboard's three report pages: **Overview**, **Detailed P&L**, and **Variance Analysis**. The examples cover financial KPIs, time intelligence, custom statement calculations, budget variances, and dynamic formatting.

## Overview — Financial performance

The Overview page uses base financial calculations, profitability ratios, and time intelligence to summarize business performance.

### Gross Profit

Calculates revenue minus cost of goods sold (COGS).

```dax
Gross Profit = [Revenue] - [COGS]
```

### EBITDA

Calculates earnings before interest, taxes, depreciation, and amortization by subtracting operating expenses (OPEX) from gross profit.

```dax
EBITDA = [Gross Profit] - [OPEX]
```

### Gross Margin %

Expresses gross profit as a percentage of revenue. `DIVIDE` handles zero or blank denominators by returning a blank result.

```dax
Gross Margin % = DIVIDE([Gross Profit], [Revenue])
```

### Revenue PY

Evaluates revenue for the corresponding period in the previous year using the shared date dimension.

```dax
Revenue PY =
CALCULATE(
    [Revenue],
    SAMEPERIODLASTYEAR(Dim_Date[Date])
)
```

### Margin Actual

Uses the disconnected `Margin_Layout` table to dynamically return the appropriate profitability ratio for each row of the margin comparison visual.

```dax
Margin Actual =
SWITCH(
    SELECTEDVALUE(Margin_Layout[Margin]),
    "Gross margin", [Gross Margin %],
    "EBITDA margin", [EBITDA Margin %],
    "Net margin", [Net Margin %]
)
```

## Detailed P&L — Financial statement logic

The Detailed P&L page uses a disconnected `PL_Layout` table to control statement rows independently of the underlying transaction tables.

### PL Actual

Dynamically returns the actual financial value for each P&L row. `SWITCH` maps statement lines to existing measures, while `CALCULATE` handles specific expense groups and `TREATAS` applies individual account names as filters.

```dax
PL Actual =
VAR SelectedLine =
    SELECTEDVALUE(PL_Layout[Line])
RETURN
    IF(
        ISBLANK(SelectedLine),
        BLANK(),
        SWITCH(
            SelectedLine,
            "Revenue", [Revenue],
            "Cost of Sales", [COGS],
            "Gross Profit", [Gross Profit],
            "Operating Expenses", [OPEX],
            "Personnel",
                CALCULATE(
                    [Actual Amount],
                    Dim_Account[Category] = "OPEX",
                    Dim_Account[PLGroup] = "Personnel"
                ),
            "EBITDA", [EBITDA],
            "Depreciation", [Depreciation],
            "Operating Profit",
                [EBITDA] - [Depreciation],
            "Interest Expense", [Interest Expense],
            "Profit Before Tax",
                [EBITDA] - [Depreciation] - [Interest Expense],
            "Income Tax", [Income Tax],
            "Net Profit", [Net Profit],
            CALCULATE(
                [Actual Amount],
                TREATAS(
                    {SelectedLine},
                    Dim_Account[AccountName]
                )
            )
        )
    )
```

### PL Budget

Uses the same statement layout to return budget values. Its structure mirrors `PL Actual`, allowing both measures to be compared on the same financial statement rows.

```dax
PL Budget =
VAR SelectedLine =
    SELECTEDVALUE(PL_Layout[Line])
RETURN
    IF(
        ISBLANK(SelectedLine),
        BLANK(),
        SWITCH(
            SelectedLine,
            "Revenue", [Revenue Budget],
            "Cost of Sales", [Budget COGS],
            "Gross Profit", [Budget Gross Profit],
            "Operating Expenses", [Budget OPEX],
            "Personnel",
                CALCULATE(
                    [Budget Amount],
                    Dim_Account[Category] = "OPEX",
                    Dim_Account[PLGroup] = "Personnel"
                ),
            "EBITDA", [Budget EBITDA],
            "Depreciation", [Budget Depreciation],
            "Operating Profit",
                [Budget EBITDA] - [Budget Depreciation],
            "Interest Expense", [Budget Interest Expense],
            "Profit Before Tax",
                [Budget EBITDA]
                    - [Budget Depreciation]
                    - [Budget Interest Expense],
            "Income Tax", [Budget Income Tax],
            "Net Profit", [Budget Net Profit],
            CALCULATE(
                [Budget Amount],
                TREATAS(
                    {SelectedLine},
                    Dim_Account[AccountName]
                )
            )
        )
    )
```

### PL Variance F-U

Calculates the difference between actual and budget, adjusting its sign according to `PL_Layout[VarianceSign]`. This allows positive results to indicate favorable performance and negative results to indicate unfavorable performance across revenue, profit, and expense lines. The measure returns blank when either input is missing and requires a single variance direction in the current filter context.

```dax
PL Variance F-U =
VAR ActualValue = [PL Actual]
VAR BudgetValue = [PL Budget]
VAR Direction = SELECTEDVALUE(PL_Layout[VarianceSign])
RETURN
    IF(
        ISBLANK(ActualValue) || ISBLANK(BudgetValue),
        BLANK(),
        (ActualValue - BudgetValue) * Direction
    )
```

### PL Variance F-U %

Expresses the favorable/unfavorable variance relative to the absolute budget amount, preserving the variance's sign without reversing it when the budget is negative.

```dax
PL Variance F-U % =
DIVIDE([PL Variance F-U], ABS([PL Budget]))
```

### PL Prior Year

Evaluates each custom P&L statement line for the corresponding period in the previous year.

```dax
PL Prior Year =
CALCULATE(
    [PL Actual],
    SAMEPERIODLASTYEAR(Dim_Date[Date])
)
```

### PL YoY %

Calculates the year-over-year percentage change for each P&L line. The absolute prior-year denominator keeps the sign of the change independent of the prior-year amount's sign; a blank or zero prior-year value produces a blank result.

```dax
PL YoY % =
VAR PriorYearValue = [PL Prior Year]
RETURN
    IF(
        NOT ISBLANK(PriorYearValue),
        DIVIDE([PL Actual] - PriorYearValue, ABS(PriorYearValue))
    )
```

## Variance Analysis — Drivers of budget performance

The Variance Analysis page evaluates differences between actual and budget performance, including operating expense overruns and account-level impacts on EBITDA.

### OPEX Over Budget

Measures spending above or below budget. Positive values indicate an overrun; negative values indicate spending below budget.

```dax
OPEX Over Budget = [OPEX Actual] - [OPEX Budget]
```

### EBITDA Bridge Value

Returns the budgeted EBITDA starting value and the signed revenue, COGS, and OPEX differences used in the EBITDA variance waterfall. Revenue shortfalls and cost overruns reduce the bridge, while favorable movements increase it.

```dax
EBITDA Bridge Value =
SWITCH(
    SELECTEDVALUE(EBITDA_Bridge[Step]),
    "Budget", [EBITDA Budget],
    "Revenue", [Revenue Actual] - [Revenue Budget],
    "COGS", [COGS Budget] - [COGS Actual],
    "OPEX", -[OPEX Over Budget],
    BLANK()
)
```

### Account EBITDA Impact

Evaluates each account's contribution to EBITDA variance using category-specific sign conventions: actual minus budget for revenue, and budget minus actual for COGS and OPEX. Negative results indicate an unfavorable impact on EBITDA.

```dax
Account EBITDA Impact =
VAR Category = SELECTEDVALUE(Dim_Account[Category])
RETURN
    SWITCH(
        Category,
        "Revenue", [Actual Amount] - [Budget Amount],
        "COGS", [Budget Amount] - [Actual Amount],
        "OPEX", [Budget Amount] - [Actual Amount],
        BLANK()
    )
```

### Unfavorable Account Impact

Isolates unfavorable account contributions by returning the absolute magnitude of negative EBITDA impacts and leaving other results blank.

```dax
Unfavorable Account Impact =
VAR Impact = [Account EBITDA Impact]
RETURN
    IF(
        Impact < 0,
        -Impact,
        BLANK()
    )
```

## Dynamic formatting

DAX measures also control visual styling in the Detailed P&L statement, allowing key financial lines to be distinguished from supporting detail rows.

### PL Row Background

Returns a background color for each P&L row, distinguishing key financial totals from supporting detail.

```dax
PL Row Background =
VAR CurrentLine =
    SELECTEDVALUE(PL_Layout[Line])
RETURN
    IF(
        CurrentLine IN {
            "Revenue",
            "Gross Profit",
            "EBITDA",
            "Operating Profit",
            "Profit Before Tax",
            "Net Profit"
        },
        "#151D28",
        "#202C3C"
    )
```

