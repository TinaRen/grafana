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

The heatmap panel visualizes the density of points over time. The X axis is
time, the Y axis is the measured value or value bucket, and each card's color
shows how many points fell into that X/Y bucket.

Use a heatmap when the distribution of values matters more than individual
series lines. Common examples include request duration buckets, response sizes,
temperatures, and other values where you want to see clusters, gaps, or outliers
over time.

## Data formats

The heatmap panel supports two input formats. Select the format in the **Axes**
tab under **Data format**.

### Timeseries

`Timeseries` is the default format. Grafana receives normal time series data and
groups every point into time buckets on the X axis and value buckets on the Y
axis.

Example input:

```text
series: api.latency
datapoints: [12, 1494410780000], [22, 1494410840000], [85, 1494410900000]
```

With this format:

- X buckets are calculated from the visible time range. If no X bucket setting is
  provided, Grafana divides the range into 30 buckets.
- `Bucket Size` on the X axis can be a number of milliseconds or an interval
  string such as `10s`, `5m`, or `1h`.
- Y buckets are calculated from the minimum and maximum values. If no Y bucket
  setting is provided, Grafana divides the range into 10 buckets.
- `Bucket Size` on the Y axis takes priority over the Y `Buckets` count.

### ES histogram

Use `ES histogram` when Elasticsearch has already grouped documents into value
buckets. In this mode Grafana expects a time series for each Y bucket. The series
name must be the numeric Y bucket bound and each datapoint value must be the
document count for that bucket at a timestamp.

Configure the Elasticsearch query like this:

1. Add a `Count` metric.
2. Add a `Histogram` group by on the numeric field you want on the Y axis, for
   example `bytes`. Set an interval that matches the desired Y bucket size.
3. Add a `Date Histogram` group by after the `Histogram` group by so each value
   bucket contains counts over time.
4. Leave `Alias` empty, or set it to only the histogram field value, for example
   `{{bytes}}`. The heatmap panel converts series names to numbers for the Y
   bucket bounds.

Elasticsearch `Histogram` group by options include `Interval`, `Min Doc Count`,
and an optional `Missing` value. The query editor defaults histogram interval to
`1000` and minimum document count to `1`.

Zero-count Elasticsearch buckets are not rendered as cards. If the panel appears
sparse, check the histogram interval and minimum document count in the query.

## Axes and buckets

The **Axes** tab controls how Grafana slices data into cards.

### Y axis

- `Show` toggles Y axis labels.
- `Unit` formats the Y values.
- `Scale` can be linear or logarithmic. Supported logarithmic bases are 2, 10,
  32, and 1024.
- `Y-Min` and `Y-Max` override the automatic Y range.
- `Decimals` overrides automatic decimal precision.
- On a linear scale, `Buckets` controls the number of Y buckets and `Bucket
  Size` fixes the size of each Y bucket.
- On a logarithmic scale, `Split Buckets` splits each power-of-base bucket into
  smaller buckets, and `Remove zero values` hides buckets for zero values.

For logarithmic scales, Grafana displays zero values in a special bottom bucket
because zero cannot be plotted on a log axis. Use `Remove zero values` if that
bucket is not useful for the data you are analyzing.

### X axis

- `Show` toggles X axis labels.
- `Buckets` controls how many time buckets Grafana creates across the selected
  time range.
- `Bucket Size` fixes the time bucket size. It accepts a number of milliseconds
  or an interval string such as `10s`, `5m`, `1h`, `1d`, or `1w`.

## Display options

The **Display** tab controls the card rendering and tooltip behavior.

### Colors

`Mode` controls how card counts become colors:

- `opacity` uses a single color and changes opacity. The scale can be `linear`
  or `sqrt`.
- `spectrum` uses a color scheme from d3-scale-chromatic. This is the default
  mode and uses the `Oranges` scheme by default.

Enable `Fill background` in spectrum mode to color the empty panel background
with the low end of the selected color scheme.

### Cards

- `Space` changes the padding between cards.
- `Round` changes the card corner radius.

### Tooltip

- `Show tooltip` enables or disables the hover tooltip.
- `Highlight cards` highlights the card under the pointer.
- `Series stats` includes the contributing series and point counts for the
  bucket.
- `Histogram` adds a small histogram for the selected time bucket.
- `Decimals` limits tooltip decimal precision.

Double-click the heatmap to zoom out by a factor of 2.

## Troubleshooting

### No data points

This warning means the query returned no datapoints. Check the dashboard time
range, query filters, and data source response.

### Data points outside time range

This warning means Grafana received datapoints, but their timestamps are outside
the panel's current time range. Common causes are a timezone mismatch or a query
that does not include the dashboard time filter.

### ES histogram shows no or unexpected buckets

For `ES histogram`, make sure the series names are numeric. If you set an
Elasticsearch alias that contains text, the panel cannot use it as a Y bucket
bound. Leave the alias empty or use only the histogram field template, such as
`{{bytes}}`.
