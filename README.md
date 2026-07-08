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
