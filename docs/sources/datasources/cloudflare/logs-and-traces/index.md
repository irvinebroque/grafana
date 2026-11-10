---
description: Search Cloudflare logs, view log context and volume, and investigate Workers traces in Grafana Explore.
keywords:
  - grafana
  - cloudflare
  - workers
  - logs
  - traces
labels:
  products:
    - enterprise
    - oss
menuTitle: Logs and traces
title: Cloudflare logs and traces
weight: 250
review_date: 2026-11-10
---

# Cloudflare logs and traces

Use the Cloudflare data source in Explore to search logs, investigate surrounding events, and follow Worker logs to trace waterfalls.

## Before you begin

Before you query logs and traces, ensure you have the following:

- A [configured Cloudflare data source](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/configure/) with access to the account you want to query.
- Logs or traces enabled for your Cloudflare application and retained for the selected time range.
- An API token with the read permissions required by the observability datasets you query.

For data collection settings, refer to [Workers logs](https://developers.cloudflare.com/workers/observability/logs/) and [Workers traces](https://developers.cloudflare.com/workers/observability/traces/).

## Search logs

The builder includes examples for Workers and Containers logs.

1. Open **Explore** and select your Cloudflare data source.
1. In **Start with**, select **Workers · logs** or **Containers · logs**.
1. Verify the **Account** and selected time range.
1. Optionally add filters, such as `scriptName = example-worker`.
1. Click **Run Cloudflare query**.
1. Expand a log row to inspect its fields.

The examples select the log dataset and **Logs** result format. Use **Row limit** to control the number of displayed rows. A row-limit warning indicates that the result might be incomplete.

### Filter from log details

For builder log queries, string fields in log details provide include and exclude filter actions. Click an action to add its field and value to the builder filters. Repeat the action to remove the filter, or select the opposite action to change it.

These actions apply literal field values. You can also edit the resulting filters in the query builder. Raw SQL, GraphQL, and Workers telemetry queries don't support this automatic query rewriting.

### View log context

Use the log context action in Explore to view surrounding Workers or Containers logs. The plugin scopes context using available request, Worker, container, or placement identifiers. The default window is 60 seconds before and after the selected row; you can adjust the context window in Explore.

Context is available for `logs.workersLogs` and `logs.containerLogs`. Other log datasets don't provide this integration.

### View log volume

For Workers and Containers builder log queries, Explore can display log volume across the selected time range. The volume query is separate from the displayed log rows, so changing the row limit doesn't define the histogram's time range. If the volume result reaches a limit, treat the histogram as incomplete and narrow the time range.

## Open a trace from a Worker log

Worker logs can include a trace ID that links the event to its spans.

1. Expand a Worker log row that contains a trace ID.
1. Click **View trace**.
1. Inspect the trace waterfall and select spans to view their attributes.

The trace link preserves the query's account. A trace might be unavailable because it wasn't sampled, is outside retention, or hasn't finished ingesting.

## Search traces

Use the trace example to search for Worker traces or load a known trace ID.

1. Select **Workers · distributed trace** in **Start with**.
1. Verify the **Account** and time range.
1. Add filters to narrow the search, then click **Run Cloudflare query**.
1. Open a trace from the results to inspect its spans.

To load a specific trace directly, enter its ID in **Trace ID** and run the query. Trace search lists root spans; loading a trace retrieves the associated span waterfall. A matching trace can have more spans than the search result shows.

## Query other log datasets

Other enabled Cloudflare log datasets can appear in the SQL catalog. Select the dataset and its timestamp and message fields in **Columns**, and verify **Time column** and **Result format**. Their schema can differ from Workers and Containers logs.

Catalog discovery doesn't enable Workers-specific context, volume, or trace correlation for every log dataset. Use **Table** or an SQL query with explicit column selection when the dataset fields don't map to the log view you need.

## Next steps

- [Add log events as annotations](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/annotations/).
- [Use other query languages](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/query-editor/).
- [Troubleshoot missing logs and traces](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/troubleshooting/#logs-or-traces-are-missing).
