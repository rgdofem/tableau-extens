# Sparkline KPI Card

A Tableau **viz extension** that renders a KPI card: a headline value, a
favorable/unfavorable variance bubble against a budget comparison, a
configurable caption (`vs $26,712K Bdgt`), and a trend sparkline.

**Hosted URL:** https://rgdofem.github.io/tableau-extens/spark-kpi/index.html

## Encoding tiles

| Tile | Type | Max | Purpose |
|---|---|---|---|
| **Value** | Continuous measure | 1 | Required. The headline value and the sparkline series. |
| **Comparison** | Continuous measure | 1 | Optional. The budget/target used for the variance bubble and caption. If not mapped, a fixed target can be set in settings; if neither exists, the bubble and caption are hidden. |
| **Date** | Temporal dimension | 1 | Optional. The sparkline x-axis and the basis for the "latest" headline. If not mapped, the sparkline is hidden and the headline uses the configured aggregation (latest degrades to sum). |

## Adding it to a worksheet

1. In a workbook (Tableau Cloud or Desktop 2024.2+), open a worksheet.
2. On the Marks card, open the mark type dropdown and choose **Add Extension**.
3. Choose **Access Local Extensions** and select `SparklineKPICard.trex`.
4. Drop your Value measure, Comparison (budget) measure, and Date dimension
   onto the matching encoding tiles on the Marks card.
5. Click the gear icon (top-right of the card) to configure: headline
   aggregation, comparison source, higher-is-better, number formatting
   (prefix/suffix/scale/decimals), caption words, bubble colors, sparkline
   styling, and card colors.

## Behavior notes

- Headline aggregation options: sum / avg / min / max / **latest** (default;
  latest = most recent Date period).
- Variance: `rawDelta = value - comparison`. Favorable when
  `(rawDelta >= 0) === higherIsBetter`; the bubble shows the raw direction
  triangle (▲ above / ▼ below) and `abs(rawDelta / comparison)` as a percent.
- Re-renders on `SummaryDataChanged` and `SettingsChanged`.
