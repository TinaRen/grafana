+++
title = "Templating"
keywords = ["grafana", "templating", "documentation", "guide"]
type = "docs"
[menu.docs]
name = "Templating"
parent = "dashboard_features"
weight = 1
+++

# Templating

<img class="no-shadow" src="/img/docs/v2/templating_var_list.png">

Dashboard Templating allows you to make your Dashboards more interactive and dynamic.

They’re one of the most powerful and most used features of Grafana, and they’ve recently gotten even more attention in Grafana 2.0 and Grafana 2.1.

You can create Dashboard Template variables that can be used practically anywhere in a Dashboard: data queries on individual Panels (within the Query Editor), the names in your legends, or titles in Panels and Rows.

You can configure Dashboard Templating by clicking the dropdown cog on the top of the Dashboard while viewing it.


## Variable types

There are several different types of Template variables: query, custom, interval, constant, data source, and ad hoc filters.

They can all be used to create dynamic variables that you can use throughout the Dashboard, but they differ in how they get the data for their values and which dashboard features they support.


### Query

 > Note: The Query type is Data Source specific. Please consult the appropriate documentation for your particular Data Source.

Query is the most common type of Template variable. Use the `Query` template type to generate a dynamic list of variables, simply by allowing Grafana to explore your Data Source metric namespace when the Dashboard loads.

For example a query like `prod.servers.*` will fill the variable with all possible values that exists in that wildcard position (in the case of the Graphite Data Source).

You can even create nested variables that use other variables in their definition. For example `apps.$app.servers.*` uses the variable $app in its own query definition.

You can utilize the special ** All ** value to allow the Dashboard user to query for every single Query variable returned. Grafana will automatically translate ** All ** into the appropriate format for your Data Source.

#### Multi-select
As of Grafana 2.1, it is now possible to select a subset of Query Template variables (previously it was possible to select an individual value or 'All', not multiple values that were less than All). This is accomplished via the Multi-Select option. If enabled, the Dashboard user will be able to enable and disable individual variables.

The Multi-Select functionality is taken a step further with the introduction of Multi-Select Tagging. This functionality allows you to group individual Template variables together under a Tag or Group name.

For example, if you were using Templating to list all 20 of your applications, you could use Multi-Select Tagging to group your applications by function or region or criticality, etc.

 > Note: Multi-Select Tagging functionality is currently experimental but is part of Grafana 2.1. To enable this feature click the enable icon when editing Template options for a particular variable.

<img class="no-shadow" src="/img/docs/v2/template-tags-config.png">

Grafana gets the list of tags and the list of values in each tag by performing two queries on your metric namespace.

The Tags query returns a list of Tags.

The Tag values query returns the values for a given Tag.

Note: a proof of concept shim that translates the metric query into a SQL call is provided. This allows you to maintain your tag:value mapping independently of your Data Source.

Once configured, Multi-Select Tagging provides a convenient way to group and your template variables, and slice your data in the exact way you want. The Tags can be seen on the right side of the template pull-down.

![](/img/docs/v2/multi-select.gif)

### Interval

Use the `Interval` type to create Template variables around time ranges (eg. `1m`,`1h`, `1d`). There is also a special `auto` option that will change depending on the current time range, you can specify how many times the current time range should be divided to calculate the current `auto` range.

![](/img/docs/v2/templated_variable_parameter.png)

### Custom

Use the `Custom` type to manually create Template variables around explicit values that are hard-coded into the Dashboard, and not dependent on any Data Source. You can specify multiple Custom Template values by separating them with a comma.

### Constant

Use the `Constant` type to define a hidden constant value. Constant variables are useful for metric prefixes or other fixed values in dashboards that you want to share or export.

When a dashboard is exported, constant variables are converted to import inputs so the value can be supplied when the dashboard is imported.

### Data source

Use the `Datasource` type to quickly change the data source for an entire dashboard. The variable's **Type** option selects a data source plugin type, such as Graphite or Prometheus, and the value dropdown is populated with configured data source instances of that type.

You can use the optional **Instance name filter** regex to limit which data source instances are available. For example, `/^prod/` only shows instances whose names start with `prod`.

To use a data source variable, set a panel's data source to the variable name, for example `$datasource`. Grafana resolves the variable before loading the panel data source, so every panel that uses `$datasource` switches when the dashboard user selects another value.

Data source variables do not support multi-value or the `All` option. Use a query or custom variable when you need multiple selected values for repeating rows or panels.

### Ad hoc filters

Use the `Ad hoc filters` type to add key/value filters that are automatically applied to all metric queries that use the selected data source. The selected data source must support ad hoc filters.

##  Repeating Panels and Repeating Rows

Template Variables can be very useful to dynamically change what you're visualizing on a given panel. Sometimes, you might want to create entire new Panels (or Rows) based on what Template Variables have been selected. This is now possible in Grafana 2.1.

Repeating rows and panels use the selected options from a variable. If the variable has the `All` option selected, Grafana creates one row or panel for each concrete option except `All`.

For multiple repeated rows or panels, use a variable type that supports **Multi-value** or **Include All option**, such as `Query` or `Custom`. A single-value variable can only produce one repeated row or panel, and data source variables are intended for switching the panel data source rather than driving repeats.

Example:

1. Create a `Query` variable named `server` that returns server names.
2. Enable **Multi-value** or **Include All option** on the variable.
3. Set a panel's **Repeat Panel** option to `server`.
4. Use `$server` in the panel title or query. Each repeated panel receives one selected server value.

## Screencast - Templated Graphite Queries

<iframe width="561" height="315" src="//www.youtube.com/embed/FhNUrueWwOk?list=PLDGkOdUX1Ujo3wHw9-z5Vo12YLqXRjzg2" frameborder="0" allowfullscreen></iframe>

