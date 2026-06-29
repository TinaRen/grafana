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

The heatmap panel visualizes how a distribution changes over time. Each card
represents the number of values that fall into a time bucket on the X axis and a
value bucket on the Y axis. Use it when you want to see value density, outliers,
or shifting distributions instead of a single line for each series.

## Data formats

Configure the data format on the **Axes** tab.

### Timeseries

Use **Timeseries** for normal time series queries from data sources such as
Graphite, InfluxDB, Prometheus, or OpenTSDB. Grafana reads the returned values
and builds the heatmap buckets in the browser.

For this format:

* The X axis buckets are time ranges.
* The Y axis buckets are ranges of metric values.
* Empty, null, undefined, and non-numeric values are skipped.

### ES histogram

Use **ES histogram** when an Elasticsearch query already returns histogram
buckets. In the Elasticsearch query editor, group by a numeric **Histogram**
aggregation and a **Date Histogram** so Grafana receives value buckets over
time. The heatmap panel uses the histogram bucket keys as Y bucket bounds and
calculates bucket sizes from the returned data.

For details on the Elasticsearch query options, see
[Using Elasticsearch in Grafana]({{< relref "../datasources/elasticsearch.md" >}}#histogram-aggregation).

## Axes and buckets

The **Axes** tab controls how values are grouped before they are drawn.

### X axis

The X axis is always time based.

* **Buckets** - Sets the number of time buckets. The default is automatic.
* **Bucket Size** - Sets an explicit time bucket size. You can enter a number of
  milliseconds or an interval such as `10s`, `5m`, or `1h`. Bucket Size has
  priority over Buckets.

### Y axis

The Y axis represents metric values.

* **Unit** - Selects the unit used for axis labels and tooltip values.
* **Scale** - Supports linear scale and log scales with bases `2`, `10`, `32`,
  and `1024`.
* **Y-Min** and **Y-Max** - Override the automatic value range.
* **Decimals** - Overrides automatic decimal precision for axis labels.
* **Buckets** - Sets the number of value buckets for linear scale.
* **Bucket Size** - Sets an explicit value bucket size for linear scale. Bucket
  Size has priority over Buckets.
* **Split Buckets** - For log scales, splits each default log bucket into the
  specified number of buckets.
* **Remove zero values** - For log scales, hides the zero-value bucket.

## Display options

The **Display** tab controls colors, card appearance, and tooltip behavior.

### Colors

The heatmap panel supports two color modes:

* **spectrum** - Uses a color scheme such as Blues, Greens, Oranges, Spectral,
  or YlOrRd. Enable **Fill background** to color empty areas with the low end of
  the selected scheme.
* **opacity** - Uses one color and changes opacity based on the bucket count.
  The opacity scale can be `linear` or `sqrt`. When using `sqrt`, the exponent
  controls how quickly cards become opaque.

### Cards

* **Space** - Adds space between cards.
* **Round** - Rounds card corners.

### Tooltip

Tooltips show the bucket time, value range, and count. Optional tooltip settings
include:

* **Highlight cards** - Highlights the bucket under the cursor.
* **Series stats** - Shows how many values from each series are in the bucket.
* **Histogram** - Adds a compact histogram for the hovered time bucket.
* **Decimals** - Sets tooltip decimal precision.

## Query and configuration tips

* Start with automatic bucket settings, then set explicit bucket sizes when the
  heatmap is too coarse or too dense.
* For wide value ranges, try a log scale and use **Split Buckets** to add more
  detail between powers of the selected log base.
* If you see a **Data points outside time range** warning, check for timezone
  mismatches or a missing time filter in the query.
* If the panel shows **No data points**, verify that the query returns numeric
  datapoints for the selected time range.
