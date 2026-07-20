# Super KPI Card

A configurable Tableau **viz extension** that renders a light-themed KPI card with a colored
status bar, driven entirely by worksheet data and settings — no build step, plain HTML/CSS/JS.

## Files

- `index.html` — the extension itself. Card view by default; append `?dialog=1` to load the
  settings dialog (same file, different view).
- `tableau.extensions.1.latest.min.js` — the Tableau Extensions API library, downloaded from the
  [tableau/extensions-api](https://github.com/tableau/extensions-api) repo's `lib/` folder and
  vendored locally (do **not** point at `tableau.github.io/extensions-api/lib/...` — that URL
  404s).
- `SuperKPICard.trex` — the extension manifest.

## Hosted URL

Once GitHub Pages is enabled (see below), the extension is served at:

```
https://rgdofem.github.io/tableau-extens/index.html
```

This is the exact URL baked into `SuperKPICard.trex`'s `<source-location>`. If you move
`index.html` into a subfolder, update both the folder path in that URL and re-download/verify
the `.trex` still resolves correctly.

## Enabling GitHub Pages

1. Push `index.html`, `tableau.extensions.1.latest.min.js`, and `SuperKPICard.trex` to the `main`
   branch of `rgdofem/tableau-extens`.
2. In the repo, go to **Settings → Pages**.
3. Under **Build and deployment**, set **Source** to "Deploy from a branch".
4. Set **Branch** to `main` and the folder to `/ (root)`. Save.
5. Wait a minute or two, then confirm `https://rgdofem.github.io/tableau-extens/index.html` loads
   in a browser (you should see "Add a measure to this card." since it's outside Tableau).

## Adding the extension to a worksheet

1. In Tableau Desktop (or Tableau Cloud/Server, 2024.2+), open a worksheet.
2. On the **Marks** card, click **Add Extension**.
3. Choose **My Extensions → Access Local Extensions** and point to your local copy of
   `SuperKPICard.trex` (or, once hosted, use whatever mechanism your Tableau site provides for
   installed/site-level extensions pointing at the manifest).
4. Drag your metric measure onto the **Value** tile in the extension's encoding shelf.
   Optionally drag a budget/target/prior-period measure onto **Comparison**, a secondary count
   onto **Context**, and a date field onto **Date**.
5. Click the gear icon that appears on hover in the top-right of the card to open the settings
   dialog and configure aggregation, comparison source, subtext mode, formatting, and colors.

### Tableau Cloud allow-listing

If you're on Tableau Cloud, a site admin must add `rgdofem.github.io` to the allowed extensions
list under **Site Settings → Extensions** before this (or any externally-hosted) extension will
load for users on that site.

### Version requirement

Viz extensions require **Tableau 2024.2 or later**. The manifest declares
`<min-api-version>1.11</min-api-version>`, which matches that minimum.

## Configuration reference

All settings are optional and have sensible defaults; open the gear icon to change them.

- **Aggregation** — how multiple summary-data rows for Value/Comparison/Context are combined:
  sum, average, min, max, count, or latest (latest uses the Date field to find the most recent
  row if present, otherwise the last row in table order).
- **Comparison source** — use the Comparison measure if one is on the shelf, or a fixed target
  number typed into settings. If you select "Fixed target number" but leave the Comparison
  measure encoding empty, the fixed number is always used (even `0`).
- **Higher is better** — sets polarity for the green/red status bar and triangle.
- **Subtext mode** — vs budget, vs target, vs prior period (with a raw-change/relative-%
  sub-toggle), context count (with a configurable noun), custom (a configurable number + symbol
  + trailing text), or none.
- **Value prefix / suffix / decimals / abbreviate** — controls both the main value and any
  absolute-difference figure shown in "vs budget" subtext, so they stay consistent with each
  other.
- **Title override / date caption prefix** — override the label text, and set the word used in
  the `(through <date>)` caption (only shown when a Date field is present).
- **Colors** — positive, negative, background, and text colors.

## A note on the acceptance-table percentages

The extension computes the relative percentage as `(value - comparison) / comparison * 100`,
verified to match exactly against the fully-worked example in the spec (MTD Census: `|34,069 -
35,188| / 35,188 = 3.2%`, diff `1,119`, red bar). A few of the other example rows in the spec's
table (YTD Census, 90 Day Retention, Counselor FTF) show a slightly different percentage than
this formula produces when fed the *rounded, displayed* value/comparison numbers from the table
— that's expected, since the real worksheet aggregates carry more precision than the rounded
figures shown in the table. Once wired to live data, the card computes off the true, unrounded
aggregated values, so this isn't a formula bug.

---

# P&L Drill Table

A second Tableau **viz extension**, living in the `pnl/` subfolder: an interactive financial
statement table with expandable categories, a switchable child dimension (Subcategory or
Location), derived variance columns, and favorability-colored variance bubbles. Plain HTML/CSS/JS
in a single `index.html`, no build step.

## Hosted URL

```
https://rgdofem.github.io/tableau-extens/pnl/index.html
```

This is the exact URL in `pnl/PnLDrillTable.trex`'s `<source-location>`. The `rgdofem.github.io`
domain is already allow-listed on the Tableau Cloud site from the Super KPI Card, so no new admin
step is needed.

## Files

- `pnl/index.html` — table view by default; `?dialog=1` loads the settings dialog.
- `pnl/tableau.extensions.1.latest.min.js` — local copy of the Extensions API library.
- `pnl/PnLDrillTable.trex` — the extension manifest.

## Adding the extension to a worksheet

1. In Tableau (2024.2+), open a worksheet, and on the **Marks** card click **Add Extension**.
2. Choose **Access Local Extensions** and pick `pnl/PnLDrillTable.trex`.
3. Drop fields on the encoding tiles (data must be at the child grain — Category ×
   Subcategory × Location). Tableau's manifest schema allows at most **4 custom encoding
   tiles**, so dimensions and measures share tiles:

   | Tile | Fields (max) | Required |
   |------|--------------|----------|
   | **Category** | Parent line item dimension (Revenue, SG&A, Total Cost of Sales, …) (1) | Yes |
   | **Drill To** | Child dimensions revealed on expand — e.g. Subcategory and/or Location (2). The header switcher toggles between them. | At least one for drill-down |
   | **Polarity** | Dimension tagging lines revenue-type vs expense-type, or true/false for higher-is-better (1) | No — defaults to higher-is-better |
   | **Measures** | Actual, Budget, Prior Year, YTD Actual, YTD Budget, Sort Order (6) | Actual only |

   Measure **roles are auto-detected from field names** (e.g. a field containing "budget"
   becomes Budget; "YTD" + "budget" becomes YTD Budget; "sort" becomes Sort Order; "PY",
   "prior", or "last year" becomes Prior Year). Only mapped roles produce columns: Budget
   enables Budget/Var $/Var %, Prior Year enables PY/PY Var %, YTD Actual + YTD Budget enable
   the YTD columns, and Sort Order controls category ordering (otherwise incoming row order
   is kept).

4. Click the gear icon in the header to configure: measure role assignments (override
   auto-detection per role), aggregation, default drill dimension and display labels,
   higher-is-better default and the Polarity value that means expense, number format
   (scale/prefix/suffix/decimals/separator), column visibility, dollar-variance-as-bubble,
   bubble colors, title, and default/persisted expand state.

## How it works

- The extension reads the worksheet's summary data at the child grain and performs the rollup
  itself: collapsed category rows sum the base measures across all children; every derived
  percentage is computed from the summed dollars after rollup, so percentages are correct at
  both levels. Total/subtotal lines (Gross Profit, Net Income, …) must already exist in the
  source data as their own category rows.
- Variances are displayed in **favorability** terms: green when favorable, red when
  unfavorable, where "favorable" respects each line's polarity (an expense under budget is
  green). Dollar variances show the favorable magnitude, with unfavorable amounts in
  accounting parentheses. The ▲/▼ triangle always reflects the raw direction of actual vs
  comparison, independent of color — so an expense line under budget shows a green ▼.
- The **Drill by** switcher re-groups the same rows by the other child dimension entirely
  client-side; switching collapses all categories.

---

# Census Facility Dashboard

A whole-tab Tableau **viz extension** in `census-facility/`: the facility census view in a single
worksheet. Plain HTML/CSS/JS in one `index.html`, no build step.

## Hosted URL

```
https://rgdofem.github.io/tableau-extens/census-facility/index.html
```

## What it renders

- **KPI row** — Day (latest date), MTD, and YTD cards. Each shows the census figure, a
  red/green budget variance line (`(966) ▼ below 35,028 budget` + `(3%) unfavorable`), and a
  **prior-year comparison line** (`vs PY 33,096 ▲ 2.9%`), also favorability-colored.
- **KPI row** — the value sits on the left and the budget/prior-year comparison lines are
  right-aligned beside it (rather than stacked underneath), so the cards fill their width
  instead of leaving the right half blank; a card with no comparison lines centers its value.
- **Breakdown panels** — one chart per dimension on the Breakdowns tile (Phase, ALOS bucket,
  Drug Type, Payor, Modality, …), snapshotted at the latest date. Per panel you choose chart
  type (horizontal bars / columns / pie / donut), order (↑/↓ arrows), grid width, Top N, sort,
  and how the tail beyond Top N is handled — **Rest**: collapse it into one "All others" item
  (which always sits at the bottom) showing its **Sum** or **Average**, or **Hide** it. Pies
  always sum the rest so the proportions stay meaningful.
- **Trend panel** — daily total actual line with a light budget area, or split into one line
  per member of a chosen dimension, capped at Top-N lines by current census so the all-sites
  view stays readable (the messy-spaghetti fix). Date window: all / YTD / trailing 12 / 24 months.

## Worksheet setup

Data must be at **Date × breakdown-dimension grain** (daily). Tiles (max 4 per the manifest
schema, so measures share one):

| Tile | Fields (max) | Required |
|------|--------------|----------|
| **Date** | The daily census date, exact day grain (1) | Yes |
| **Measures** | Census, Budget, Prior Year (4) | Census only |
| **Breakdowns** | Up to six dimensions to break census out by (6) | No |

Measure roles are auto-detected from names (budget/plan/target → Budget; PY/prior/last year →
Prior Year) and can be reassigned in the settings dialog.

### Budget grain (important when Budget is monthly-per-location)

Census is captured per patient per day, but budget is typically **one number per month per
location**, related to census on site + year + month. In a Tableau relationship that monthly
figure is *repeated* on every breakdown row for the day (Modality × Phase × Drug × Payor …), so
naively summing it multiplies the budget by the number of breakdown rows (e.g. a 35,028 budget
shows as ~200K). The **Budget grain** setting handles this:

- **De-duplicate** (default) — the budget is resolved at **month grain**: one value per
  (month × grain group), summed across groups, then applied to every day of that month. For a
  **single site**, leave "Budget varies by" empty → one value per month. For an **all-sites**
  view where Location is on the Breakdowns tile, check **Location** so each location contributes
  once and they sum across locations.
- **Add it up** — only if your budget is genuinely stored at the row grain (uncommon for census).

Resolving at month grain (rather than per day) is deliberate: the budget is a monthly figure, so
this keeps it **constant within a month** — the trend budget steps once a month instead of
wiggling daily, and MTD/YTD stay correct even when some days are missing rows (a partial day no
longer drags the average down). If the current-month budget still looks low, the budget is being
split across a dimension that isn't being summed — check that dimension under "Budget varies by".

Budget then uses the same MTD/YTD aggregation as census; with the default **ADC (average)** each
day carries the monthly target and the average over the window returns that target.

### Trend position and axis

The trend panel can sit **above the breakdown panels** (a middle layer between the KPI cards and
the breakdown grid) or **below** them — set it under Trend panel → Position. The x-axis label
granularity is set by Trend panel → **Axis labels**: Auto (year for multi-year spans, month
otherwise), Year, Quarter (`Q3 '26`), or Month (`Jul`, with a year marker each January). Month
and quarter labels are thinned to fit the panel width. The regional trend has the same Axis
labels control.

## MTD / YTD / prior-year windows

The extension computes the windows itself from the Date field: Day = latest date with data,
MTD = that date's month-to-date, YTD = year-to-date. MTD/YTD aggregation is configurable —
ADC (average of daily totals, the default), Sum, or Latest day. Budget aggregates the same way.
**Prior year**: if a Prior Year measure is mapped it is aggregated over the *same* window;
otherwise PY is derived from the Date history (same window shifted one year back) — so keep at
least last year's dates in the worksheet (a relative date filter of ~2 years works well, and
also keeps the summary-data volume down at fine dimensional grain).

---

# Census Regional Dashboard

The corporate/regional counterpart in `census-regional/`: same KPI engine plus a variance chart,
an actual-vs-budget trend, and a multi-measure site crosstab.

## Hosted URL

```
https://rgdofem.github.io/tableau-extens/census-regional/index.html
```

## What it renders

- **KPI row** — identical Day/MTD/YTD cards with budget variance and prior-year lines, using the
  same value-left / comparison-right layout as the facility cards.
- **Variance chart** — diverging red/green bars of actual − budget at the latest date, grouped
  by a Rows dimension (default the first, e.g. Region), sortable A→Z / worst-first / best-first.
- **Trend chart** — daily total census line over a light budget area, with the same date-window
  options as the facility trend.
- **Crosstab** — one row per dimension tuple (Region / Location / Program) at the latest date:
  Census, Budget, favorability-colored Variance (and optional Var %), then **every measure not
  assigned a role as an extra column** — payor mix %, ALOS, whatever you drop on the tile.
  Percent-looking fields format automatically (0–1 ratios are scaled to %); overrides per field
  in settings. Every column header is click-sortable and **renamable** (Crosstab → Column
  headers); the default sort and an optional totals row are configurable. The crosstab **scrolls
  internally** (sticky header) so the dashboard fits a fixed slide without the page scrolling.

### Fitting a slide (e.g. 1600×900)

The regional dashboard fills its Tableau zone as a fixed-height column: the KPI row, variance/
trend row, and crosstab stack, and the crosstab takes the remaining height and scrolls inside
itself (vertically for extra rows, horizontally for extra columns). Set your Tableau dashboard to
a fixed size (e.g. 1600×900) and the layout fits with no outer scrollbar. Keep the middle-row
height (Trend chart → Middle row height) modest so the crosstab keeps enough room.

## Worksheet setup

Data at **Date × site grain** (daily):

| Tile | Fields (max) | Required |
|------|--------------|----------|
| **Date** | The daily census date, exact day grain (1) | Yes |
| **Rows** | Crosstab row dimensions in display order, e.g. Region, Location, Program (3) | For chart/crosstab |
| **Measures** | Census, Budget, Prior Year + any extra crosstab measures (12) | Census only |

The prior-year note from the facility extension applies here too — map a PY measure or keep last
year's dates in the worksheet.

### Budget grain

Same coarse-budget handling as the facility extension, but it defaults to **Add it up** because
the natural regional grain (Region / Location) usually matches the budget's grain, so no fan-out
occurs. If you add a Rows dimension *finer* than the budget — e.g. **Program** under a
per-location budget — the budget is repeated across those rows and the totals/KPIs would double.
In that case switch **Budget grain** to **De-duplicate** and check the dimension(s) the budget is
unique by (e.g. Region + Location); the KPI cards, variance chart, and crosstab totals then take
one budget value per group. Per-row budget cells still show the location's figure on each finer
row, so keep the crosstab at the budget's grain if you want a clean per-row comparison.

## Testing either extension outside Tableau

Copy the extension's `index.html` into a scratch folder next to a mock file named
`tableau.extensions.1.latest.min.js` that stubs `initializeAsync`, `worksheetContent`,
`settings`, and `ui`, then serve the folder with any static server. Both dashboards render
fully from the mocked summary data; `?dialog=1` loads the settings dialog the same way.
