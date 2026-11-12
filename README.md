# Medford Budget Explorer / Budget Builder

General Fund revenue, spending, and department comparisons for FY2020–FY2027. This replaces the earlier Budget Builder prototype.

## Run

Open `index.html` in a browser, or run `python3 -m http.server 8000` and visit http://localhost:8000. No build or dependencies are required. Open `index.html#flows` to start at the Sankey.

## Explore

- Treemap with category details.
- Expandable Sankey: spending function → department → prime account; revenue category → source detail. Click bands, labels, or the explicit Expand buttons. Collapse a branch or all branches. Both sides share one dollar scale.
- Department trends, year comparisons, searchable tables, and CSV exports.

## Files

`app.js` contains interactions and chart rendering; `style.css` contains styles; `data.js` contains the additive records and source metadata. `data/Medford-Budget-Database.zip` includes SQLite, JSON exports, raw inputs, and rebuilding instructions. See [data methodology](docs/data-methodology.md).

## Data limits

2,344 additive General Fund facts; 30 available revenue/expense totals reconcile. FY20–FY21 budget revenue is missing. FY26 actuals may be partial; FY27 actuals are unavailable. Published budgets have unverified adoption status. Department naming comparisons do not establish unchanged responsibilities. Revenue minus expense is not a verified operating balance.

## Hosting

This is a static site ready for GitHub Pages from `main` / root if enabled in repository settings. Repository storage does not itself confirm a deployment. No ChatGPT Sites credentials or hosting configuration are needed. The existing private ChatGPT Site is a separate deployment.

## Next phase

Build-and-balance-your-own-budget simulation is planned, not implemented here.

Previous repository contents are retained on `backup/pre-medford-explorer-2026-11-12` and in Git history.
