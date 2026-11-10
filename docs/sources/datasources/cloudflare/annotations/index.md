---
description: Display Cloudflare log events as annotations on Grafana dashboards.
keywords:
  - grafana
  - cloudflare
  - annotations
labels:
  products:
    - enterprise
    - oss
menuTitle: Annotations
title: Cloudflare annotations
weight: 400
review_date: 2026-11-10
---

# Cloudflare annotations

Annotations overlay events on dashboard graphs so you can correlate Cloudflare log messages with changes in metrics. The Cloudflare data source uses a table query to provide annotation timestamps and text.

## Before you begin

Before you create annotations, ensure you have the following:

- A [configured Cloudflare data source](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/configure/).
- A dashboard you can edit.
- Access to log events in the selected time range.

For general annotation settings, refer to [Annotate visualizations](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/visualizations/dashboards/build-dashboards/annotate-visualizations/).

## Create an annotation query

Use an annotation query to retrieve the events you want to display on your dashboard.

1. Open the dashboard and select **Editing** in the dashboard mode drop-down list.
1. Click **Add new element** and select **Annotation query**.
1. Enter an annotation name and configure its color and display settings.
1. Click **Open query editor** and select your Cloudflare data source.
1. Select **SQL** in **Query language** and enter the query below.
1. Set **Time column** to `time`.
1. Click **Test annotation query** and verify the returned events.
1. Close the query editor and save the dashboard.

The following query uses Worker log timestamps and messages:

```sql
SELECT timestamp AS time,
       message AS text
FROM logs.workersLogs
ORDER BY timestamp DESC
LIMIT 100
```

The plugin supplies the dashboard time range and uses **Table** format for annotation queries. Return a timestamp column named `time` and a string column named `text`.

## Filter annotation events

Use SQL filters to select events relevant to your dashboard. For example, add a parameterized Worker filter before `ORDER BY`:

```sql
WHERE scriptName = $worker_name
```

Set **SQL parameters (JSON)** to the following object:

```json
{
  "worker_name": "example-worker"
}
```

Replace `example-worker` with your Worker name. To use a dashboard variable, refer to [Cloudflare template variables](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/template-variables/).

The query returns at most 100 events. Narrow the time range or add filters when you need to focus on specific events. Annotations reflect retained Cloudflare data and can be affected by logging and sampling settings.

## Next steps

- [Explore Cloudflare logs and traces](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/logs-and-traces/).
- [Configure Cloudflare alerting](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/alerting/).
