# Financial P&L Dashboard

Interactive financial Profit & Loss reporting built with **Microsoft Power BI** and **synthetic sample data**.

The dashboard is designed to support financial performance reviews by combining profitability indicators, detailed P&L reporting, and budget variance analysis. All figures represent generated sample data, not real organizational performance.

## Business questions

- How do actual revenue, gross profit, operating expenses, and EBITDA compare with budget?
- Where are the largest favorable and unfavorable differences?
- Which departments or expense categories warrant closer review?
- What might explain the gap between actual and budgeted profitability?

## Report pages

| Page | Purpose |
| --- | --- |
| **Financial Overview** | Summarizes the principal financial indicators for a quick performance assessment. |
| **P&L Statement** | Presents income statement line items with actual, budget, variance, and variance percentage. |
| **Variance Analysis** | Explores actual-versus-budget differences with emphasis on the drivers of expense and profitability variance. |

Read the [page-by-page breakdown](docs/report-pages.md) for example business questions addressed by each page.

## Financial metrics

- Revenue
- Gross profit
- Operating expenses (OPEX)
- EBITDA
- Actual versus budget variance (absolute and percentage)

Monetary measures are displayed in **USD**.

## Tools and techniques

- **Power BI Desktop** for visual report development
- **DAX** for calculations, KPIs, and budget comparisons
- **Data modelling** to structure reporting data
- **Report design** for financial statements, KPI cards, navigation, and analytical visuals

## Data

The report uses generated synthetic data. Values, trends, and conclusions are illustrative and should not be interpreted as actual business results.

## Project files and documentation

| Path | Contents |
| --- | --- |
| `docs/report-pages.md` | Purpose and business questions for each page |
| `docs/data-model.md` | Data-model documentation checklist, pending inspection of the finished project |
| `dax/measures.md` | DAX documentation template, pending confirmation of the exact measures |
| `screenshots/` | Planned screenshots of the completed report pages |
| `powerbi/` | Planned Power BI report files, after checking metadata and data-sharing safety |

**Publication note:** The interactive report is already published externally. A direct report link is intentionally omitted until the published report's metadata has been checked for identifying information.
