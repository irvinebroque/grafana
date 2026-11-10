---
description: Create Cloudflare query variables and use them in Grafana dashboards.
keywords:
  - grafana
  - cloudflare
  - template
  - variables
labels:
  products:
    - enterprise
    - oss
menuTitle: Template variables
title: Cloudflare template variables
weight: 350
review_date: 2026-11-10
---

# Cloudflare template variables

Template variables let you reuse a dashboard for different Workers, accounts, or other Cloudflare dimensions. Grafana displays variable values in drop-down lists at the top of the dashboard.

Before you begin, [configure a Cloudflare data source](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/configure/) and create a dashboard. For general variable concepts, refer to [Variables](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/visualizations/dashboards/variables/).

## Create a query variable

Cloudflare query variables use the same query editor as panel queries. The following example lists Worker names with activity in the selected time range.

1. Open your dashboard's settings and select **Variables**.
1. Add a variable named `worker` and set its type to **Query**.
1. Select your Cloudflare data source.
1. Select **SQL** in **Query language** and **Table** in **Result format**.
1. Enter the query below.
1. Set **Refresh** to **On time range change** if you want the values to reflect the dashboard's selected time range.
1. Preview the values and save the variable.

Use `__text` for the displayed label and `__value` for the value used in queries:

```sql
SELECT scriptName AS __text,
       scriptName AS __value
FROM events.workersInvocations
GROUP BY scriptName
LIMIT 1000
```

This query lists Workers present in analytics for the selected range. It doesn't list every deployed Worker.

## Use a variable in SQL parameters

For a single-select `worker` variable, use a named SQL parameter in your metric query:

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
  "worker_name": ${worker:json}
}
```

Grafana formats `${worker:json}` as a JSON string before the plugin parses the parameters. Don't add another pair of quotes around it. The `$worker_name` in the SQL query binds to this parameter. Use a parameter name, such as `worker_name`, that differs from the dashboard variable name so Grafana doesn't interpolate the SQL placeholder itself. Don't enable **Multi-value** or **Include All option** for this equality example.

Select **Time series** and set **Time column** to `time`. Refer to [Variable syntax](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/visualizations/dashboards/variables/variable-syntax/) for formatting options.

## Use variables in other query fields

The plugin supports dashboard variables in account IDs, zone IDs, dataset names, trace IDs, builder filter values, SQL text, and JSON request fields.

- **Builder filters:** Enter `$worker` in **Value** for a `scriptName` equality filter. Use a single-select variable for this filter.
- **Account and zone selection:** Enter an ID variable such as `$account` in **Account**. The saved API token must have access to every selected resource.
- **GraphQL:** Reference dashboard variables in **Additional variables (JSON)** and declare the corresponding GraphQL variables in the query.
- **Workers telemetry:** Use JSON-formatted variables in the request body.

SQL text uses SQL string formatting by default. To filter multiple Worker names with a multi-value variable, use the following predicate in a raw SQL query:

```sql
WHERE scriptName IN (${worker:sqlstring})
```

Don't wrap `${worker:sqlstring}` in additional quotes. The scalar equality parameter example above and builder equality filters don't expand a multi-value selection into an `IN` condition.

## Next steps

- [Build Cloudflare queries](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/query-editor/).
- [Add and manage variables](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/visualizations/dashboards/variables/add-template-variables/).
