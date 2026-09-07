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

The heatmap panel shows how many values fall into value buckets over time. It is
useful when the individual samples are less important than the distribution, for
example request durations, response sizes, or sensor readings.

Each heatmap card represents one time bucket on the X axis and one value bucket
on the Y axis. The card color or opacity is based on how many data points are in
that bucket.

## Data formats

The heatmap panel supports two input formats.

### Timeseries

Use the default `Timeseries` format when the datasource returns one or more time
series with numeric values. Grafana groups every point into:

- an X bucket, based on the point timestamp and the X bucket size
- a Y bucket, based on the point value and the Y bucket size or log scale

Example datapoints:

```javascript
[
  { target: "api.latency", datapoints: [[120, 1494478800000], [210, 1494478860000]] },
  { target: "worker.latency", datapoints: [[80, 1494478800000], [160, 1494478860000]] }
]
```

With an X bucket size of `1m` and a Y bucket size of `100`, values from `100` to
`199` are counted in one card and values from `200` to `299` are counted in the
next card for each minute.

### ES histogram

Use `ES histogram` when an Elasticsearch query already produces histogram
buckets. Configure the query with:

1. a `Histogram` bucket aggregation on the numeric field for the Y axis
2. a `Date Histogram` bucket aggregation last, for the X axis
3. a `Count` metric to draw

The heatmap panel treats each returned series name as the Y bucket boundary and
each datapoint value as the count for that Y bucket at a timestamp. Keep the
series alias numeric. Leaving the Elasticsearch alias empty works when the
histogram bucket key is numeric; otherwise use an alias template that resolves to
the numeric bucket key.

> Note: Elasticsearch buckets with a count of `0` are not drawn by the heatmap
> converter.

If the last Elasticsearch bucket aggregation is not `Date Histogram`, Grafana
returns table-style documents instead of time series, and the heatmap panel
cannot place those results on the time axis.

## Axes options

Open the `Axes` tab to control bucket sizing and axis formatting.

### X axis

- `Show` toggles X axis grid lines.
- `Buckets` sets the target number of time buckets. The default is `30`.
- `Bucket Size` overrides `Buckets`. It accepts either a number of milliseconds
  or an interval such as `10s`, `5m`, or `1h`.

When `Bucket Size` is empty, Grafana calculates the time bucket size from the
dashboard time range divided by the bucket count.

### Y axis

- `Show` toggles Y axis grid lines.
- `Unit` formats Y axis and tooltip values.
- `Scale` can be `linear`, `log (base 2)`, `log (base 10)`,
  `log (base 32)`, or `log (base 1024)`.
- `Y-Min` and `Y-Max` override the automatic Y range.
- `Decimals` overrides automatic decimal precision.

For the linear scale:

- `Buckets` sets the target number of value buckets. The default is `10`.
- `Bucket Size` overrides `Buckets` and sets the value interval directly.

For log scales:

- `Split Buckets` divides each power-of-base bucket into smaller buckets. For
  example, a value of `2` splits each log bucket into two buckets.
- `Remove zero values` hides the zero bucket. When it is off, Grafana merges zero
  values into the first visible log bucket and labels the first tick as `0`.

## Display options

The `Display` tab controls how cards are colored, sized, and explained in the
tooltip.

### Colors

`Mode` controls how card counts are mapped visually:

- `opacity` uses one configured color and varies the opacity by bucket count.
- `spectrum` uses a D3 color scheme such as `Oranges`, `Blues`, or `Spectral`.

For opacity mode, `Scale` can be `linear` or `sqrt`; the `sqrt` option uses the
configured exponent. For spectrum mode, `Fill background` paints the panel
background with the lowest color in the selected scheme.

### Cards

- `Space` controls padding between cards.
- `Round` controls the card corner radius.

### Tooltip

- `Show tooltip` enables bucket details on hover.
- `Highlight cards` darkens the hovered card.
- `Series stats` shows how many points from each series contributed to the
  bucket. This is available for the `Timeseries` format.
- `Histogram` adds a small value distribution for the hovered time bucket.
- `Decimals` limits tooltip decimal precision.

The tooltip shows the bucket time, value range, and count. If shared dashboard
tooltips are enabled, heatmap panels also participate in the shared crosshair and
tooltip behavior.

## Interactions

- Drag across the panel to zoom into a time range.
- Double-click the panel to zoom out.
- Hover over cards to show bucket details.

## Troubleshooting

### No data points

The panel shows `No data points` when the datasource returns no datapoints for
all series. Check the dashboard time range, query filters, and datasource
response.

### Data points outside time range

The panel shows `Data points outside time range` when returned datapoints are far
earlier than the current panel range. This is usually caused by a timezone
mismatch or a missing time filter in the query.

### Elasticsearch histogram appears empty

Check that:

- the panel `Data format` is `ES histogram`
- the Elasticsearch query has a numeric `Histogram` bucket first
- `Date Histogram` is the last bucket aggregation
- the returned series aliases are numeric Y bucket boundaries
- the selected time range contains non-zero bucket counts
