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

The heatmap panel visualizes the density of values over time. Grafana groups
data points into time buckets on the X axis and value buckets on the Y axis,
then colors each card by how many values fall into that bucket.

Use a heatmap when you need to see the distribution of many measurements, such
as request durations, queue sizes, or metric values whose outliers are hidden by
line charts.

## Supported data formats

The **Axes** tab has a **Data format** setting with two modes.

### Timeseries

Use **Timeseries** when your query returns ordinary time series. Grafana reads
each datapoint as `[value, timestamp]`, ignores `null`, `undefined`, and `NaN`
values, and places each value into the matching X and Y bucket.

This format works well when the raw values are already returned by the data
source and Grafana should calculate the heatmap buckets in the browser.

### ES histogram

Use **ES histogram** for Elasticsearch queries that already aggregate values
into histogram buckets. Configure the Elasticsearch query with:

1. a **Histogram** bucket on the numeric field that should become the Y axis;
2. a **Date Histogram** bucket as the last bucket aggregation, so Grafana
   receives time series rather than table-style documents;
3. an empty Elasticsearch **Alias** field, or an alias template that resolves to
   the numeric histogram bucket, such as `{{latency}}`.

The heatmap panel converts each Elasticsearch series name to a number and uses
that number as the Y bucket boundary. If the alias is non-numeric, the panel
cannot place the series on the Y axis. Zero-count Elasticsearch buckets are not
drawn.

## Bucket configuration

### X axis

- **Buckets** sets the target number of time buckets. If unset, Grafana uses 30.
- **Bucket Size** overrides **Buckets**. It accepts a number of milliseconds or
  an interval such as `10s`, `5m`, or `1h`.
- **Show** hides or shows the X axis grid and labels.

### Y axis

- **Unit** formats Y-axis values and tooltip bucket bounds.
- **Scale** supports `linear`, `log (base 2)`, `log (base 10)`,
  `log (base 32)`, and `log (base 1024)`.
- **Y-Min** and **Y-Max** override the automatically calculated Y-axis range.
- **Decimals** overrides automatic decimal precision for Y-axis labels.
- For a linear scale, **Buckets** sets the target number of Y buckets. If unset,
  Grafana uses 10.
- For a linear scale, **Bucket Size** overrides **Buckets**.
- For a log scale, **Split Buckets** divides each power-of-base bucket into the
  specified number of smaller buckets.
- For a log scale, **Remove zero values** hides the zero-value bucket. When this
  option is off, Grafana merges zero values into a bottom bucket so they can be
  shown on a logarithmic axis.

## Display options

The **Display** tab controls how cards and tooltips are drawn.

### Colors

- **Opacity** mode uses one color and varies opacity by bucket count.
- **Spectrum** mode uses a color scheme from the D3 color palettes and colors
  each card by bucket count.
- **Scale** can be `linear` or `sqrt` in opacity mode. The `sqrt` scale uses the
  **Exponent** value.
- **Fill background** fills the plot background with the lowest spectrum color.

### Cards

- **Space** controls card padding.
- **Round** controls card corner radius.

### Tooltip

- **Show tooltip** enables bucket tooltips.
- **Highlight cards** changes the hovered card border and fill.
- **Series stats** shows how many points each input series contributed to the
  hovered bucket.
- **Histogram** adds a small per-time-bucket histogram to the tooltip.
- **Decimals** limits tooltip decimal precision.

## Interactions

- Drag across the panel to zoom the dashboard time range to the selected time
  window.
- Double-click the panel to zoom out.
- Shared crosshair and shared tooltip modes follow the dashboard tooltip
  setting, the same as graph panels.

## Troubleshooting

### No data points

The panel shows **No data points** when the query returns no datapoints. Check
the query, dashboard time range, and data source response.

### Data points outside time range

The panel shows **Data points outside time range** when returned datapoints fall
outside the dashboard range. This is commonly caused by a missing time filter in
the query or a timezone mismatch.

### Elasticsearch histogram is empty or misplaced

For **ES histogram** mode, keep the numeric **Histogram** bucket before the
**Date Histogram** bucket, and leave **Alias** empty unless it resolves to the
numeric histogram bucket. A text alias such as `latency bucket` cannot be parsed
as a Y-axis boundary.
