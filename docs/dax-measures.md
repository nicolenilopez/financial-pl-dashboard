# DAX Measures & Calculation Logic

This document highlights selected DAX measures used in the Financial P&L Dashboard. The calculations support financial KPIs, prior-year comparisons, custom P&L statement rows, favorable/unfavorable (F/U) variances, and EBITDA variance analysis.

The examples are organized around the report's three pages: **Overview**, **Detailed P&L**, and **Variance Analysis**. Additional measures support conditional formatting and the presentation of financial results.

## Overview — Financial performance

The Overview page uses base financial calculations, profitability ratios, and time intelligence to summarize business performance.

**Gross Profit** - calculates the amount remaining after subtracting cost of goods sold (COGS) from revenue.

```dax
Gross Profit = [Revenue] - [COGS]
```

**EBITDA** - calculates earnings before interest, taxes, depreciation, and amortization by subtracting operating expenses (OPEX) from gross profit.

```dax
EBITDA = [Gross Profit] - [OPEX]
```

**Gross Margin %** - expresses gross profit as a percentage of revenue. `DIVIDE` handles zero or blank denominators by returning a blank result.

```dax
Gross Margin % = DIVIDE([Gross Profit], [Revenue])
```

**Revenue PY** - evaluates revenue for the corresponding period in the previous year using the report's date dimension. This measure supports year-over-year comparisons in the Overview page.

```dax
Revenue PY =
CALCULATE(
    [Revenue],
    SAMEPERIODLASTYEAR(Dim_Date[Date])
)
```

**Margin Actual** - uses the disconnected `Margin_Layout` table to dynamically return the appropriate profitability ratio for each row of the margin comparison visual.

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

**PL Actual** - dynamically returns the actual financial value for each P&L row. `SWITCH` maps statement lines to existing measures, while `CALCULATE` handles specific expense groups and `TREATAS` applies individual account names as filters.

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

**PL Budget** - uses the same statement layout to return budget values. Its structure mirrors `PL Actual`, allowing both measures to be compared on the same financial statement rows.

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

**PL Variance F-U** - calculates the actual vs. budget difference and applies the direction defined in `PL_Layout[VarianceSign]`. This allows positive results to indicate favorable performance and negative results to indicate unfavorable performance across revenue, profit, and expense lines. The measure returns blank when either input is missing.

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

**PL Variance F-U %** - expresses the favorable/unfavorable variance relative to the absolute budget amount, preserving the variance's sign without reversing it when the budget is negative.

```dax
PL Variance F-U % =
DIVIDE([PL Variance F-U], ABS([PL Budget]))
```

**PL Prior Year** - evaluates each custom P&L statement line for the corresponding period in the previous year.

```dax
PL Prior Year =
CALCULATE(
    [PL Actual],
    SAMEPERIODLASTYEAR(Dim_Date[Date])
)
```

**PL YoY %** - calculates the year-over-year percentage change for each P&L line. The absolute prior-year denominator keeps the sign of the change independent of the prior-year amount's sign; a blank or zero prior-year value produces a blank result.

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

**OPEX Over Budget** - measures spending above or below budget. Positive values indicate an overrun; negative values indicate spending below budget.

```dax
OPEX Over Budget = [OPEX Actual] - [OPEX Budget]
```

**EBITDA Bridge Value** - returns the budgeted EBITDA starting value and the signed revenue, COGS, and OPEX differences used in the EBITDA variance waterfall. Revenue shortfalls and cost overruns reduce the bridge, while favorable movements increase it.

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

**Account EBITDA Impact** - evaluates each account's contribution to EBITDA variance using category-specific sign conventions: actual minus budget for revenue, and budget minus actual for COGS and OPEX. Negative results indicate an unfavorable impact on EBITDA.

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

**Unfavourable Account Impact** - returns the magnitude of a negative account-level EBITDA impact, leaving zero and favorable impacts blank. This measure can be used to isolate unfavorable account contributions.

```dax
Unfavourable Account Impact =
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

**PL Row Background** - assigns a darker background to major financial statement lines, including Revenue, Gross Profit, EBITDA, Operating Profit, Profit Before Tax, and Net Profit.

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

This approach keeps the visual hierarchy of the P&L statement consistent with its underlying financial structure.
