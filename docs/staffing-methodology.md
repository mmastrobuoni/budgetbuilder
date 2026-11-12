> Publishing note: CSV tables are published here. The complete SQLite/evidence package has been prepared separately; the website’s existing financial database ZIP has not yet been replaced. Rebuild instructions below apply to that complete package.

# Staffing coverage and definitions

Added November 12, 2026. The database contains **514 source observations**: 390 counts from 195 departmental PDF schedules and 124 counts from the FY2027 digital City Personnel Summary. These are overlapping versions, not 514 jobs. Reported fiscal years span FY2020–FY2027; FY2020 comes from comparative columns in the FY2021 book.

## Positions are not necessarily FTEs

The PDF schedules primarily report positions, including part-time employees, interns, elected officials, grant-funded positions, and positions budgeted at $1. No comparable full-time-equivalent series has been verified. `reported_count` preserves the printed count, `unit` describes what is known, and **`fte` remains NULL**. Counts were not estimated by dividing salaries or assuming part-time hours.

School staffing remains a coverage gap. Missing departments/years are not zero. Enterprise schedules are flagged separately. Other schedules can include non-General-Fund funding without enough information to allocate positions. Combined DPW schedules overlap component departments; custodians can also appear in multiple schedules. **Do not sum these records into a citywide or General Fund FTE total.**

## Sources and reproducibility

`data/staffing/pdf-transcription.csv` retains printed department totals and notes. `evidence/` contains the corresponding table images (and FY2021 page images/text). `sources.json` records book URLs, PDF hashes, and coverage. Page numbers are one-based PDF pages, not printed footer numbering. `source_total_label` retains the original total caption. The digital summary's raw panel is retained separately in `city-summary-source.json`; its unit is unspecified.

All extracted table images were visually reviewed. Printed totals are preserved even where detail rows disagree, with explanatory notes. This is departmental schedule-total extraction, not a complete position-by-position roster transcription.

## Query tables and views

| Object | Purpose |
|---|---|
| `staffing_observations` | All source versions, evidence references, funding/scope flags and notes |
| `staffing_department_history` | One selected PDF observation per department/year |
| `staffing_source_conflicts` | Department/years with differing source counts; differences may reflect scope or versions |

History prefers the current column in that fiscal year's own book, then the latest available comparative schedule. It does not establish adopted staffing. Stale, contradictory or missing year headings are retained in observations but excluded from history through explicit `column-overrides.json` rules. One FY2026 Health heading is inferred from its change column and flagged. Combined DPW totals and the digital summary are excluded from this selected history.

There are 229 selected department/year records and 45 department/year groups with differing source values. For example, the FY2027 Library schedule reports 41 positions while the digital summary reports 30. Library explicitly warns that its part-time counts are not FTEs. Neither count is silently substituted for the other.

```sql
SELECT fiscal_year, comparison_department, reported_count, unit,
       version_label, source_book_year, pdf_page, notes
FROM staffing_department_history
ORDER BY comparison_department, fiscal_year;

SELECT * FROM staffing_observations
WHERE comparison_department='Library' AND fiscal_year=2027;
```

CSV and JSON versions of observations and selected history are provided beside the evidence. Joining staffing to financial facts requires aggregation first: joining multiple source versions to dollar rows will multiply spending amounts. Department label matching alone does not establish identical funding or organizational scope.

## Rebuild

In this repository, run `python3 scripts/build_staffing.py` to regenerate exports and update the downloadable archive. In the extracted archive, `python3 database/build_database.py` rebuilds financial data and reapplies staffing. Alternatively, run `python3 scripts/build_staffing.py --database database/medford_budget.sqlite` there. Python standard library only. Staffing augmentation verifies that every existing financial table remains unchanged and runs SQLite integrity checks.
