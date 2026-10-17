# Financial P&L Dashboard

An interactive **Power BI financial reporting project** built with synthetic sample data. The dashboard explores revenue, profitability, and performance against budget through three connected report pages.

All figures are illustrative and do not represent the financial results of a real organization. Financial amounts are displayed in **USD**.

## Business objectives

- Monitor Revenue, Gross Profit, EBITDA, and Net Profit against budget and the previous year.
- Compare monthly revenue and EBITDA performance with budget.
- Understand how costs and expenses contribute to Net Profit.
- Review detailed P&L lines and investigate financial and departmental budget variances.

## Report pages

| Navigation tab | Page heading | Focus |
| --- | --- | --- |
| **Overview** | Financial Performance Overview | Financial KPI cards, monthly Revenue and EBITDA comparisons, a revenue-to-net-profit waterfall, and profitability margins. |
| **Detailed P&L** | Profit & Loss Statement | A detailed statement comparing Actual, Budget, Variance, Variance %, Prior Year, and YoY % across financial lines and subcategories. |
| **Variance Analysis** | Budget Variance Analysis | Revenue variance, OPEX over budget, and EBITDA variance KPIs; an EBITDA variance waterfall; unfavorable account variances; department expense comparison and detail. |

See [Report pages and business questions](docs/report-pages.md) for a page-by-page explanation.

## Report screenshots

### 1. Overview — Financial Performance Overview

![Financial Performance Overview](screenshots/Financial%20P%26L%20-%20Overview.png)

Executive view of financial KPIs, monthly Revenue and EBITDA comparisons, the profit waterfall, and profitability margins.

### 2. Detailed P&L — Profit & Loss Statement

![Profit & Loss Statement](screenshots/Financial%20P%26L%20-%20Detailed%20P%26L.png)

Detailed Actual, Budget, Variance, Variance %, Prior Year, and YoY % comparisons across P&L categories.

### 3. Variance Analysis — Budget Variance Analysis

![Budget Variance Analysis](screenshots/Financial%20P%26L%20-%20Variance%20Analysis.png)

Revenue, OPEX, and EBITDA variance KPIs, EBITDA shortfall drivers, unfavorable account variances, and department-level expenses.

## Key financial metrics

- Revenue, Gross Profit, EBITDA, and Net Profit
- Cost of Sales and Operating Expenses (OPEX)
- Gross Margin, EBITDA Margin, and Net Margin
- Actual vs. Budget variance and variance percentage
- Prior Year and year-over-year (YoY) comparison

## Tools and techniques

- **Power BI Desktop** — interactive report development
- **DAX** — financial measures and variance calculations
- **Data modelling** — structuring data for financial analysis
- **Report design** — KPI cards, financial statement matrix, waterfall and comparison charts, navigation buttons, and Year/Month slicers

## Technical documentation

- [Report pages and business questions](docs/report-pages.md)
- [Selected DAX measures and calculation logic](docs/dax-measures.md)

## Dataset

The dashboard uses **generated synthetic data** for demonstration purposes. Its financial figures and trends are fictional.
