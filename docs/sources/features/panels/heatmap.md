+++
title = "Heatmap Panel"
keywords = ["grafana", "dashboard", "documentation", "panels", "heatmap panel", "heatmap"]
type = "docs"
[menu.docs]
name = "Heatmap"
parent = "panels"
weight = 3
+++

# Heatmap Panel

The heatmap panel groups time series values into buckets and renders each bucket as a colored card.
The X axis is time, the Y axis is the value range, and the card color represents how many values fall
into that X/Y bucket.

Use a heatmap when you want to see the distribution of values over time, for example request latency
distribution, sensor readings, or any metric where the shape of the distribution is more useful than a
single line.

## Data formats

The heatmap panel supports two input formats. Choose the format in the `Axes` tab under `Data format`.

### Timeseries

Use `Timeseries` when the query returns regular time series datapoints. Grafana calculates the heatmap
buckets in the browser:

- Each datapoint timestamp is placed into an X-axis time bucket.
- Each datapoint value is placed into a Y-axis value bucket.
- Null, undefined, or non-numeric values are ignored.

This format works with any datasource that returns numeric time series.

### ES histogram

Use `ES histogram` when Elasticsearch already returns histogram-style buckets. In this mode each
returned series is treated as one Y-axis bucket:

- The series name must be the numeric lower bound for the Y bucket.
- Each datapoint value is the count for that Y bucket at the datapoint timestamp.
- Empty or zero-count buckets do not create cards.

For example, if Elasticsearch returns one series named `100` and another named `200`, the heatmap uses
those names as Y bucket bounds. The datapoint timestamps become X bucket bounds, and the datapoint values
become the bucket counts.

When using `ES histogram`, Grafana derives the X bucket size from the time buckets that remain after
empty and zero-count buckets are skipped. The Y bucket size is derived from the returned series aliases.
For linear axes it uses the smallest distance between adjacent bounds. For logarithmic Y axes it uses
the smallest logarithmic distance and maps that to the split-bucket setting.

## Axes and buckets

The `Axes` tab controls bucket sizing and axis display.

### X axis

- `Show` toggles the time-axis grid and tick lines.
- `Buckets` sets the target number of X buckets. If left empty, Grafana uses 30 buckets.
- `Bucket Size` sets an explicit X bucket size and takes priority over `Buckets`. It accepts a number
  of milliseconds or an interval such as `10s`, `5m`, or `1h`.

### Y axis

- `Show` toggles the value-axis grid and tick lines.
- `Unit` controls value formatting.
- `Scale` supports linear scale and logarithmic scales with bases 2, 10, 32, and 1024.
- `Y-Min` and `Y-Max` override the automatic Y-axis range. On logarithmic scales, Grafana rounds these
  values to powers of the selected log base.
- `Decimals` overrides automatic decimal precision for axis labels.

For a linear Y axis:

- `Buckets` sets the target number of Y buckets. If left empty, Grafana uses 10 buckets.
- `Bucket Size` sets an explicit Y bucket size and takes priority over `Buckets`.

For a logarithmic Y axis:

- `Split Buckets` splits each logarithmic interval into smaller buckets.
- `Remove zero values` hides the special zero bucket. When this option is off, Grafana keeps a zero
  bucket so zero values can still be displayed on a log scale.

## Display options

The `Display` tab controls card color, spacing, and tooltip behavior.

### Colors

`Mode` controls how bucket counts are mapped to color:

- `opacity` uses one configured color and varies the opacity by bucket count. The opacity scale can be
  `linear` or `sqrt`.
- `spectrum` maps bucket counts through a color scheme. The available schemes are based on
  d3-scale-chromatic palettes.

`Fill background` is available in `spectrum` mode and fills the panel background with the color for zero
values.

### Cards

- `Space` controls the padding between cards.
- `Round` controls the card corner radius.

### Tooltip

- `Show tooltip` enables or disables heatmap tooltips.
- `Highlight cards` highlights the bucket under the cursor.
- `Series stats` shows how many values from each source series contributed to the bucket when that
  information is available.
- `Histogram` adds a small histogram for the hovered time bucket.
- `Decimals` controls tooltip value precision.

## Troubleshooting

### The panel shows "No data points"

The query returned no datapoints. Check the datasource query and dashboard time range.

### The panel warns that data points are outside the time range

Grafana detected datapoints that are older than the panel range. This is commonly caused by a timezone
mismatch or a query that does not filter by the dashboard time range.

### ES histogram data renders an empty heatmap

Check that each returned series name is a numeric Y bucket bound and that datapoint values are counts.
Zero or empty counts are skipped, so a query that only returns zero counts will not draw cards.

### Bucket sizes look unexpected

For `Timeseries`, explicit `Bucket Size` values override the bucket count settings. For `ES histogram`,
Grafana derives the X bucket size from non-empty time buckets and the Y bucket size from returned series
aliases, so sparse or uneven histogram output can make the smallest derived distance determine the
displayed card size.
