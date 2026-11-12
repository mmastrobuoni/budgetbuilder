# Medford budget database: FY2020–FY2027

Built November 12, 2026. Open `medford_budget.sqlite` with SQLite, DB Browser for SQLite, or an application SQLite driver. JSON exports support a future web explorer without a database server.

## What is included

- 26,792 source observations retained unchanged in `raw_statistics`.
- 2,552 non-overlapping category facts across available leaf funds, including 2,344 General Fund facts.
- Revenue, expense, fund, original department hierarchy, comparison department, personnel/ordinary expense classification, fiscal year, budget/actual version, and source row ID.
- All eight budget source references; six digital-book definitions and their historical series selections.
- Original department page titles and source URLs by book year in `department_pages`.
- 55 General Fund comparison departments/categories with nonzero observations. These include transfers, insurance, debt, and other citywide accounts, not just organizational departments.

This is a category-level financial database. It does not yet transcribe every staffing roster, capital project, grant schedule, or account printed in the PDFs. Historical actuals come from the publicly available digital-book dataset as collected, not frozen annual ledger snapshots. PDF-only FY20–FY21 proposed revenue schedules have not been transcribed; the missing digital revenue budgets remain explicitly missing.

## Ready-to-query views

| View | Purpose |
|---|---|
| `general_fund_detail` | Additive revenue and spending records; dollars and original source IDs |
| `general_fund_totals` | Annual revenue and expense totals, separately by budget/actual |
| `general_fund_revenue` | Revenue group and detailed source over time |
| `department_history` | Department by personnel, ordinary expenses, and other classifications |
| `department_totals` | Comparable department labels by fiscal year and measure |
| `department_year_over_year` | Dollar/percent changes versus the immediately preceding year |
| `budget_to_actual` | Budget and actual by department; variance suppressed for incomplete actuals |
| `department_presence` | First/last nonzero observation and original names, not legal creation/abolition dates |

## Validation and important limits

The additive facts use the root financial cube, leaf fund/department/object categories, and valid dimension IDs. This avoids counting category parents, alternative dimension summaries, and unclassified duplicate totals. Duplicate dimensional keys are rejected. Raw source records remain available for review.

All **30 General Fund revenue/expense source totals** reconcile to their component records within two cents. This validates extraction and aggregation, not audited accuracy or legal adoption. FY20–FY21 zero source budget revenue totals also reconcile arithmetically but are explicitly treated as **missing revenue budgets**, not real zero revenue. Integer cents are used for aggregation; original decimal amounts are retained because some source budgets contain fractions of a cent.

- **FY20–FY21 budget revenues:** absent from the usable digital series. Analytical revenue views return NULL. Expense budgets and actuals are available.
- **FY26 actuals:** flagged `partial_or_unverified`; excluded from default year-over-year comparisons and final variance calculations. No cutoff date has been verified.
- **FY27 actuals:** absent, not zero.
- **Budget versions:** retained as published `Budgeted`, with adoption status unverified. FY22–FY24 PDF covers say Proposed Version despite online titles saying Final. Later books may restate historical budgets.
- **Actuals:** historical budget-book figures, not independently audited here.
- **Gross presentation:** General Fund revenue includes gross state aid; expenses include state/county assessments and transfers. Do not mix these totals with net-state-aid appropriation summaries. Revenue less expense is not automatically a structural operating surplus/deficit.
- **Department comparisons:** explicit spelling/name normalizations combine Executive/Mayor, Elections variants, School/Education, Cable/PEG, and similar labels. Original names and category IDs remain available. These mappings are analytical choices, not proof of unchanged responsibilities.
- **Reorganizations:** Building Maintenance and Facilities remain separate, as do PDS, Planning, and Energy and Environment. Book page mappings allow investigation of combined department presentations. Missing records are not automatically filled with zeros.
- **Transfers:** kept separate from department services. FY25 transfers increase materially; do not interpret their growth as recurring service investment without tracing their purpose.

The latest source hierarchy is used for category ancestry. `department_pages` preserves book-specific presentation, but historical chart-of-accounts snapshots are not available. A renamed category may therefore carry today's label into historical records.

## Example queries

```sql
-- General Fund budget and actual totals
SELECT * FROM general_fund_totals ORDER BY fiscal_year, measure, flow;

-- Where revenue came from
SELECT fiscal_year, measure, revenue_group, SUM(amount) AS dollars
FROM general_fund_revenue
GROUP BY fiscal_year, measure, revenue_group;

-- Department investment changes between FY20 and FY25 actuals
SELECT a.comparison_department, b.amount AS fy20, a.amount AS fy25,
       a.amount-b.amount AS change_dollars
FROM department_totals a JOIN department_totals b
  ON a.comparison_department=b.comparison_department AND a.measure=b.measure
WHERE a.fiscal_year=2025 AND b.fiscal_year=2020 AND a.measure='actual'
ORDER BY change_dollars DESC;

-- Inspect source category and the page that includes it in a given book
SELECT f.*, p.page_title, p.source_url
FROM general_fund_detail f
JOIN department_pages p ON p.department_category_id=f.department_category_id
WHERE f.fiscal_year=2025 AND f.measure='actual' AND p.book_year=2027;

-- Audit controls and known issues
SELECT * FROM reconciliation WHERE fund_id=1088668;
SELECT * FROM data_issues;
```

For treemaps and Sankeys, filter to a single year and measure, then sum **facts only** along fund → original group/comparison department → expense category. Never add `raw_statistics` or parent totals to detail. Revenue-source → General Fund and General Fund → expense-department flows are useful; do not imply that unrestricted revenue sources fund particular departments. Show any difference between revenue and expenses explicitly.

For trends, default to FY20–FY25 actuals or separately labeled budget-to-budget comparisons. A zero prior-year denominator produces NULL percent change.

## Rebuild

The archive includes `database/build_database.py`, `collection/published-book-statistics.json`, `collection/source-inventory.json`, and required `raw` metadata. From the extracted `Medford-Budget-Database` directory, run:

```sh
python3 database/build_database.py
```

Uses Python's standard library only. Rebuilding replaces the generated SQLite and JSON outputs in that directory. Raw inputs are unchanged. PDF references are in `collection/source-inventory.json`; original PDFs are not duplicated in this database package.
