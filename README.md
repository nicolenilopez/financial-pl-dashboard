# Financial P&L Dashboard

An interactive **Power BI financial dashboard** for monitoring profitability, comparing actual performance against budget and prior-year results, and identifying the key drivers of financial variances.

Built using **synthetic financial data** for portfolio demonstration. All results are illustrative, not those of a real organization. Financial amounts are reported in **USD**.

## Dashboard preview

![Financial Performance Overview](screenshots/Financial%20P%26L%20-%20Overview.png)

The **Overview** brings together Revenue, Gross Profit, EBITDA, and Net Profit KPIs, monthly Actual vs. Budget comparisons, a revenue-to-net-profit waterfall, and profitability margins.

## Business objectives

- Monitor Revenue, Gross Profit, EBITDA, and Net Profit against budget and the previous year.
- Analyze monthly Revenue and EBITDA performance against budget.
- Understand how costs and expenses affect profitability.
- Investigate financial statement variances by account and operating expenses by department.

## Report pages

| Page | What it helps analyze |
| --- | --- |
| **Overview** | Financial performance, monthly trends, profit drivers, and margins. |
| **Detailed P&L** | Actual, Budget, Variance, Variance %, Prior Year, and YoY % across financial statement lines. |
| **Variance Analysis** | EBITDA variance drivers, unfavorable account impacts, and departmental operating expenses. |

### Detailed P&L — Profit & Loss Statement

![Profit & Loss Statement](screenshots/Financial%20P%26L%20-%20Detailed%20P%26L.png)

A structured financial statement with summary and detail lines, budget comparisons, and favorable/unfavorable variance formatting.

### Variance Analysis — Budget Variance Analysis

![Budget Variance Analysis](screenshots/Financial%20P%26L%20-%20Variance%20Analysis.png)

A closer look at Revenue, OPEX, and EBITDA deviations, with an EBITDA bridge, account-level impacts, and department expense comparisons.

For the purpose and business questions behind each page, see [Report pages and business questions](docs/report-pages.md).

## Technical implementation

- **Power BI Desktop:** Three-page report with navigation tabs, Year and Month slicers, KPI cards, a financial statement matrix, and waterfall and comparison charts.
- **Data modeling:** Separate actuals (`Fact_GL`) and monthly budgets (`Fact_Budget`) connected to shared Date, Account, and Department dimensions.
- **DAX:** Profitability and margin measures, prior-year comparisons, budget variances, and signed EBITDA variance impacts.
- **Custom financial layouts:** Disconnected layout tables organize P&L lines, profitability margins, and waterfall steps. The P&L layout controls display order and favorable/unfavorable variance direction.

## Key financial metrics

Revenue · Gross Profit · EBITDA · Net Profit · Gross Margin % · EBITDA Margin % · Net Margin % · Actual vs. Budget · Prior Year · YoY %

## Technical documentation

- [Data model and relationships](docs/data-model.md)
- [Selected DAX measures and calculation logic](docs/dax-measures.md)
- [Report pages and business questions](docs/report-pages.md)

## Data and scope

The dashboard uses **generated synthetic financial data**. Figures and trends are fictional and intended solely to demonstrate Power BI financial reporting and analysis. Budget amounts are recorded at a monthly level, so the report is designed for Year and Month filtering.
