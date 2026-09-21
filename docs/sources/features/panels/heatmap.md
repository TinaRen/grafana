+++
title = "Heatmap Panel"
keywords = ["grafana", "dashboard", "documentation", "panels", "heatmap"]
type = "docs"
[menu.docs]
name = "Heatmap"
parent = "panels"
weight = 5
+++

# Heatmap Panel

The heatmap panel visualizes a time series as buckets over time. Each card on
the graph represents all values that fall into one X bucket (time) and one Y
bucket (value). The card color or opacity is based on how many values are in
that bucket.

Use a heatmap when you want to see distributions over time, for example request
duration buckets, response size ranges, or Elasticsearch histogram buckets.

## Data formats

The heatmap panel supports two input formats.

### Timeseries

The `Timeseries` format converts regular time series datapoints into heatmap
buckets in the browser. Grafana groups datapoints first by time, then by value.

- X buckets are based on the dashboard time range and the X axis bucket settings.
- Y buckets are based on the datapoint values and the Y axis bucket settings.
- `null`, `undefined`, and `NaN` values are ignored.
- If a query returns datapoints outside the visible time range, the panel shows a
  warning. This is commonly caused by a timezone mismatch or a missing time
  filter in the query.

### ES histogram

The `ES histogram` format is for Elasticsearch queries that already return
histogram buckets. Use this format when the Elasticsearch query groups documents
by a numeric histogram and a date histogram.

Configure the Elasticsearch query with the numeric `Histogram` bucket before the
`Date Histogram` bucket. The numeric histogram bucket becomes the Y bucket and
the date histogram bucket becomes the X bucket. Leave the Elasticsearch `Alias`
empty, or set it to a numeric template, because the heatmap panel reads the
series alias as the numeric Y bucket bound.

Grafana skips Elasticsearch histogram buckets with a zero count when converting
them to heatmap data.

## Axes and buckets

Open the `Axes` editor tab to configure bucket sizes and axis display.

### X axis

- `Show` toggles the X axis grid and labels.
- `Buckets` sets the number of time buckets. If unset, Grafana uses 30 buckets.
- `Bucket Size` sets an explicit bucket size and takes priority over `Buckets`.
  It can be a number of milliseconds or an interval string such as `10s`, `5m`,
  or `1h`.

### Y axis

- `Show` toggles the Y axis grid and labels.
- `Unit` controls value formatting for the Y axis and tooltip values.
- `Scale` can be linear or logarithmic (`log (base 2)`, `log (base 10)`,
  `log (base 32)`, or `log (base 1024)`).
- `Y-Min` and `Y-Max` override the automatic Y range.
- `Decimals` overrides automatic decimal precision for axis labels.

For a linear scale:

- `Buckets` sets the number of Y buckets. If unset, Grafana uses 10 buckets.
- `Bucket Size` sets an explicit Y bucket size and takes priority over
  `Buckets`.

For a logarithmic scale:

- `Split Buckets` divides each logarithmic bucket. For example, with log base 2,
  the default buckets are `1-2`, `2-4`, `4-8`, and so on; a split factor of `2`
  divides each range in half.
- `Remove zero values` removes zero-value buckets. If disabled, Grafana merges
  zero values into the bottom bucket so they can still be shown on a log scale.

## Display options

Open the `Display` editor tab to configure colors, cards, and tooltip behavior.

### Colors

The heatmap can color cards in two modes:

- `opacity` uses one color and varies the opacity by bucket count.
- `spectrum` uses a color scheme and maps bucket count to a color.

In `opacity` mode, `Scale` can be `linear` or `sqrt`. The `sqrt` scale uses the
configured exponent. In `spectrum` mode, `Fill background` fills the plot
background with the lowest color in the selected scheme.

### Cards

- `Space` controls padding between cards.
- `Round` controls card corner radius.

### Tooltip

- `Show tooltip` enables bucket tooltips.
- `Highlight cards` changes the hovered card stroke and fill color.
- `Series stats` adds per-series counts for the bucket when that information is
  available.
- `Histogram` shows a small histogram for the hovered time bucket.
- `Decimals` controls tooltip value precision.

## Interactions

Hover over the panel to see the crosshair and tooltip. If the dashboard shared
tooltip is enabled, other graph panels can drive the heatmap tooltip and
crosshair.

Drag across the heatmap to zoom to a time range. Double-click the panel to zoom
out.

## Troubleshooting

### No data points

The panel shows `No data points` when the query returns no datapoints. Check the
query, dashboard time range, and datasource time filter.

### Data points outside time range

The panel shows `Data points outside time range` when returned datapoints are
older than the dashboard range. This can happen when the query is missing a time
filter or when the datasource and Grafana use different timezones.

### Empty or incorrect ES histogram

For `ES histogram`, make sure the query uses a numeric `Histogram` bucket before
the `Date Histogram` bucket, and make sure the series alias is numeric. If the
alias is text, Grafana cannot convert it into a Y bucket bound.
