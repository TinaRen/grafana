+++
title = "Using InfluxDB in Grafana"
description = "Guide for using InfluxDB in Grafana"
keywords = ["grafana", "influxdb", "guide"]
type = "docs"
aliases = ["/datasources/influxdb"]
[menu.docs]
name = "InfluxDB"
parent = "datasources"
weight = 3
+++

# Using InfluxDB in Grafana

Grafana ships with very feature rich data source plugin for InfluxDB. Supporting a feature rich query editor, annotation and templating queries.

## Adding the data source
![](/img/docs/v2/add_Influx.jpg)

1. Open the side menu by clicking the the Grafana icon in the top header.
2. In the side menu under the `Dashboards` link you should find a link named `Data Sources`.

    > NOTE: If this link is missing in the side menu it means that your current user does not have the `Admin` role for the current organization.

3. Click the `Add new` link in the top header.

Name | Description
------------ | -------------
Name | The data source name, important that this is the same as in Grafana v1.x if you plan to import old dashboards.
Default | Default data source means that it will be pre-selected for new panels.
Url | The http protocol, ip and port of you influxdb api (influxdb api port is by default 8086)
Access | Proxy = access via Grafana backend, Direct = access directly from browser.
Database | Name of your influxdb database
User | Name of your database user
Password | Database user's password
Default group by time | Optional lower limit for the automatically calculated `GROUP BY time()` interval. Use the `>10s` syntax to keep Grafana from choosing a smaller interval than your write frequency.

 > Proxy access means that the Grafana backend will proxy all requests from the browser, and send them on to the Data Source. This is useful because it can eliminate CORS (Cross Origin Site Resource) issues, as well as eliminate the need to disseminate authentication details to the Data Source to the browser.

 > Direct access is still supported because in some cases it may be useful to access a Data Source directly depending on the use case and topology of Grafana, the user, and the Data Source.

You can enter more than one InfluxDB URL in the URL field by separating the URLs with commas. Grafana rotates through the list when it sends queries. Save & Test verifies the data source by running `SHOW MEASUREMENTS LIMIT 1`.


## Query Editor

![](/assets/img/blog/v2.6/influxdb_editor_v3.gif)

You find the InfluxDB editor in the metrics tab in Graph or Singlestat panel's edit mode. You enter edit mode by clicking the
panel title, then edit. The editor allows you to select metrics and tags.

The visual editor builds queries from the rows in the editor. New queries start with the `default` retention policy, a `value` field, `mean()` aggregation, `GROUP BY time($__interval)`, and `fill(null)`.

### Retention policies and measurements
The `FROM` row contains a retention policy selector and a measurement selector. The policy list is loaded with `SHOW RETENTION POLICIES on "<database>"`. Measurement names are quoted automatically unless you enter a regular expression such as `/cpu.*/`.

### Filter data (WHERE)
To add a tag filter click the plus icon to the right of the `WHERE` condition. You can remove tag filters by clicking on
the tag key and select `--remove tag filter--`.

**Regex matching**

You can type in regex patterns for metric names or tag filter values, be sure to wrap the regex pattern in forward slashes (`/`). Grafana
will automatically adjust the filter tag condition to use the InfluxDB regex match condition operator (`=~`).

When you add more than one tag filter, the editor inserts an `AND` condition. Click the condition segment to change it to `OR`. Tag values that are not regular expressions are quoted in the generated query.

### Field & Aggregation functions
In the `SELECT` row you can specify what fields and functions you want to use. If you have a
group by time you need an aggregation function. Some functions like derivative require an aggregation function.

The editor tries simplify and unify this part of the query. For example:
![](/img/docs/influxdb/select_editor.png)

The above will generate the following InfluxDB `SELECT` clause:

```sql
SELECT derivative(mean("value"), 10s) /10 AS "REQ/s" FROM ....
```

#### Select multiple fields
Use the plus button and select Field > field to add another SELECT clause. You can also
specify an asterix `*` to select all fields.

### Group By and fill
To group by a tag click the plus icon at the end of the GROUP BY row. Pick a tag from the dropdown that appears.
You can remove the group by by clicking on the `tag` and then click on the x icon.

The editor supports `GROUP BY time()`, tag grouping, and `fill()`. Supported fill values include `none`, `null`, `0`, `previous`, and `linear`. Removing the time grouping also removes aggregation and selector functions from the SELECT row because those functions require a time grouping in the visual editor.

Use a lower bound in the query options, for example `>60s`, when you use `fill(0)` or `fill(null)` and write metrics at a fixed interval. This keeps stacked graphs aligned without asking InfluxDB for more buckets than your data can populate.

### Sort order and series limits
Use `ORDER BY time` to choose ascending or descending time order. Use `LIMIT` to limit points returned per series and `SLIMIT` to limit the number of series returned by InfluxDB. Leave these fields empty when you do not want Grafana to add the corresponding clause.

### Text Editor Mode (RAW)
You can switch to raw query mode by clicking hamburger icon and then `Switch editor mode`.

> If you use Raw Query be sure your query at minimum have `WHERE $timeFilter`
> Also please always have a group by time and an aggregation function, otherwise InfluxDB can easily return hundreds of thousands
> of data points that will hang the browser.

When you switch from visual mode to raw query mode, Grafana copies the query generated by the visual editor into the text field. Template variables are interpolated before Grafana sends the query to InfluxDB.

### Alias patterns

- $m = replaced with measurement name
- $measurement = replaced with measurement name
- $1 - $9 = replaced with part of the measurement name, split on dots
- $col = replaced with column name
- $tag_hostname = replaced with the value of the hostname tag
- You can also use [[tag_hostname]] pattern replacement syntax

### Table query / raw data

![](/assets/img/blog/v2.6/table_influxdb_logs.png)

You can remove the group by time by clicking on the `time` part and then the `x` icon. You can
change the option `Format As` to `Table` if you want to show raw data in the `Table` panel.


## Templating
You can create a template variable in Grafana and have that variable filled with values from any InfluxDB metric exploration query.
You can then use this variable in your InfluxDB metric queries.

For example you can have a variable that contains all values for tag `hostname` if you specify a query like this
in the templating edit view.
```sql
SHOW TAG VALUES WITH KEY = "hostname"
```

You can also create nested variables. For example if you had another variable, for example `region`. Then you could have
the hosts variable only show hosts from the current selected region with a query like this:

```sql
SHOW TAG VALUES WITH KEY = "hostname"  WHERE region =~ /$region/
```

> Always use `regex values` or `regex wildcard` for All format or multi select format.

![](/img/docs/influxdb/templating_simple_ex1.png)

InfluxDB template queries use the same query endpoint as panel queries. Grafana parses unique values from the first result series. For InfluxDB 1.1 `SHOW TAG VALUES` responses, Grafana uses the returned tag value column instead of the tag key column.

### Ad hoc filters
InfluxDB supports Grafana's ad hoc filters. Grafana discovers tag keys with `SHOW TAG KEYS` and tag values with `SHOW TAG VALUES WITH KEY = "<key>"`. During panel queries, the active ad hoc filters for the data source are appended to the time filter with `AND`.

## Annotations
Annotations allows you to overlay rich event information on top of graphs.

An example query:

```SQL
SELECT title, description from events WHERE $timeFilter order asc
```

The annotation query is required. Use `$timeFilter` in the query so Grafana can replace it with the dashboard time range before sending the query to InfluxDB.

If the query returns more than one column, configure the column mappings in the annotation editor:

- **Title** maps the event title column. If it is blank, Grafana uses the first non-time and non-`sequence_number` column.
- **Tags** maps one or more comma-separated tag columns. Empty tag values are ignored, and comma-separated values inside a tag column are split into separate tags.
- **Text** maps the event text column.

