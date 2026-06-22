+++
title = "Heatmap Panel"
keywords = ["grafana", "dashboard", "documentation", "panels", "heatmap panel", "histogram"]
type = "docs"
[menu.docs]
name = "Heatmap"
parent = "panels"
weight = 5
+++

# Heatmap Panel

The heatmap panel shows the distribution of numeric values over time. It groups data points into time
buckets on the X axis and value buckets on the Y axis, then colors each card by the number of values in
that bucket.

Use a heatmap when you want to see patterns in a distribution, for example request latency, payload size,
or sensor values over time. If you only need the latest value or a single aggregate, use a graph or
singlestat panel instead.

## Data formats

The heatmap panel supports two data formats. Select the format on the **Axes** tab.

### Timeseries

Use `Timeseries` for normal metric queries. Grafana reads every returned time series, ignores null or
non-numeric values, and places each point into an X bucket by timestamp and a Y bucket by value.

Example:

1. Query one or more latency series.
2. Set **Format** to `Timeseries`.
3. Set **X Axis > Bucket Size** to `5m` to group points into five minute windows.
4. Set **Y Axis > Bucket Size** to `100` to group latency values into 100 ms buckets.

If **Bucket Size** is empty, Grafana calculates the bucket size from the selected time range and the
number of buckets. The default is 30 X buckets and 10 Y buckets.

### ES histogram

Use `ES histogram` when an Elasticsearch query already returns histogram buckets. In this mode the
panel treats each returned series name as the Y bucket boundary and each datapoint value as the count
for that bucket at a timestamp.

For Elasticsearch, configure the metric query so that the final bucket aggregation is a date histogram:

1. Add a `Count` metric.
2. Add **Group by** `Histogram` on the numeric field you want on the Y axis, such as `bytes` or
   `response_time`.
3. Add **Then by** `Date Histogram` on the time field.
4. In the heatmap panel, set **Axes > Data format > Format** to `ES histogram`.

Grafana calculates the X and Y bucket sizes from the returned histogram bucket boundaries. Empty
histogram buckets with a zero count are not drawn as cards.

## Axes

The **Axes** tab controls how values are bucketed and how axis labels are displayed.

### X axis

- `Show` - Toggles the time axis.
- `Buckets` - Number of time buckets to split the visible time range into when `Bucket Size` is empty.
- `Bucket Size` - Explicit time bucket size. This can be a number of milliseconds or an interval such
  as `10s`, `5m`, or `1h`. This option has priority over `Buckets`.

### Y axis

- `Show` - Toggles the value axis.
- `Unit` - Unit formatter for Y axis and tooltip values.
- `Scale` - Linear scale or logarithmic scale with base 2, 10, 32, or 1024.
- `Y-Min` / `Y-Max` - Optional fixed Y axis range. Leave empty for automatic range selection.
- `Decimals` - Optional decimal precision for axis labels.
- `Buckets` - Number of Y buckets to create on a linear scale when `Bucket Size` is empty.
- `Bucket Size` - Explicit Y bucket size on a linear scale. This option has priority over `Buckets`.

For logarithmic scales, the linear bucket size options are replaced by:

- `Split Buckets` - Splits each power-of-log bucket into smaller buckets. For example, with log base 2
  and split factor `2`, the range between 1 and 2 is split into two buckets.
- `Remove zero values` - Removes the zero bucket from the rendered heatmap. If disabled, Grafana keeps
  a zero bucket so zero values can still be represented on a log scale.

## Display

The **Display** tab controls the color and card rendering.

### Colors

The heatmap can color cards in two modes:

- `spectrum` - Uses a color scheme where color changes as the bucket count increases. This is the
  default mode.
- `opacity` - Uses one color and changes opacity as the bucket count increases.

In `spectrum` mode, choose a color scheme and optionally enable `Fill background` to color the panel
background with the lowest value in the selected color scale. In `opacity` mode, choose the card color
and either a `linear` or `sqrt` opacity scale. The `sqrt` scale can use an `Exponent` value.

### Cards

- `Space` - Space between heatmap cards. Leave empty for the default spacing.
- `Round` - Corner radius for heatmap cards. Leave empty for square cards.

### Tooltip

- `Show tooltip` - Toggles the heatmap tooltip.
- `Highlight cards` - Highlights the card under the cursor.
- `Series stats` - Shows how many values each source series contributed to the hovered bucket.
- `Histogram` - Shows the distribution of values inside the hovered time bucket.
- `Decimals` - Maximum decimal precision in tooltip values.

The heatmap also participates in shared dashboard tooltips. When dashboard tooltip mode is set to shared
crosshair and tooltip, the panel can show the matching bucket for a hover event from another graph.

## Interactions and troubleshooting

- Drag across the panel to zoom into a time range.
- Double-click the panel to zoom out.
- If the panel shows `No data points`, the query returned no datapoints for the selected time range.
- If the panel warns about `Data points outside time range`, check the query time filter and datasource
  timezone settings.
- Null, undefined, and non-numeric values are skipped when Grafana builds timeseries heatmap buckets.
- For `ES histogram`, ensure the histogram aggregation is on a numeric field and the last bucket
  aggregation is a date histogram. Otherwise Grafana may return table-like documents instead of
  time series that the heatmap can render.
