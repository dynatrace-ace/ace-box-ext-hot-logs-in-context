# Analyze Logs in Context Introduction

The goal of the following Hands-on labs is to dive deeper into DQL functionality and advanced query building. We'll utilize different apps to visualize logging data in use-case specific ways. Our labs will focus on log analysis use-cases and utilize the Dynatrace entity model for means of searching, filtering, trends, and finding answers in the sea of log content within a typical enterprise.

After completing the labs, the instructor will review the environment settings and discuss how to configure rules for ingest, processing, custom attributes, and more.

Any dashboards or notebooks created throughout the labs can be downloaded from this environment and imported into another for later review. Notebooks are the preferred app for saving DQL queries as the results will be attached to the document and persisted on import unless the query is explicitly re-ran.

### Explore Logs

Dynatrace recently introduced the log & metric explorer for an easy point and click way to quickly find log records or build charts through metrics wihtout writing DQL. Let's begin by exploring some recent log records:

1. Open the notebooks app and click the `+` icon and select `Logs` in the explore section of this menu. This will open the latest 20 log records found within the entire environment. 
2. Change the status to `ERROR` and click `Run`.
3. Change the limit to `100`.
4. Within the Explore Logs section, click the `+` and select Summarize. Change `All Records` to `k8s.deployment.name` and click `Run`.
5. Within the resulting table, click the `count` column and select `Sort Descending`.
6. At the top of the section you'll find options for timeframe, hide/show input, Options, and an 3 dot menu for additional options. Select the 3 dot menu and choose `Create DQL Section`. Based on the filters we applied Dynatrace will generate a new section of the resulting DQL. 

```
fetch logs
| filter in(status, array("ERROR"))
| summarize count = count(), by:{k8s.deployment.name}
| limit 100
```
Next, we'll review the DQL fundamental commands and introduce advanced query building throughout the remainder of the session.


### DQL Basics Review

1. Within the same notebook, click the `+` and select `query grail` to open a new section to build dql queries.

There are 4 commands we can begin with: Data, Describe, Fetch, and Timeseries.

- **Data**: The data command is generating sample data during query runtime. It is intended to test and document query scenarios based on a small, exemplary dataset.
- **Describe**: Describes the on-read schema extraction definition for a given data object. It returns the specified fields and their consecutive datatypes.
- **Fetch**: Loads data from the specified resource.
  - Logs, BizEvents, dt.entity, Events
- **Timeseries**: combines loading, filtering and aggregating metrics data into a time series output.

2. To retrieve log records for the default timeframe of 2 hours input the following query:

```
fetch logs
```

Query results are limited to 1000 records by default. You can also use `scanLimitGBytes` and/or `samplingratio` as parameters to your fetch command to limit the result set. Using these optional parameters are a good practice when developing a query.

### Timeframe and basic filters

Timeframe can optionally be directly included in the query. If it is excluded the global timeframe selector will be inherited by the log viewer.

To override the global timeframe selector try the following query using `from:`

```
fetch logs, scanLimitGBytes: 500, samplingRatio: 1000, from: now() -2h
| sort timestamp desc
```

Now, let's try finding all log records where the 'status' = error:

```
fetch logs, scanLimitGBytes: 500, samplingRatio: 1000, from: now() -2h
| filter status == "ERROR"
| sort timestamp desc
```

**Note: field values are case-sensitve**

Combine filters with AND/OR logic:

Show all logs related to any kubernetes container & have log level = error:

```
fetch logs, scanLimitGBytes: 500, samplingRatio: 1000, from: now() -2h
| filter isNotNull(k8s.container.name) AND status == "ERROR"
| sort timestamp desc
```

In the above query the isNotNull() is used to ensure the k8s property of container exists and is not null.

### Clean up results with field selectors

`fields:` command allows you to select which fields to display results for.

Starting with the DQL Query:

```
fetch logs, scanLimitGBytes: 500, samplingRatio: 1000, from: now() -2h
| filter isNotNull(k8s.container.name) AND status == "ERROR"
| sort timestamp desc
```

Add the `fields:` command to the query and only select the following fields:

- timestamp
- content
- k8s.container.name
- k8s.namespace.name

The resulting query should look like:

```
fetch logs, scanLimitGBytes: 500, samplingRatio: 1000, from: now() -2h
| filter isNotNull(k8s.container.name) AND status == "ERROR"
| fields timestamp, content, k8s.container.name, k8s.namespace.name
| sort timestamp desc
```

You can also label your columns or create referenceable variables inside your query:

```
fetch logs, scanLimitGBytes: 500, samplingRatio: 1000, from: now() -2h
| filter isNotNull(k8s.container.name) AND status == "ERROR"
| fields timestamp, content, containerName = k8s.container.name, namespace = k8s.namespace.name
| sort timestamp desc
```

Additional Field commands:

- fieldsAdd
- fieldsRemove
- fieldsRename
- fieldsKeep

Helpful Documentation Reference:

- [DQL Commands](https://docs.dynatrace.com/docs/platform/grail/dynatrace-query-language/commands)
- [DQL Reference](https://docs.dynatrace.com/docs/platform/grail/dynatrace-query-language/dql-reference)
- [DQL Data Types](https://docs.dynatrace.com/docs/platform/grail/dynatrace-query-language/data-types)
- [DQL Best Practices](https://docs.dynatrace.com/docs/platform/grail/dynatrace-query-language/dql-best-practices)
- [Dynatrace Pattern Language](https://docs.dynatrace.com/docs/platform/grail/dynatrace-pattern-language)
