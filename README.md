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
   Subcategory × Location):

   | Tile | Field | Required |
   |------|-------|----------|
   | **Category** | Parent line item dimension (Revenue, SG&A, Total Cost of Sales, …) | Yes |
   | **Subcategory** | Child dimension option A | At least one child dim for drill |
   | **Location** | Child dimension option B | At least one child dim for drill |
   | **Polarity** | Dimension tagging lines revenue-type vs expense-type (or true/false) | No — defaults to higher-is-better |
   | **Actual** | Current-period actual dollars (measure) | Yes |
   | **Budget** | Current-period budget (measure) — enables Budget, Var $, Var % | No |
   | **Prior Year** | Prior-year value (measure) — enables PY, PY Var % | No |
   | **YTD Actual** | Year-to-date actual (measure) — enables YTD Act | No |
   | **YTD Budget** | Year-to-date budget (measure) — with YTD Actual enables YTD Bdgt, YTD Var $, YTD Var | No |
   | **Sort Order** | Numeric measure controlling category order (otherwise incoming row order is kept) | No |

4. Click the gear icon in the header to configure: aggregation, default drill dimension and
   labels, higher-is-better default and the Polarity value that means expense, number format
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
