+++
title = "Heatmap Panel"
keywords = ["grafana", "dashboard", "documentation", "panels", "heatmap"]
type = "docs"
[menu.docs]
name = "Heatmap"
parent = "panels"
weight = 4
+++

# Heatmap Panel

The Heatmap panel visualizes the density of values over time. It groups data
points into time buckets on the X axis and value buckets on the Y axis, then
colors each card by the number of values in that bucket.

Use a heatmap when you want to see the distribution of many observations, such
as request latencies, payload sizes, temperatures, or any other metric where the
shape of the values is more useful than a single line.

## Data formats

The **Axes** tab has a **Data format** option. The panel supports two formats:

- `Timeseries` - The default format. Each returned time series contributes its
  numeric datapoints to the heatmap. Grafana creates the X and Y buckets in the
  browser.
- `ES histogram` - Use this with Elasticsearch histogram aggregations. Configure
  the Elasticsearch query with a `Histogram` bucket for the numeric value first,
  followed by a `Date Histogram` bucket for time. The heatmap uses the histogram
  bucket key as the Y bucket and the date histogram bucket as the X bucket.

For Elasticsearch histogram data, keep the series names numeric. The panel reads
each series alias as the Y bucket boundary, so an alias such as `100` works but
a text alias such as `latency bucket` cannot be converted into a heatmap row.

## Bucket options

Bucket settings control how many cards are drawn.

### X axis buckets

- `Buckets` - The target number of time buckets. If neither X bucket option is
  set, Grafana uses 30 buckets across the selected time range.
- `Bucket Size` - A fixed time bucket size. This setting has priority over
  `Buckets` and accepts either a number of milliseconds or an interval such as
  `10s`, `5m`, or `1h`.

### Y axis buckets

- `Buckets` - The target number of value buckets. If neither Y bucket option is
  set, Grafana uses 10 buckets across the data range.
- `Bucket Size` - A fixed numeric bucket size. This setting has priority over
  `Buckets` when the Y scale is linear.
- `Scale` - Choose `linear`, `log (base 2)`, `log (base 10)`, `log (base 32)`,
  or `log (base 1024)`.
- `Split Buckets` - Available for log scales only. Splits each logarithmic step
  into more buckets.
- `Remove zero values` - Available for log scales only. Hide the special zero
  bucket that Grafana uses because zero cannot be displayed on a logarithmic
  scale.

If all returned values are the same, the panel still creates visible buckets. A
non-zero single value uses that value divided by the default Y bucket count; a
zero-only result uses a bucket size of `1`.

## Axes and display

The **Axes** tab also controls:

- `Show` for X and Y axes.
- Y-axis `Unit`, `Y-Min`, `Y-Max`, and `Decimals`.

The **Display** tab controls how cards are drawn:

- `Mode` - `spectrum` maps bucket counts to a color scheme. `opacity` uses one
  color and changes opacity by bucket count.
- `Scheme` - The color scheme used by `spectrum` mode. Sequential schemes are
  inverted automatically on dark themes.
- `Fill background` - Fills empty heatmap space with the low-end color when
  using `spectrum` mode.
- `Space` and `Round` - Card spacing and corner radius.

## Tooltip and interactions

Tooltips can show the bucket time, value range, and count. Enable optional
tooltip fields from the **Display** tab:

- `Highlight cards` - Darkens the card under the cursor.
- `Series stats` - Shows how many values each original series contributed to the
  hovered bucket.
- `Histogram` - Shows the distribution for all Y buckets in the hovered X
  bucket.
- `Decimals` - Limits tooltip value precision.

Drag across the heatmap to zoom into a time range. Double-click the panel to
zoom out. When dashboard shared crosshair or shared tooltip mode is enabled,
the heatmap participates in the same hover events as graph panels.

## Troubleshooting

### No data points

The panel shows `No data points` when the query returns no datapoints. Check the
query, time range, and datasource response.

### Data points outside time range

The panel warns about data outside the current time range when returned
datapoints are older than the dashboard range. This is commonly caused by a
timezone mismatch or a missing time filter in the query.

### Elasticsearch heatmap rows are missing or grouped incorrectly

For `ES histogram` mode, verify that:

- The Elasticsearch query uses a `Histogram` bucket for the numeric value and a
  `Date Histogram` bucket for time.
- The `Date Histogram` bucket is the last bucket aggregation, so Grafana receives
  time series data instead of table data.
- The series alias is numeric or left as the numeric histogram bucket key.

Zero-count Elasticsearch buckets are skipped by the heatmap converter, so empty
histogram buckets do not draw cards.
