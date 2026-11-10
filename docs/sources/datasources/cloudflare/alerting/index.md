---
description: Create Grafana-managed alert rules from Cloudflare metric queries.
keywords:
  - grafana
  - cloudflare
  - alerting
  - metrics
labels:
  products:
    - enterprise
    - oss
menuTitle: Alerting
title: Cloudflare alerting
weight: 450
review_date: 2026-11-10
---

# Cloudflare alerting

The Cloudflare data source supports Grafana-managed alert rules for numeric metric queries. Use alerts to monitor Worker requests, CPU time, errors, and other available Cloudflare metrics.

## Before you begin

Before you create an alert rule, ensure you have the following:

- Permissions to create alert rules in Grafana.
- A [configured Cloudflare data source](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/configure/) with a token that can access the required dataset.
- A metric query that returns the expected numeric time series in Explore.

For alerting concepts, refer to [Grafana Alerting fundamentals](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/alerting/fundamentals/).

## Create an alert rule

Start with a metric query that you can evaluate against a threshold.

1. Open **Alerting > Alert rules** and create a Grafana-managed alert rule.
1. Enter a rule name and select your Cloudflare data source.
1. Select **Builder** and choose a metric example, such as **Workers · requests**. Alternatively, enter an SQL metric query.
1. Select **Time series** and verify the query's account, filters, and time column.
1. Run the query preview.
1. Add a **Reduce** expression, such as **Last** or **Mean**, and a threshold condition.
1. Configure the evaluation interval, pending period, and no-data and error behavior.
1. Configure notification routing and save the rule.

For the complete workflow, refer to [Create Grafana-managed alert rules](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/alerting/alerting-rules/create-grafana-managed-rule/).

## Query requirements

Alert evaluation runs on the Grafana server with the data source's saved API token. Use queries that return timestamps and numeric metric values, with dimensions that identify each series.

- **Result format:** Use **Time series** for metric queries. Log rows and trace waterfalls aren't numeric alert inputs.
- **Variables:** Replace dashboard template variables with fixed values or query dimensions. Dashboard variables aren't available during alert evaluation.
- **Dimensions:** Group by fields such as `scriptName` when you want separate alert instances for each Worker. Avoid unbounded identifiers such as individual request IDs.
- **Time range:** Use a range that accounts for Cloudflare ingestion delay, sampling, and dataset retention.

## Example: Worker request volume

The following SQL query returns request counts for each Worker over time:

```sql
SELECT $__timeGroup(timestamp) AS time,
       scriptName,
       COUNT(*) AS requests
FROM events.workersInvocations
GROUP BY time, scriptName
ORDER BY time
LIMIT 1000
```

Set **Time column** to `time` and **Series dimensions** to `scriptName`. Reduce each series and set a threshold appropriate for your workload. The query returns counts per time bucket; it doesn't return requests per second.

## Handle missing data and errors

Cloudflare can return no rows when a product has no activity, logging isn't enabled, or the selected time range exceeds retention. An empty result doesn't always mean that a metric is zero.

Configure [no-data and error handling](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/alerting/fundamentals/alert-rule-evaluation/nodata-and-error-states/) for your workload. If a rule returns authentication, rate-limit, or timeout errors, test the same query in Explore and refer to [Troubleshooting](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/troubleshooting/).

## Next steps

- [Configure notification policies](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/alerting/configure-notifications/).
- [Refine your Cloudflare query](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/query-editor/).
