---
description: Build Cloudflare queries and use SQL, GraphQL, Analytics Engine SQL, and Workers telemetry in Grafana.
keywords:
  - grafana
  - cloudflare
  - queries
  - sql
  - graphql
labels:
  products:
    - enterprise
    - oss
menuTitle: Query editor
title: Cloudflare query editor
weight: 200
review_date: 2026-11-10
---

# Cloudflare query editor

The Cloudflare query editor lets you build queries visually or write requests for Cloudflare's analytics and observability APIs. You can use the editor in dashboards and Explore.

Before you begin, [configure a Cloudflare data source](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/configure/) and select it in a panel or Explore. Choose a time range that contains data for your account.

## Choose a query language

Use **Query language** to select the API and editor for your query.

| Query language           | Use case                                                           |
| ------------------------ | ------------------------------------------------------------------ |
| **Builder**              | Discover SQL datasets and select metrics, dimensions, and filters. |
| **SQL**                  | Write parameterized queries for the Cloudflare SQL API beta.       |
| **GraphQL**              | Write queries for the GraphQL Analytics API.                       |
| **Analytics Engine SQL** | Query the original Analytics Engine SQL endpoint.                  |
| **Workers telemetry**    | Run a JSON request from Workers Query Builder.                     |

Use **Start with** to load an example. Selecting an example replaces the relevant query settings. Switching between Builder and SQL doesn't convert the builder query into editable SQL. To inspect generated SQL, open the [Query inspector](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/visualizations/panels-visualizations/panel-inspector/#inspect-query-request-and-response-data).

## Set the scope and result format

Common settings control which account or zone Grafana queries and how it displays results.

| Setting                                  | Description                                                                                                                                                                |
| ---------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Scope**                                | Select **Account** or **Zone** according to the dataset or GraphQL query.                                                                                                  |
| **Account**                              | Overrides the configured default account. Leave empty to use the default.                                                                                                  |
| **Zone ID**                              | Overrides the configured default zone for zone-scoped queries.                                                                                                             |
| **Browse accounts** and **Browse zones** | Load resources available to the token. You can enter an ID directly if browsing permissions aren't available.                                                              |
| **Result format**                        | Select **Time series**, **Table**, **Logs**, or **Trace**. Selecting a format doesn't change the underlying API or dataset.                                                |
| **Time column**                          | Identifies the timestamp column for time series and logs. Use dot notation for flattened GraphQL fields, such as `dimensions.datetimeMinute`.                              |
| **Series dimensions**                    | For raw queries, lists result columns to use as time series labels, separated by commas. Include numeric dimensions here when they identify a series rather than a metric. |

Time series queries need a timestamp column and numeric values. Use **Table** for results without timestamps. Native trace waterfalls require span data; refer to [Logs and traces](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/logs-and-traces/).

## Build a query

The builder uses the Cloudflare SQL dataset catalog to populate dataset, field, and aggregation choices.

1. Select **Builder** in **Query language**.
1. Select an example in **Start with**, or choose a **Dataset**.
1. Verify the account or zone scope and **Time column**.
1. For metrics, select an **Aggregation** and, when required, a numeric **Metric field**. Set an **Alias** to name the result.
1. Optionally add metrics and select **Group by** fields. Each unique dimension combination becomes a labeled series.
1. Optionally click **Add filter** and choose a **Filter field**, **Operator**, and **Value**.
1. Set **Row limit** and click **Run Cloudflare query**.

The builder supports `count`, `sum`, `avg`, `min`, `max`, `p50`, `p90`, `p95`, and `p99`. State datasets can restrict the available aggregations. Use distinct aliases for metrics and avoid aliases that duplicate dimension or time column names.

Filters support comparisons, `LIKE`, `NOT LIKE`, `IS NULL`, and `IS NOT NULL`. Multiple filters are combined with `AND`. The builder sends filter values as SQL parameters. Use SQL mode for more complex conditions.

Expand **Dataset schema** to inspect column names, types, and descriptions. Click **Discover custom attributes** to load available custom attributes for the selected dataset. Click **Reload catalog** to refresh the dataset list after changing product access or configuration.

The SQL API accounts for sampling in `count`, `sum`, and `avg`. The builder also uses explicit sampling weights for supported percentile metrics. Sampling can still affect precision and the events available for log and trace searches.

## Write SQL queries

Select **SQL** to query the Cloudflare SQL API beta. Use schema-qualified names from the dataset catalog, such as `events.workersInvocations` or `logs.workersLogs`.

The following query charts requests for one Worker. Select **Time series** and set **Time column** to `time`:

```sql
SELECT $__timeGroup(timestamp) AS time,
       COUNT(*) AS requests
FROM events.workersInvocations
WHERE scriptName = $worker_name
GROUP BY time
ORDER BY time
LIMIT 1000
```

Set **SQL parameters (JSON)** to the following object:

```json
{
  "worker_name": "example-worker"
}
```

Replace `example-worker` with your Worker name. The `$worker_name` placeholder is a named SQL parameter, separate from Grafana dashboard variables. To use a dashboard variable as its value, refer to [Template variables](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/template-variables/).

The plugin supplies account or zone scope and the selected Grafana time range in the API request. You don't need to add account, zone, or time predicates to this SQL query. Use one read-only query per request. For supported functions and syntax, refer to the [Cloudflare SQL reference](https://developers.cloudflare.com/analytics/sql-api/sql-reference/).

### SQL macros

Macros expand on the Grafana server before it sends the query to Cloudflare.

| Macro                          | Description                                                                                                                                          |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `$__timeGroup(timestamp)`      | Groups a timestamp column into buckets using the Grafana query interval, with a minimum of one second. Replace `timestamp` with a column identifier. |
| `$__interval_s`                | Expands to the query interval in seconds.                                                                                                            |
| `$__interval_ms`               | Expands to the query interval in milliseconds.                                                                                                       |
| `$__fromTime` and `$__toTime`  | Expand to quoted UTC date-time strings for the start and end of the selected time range.                                                             |
| `$__from_s` and `$__to_s`      | Expand to Unix timestamps in seconds for the selected time range.                                                                                    |
| `$__accountId` and `$__zoneId` | Expand to quoted account and zone IDs for the query.                                                                                                 |

### Editor shortcuts

The SQL editor provides completion for datasets, columns, macros, and dashboard variables.

- Press **Ctrl+Space** to open suggestions.
- Press **Shift+Enter** or **Ctrl+Enter** on Windows and Linux, or **Shift+Enter** or **Cmd+Enter** on macOS, to run the query.

## Query GraphQL analytics

Select **GraphQL** to use the Cloudflare GraphQL Analytics API. The plugin provides `accountTag`, `zoneTag`, `from`, and `to` variables from the selected scope and Grafana time range. The `from` and `to` values are UTC date-time strings.

The following query returns Worker request and error counts:

```graphql
query Workers($accountTag: string!, $from: Time!, $to: Time!) {
  viewer {
    accounts(filter: { accountTag: $accountTag }) {
      workersInvocationsAdaptive(
        limit: 1000
        filter: { datetime_geq: $from, datetime_leq: $to }
        orderBy: [datetimeMinute_ASC]
      ) {
        dimensions {
          datetimeMinute
          scriptName
        }
        sum {
          requests
          errors
        }
      }
    }
  }
}
```

Use the following result settings:

| Setting               | Value                                        |
| --------------------- | -------------------------------------------- |
| **Result format**     | **Time series**                              |
| **Result data path**  | `viewer.accounts.workersInvocationsAdaptive` |
| **Time column**       | `dimensions.datetimeMinute`                  |
| **Series dimensions** | `dimensions.scriptName`                      |

The result data path is relative to the GraphQL response's `data` object. The plugin traverses arrays automatically and flattens nested field names using dots.

Use **Additional variables (JSON)** for other variables declared by your query. The plugin supplies the four built-in variables, so additional variables don't override them. Set **Operation name** when a document contains multiple query operations. Mutations and subscriptions aren't supported.

For datasets, permissions, and query examples, refer to the [Cloudflare GraphQL Analytics API documentation](https://developers.cloudflare.com/analytics/graphql-api/).

## Query Analytics Engine

Select **Analytics Engine SQL** to use the original Analytics Engine SQL endpoint. This endpoint uses a different SQL dialect and dataset names from the unified SQL API.

The following query counts sampled events. Select **Time series** and set **Time column** to `time`:

```sql
SELECT $__timeGroup(timestamp) AS time,
       SUM(_sample_interval) AS events
FROM <DATASET_NAME>
WHERE timestamp >= toDateTime($__fromTime)
  AND timestamp <= toDateTime($__toTime)
GROUP BY time
ORDER BY time
LIMIT 1000
```

Replace _`<DATASET_NAME>`_ with your Analytics Engine dataset name. Keep the explicit time filter: this endpoint doesn't use the automatic request time range of the SQL API. Use `_sample_interval` to account for Analytics Engine sampling where required by your aggregation. Refer to [Analytics Engine SQL API](https://developers.cloudflare.com/analytics/analytics-engine/sql-api/).

To query an Analytics Engine dataset through the unified API instead, select **SQL** and use its catalog name, such as `events.analyticsEngine.<DATASET_NAME>`.

## Query Workers telemetry

Select **Workers telemetry** to run JSON requests from [Workers Query Builder](https://developers.cloudflare.com/workers/observability/query-builder/).

This query language uses an endpoint that currently requires **Workers Observability Write**. The SQL-backed log and trace examples use a different endpoint. For permission details, refer to [Select token permissions](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/configure/#select-token-permissions).

The following request retrieves Worker log events:

```json
{
  "view": "events",
  "limit": 100,
  "parameters": {
    "datasets": ["cloudflare-workers"],
    "filters": [],
    "calculations": []
  }
}
```

Select **Logs**, set **Result data path** to `events.events`, and set **Time column** to `timestamp`. The plugin supplies the selected Grafana time range as the request's `timeframe` and adds a query ID when one isn't provided. Use a JSON object for the request.

For native trace waterfalls, use the SQL-backed trace example described in [Logs and traces](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/logs-and-traces/).

## Next steps

- [Explore logs and traces](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/logs-and-traces/).
- [Use dashboard variables in queries](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/template-variables/).
- [Troubleshoot query results](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/troubleshooting/).
