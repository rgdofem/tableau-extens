# MOR KPI Card

A Tableau **viz extension** for the MOR (Monthly Operating Review) report — designed to sit
next to the P&L Drill Table. One card that **switches between many KPIs**: pick a category,
pick a KPI, and see the current-month value, a configurable comparison, a rising/falling/flat
indicator, and an actual-vs-budget/prior-year trend with an optional month-end forecast.

**Hosted URL:** https://rgdofem.github.io/tableau-extens/mor-kpi/index.html

## Encoding tiles

| Tile | Type | Max | Purpose |
|---|---|---|---|
| **Date** | Temporal dimension | 1 | Required. Month grain. Drives the current/prior/3-month-average windows and the trend x-axis. |
| **Measures** | Continuous measure | 16 | Every KPI measure (Admits, Terms, Headcount, …) plus any budget and prior-year measures. Which measures are KPIs, how they group, and how each compares are all set in the settings dialog. |

The worksheet's own filters/context scope the data, so no dimension tile is needed — every row
is aggregated to monthly per-measure totals (a plain sum within each month; with one row per
month that's the value itself).

## What it renders

- A **category pill row** and a **KPI pill row** at the top (each hidden when there's only one).
  Selection is session state — it resets when you change settings.
- The active KPI's **current-month value**, its label, a **comparison** line, and a
  **▲ / ▼ / ▬** glyph plus a percentage bubble (favorable/unfavorable colored, flat inside the
  configurable band).
- A **trend line** of the actual monthly series, optionally overlaid with a budget area or a
  prior-year line, and — when forecast is on — a **dashed projection** to the month-end forecast.

## Per-KPI configuration (gear → KPIs)

Each KPI card in the dialog sets:

- **Label / Category** — the category groups KPIs into the top selector.
- **KPI type** — *Single measure* or *Ratio*.
  - *Single measure*: pick an **Actual measure** and an **Aggregation** (how it reduces within
    a month): **Sum** (additive totals like Admits — forecast uses a run-rate), **Average of
    days** (point-in-time like ADC — forecast flat), **Max / one value per month** (a monthly
    level repeated across days), or **Latest day**.
  - *Ratio*: **numerator ÷ denominator**, each measure with its **own** monthly reduction, plus
    an optional **× 2nd denominator** factor and a **× multiplier** (e.g. 100 for a percentage).
    This is how you build RPPD (Revenue `Max` ÷ Census `Sum`), occupancy, cost-per-day, etc. —
    it solves the "different parts need different reductions" problem a single measure can't.
- **Compare to** — Prior month, Last 3-month average, Budget, or a Specific prior month
  (with a months-back offset). Ratios compare against the ratio at that period.
- **Period lag** — for measures that trail the report month (e.g. revenue only through last
  month): *Report month* (default), *1–6 months back*, or *Auto* (the latest month this KPI
  actually has data for). A lagged KPI shows an "As of <Mon Year>" note and skips the forecast.
- **Budget / Prior-year measures** — optional; used for the budget comparison and the trend overlay.
- **Budget is a monthly level (don't sum)** — on by default. A budget stored once per month but
  repeated across daily rows is taken as one value per month instead of summed (fixes a budget
  showing N× too big). Turn off only if the budget is a genuine daily target.
- **Trend overlay** — budget area, prior-year line, or actual only.
- **Up is good** — cardinality for favorable/unfavorable coloring.
- **Month-end forecast** — project the current partial month to month end (single-measure Sum
  KPIs only). When on, the variance is measured off the **forecast**, not the raw partial-month
  actual (a partial month always looks low against a full prior month — this makes it fair).
- **Value format** — prefix / suffix / scale (K/M) / decimals, per KPI (a count and a rate
  format very differently).

On first use, if no KPIs are configured the card **auto-seeds** one KPI per actual-looking
measure, pairing same-named budget/prior-year measures automatically.

## The month-end forecast

The data warehouse is always **one day behind**, so the forecast treats **yesterday** as the
last day with data. For a sum KPI the projection is `MTD ÷ (days-elapsed ÷ days-in-month)`,
where days-elapsed is yesterday's day-of-month (auto), or a manual override in
General → *Forecast: days elapsed override*. When yesterday falls in a prior month, the current
data month is treated as complete (no projection). Average/point-in-time KPIs project flat.

## Testing outside Tableau

Copy `index.html` into a scratch folder next to a mock `tableau.extensions.1.latest.min.js`
that stubs `initializeAsync`, `worksheetContent`, `settings`, and `ui` with a monthly
multi-measure dataset, then serve the folder with any static server. `?dialog=1` loads the
settings dialog.
