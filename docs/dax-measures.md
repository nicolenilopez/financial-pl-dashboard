# DAX Measures & Calculation Logic

This document highlights selected DAX measures used in the Financial P&L Dashboard, demonstrating the calculation logic behind financial KPIs, profitability analysis, budget variances, and year-over-year comparisons.

The measures are organized by report functionality, covering the **Overview**, **Detailed P&L**, and **Variance Analysis** pages.

## Core financial calculations

**Gross Profit** — revenue after cost of goods sold.

```dax
Gross Profit = [Revenue] - [COGS]
```

**EBITDA** — gross profit less operating expenses.

```dax
EBITDA = [Gross Profit] - [OPEX]
```

**Gross Margin %** — share of revenue retained as gross profit.

```dax
Gross Margin % = DIVIDE([Gross Profit], [Revenue])
```

**Revenue PY** — revenue for the corresponding prior-year period.

```dax
Revenue PY =
CALCULATE(
    [Revenue],
    SAMEPERIODLASTYEAR(Dim_Date[Date])
)
```

## Detailed P&L

**PL Variance F-U** — normalizes actual-versus-budget differences using the configured favorable/unfavorable sign for each financial statement line.

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

**PL Variance F-U %** — compares the signed variance with the absolute budget value.

```dax
PL Variance F-U % =
DIVIDE([PL Variance F-U], ABS([PL Budget]))
```

**PL YoY %** — compares current values with prior-year figures, using the absolute prior-year amount as the denominator.

```dax
PL YoY % =
VAR PriorYearValue = [PL Prior Year]
RETURN
    IF(
        NOT ISBLANK(PriorYearValue),
        DIVIDE([PL Actual] - PriorYearValue, ABS(PriorYearValue))
    )
```

## Variance Analysis

**OPEX Over Budget** — positive results represent expenses exceeding budget.

```dax
OPEX Over Budget = [OPEX Actual] - [OPEX Budget]
```

**EBITDA Bridge Value** — supplies the components for the budget-to-actual EBITDA waterfall.

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

**Account EBITDA Impact** — evaluates revenue and expense lines with opposite variance signs so that negative values indicate unfavorable EBITDA impacts.

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
