+++
title = "Using Elasticsearch in Grafana"
description = "Guide for using Elasticsearch in Grafana"
keywords = ["grafana", "elasticsearch", "guide"]
type = "docs"
aliases = ["/datasources/elasticsearch"]
[menu.docs]
name = "Elasticsearch"
parent = "datasources"
weight = 3
+++

# Using Elasticsearch in Grafana

Grafana ships with advanced support for Elasticsearch. You can do many types of
simple or complex elasticsearch queries to visualize logs or metrics stored in elasticsearch. You can
also annotate your graphs with log events stored in elasticsearch.

## Adding the data source

![](/img/docs/v2/add_Graphite.jpg)

1. Open the side menu by clicking the the Grafana icon in the top header.
2. In the side menu under the `Dashboards` link you should find a link named `Data Sources`.

    > NOTE: If this link is missing in the side menu it means that your current user does not have the `Admin` role for the current organization.

3. Click the `Add new` link in the top header.
4. Select `Elasticsearch` from the dropdown.

Name | Description
------------ | -------------
Name | The data source name, important that this is the same as in Grafana v1.x if you plan to import old dashboards.
Default | Default data source means that it will be pre-selected for new panels.
Url | The http protocol, ip and port of you elasticsearch server.
Access | Proxy = access via Grafana backend, Direct = access directly from browser.

Proxy access means that the Grafana backend will proxy all requests from the browser, and send them on to the Data Source. This is useful because it can eliminate CORS (Cross Origin Site Resource) issues, as well as eliminate the need to disseminate authentication details to the Data Source to the browser.

Direct access is still supported because in some cases it may be useful to access a Data Source directly depending on the use case and topology of Grafana, the user, and the Data Source.

### Direct access
If you select direct access you must update your Elasticsearch configuration to allow other domains to access
Elasticsearch from the browser. You do this by specifying these to options in your **elasticsearch.yml** config file.

    http.cors.enabled: true
    http.cors.allow-origin: "*"

### Index settings

![](/img/docs/elasticsearch/elasticsearch_ds_details.png)

Here you can specify a default for the `time field` and specify the name of your elasticsearch index. You can use
a time pattern for the index name or a wildcard.

## Metric Query editor

![](/img/docs/elasticsearch/query_editor.png)

The Elasticsearch query editor allows you to select multiple metrics and group by multiple terms or filters. Use the plus and minus icons to the right to add / remove
metrics or group bys. Some metrics and group by have options, click the option text to expand the the row to view and edit metric or group by options.

## Pipeline metrics

If you have Elasticsearch 2.x and Grafana 2.6 or above then you can use pipeline metric aggregations like
**Moving Average** and **Derivative**. Elasticsearch pipeline metrics require another metric to be based on. Use the eye icon next to the metric
to hide metrics from appearing in the graph. This is useful for metrics you only have in the query to be used
in a pipeline metric.

![](/img/docs/elasticsearch/pipeline_metrics_editor.png)

## Histogram aggregations

Use the `Histogram` bucket aggregation to group documents by numeric field
ranges instead of by time. In the query editor, add a `Group by` row, select
`Histogram`, choose a numeric field, and set:

- `Interval` - The numeric width of each bucket.
- `Min Doc Count` - The minimum document count required for a bucket to be
  returned. The query builder defaults this to `1` in the editor.

Grafana sends the selected field, interval, and minimum document count to
Elasticsearch. Dashboard JSON can also contain a `missing` setting for histogram
buckets; when present, Grafana forwards it to Elasticsearch so documents without
the selected field are treated as if they had that value.

### Visualizing histograms as a heatmap

The [Heatmap panel]({{< relref "features/panels/heatmap.md" >}}) can render an
Elasticsearch histogram as value buckets over time. Configure the query so the
deepest bucket is a `Date Histogram`:

1. Add a `Count` metric.
2. Add a `Histogram` bucket aggregation for the numeric value field.
3. Add a `Date Histogram` bucket aggregation for the time field.
4. In the heatmap panel **Axes** tab, set `Data format` to `ES histogram`.

For example, to chart request size distribution, group by `Histogram` on
`bytes` with interval `1000`, then group by `Date Histogram` on `@timestamp`.
Grafana uses the histogram bucket keys as Y bucket bounds and the date histogram
buckets as X bucket bounds.

Leave the query `Alias` field empty, or set it to the histogram field template,
such as `{{bytes}}`. The heatmap panel parses the series name as the numeric Y
bucket bound.

If the final bucket aggregation is not a `Date Histogram`, Elasticsearch results
are returned as document-style rows instead of time series and cannot be used by
the heatmap `ES histogram` format.

## Templating

The Elasticsearch datasource supports two types of queries you can use to fill template variables with values.

### Possible values for a field

```json
{"find": "terms", "field": "@hostname"}
```

### Fields filtered by type
```json
{"find": "fields", "type": "string"}
```

### Fields filtered by type, with filter
```json
{"find": "fields", "type": "string", "query": <lucene query>}
```

### Multi format / All format
Use lucene format.


