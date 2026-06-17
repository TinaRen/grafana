+++
title = "Heatmap Panel"
keywords = ["grafana", "dashboard", "documentation", "panels", "heatmap panel"]
type = "docs"
[menu.docs]
name = "Heatmap"
parent = "panels"
weight = 4
+++

# Heatmap Panel

The Heatmap panel visualizes the density of numeric time series values over time. It groups returned
datapoints into buckets on the X axis (time) and Y axis (value), then draws one card for each bucket.
The card color or opacity is based on the number of values in the bucket, not on the average or sum of
the values.

Use it for distributions where the shape matters more than a single line, for example request latency,
temperature readings, queue sizes, or other metrics that produce many samples over the selected time
range.

## Data formats

The **Axes** tab contains the **Data format** setting.

### Timeseries

`Timeseries` is the default format. Each datapoint returned by the query is treated as one sample:

```text
[value, timestamp]
```

Grafana places the sample into an X bucket based on the timestamp and into a Y bucket based on the
numeric value. Null, undefined, and non-numeric values are skipped.

Example: if you query per-request latency as a time series, the X axis shows when requests happened,
the Y axis shows latency values, and darker cards indicate more requests in that time/value bucket.

### ES histogram

`ES histogram` is for Elasticsearch queries that already return histogram buckets. In this format,
each series name is interpreted as a Y bucket boundary and each datapoint value is interpreted as the
count for that bucket at the datapoint timestamp.

Use this format when Elasticsearch has already aggregated the distribution. For raw numeric time
series, use `Timeseries` so Grafana can build the heatmap buckets itself.

## Bucket options

Bucket settings control the resolution of the heatmap.

### X axis buckets

- **Buckets** sets the target number of time buckets. When left empty, Grafana uses 30 buckets.
- **Bucket Size** sets an explicit time bucket size and has priority over **Buckets**.

The X axis **Bucket Size** can be a number of milliseconds or an interval string such as `10s`, `5m`,
or `1h`. Supported interval units are `ms`, `s`, `m`, `h`, `d`, `w`, `M`, and `y`.

### Y axis buckets

For a linear Y axis:

- **Buckets** sets the target number of value buckets. When left empty, Grafana uses 10 buckets.
- **Bucket Size** sets an explicit value bucket size and has priority over **Buckets**.

For a logarithmic Y axis, Grafana groups values by powers of the selected log base. The **Split
Buckets** option divides each logarithmic bucket into smaller buckets. The available scales are
linear, log base 2, log base 10, log base 32, and log base 1024.

Zero values cannot be placed on a logarithmic scale. By default Grafana folds them into the bottom
bucket and labels the first tick as `0`. Enable **Remove zero values** if those zero-value samples
should be hidden instead.

## Axis display

The **Axes** tab also controls display settings:

- **Show** toggles each axis.
- **Unit** formats Y axis values.
- **Y-Min** and **Y-Max** override the automatically calculated Y range.
- **Decimals** overrides automatic decimal precision on the Y axis.

## Display options

The **Display** tab controls the card appearance and tooltip behavior.

### Colors

The Heatmap panel has two color modes:

- **Opacity** uses one selected color and changes opacity based on bucket count. The scale can be
  linear or square-root.
- **Spectrum** uses a color scheme from the panel. Some schemes are automatically inverted on the dark
  theme so high-density buckets stay visually prominent.

With **Fill background** enabled in spectrum mode, Grafana fills empty space with the lowest color in
the selected scheme.

### Cards

- **Space** controls padding between cards. When empty, Grafana uses the default spacing.
- **Round** controls card corner radius. When empty, cards have square corners.

### Tooltip

The tooltip can show:

- the bucket time;
- the Y bucket range;
- the count of samples in that bucket;
- per-series counts when **Series stats** is enabled;
- a small histogram for the hovered time bucket when **Histogram** is enabled.

When **Highlight cards** is enabled, hovering a card temporarily darkens the card and adds a brighter
stroke so the selected bucket is easier to see.

## Troubleshooting

### No data points

This warning means the query returned no datapoints for the selected time range. Check the dashboard
time range and the query filter.

### Data points outside time range

This warning can occur when returned datapoints are outside the dashboard time range. Common causes
are a timezone mismatch or a missing time filter in the query.

### Heatmap is too coarse or too dense

Adjust X and Y bucket settings. Smaller bucket sizes show more detail but can create many cards.
Larger bucket sizes make the density pattern easier to scan but hide fine-grained variation.

### Colors do not match value magnitude

Color intensity represents the number of samples in each bucket. The sample values determine the
bucket's Y position. To show larger values with a different color, use thresholds in another panel
type such as Graph or Singlestat.
