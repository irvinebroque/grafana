---
description: Query and visualize Cloudflare metrics, logs, traces, and analytics in Grafana.
keywords:
  - grafana
  - cloudflare
  - workers
  - analytics
labels:
  products:
    - enterprise
    - oss
menuTitle: Cloudflare
title: Cloudflare data source
weight: 310
review_date: 2026-11-10
---

# Cloudflare data source

The Cloudflare data source lets you query metrics, logs, traces, and analytics retained in Cloudflare from Grafana dashboards and Explore. Use the visual query builder or write queries for the Cloudflare SQL API, GraphQL Analytics API, Analytics Engine, and Workers observability.

The [Cloudflare data source plugin](https://github.com/irvinebroque/grafana-cloudflare-datasource) is a community plugin that you install separately. It requires Grafana 13.3 or later. These instructions describe plugin version 0.1.2.

{{< admonition type="note" >}}
The plugin is unsigned and requires a self-managed Grafana OSS or Grafana Enterprise instance configured to load it. You can't install this unsigned plugin in Grafana Cloud. The query builder and SQL editor use the Cloudflare SQL API, which is in beta.
{{< /admonition >}}

## Get started

The following documents help you get started with the Cloudflare data source:

- [Configure the data source](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/configure/) - Install the plugin, configure an API token, and connect your account.
- [Query editor](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/query-editor/) - Build queries or write SQL, GraphQL, and Workers telemetry requests.
- [Logs and traces](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/logs-and-traces/) - Search logs, view surrounding context, and open trace waterfalls.
- [Template variables](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/template-variables/) - Create dynamic dashboards with Cloudflare query results.
- [Annotations](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/annotations/) - Overlay Cloudflare log events on graphs.
- [Alerting](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/alerting/) - Create alert rules from Cloudflare metrics.
- [Troubleshooting](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/troubleshooting/) - Resolve installation, authentication, and query errors.

## Supported query types

Select a query language based on the API and data you want to query.

| Query language           | Description                                                                                |
| ------------------------ | ------------------------------------------------------------------------------------------ |
| **Builder**              | Discover SQL datasets and select metrics, dimensions, and filters without writing SQL.     |
| **SQL**                  | Query the Cloudflare SQL API using schema-qualified dataset names and optional parameters. |
| **GraphQL**              | Query the GraphQL Analytics API, including account and zone analytics.                     |
| **Analytics Engine SQL** | Query custom Analytics Engine datasets through the original Analytics Engine SQL API.      |
| **Workers telemetry**    | Run JSON requests from Workers Query Builder.                                              |

## Supported Cloudflare data

The query builder discovers datasets from your Cloudflare account. Available data depends on your token permissions, Cloudflare plan, product configuration, and retention period. The catalog can include Workers, Durable Objects, D1, R2, KV, Queues, Workflows, Pipelines, Workers AI, HTTP traffic, and other Cloudflare analytics. Account-defined Analytics Engine and enabled log datasets can also appear in the catalog.

The plugin includes query examples for common products. Workers and Containers logs have additional Explore features, and Workers traces support trace search and span waterfalls. Other discovered log datasets can require you to select their timestamp and message columns explicitly.

Use the GraphQL editor for analytics that aren't available through SQL. It accepts queries for Cloudflare's GraphQL schema but doesn't provide a product-specific dataset browser. Refer to [Cloudflare SQL datasets](https://developers.cloudflare.com/analytics/sql-api/datasets/) and [GraphQL dataset discovery](https://developers.cloudflare.com/analytics/graphql-api/features/discovery/) for availability.

The plugin queries Cloudflare APIs. It doesn't read exported log files, scrape host metrics, or query operational resource configuration. For example, use a Prometheus data source for collected `cloudflared` metrics.

## Additional features

After you configure the Cloudflare data source, you can:

- Use [Explore](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/visualizations/explore/) to investigate data without building a dashboard.
- Create [visualizations](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/visualizations/panels-visualizations/visualizations/) from query results.
- Add [template variables](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/template-variables/) to switch between Workers or accounts.
- Add [annotations](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/annotations/) to correlate log events with metrics.
- Configure [Grafana Alerting](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/alerting/) for numeric metric queries.

## Import the Cloudflare overview dashboard

The plugin includes a **Cloudflare overview** dashboard with example queries.

1. Open **Connections > Data sources** and select your Cloudflare data source.
1. Open the **Dashboards** tab.
1. Import **Cloudflare overview**.
1. Open the dashboard and select a time range with Cloudflare activity.

Panels can return no data when a product isn't enabled or has no activity in the selected time range. Adjust or remove those panels for your account.

## Related resources

- [Cloudflare data source repository](https://github.com/irvinebroque/grafana-cloudflare-datasource).
- [Cloudflare SQL API documentation](https://developers.cloudflare.com/analytics/sql-api/).
- [Cloudflare GraphQL Analytics API documentation](https://developers.cloudflare.com/analytics/graphql-api/).
- [Cloudflare Workers observability documentation](https://developers.cloudflare.com/workers/observability/).
