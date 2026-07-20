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

The heatmap panel visualizes the distribution of values over time. It groups
datapoints into time buckets on the X axis and value buckets on the Y axis, then
draws one colored card for each bucket. Use it when the density or shape of a
value distribution is more important than individual time series lines.

## Data formats

The heatmap panel supports two data formats in the **Axes** tab.

### Timeseries

`Timeseries` is the default format. Grafana reads normal time series datapoints
and calculates both the time buckets and value buckets in the browser.

Use this format when your query returns one or more numeric time series, for
example latency, request size, queue depth, or load values over time.

### ES histogram

`ES histogram` is for Elasticsearch queries that already return a histogram of
value buckets over time. Grafana expects one time series per value bucket. The
series name must be the numeric lower bound of the value bucket, and each point
value must be the document count for that time bucket.

The Elasticsearch query editor can produce this shape by using:

1. A `Count` metric.
2. A `Histogram` bucket aggregation on the numeric field to use for the Y axis.
3. A `Date Histogram` bucket aggregation after the `Histogram` bucket, using the
   data source time field for the X axis.

For example, to show request sizes over time, group by `Histogram` on the
`bytes` field with an interval of `1000`, then group by `Date Histogram` on
`@timestamp`.

Leave the Elasticsearch query `Alias` field empty, or set it to a template that
expands to the numeric histogram key, such as `{{bytes}}`. The heatmap parser
uses the series name as the Y bucket bound.

## Axes

The **Axes** tab controls bucket sizing and axis display.

### X axis

- `Show` - Toggles the X axis grid and tick lines.
- `Buckets` - Sets the target number of time buckets. The default is 30.
- `Bucket Size` - Sets an explicit time bucket size. It accepts a number of
  milliseconds or an interval string such as `10s`, `5m`, or `1h`. This option
  has priority over `Buckets`.

For `ES histogram` data, Grafana calculates the X bucket size from the returned
Elasticsearch date histogram buckets.

### Y axis

- `Show` - Toggles the Y axis grid and tick lines.
- `Unit` - Sets the value unit used on the axis and tooltip.
- `Scale` - Supports `linear`, `log (base 2)`, `log (base 10)`,
  `log (base 32)`, and `log (base 1024)`.
- `Y-Min` and `Y-Max` - Override the automatically calculated Y range.
- `Decimals` - Overrides automatic decimal precision.
- `Buckets` - Sets the target number of value buckets for linear scale. The
  default is 10.
- `Bucket Size` - Sets an explicit value bucket size for linear scale. This
  option has priority over `Buckets`.
- `Split Buckets` - For log scales, splits each default log bucket into smaller
  buckets.
- `Remove zero values` - For log scales, hides the zero-value bucket.

For `ES histogram` data, Grafana calculates the Y bucket size from the numeric
series aliases returned by the Elasticsearch histogram buckets. With a log Y
axis, the calculated size is used as the log split factor.

## Display

The **Display** tab controls card appearance, colors, and tooltip behavior.

### Colors

The heatmap panel supports two color modes:

- `spectrum` - Colors buckets with a d3 color scheme such as `Oranges`,
  `Blues`, or `Spectral`. `Fill background` applies the lowest color level to
  empty space behind the cards.
- `opacity` - Uses one selected card color and varies opacity by bucket count.
  The opacity scale can be `linear` or `sqrt`; the `sqrt` scale exposes an
  `Exponent` setting.

### Cards

- `Space` - Sets the padding between cards.
- `Round` - Rounds card corners.

### Tooltip

- `Show tooltip` - Enables bucket hover details.
- `Highlight cards` - Highlights cards while hovering.
- `Series stats` - Shows how many values from each source series are in the
  bucket. This is useful for the `Timeseries` format.
- `Histogram` - Shows the distribution for the hovered time bucket.
- `Decimals` - Limits tooltip decimal precision.

## Troubleshooting

### No data points

The panel shows a `No data points` warning when the query returns no datapoints.
Check the query and dashboard time range.

### Data points outside time range

The panel warns when returned datapoints are outside the selected dashboard time
range. This can be caused by a timezone mismatch or a missing time filter in the
query.

### Elasticsearch histogram is shown as a table

For the heatmap `ES histogram` format, the deepest Elasticsearch bucket must be a
`Date Histogram`. If the last bucket is a `Histogram`, Grafana treats the result
as document-style rows instead of time series. Configure the buckets as
`Histogram` first and `Date Histogram` last.

If the panel is blank or the Y axis looks wrong, also check the Elasticsearch
query `Alias` field. The series names must be numeric because Grafana uses them
as Y bucket bounds.
