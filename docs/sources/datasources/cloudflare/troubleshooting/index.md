---
description: Resolve installation, authentication, query, and missing-data issues with the Cloudflare data source.
keywords:
  - grafana
  - cloudflare
  - troubleshooting
  - errors
labels:
  products:
    - enterprise
    - oss
menuTitle: Troubleshooting
title: Troubleshoot Cloudflare data source issues
weight: 500
review_date: 2026-11-10
---

# Troubleshoot Cloudflare data source issues

This document provides solutions to common issues with the Cloudflare data source. For setup instructions, refer to [Configure the Cloudflare data source](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/configure/).

## Plugin doesn't appear or start

If Cloudflare isn't available when you add a data source, verify the installation.

| Cause                                           | Solution                                                                                                                                      |
| ----------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| Plugin files are in the wrong directory.        | Place `plugin.json` directly inside `cloudflare-cloudflare-datasource` in the configured Grafana plugins directory.                           |
| Grafana rejects the unsigned plugin.            | Add `cloudflare-cloudflare-datasource` to `allow_loading_unsigned_plugins` and restart Grafana. This plugin requires a self-managed instance. |
| Grafana is too old.                             | Use Grafana 13.3 or later.                                                                                                                    |
| The backend executable is missing or can't run. | Build the backend for the server's operating system and architecture, and verify that the Grafana user has execute permission.                |

Review the Grafana server logs for plugin loading errors after restarting.

## Authentication and permission errors

Authentication failures can affect connection tests, dataset discovery, queries, or resource browsing separately.

| Symptom                                                   | Solution                                                                                                                             |
| --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| **Save & test** asks for a default account ID.            | Set **Default account ID** to the account's 32-character lowercase hexadecimal ID.                                                   |
| Requests return `401` or `403`.                           | Verify that the token is valid, hasn't expired or been revoked, and includes the required read permissions and resource scope.       |
| The SQL catalog test succeeds, but a product query fails. | Verify the product's additional permissions, plan availability, and dataset access. The connection test doesn't query every product. |
| **Browse accounts** or **Browse zones** fails.            | Add the relevant resource-listing permissions, or enter an authorized account or zone ID directly.                                   |
| A query override returns a permission error.              | Confirm the saved token has access to the query's account or zone, rather than only the configured default.                          |

For token setup, refer to [Cloudflare SQL API setup](https://developers.cloudflare.com/analytics/sql-api/get-started/) and the documentation for the API you're querying.

## Datasets or fields are missing

The SQL catalog depends on the account and Cloudflare deployment. Product enablement, plan access, token permissions, and custom datasets can change the available entries.

1. Verify the query's **Account** and **Scope**.
1. Click **Reload catalog** after changing permissions or product configuration.
1. Expand **Dataset schema** and click **Discover custom attributes** when you need custom fields.
1. Confirm availability in [Cloudflare SQL datasets](https://developers.cloudflare.com/analytics/sql-api/datasets/).

The SQL catalog doesn't list GraphQL-only datasets. Use **GraphQL** and refer to [GraphQL dataset discovery](https://developers.cloudflare.com/analytics/graphql-api/features/discovery/) for those datasets.

## Query errors or unexpected results

Use the query editor settings and the syntax for the selected API to diagnose query failures.

| Symptom                                                     | Solution                                                                                                                                         |
| ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| SQL reports an unknown dataset or syntax error.             | Use a schema-qualified catalog name in **SQL** mode. In **Analytics Engine SQL**, use the original dataset name and that endpoint's SQL dialect. |
| SQL parameters fail to parse.                               | Use a JSON object in **SQL parameters (JSON)**. For dashboard variables, use `${variable:json}` without additional quotes.                       |
| Time series don't render.                                   | Return a timestamp and numeric values. Verify **Time column**, **Result format**, and **Series dimensions**.                                     |
| GraphQL results are empty despite a populated API response. | Set **Result data path** relative to `data`, without a `data.` prefix. Use flattened field names such as `dimensions.datetimeMinute`.            |
| GraphQL requires an operation name.                         | Set **Operation name** when the document has multiple query operations.                                                                          |
| Analytics Engine returns data outside the expected range.   | Include explicit timestamp predicates. The original Analytics Engine endpoint doesn't apply the request time range of the unified SQL API.       |
| Counts differ from raw event row counts.                    | Account for Cloudflare sampling. Unified SQL aggregates can represent weighted events, while log searches contain only retained events.          |

Open the [Query inspector](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/visualizations/panels-visualizations/panel-inspector/#inspect-query-request-and-response-data) to review the request, response, and executed query where available. For detailed syntax, refer to [Cloudflare query editor](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/query-editor/).

## Results reach a row limit

The builder's **Row limit**, a raw query's `LIMIT`, and the data source's **Maximum returned rows** can restrict results. A result that reaches the configured safety limit displays a warning and might be incomplete.

Narrow the time range, add filters, or aggregate data before raising limits. Increasing the row limit doesn't bypass Cloudflare's API limits. Large results can also reach response-size or frame limits; select fewer columns and reduce the result size if that happens.

## Rate limits, timeouts, and upstream errors

Cloudflare can reject or delay requests because of API limits, query cost, or service availability.

- **`429` responses:** Reduce the dashboard refresh frequency, number of simultaneous queries, or alert evaluation frequency.
- **Timeouts:** Narrow the time range, add filters, reduce the number of groups, or increase **Request timeout (seconds)** within its 300-second maximum.
- **`5xx` responses:** Retry the query and verify the product's availability. A service error doesn't establish that a dataset is unsupported.

If an error includes a Cloudflare Ray ID, retain it when you request help. Don't include API tokens in reports.

## Logs or traces are missing

Log and trace queries depend on data collection as well as API access.

1. Verify that logs or traces are enabled for the application.
1. Select a time range with recent activity that is within the dataset retention period.
1. Allow for ingestion delay and verify that sampling includes the events you expect.
1. Confirm the query uses the same account as the application.
1. For logs, verify the selected timestamp and message columns.

The **View trace** link can't retrieve a trace that wasn't retained or sampled. Native log context and volume are available for Workers and Containers datasets; selecting **Logs** doesn't add those features to other datasets.

For collection settings, refer to [Workers logs](https://developers.cloudflare.com/workers/observability/logs/) and [Workers traces](https://developers.cloudflare.com/workers/observability/traces/).

## Alert rules fail or return no data

Run the same metric query in Explore and verify it returns numeric time series with stable labels. Remove dashboard template variables from alert queries and verify the saved token's access. Configure no-data and error behavior according to the workload.

For query requirements, refer to [Cloudflare alerting](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/alerting/).

## Get help

Include the plugin and Grafana versions, query language, redacted query, time range, error message, and any Cloudflare Ray ID when you [report an issue in the plugin repository](https://github.com/irvinebroque/grafana-cloudflare-datasource/issues). Remove credentials and account-specific data from requests and responses before sharing them.
