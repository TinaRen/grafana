+++
title = "JSON Model"
keywords = ["grafana", "dashboard", "documentation", "json", "model"]
type = "docs"
[menu.docs]
name = "JSON Model"
parent = "dashboard_features"
weight = 100
+++

# Dashboard JSON

A dashboard in Grafana is represented by a JSON object, which stores metadata of its dashboard. Dashboard metadata includes dashboard properties, metadata from rows, panels, template variables, panel queries, etc.

To view the JSON of a dashboard, follow the steps mentioned below:

  1. Go to a dashboard
  2. Click on `Manage dashboard` menu on the top navigation bar
  3. Select `View JSON` from the dropdown menu

## JSON fields

When a user creates a new dashboard, a new dashboard JSON object is initialized with the following fields:

> Note: In the following JSON, id is shown as null which is the default value assigned to it until a dashboard is saved. Once a dashboard is saved, an integer value is assigned to the `id` field.

```
{
  "id": null,
  "title": "New dashboard",
  "description": "A dashboard for important service metrics.",
  "tags": [],
  "style": "dark",
  "timezone": "browser",
  "editable": true,
  "hideControls": false,
  "graphTooltip": 1,
  "rows": [],
  "time": {
    "from": "now-6h",
    "to": "now"
  },
  "timepicker": {
    "time_options": [],
    "refresh_intervals": []
  },
  "templating": {
    "list": []
  },
  "annotations": {
    "list": []
  },
  "schemaVersion": 7,
  "version": 0,
  "links": []
}
```
Each field in the dashboard JSON is explained below with its usage:

| Name | Usage |
| ---- | ----- |
| **id** | unique dashboard id, an integer |
| **title** | current title of dashboard |
| **description** | optional dashboard description configured in dashboard settings |
| **tags** | tags associated with dashboard, an array of strings |
| **style** | theme of dashboard, i.e. `dark` or `light` |
| **timezone** | timezone of dashboard, i.e. `utc` or `browser` |
| **editable** | whether a dashboard is editable or not |
| **hideControls** | whether row controls on the left in green are hidden or not |
| **graphTooltip** | shared cursor behavior for graph panels: `0` is default, `1` is shared crosshair, `2` is shared crosshair and shared tooltip |
| **rows** | row metadata, see [rows section](#rows) for details |
| **time** | time range for dashboard, i.e. last 6 hours, last 7 days, etc |
| **timepicker** | timepicker metadata, see [timepicker section](#timepicker) for details |
| **templating** | templating metadata, see [templating section](#templating) for details |
| **annotations** | annotations metadata, see [annotations section](#annotations) for details |
| **schemaVersion** | version of the dashboard JSON schema; Grafana uses this when migrating older dashboards |
| **version** | dashboard revision number; it is incremented each time the dashboard is saved |
| **links** | dashboard links displayed in the dashboard submenu |

### rows

`rows` field consists of an array of JSON object representing each row in a dashboard, such as shown below:

```json
 "rows": [
    {
      "collapse": false,
      "editable": true,
      "height": "200px",
      "panels": [],
      "title": "New row"
    },
    {
      "collapse": true,
      "editable": true,
      "height": "300px",
      "panels": [],
      "title": "New row"
    }
  ]
```

Usage of the fields is explained below:

| Name | Usage |
| ---- | ----- |
| **collapse** | whether row is collapsed or not |
| **editable** | whether a row is editable or not |
| **height** | height of the row in pixels |
| **panels** | panels metadata, see [panels section](#panels) for details |
| **title** | title of row |

#### panels

Panels are the building blocks a dashboard. It consists of datasource queries, type of graphs, aliases, etc. Panel JSON consists of an array of JSON objects, each representing a different panel in a row. Most of the fields are common for all panels but some fields depends on the panel type. Following is an example of panel JSON representing a `graph` panel type:

```json
"panels": [
        {
          "aliasColors": {},
          "bars": false,
          "datasource": null,
          "description": "Shows memory usage for the selected hosts.",
          "editable": true,
          "error": false,
          "fill": 0,
          "grid": {
            "leftLogBase": 1,
            "leftMax": null,
            "leftMin": null,
            "rightLogBase": 1,
            "rightMax": null,
            "rightMin": null,
            "threshold1": null,
            "threshold1Color": "rgba(216, 200, 27, 0.27)",
            "threshold2": null,
            "threshold2Color": "rgba(234, 112, 112, 0.22)"
          },
          "id": 1,
          "legend": {
            "avg": false,
            "current": false,
            "max": false,
            "min": false,
            "show": true,
            "total": false,
            "values": false
          },
          "lines": true,
          "linewidth": 1,
          "links": [],
          "nullPointMode": "connected",
          "percentage": false,
          "pointradius": 5,
          "points": false,
          "renderer": "flot",
          "seriesOverrides": [],
          "span": 4,
          "stack": false,
          "steppedLine": false,
          "targets": [
            {
              "aggregator": "max",
              "alias": "$tag_instance_id",
              "currentTagKey": "",
              "currentTagValue": "",
              "downsampleAggregator": "avg",
              "downsampleInterval": "",
              "errors": {},
              "metric": "memory.percent-used",
              "refId": "A",
              "shouldComputeRate": false,
              "tags": {
                "app": "$app",
                "env": "stage",
                "instance_id": "*"
              }
            }
          ],
          "timeFrom": null,
          "timeShift": null,
          "title": "Memory Utilization",
          "tooltip": {
            "shared": true,
            "value_type": "cumulative"
          },
          "type": "graph",
          "x-axis": true,
          "y-axis": true,
          "y_formats": [
            "percent",
            "short"
          ]
        },
        {
          "aliasColors": {},
          "bars": false,
          "datasource": null,
          "editable": true,
          "error": false,
          "fill": 0,
          "grid": {
            "leftLogBase": 1,
            "leftMax": null,
            "leftMin": null,
            "rightLogBase": 1,
            "rightMax": null,
            "rightMin": null,
            "threshold1": null,
            "threshold1Color": "rgba(216, 200, 27, 0.27)",
            "threshold2": null,
            "threshold2Color": "rgba(234, 112, 112, 0.22)"
          },
          "id": 2,
          "legend": {
            "avg": false,
            "current": false,
            "max": false,
            "min": false,
            "show": true,
            "total": false,
            "values": false
          },
          "lines": true,
          "linewidth": 1,
          "links": [],
          "nullPointMode": "connected",
          "percentage": false,
          "pointradius": 5,
          "points": false,
          "renderer": "flot",
          "seriesOverrides": [],
          "span": 4,
          "stack": false,
          "steppedLine": false,
          "targets": [
            {
              "aggregator": "avg",
              "alias": "$tag_instance_id",
              "currentTagKey": "",
              "currentTagValue": "",
              "downsampleAggregator": "avg",
              "downsampleInterval": "",
              "errors": {},
              "metric": "memory.percent-cached",
              "refId": "A",
              "shouldComputeRate": false,
              "tags": {
                "app": "$app",
                "env": "prod",
                "instance_id": "*"
              }
            }
          ],
          "timeFrom": null,
          "timeShift": null,
          "title": "Memory Cached",
          "tooltip": {
            "shared": true,
            "value_type": "cumulative"
          },
          "type": "graph",
          "x-axis": true,
          "y-axis": true,
          "y_formats": [
            "short",
            "short"
          ]
        },
```

Usage of each field is explained below:

| Name | Usage |
| ---- | ----- |
| **id** | unique panel id within the dashboard |
| **title** | panel title shown in the panel header |
| **description** | optional Markdown help text shown from the panel info corner and inspector |
| **type** | panel plugin type, for example `graph`, `singlestat`, or `table` |
| **datasource** | datasource name for the panel queries; `null` uses the default datasource |
| **targets** | query definitions sent to the datasource |
| **span** | panel width in the 12-column dashboard grid |
| **height** | panel height; if omitted, the row height is used |
| **editable** | whether the panel can be edited |
| **error** | panel error state shown in the panel info corner |
| **links** | drilldown links configured in the panel General tab |
| **timeFrom** | optional panel-specific relative time override |
| **timeShift** | optional panel-specific time shift |
| **tooltip** | graph panel tooltip settings |

#### Panel descriptions and links

Panel descriptions are configured on the panel editor's **General** tab. The
description supports Markdown and links, and Grafana interpolates template
variables before rendering it in the panel information view. Use this field for
short runbook notes, query assumptions, or links that explain why the panel
exists.

When a panel has an error, a description, or panel links, Grafana shows an
information corner in the panel. Hovering the corner opens a tooltip that is
pinned inside the browser window; clicking it opens the panel inspector on the
**Panel Description** tab. Panel errors take precedence over descriptions, and
descriptions are shown before panel links.

Panel links can point to another dashboard or to an absolute URL. Links can keep
the current time range, include current template variable values, append custom
URL parameters, and open in a new tab.

### timepicker

Description: TODO

```json
"timepicker": {
    "collapse": false,
    "enable": true,
    "notice": false,
    "now": true,
    "refresh_intervals": [
      "5s",
      "10s",
      "30s",
      "1m",
      "5m",
      "15m",
      "30m",
      "1h",
      "2h",
      "1d"
    ],
    "status": "Stable",
    "time_options": [
      "5m",
      "15m",
      "1h",
      "3h",
      "6h",
      "12h",
      "24h",
      "2d",
      "3d",
      "4d",
      "7d",
      "30d"
    ],
    "type": "timepicker"
  }
```

Usage of the fields is explained below:

| Name | Usage |
| ---- | ----- |
| **collapse** | whether timepicker is collapsed or not |
| **enable** | whether timepicker is enabled or not |
| **notice** | TODO |
| **now** | TODO |
| **refresh_intervals** | TODO |
| **status** | TODO |
| **time_options** | TODO |
| **type** | TODO |

### templating

`templating` fields contains array of template variables with their saved values along with some other metadata, for example:

```json
 "templating": {
    "enable": true,
    "list": [
      {
        "allFormat": "wildcard",
        "current": {
          "tags": [],
          "text": "prod",
          "value": "prod"
        },
        "datasource": null,
        "includeAll": true,
        "name": "env",
        "options": [
          {
            "selected": false,
            "text": "All",
            "value": "*"
          },
          {
            "selected": false,
            "text": "stage",
            "value": "stage"
          },
          {
            "selected": false,
            "text": "test",
            "value": "test"
          }
        ],
        "query": "tag_values(cpu.utilization.average,env)",
        "refresh": false,
        "refresh": false,
        "type": "query"
      },
      {
        "allFormat": "wildcard",
        "current": {
          "text": "apache",
          "value": "apache"
        },
        "datasource": null,
        "includeAll": false,
        "multi": false,
        "multiFormat": "glob",
        "name": "app",
        "options": [
          {
            "selected": true,
            "text": "tomcat",
            "value": "tomcat"
          },
          {
            "selected": false,
            "text": "cassandra",
            "value": "cassandra"
          }
        ],
        "query": "tag_values(cpu.utilization.average,app)",
        "refresh": false,
        "regex": "",
        "type": "query"
      }
    ]
  }
```

Usage of the above mentioned fields in the templating section is explained below:

| Name | Usage |
| ---- | ----- |
| **enable** | whether templating is enabled or not |
| **list** | an array of objects representing, each representing one template variable |
| **allFormat** | format to use while fetching all values from datasource, eg: `wildcard`, `glob`, `regex`, `pipe`, etc. |
| **current** | shows current selected variable text/value on the dashboard |
| **datasource** | shows datasource for the variables |
| **includeAll** | whether all value option is available or not |
| **multi** | whether multiple values can be selected or not from variable value list |
| **multiFormat** | format to use while fetching timeseries from datasource |
| **name** | name of variable |
| **options** | array of variable text/value pairs available for selection on dashboard |
| **query** | datasource query used to fetch values for a variable |
| **refresh** | TODO |
| **regex** | TODO |
| **type** | type of variable, i.e. `custom`, `query` or `interval` |

### annotations

TODO
