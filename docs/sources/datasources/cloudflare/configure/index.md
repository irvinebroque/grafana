---
description: Install the Cloudflare plugin and configure authentication, account settings, and provisioning in Grafana.
keywords:
  - grafana
  - cloudflare
  - configure
  - authentication
labels:
  products:
    - enterprise
    - oss
menuTitle: Configure
title: Configure the Cloudflare data source
weight: 100
review_date: 2026-11-10
---

# Configure the Cloudflare data source

This document explains how to install the Cloudflare plugin, connect your account, and provision the data source in Grafana.

## Before you begin

Before you begin, ensure you have the following:

- **Grafana instance:** A self-managed Grafana OSS or Grafana Enterprise instance running Grafana 13.3 or later.
- **Grafana permissions:** The organization administrator role or equivalent permissions to configure data sources, and access to the server's plugin directory and configuration.
- **Cloudflare account:** An account ID and permission to create an API token for the data you want to query.
- **Build tools:** Node.js 24 or later, `pnpm`, and Go 1.27.3 or later to build the plugin from source.

## Install the plugin

The plugin is installed separately from Grafana. Build the plugin from the [Cloudflare data source repository](https://github.com/irvinebroque/grafana-cloudflare-datasource):

```sh
git clone https://github.com/irvinebroque/grafana-cloudflare-datasource.git
cd grafana-cloudflare-datasource
pnpm install --frozen-lockfile
BUILD_ALL=1 pnpm exec vp run build
```

The `dist/` directory contains the installable plugin, including backend executables for Linux and macOS on AMD64 and ARM64, and Windows on AMD64.

1. Copy the contents of `dist/` into a directory named `cloudflare-cloudflare-datasource` within your Grafana plugins directory. The `plugin.json` file must be directly inside that directory.
1. Ensure the Grafana server user can read the plugin files and execute the backend binary for its operating system and architecture.
1. Add the plugin ID to `allow_loading_unsigned_plugins` in your Grafana configuration. If you already allow other unsigned plugins, append this ID to the comma-separated list.

Use the following configuration to allow the Cloudflare plugin:

```ini
[plugins]
allow_loading_unsigned_plugins = cloudflare-cloudflare-datasource
```

Restart Grafana after you install the plugin or change this configuration. For plugin directory and configuration details, refer to [Configure Grafana](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/setup-grafana/configure-grafana/). For information about unsigned plugins, refer to [Plugin signatures](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/administration/plugin-management/plugin-sign/).

{{< admonition type="note" >}}
The plugin is unsigned. This installation method requires a self-managed instance and isn't available in Grafana Cloud.
{{< /admonition >}}

## Set up Cloudflare authentication

All requests to Cloudflare run through the Grafana plugin backend. Grafana encrypts the saved API token in its database.

1. Create a Cloudflare API token with **Account Analytics Read** for the account you want to query. Refer to [Cloudflare SQL API setup](https://developers.cloudflare.com/analytics/sql-api/get-started/).
1. Add the permissions for the query languages, datasets, and resource selectors you use, as described below.
1. Scope the token to the accounts and zones you want the data source to access.
1. Copy your account ID from the Cloudflare dashboard. If you plan to query zone data, copy the zone ID as well. Refer to [Find account and zone IDs](https://developers.cloudflare.com/fundamentals/account/find-account-and-zone-ids/).

Account and zone IDs are 32-character lowercase hexadecimal identifiers. Query-level account or zone overrides use the same saved token, so the token must have permission for each selected resource.

### Select token permissions

Start with **Account > Account Analytics > Read**. Add permissions according to the features you use.

| Feature                              | Permission                                           | When to use it                                                                                                                                    |
| ------------------------------------ | ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| SQL catalog and connection test      | **Account > Account Analytics > Read**               | Required for dataset discovery and **Save & test**. GraphQL account analytics and the original Analytics Engine SQL API also use this permission. |
| Account selector                     | **Account > Account Settings > Read**                | Optional account metadata access for **Browse accounts**. You can enter an account ID directly instead.                                           |
| Zone selector                        | **Zone > Zone > Read**                               | Required for **Browse zones**. You can enter a zone ID directly instead. This permission doesn't grant analytics access.                          |
| **Workers telemetry** query language | **Workers Observability Write** (account permission) | Required by the telemetry query endpoint according to its API reference. Add this permission only if you use that query language.                 |

For the permission definitions, refer to [Cloudflare API token permissions](https://developers.cloudflare.com/fundamentals/api/reference/permissions/). For the zone selector requirement, refer to [List zones](https://developers.cloudflare.com/api/resources/zones/methods/list/).

The [Workers telemetry query endpoint](https://developers.cloudflare.com/api/resources/workers/subresources/observability/subresources/telemetry/methods/query/) currently lists **Workers Observability Write**, rather than **Workers Observability Read**, as its accepted permission. The Workers logs, Containers logs, and trace examples use the SQL API. Using those examples doesn't by itself require the telemetry permission.

SQL datasets and fields can require additional product permissions. Add the permissions required by the data you query. For zone-scoped SQL queries, Cloudflare accepts **Zone > Analytics > Read** for the selected zones or **Account Analytics Read** for their owning account. Retain **Account Analytics Read** for the account-scoped catalog and connection test.

The connection test doesn't verify every dataset or API. After saving, run a query for each feature you intend to use. Permissions don't replace product enablement, plan access, or data retention.

## Add the data source

To add the Cloudflare data source:

1. Click **Connections** in the left-side menu.
1. Click **Data sources**.
1. Click **Add data source**.
1. Search for and select **Cloudflare**.

You're taken to the **Settings** tab where you configure the data source.

## Configure the data source in the UI

Use the following settings to configure your connection and query limits.

| Setting                       | Description                                                                                                                          |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| **Name**                      | Sets the name you use to identify the data source in panels and queries.                                                             |
| **Default**                   | Selects this data source by default for new panels.                                                                                  |
| **API token**                 | Authenticates requests to Cloudflare. Grafana stores this value as an encrypted secret.                                              |
| **Default account ID**        | Sets the account used when a query's **Account** field is empty. Required for **Save & test**.                                       |
| **Default zone ID**           | Optionally sets the zone used when a zone-scoped query's **Zone ID** field is empty.                                                 |
| **Request timeout (seconds)** | Sets the request timeout. The default is `60`, and the maximum is `300`.                                                             |
| **Maximum returned rows**     | Limits returned rows per query. The default is `10000`, and the maximum is `100000`. Queries that reach the limit display a warning. |

Click **Save & test**. A successful test displays the following message:

```text
Connected to Cloudflare SQL API. Dataset access depends on your token permissions and plan.
```

The test verifies access to the SQL dataset catalog for the default account. Run queries for the products you need to verify their permissions and data availability separately.

## Provision the data source

You can configure the data source with a YAML file in your Grafana provisioning directory. For details, refer to [Provisioning Grafana](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/administration/provisioning/#data-sources).

The following example reads the token from the Grafana server's environment:

```yaml
apiVersion: 1

datasources:
  - name: Cloudflare
    uid: cloudflare
    type: cloudflare-cloudflare-datasource
    access: proxy
    jsonData:
      accountId: <ACCOUNT_ID>
      zoneId: <ZONE_ID>
      timeoutSeconds: 60
      maxRows: 10000
    secureJsonData:
      apiToken: $CLOUDFLARE_API_TOKEN
```

Replace _`<ACCOUNT_ID>`_ with your Cloudflare account ID and _`<ZONE_ID>`_ with your zone ID. Omit `zoneId` if you don't use zone-scoped queries. Set `CLOUDFLARE_API_TOKEN` in the Grafana server environment before starting Grafana. Store the token in `secureJsonData`, rather than `jsonData`.

## Next steps

After you configure the data source, you can:

- [Build your first query](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/query-editor/).
- [Explore logs and traces](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/logs-and-traces/).
- [Import the Cloudflare overview dashboard](https://grafana.com/docs/grafana/<GRAFANA_VERSION>/datasources/cloudflare/#import-the-cloudflare-overview-dashboard).
