# DAX Measures & Calculation Logic

This document describes the DAX calculations underpinning the Financial P&L Dashboard, including profitability metrics, prior-year comparisons, favorable/unfavorable (F/U) variance logic, and the EBITDA variance bridge.

The examples are grouped by their role in the report: **Overview**, **Detailed P&L**, and **Variance Analysis**.

## Overview - Financial performance

**Gross Profit** - calculates the amount remaining after subtracting cost of goods sold (COGS) from revenue.

```dax
Gross Profit = [Revenue] - [COGS]
```

**EBITDA** - calculates earnings before interest, taxes, depreciation, and amortization by subtracting operating expenses (OPEX) from gross profit. Depreciation, interest, and tax are accounted for separately in the report's profit calculations.

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

## Detailed P&L - Financial statement logic

**PL Variance F-U** - calculates the actual-versus-budget difference and applies the direction defined in `PL_Layout[VarianceSign]`. This allows positive results to indicate favorable performance and negative results to indicate unfavorable performance across revenue, profit, and expense lines. The measure returns blank when either input is missing.

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

## Variance Analysis - Drivers of budget performance

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
