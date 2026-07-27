+++
title = "Heatmap Panel"
keywords = ["grafana", "dashboard", "documentation", "panels", "heatmap"]
type = "docs"
[menu.docs]
name = "Heatmap"
parent = "panels"
weight = 3
+++

# Heatmap Panel

The Heatmap panel visualizes a distribution over time. It groups datapoints into
time buckets on the X axis and value buckets on the Y axis, then colors each card
by how many values fall into that bucket.

Use a heatmap when you want to see how a value distribution changes over time,
for example request durations, response sizes or event counts split by numeric
ranges.

## Data formats

The panel supports two input formats. Choose the format in the **Axes** tab under
**Data format**.

### Timeseries

Use **Timeseries** when your query returns one or more normal time series. The
panel reads every datapoint value and calculates the heatmap buckets inside
Grafana.

For timeseries data:

- The X bucket is based on the datapoint timestamp.
- The Y bucket is based on the datapoint value.
- The default X bucket count is 30 when **X Axis / Buckets** and **Bucket Size**
  are left empty.
- The default Y bucket count is 10 when **Y Axis / Buckets** and **Bucket Size**
  are left empty.
- **Bucket Size** has priority over **Buckets** when both are set.

X bucket size can be a number of milliseconds or an interval string such as
`10s`, `5m` or `1h`.

### Elasticsearch histogram

Use **ES histogram** when Elasticsearch has already grouped documents into
numeric histogram buckets. In this mode the panel expects each returned time
series name to be a numeric Y bucket bound, and each datapoint value to be the
document count for that Y bucket at the datapoint timestamp.

To build this query with the Elasticsearch query editor:

1. Add a **Count** metric.
2. Add a **Histogram** group by for the numeric field you want on the Y axis.
3. Add a **Date Histogram** group by after the numeric histogram.
4. Leave the query alias empty, or set it to a numeric template such as
   `{{bytes}}` where `bytes` is the histogram field.
5. In the Heatmap **Axes** tab, set **Data format** to **ES histogram**.

The **Date Histogram** must be the last group by. Grafana turns the nested
numeric histogram buckets into separate time series and uses the histogram bucket
key as the series name. If the series name is not numeric, the Heatmap panel
cannot calculate the Y buckets correctly.

## Axes

The **Axes** tab controls the value scale and bucket layout.

### Y axis

- **Unit** formats the Y axis values and tooltip values.
- **Scale** can be linear or logarithmic. Supported log bases are 2, 10, 32 and
  1024.
- **Y-Min** and **Y-Max** override the automatically calculated Y range.
- **Decimals** overrides automatic decimal precision.
- On a linear scale, **Buckets** sets the target number of Y buckets and
  **Bucket Size** sets an explicit bucket size.
- On a logarithmic scale, **Split Buckets** splits each power-of-log bucket into
  smaller buckets. **Remove zero values** hides the zero bucket.

### X axis

- **Buckets** sets the target number of time buckets.
- **Bucket Size** sets an explicit bucket size and accepts either milliseconds or
  an interval such as `10s`, `5m` or `1h`.
- **Bucket Size** has priority over **Buckets**.

## Display options

The **Display** tab controls colors, card shape and tooltip behavior.

### Colors

The Heatmap panel has two color modes:

- **spectrum** maps bucket counts to a color scheme.
- **opacity** uses one color and changes the card opacity by bucket count.

In **spectrum** mode you can choose one of the built-in d3 color schemes and use
**Fill background** to color empty background space. Some color schemes are
inverted automatically on dark themes so that higher values remain visually
prominent.

In **opacity** mode you can choose the card color and use either a `linear` or
`sqrt` scale. The `sqrt` scale uses the **Exponent** setting.

### Cards

- **Space** changes the padding between heatmap cards.
- **Round** changes the card corner radius.

### Tooltip

Enable **Show tooltip** to inspect individual cards. Tooltip options include:

- **Highlight cards** to highlight the hovered card.
- **Series stats** to show how many values from each source series contributed to
  the bucket.
- **Histogram** to show the value distribution for the hovered time bucket.
- **Decimals** to limit tooltip decimal precision.

## Troubleshooting

### No data points

The panel shows a warning when the query returns no datapoints. Verify the panel
time range and datasource query first.

### Data points outside time range

The panel warns when returned datapoints appear to be outside the selected time
range. This is commonly caused by a timezone mismatch or a missing time filter in
the query.

### Elasticsearch heatmap shows empty or incorrect Y buckets

When using **ES histogram**, make sure the numeric **Histogram** group by is
before the **Date Histogram** group by, and that the series alias resolves to a
number. The Heatmap panel uses the series name as the Y bucket bound.
