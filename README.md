# Saudi Arabia vs GCC Economic Dashboard (World Bank API)

## Project Overview
This project packages a complete, production-ready Streamlit dashboard that compares Saudi Arabia with the rest of the GCC across three headline macro indicators: real GDP growth, inflation, and unemployment. The design goal is practical communication for non-technical stakeholders: one page, clear filtering, visible trend context, and automatic narrative insights that explain what changed and how Saudi Arabia currently sits relative to regional peers. The repository includes both the data engineering layer and the presentation layer, so the same project can be used for repeatable refreshes, executive walkthroughs, and cloud sharing.

## Data Source
The underlying data is sourced directly from the World Bank Open Data API (`api.worldbank.org`) and then persisted locally in `data/world_bank_gcc_raw.csv`. The dashboard tracks six ISO3 countries: `SAU`, `ARE`, `QAT`, `KWT`, `BHR`, and `OMN`. It pulls exactly three World Bank indicator codes so analytical scope remains clear and business-facing:

- `NY.GDP.MKTP.KD.ZG` for GDP growth (annual %)
- `FP.CPI.TOTL.ZG` for Inflation, consumer prices (annual %)
- `SL.UEM.TOTL.ZS` for Unemployment, total (% of total labor force)

The fetch pipeline supports 2000–2024 records and the dashboard defaults the interactive range to 2010 through the latest available year in the file.

## Steps & Methodology
The workflow starts with `scripts/fetch_data.py`, which uses `requests` with explicit pagination (`page`, `per_page`) for each country-indicator pair. The script is defensive by design: it validates HTTP responses, checks JSON structure, raises clear errors when payloads are malformed or empty, and writes a standardized CSV schema with the exact columns `country_code`, `country_name`, `indicator_code`, `indicator_name`, `year`, and `value`.

Transformation choices are intentionally light so stakeholders can trust traceability back to source. Values are kept at annual frequency, years are coerced to integers, and missing values are preserved rather than imputed. In charts, gaps are shown as gaps, which avoids implying continuity where reporting is incomplete. The dashboard logic then computes Saudi KPI cards from the filtered time window (latest value, latest year, year-over-year delta from the prior available point, and rolling five-year average), builds cross-country trend lines for each selected indicator, and creates a latest-snapshot comparison where each country contributes its most recent available value in range.

For analytical depth without complexity, the app also calculates a Saudi-only indicator correlation heatmap over the selected years and generates plain-language “Key insights” bullets. These bullets blend absolute movement (up/down vs prior point), relative context (position among selected peers), and self-benchmarking (distance from Saudi Arabia’s five-year average), making them suitable for policy or executive discussions.

Tool selection favors maintainability: `pandas` for tabular shaping, `plotly` for interactive visuals, and `streamlit` for fast delivery and easy hosting. The interface uses wide layout, concise labels, and stakeholder-friendly captions so users can interpret results without reading code.

## Dashboard Screenshots
Add screenshots to `/assets` and keep the filenames below so the README renders automatically on GitHub:

`assets/dashboard-overview.png`  
`assets/dashboard-trends.png`  
`assets/dashboard-insights.png`

When images are added, these Markdown references will display them:

![Dashboard Overview](assets/dashboard-overview.png)
![Dashboard Trends](assets/dashboard-trends.png)
![Dashboard Insights](assets/dashboard-insights.png)

## Design Screenshots (optional)
If you also want to document the visual design process, place optional mockups or layout iterations in `/assets`, for example `assets/design-wireframe.png` and `assets/design-final.png`, then reference them here:

![Design Wireframe](assets/design-wireframe.png)
![Design Final](assets/design-final.png)

## Key Insights
The dashboard is built to surface three recurring stakeholder-level narratives. First, Saudi Arabia’s latest reading for each indicator is immediately visible and anchored to the most recent year available. Second, recent movement is contextualized with a year-over-year delta so directional change is obvious at a glance. Third, peer comparison avoids stale snapshots by using each country’s latest available in-range observation, which is often the fairest approach when publication timing differs across national series. Together, these mechanics help answer practical questions such as whether Saudi inflation momentum is easing, whether unemployment has diverged from GCC peers, and whether growth performance is above or below Saudi Arabia’s own recent baseline.

## Live Dashboard Link placeholder
Deploy the app and replace this placeholder with your public URL:

`https://your-streamlit-cloud-url.streamlit.app`

## Assumptions & Limitations
This project assumes the World Bank API remains reachable and that indicator definitions are stable over time. The app does not interpolate missing years, does not seasonally adjust data, and does not normalize country-level structural differences beyond the indicator definitions already provided by the source. Year-over-year deltas use the prior available observation in range, which can span more than one calendar year if reporting gaps exist. Correlation results should be interpreted as directional context, not causal inference, especially when the selected period is short.

## How to run locally
From the repository root, create an environment, install pinned dependencies, refresh data, and launch Streamlit:

```bash
cd financial-dashboard-worldbank-gcc
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python scripts/fetch_data.py
streamlit run app.py
```

For Streamlit Cloud deployment, push this folder to GitHub, create a new Streamlit app at `share.streamlit.io`, point it to `app.py`, and keep `requirements.txt` at the project root so dependencies install automatically. After deployment, copy the generated URL into the “Live Dashboard Link placeholder” section above.
