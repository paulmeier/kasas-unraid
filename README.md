# kasas-unraid

<p align="center">
  <img src="icons/kasas.png" alt="kasas" width="200" />
</p>

[![Lint](https://github.com/paulmeier/kasas-unraid/actions/workflows/lint.yml/badge.svg)](https://github.com/paulmeier/kasas-unraid/actions/workflows/lint.yml)
[![Release Please](https://github.com/paulmeier/kasas-unraid/actions/workflows/release-please.yml/badge.svg)](https://github.com/paulmeier/kasas-unraid/actions/workflows/release-please.yml)
[![Release](https://img.shields.io/github/v/release/paulmeier/kasas-unraid)](https://github.com/paulmeier/kasas-unraid/releases)

Unraid Community Applications template for [kasas](https://github.com/paulmeier/kasas) — a self-hosted, single-binary financial ledger. kasas syncs transactions from a [SimpleFIN](https://www.simplefin.org/) bridge into local SQLite (or Postgres) and exposes a REST API, a built-in [MCP](https://modelcontextprotocol.io/) server, a read-only web dashboard, a canonical event stream, outbound webhooks, and sandboxed plugins. It is the ledger that budgeting, tax, fraud-detection, forecasting, notification, and AI-agent apps build on — not a budgeting app itself.

## Template

- **Container image**: `ghcr.io/paulmeier/kasas:latest` (multi-arch: amd64 + arm64)
- **Project**: https://github.com/paulmeier/kasas
- **Container registry**: https://github.com/paulmeier/kasas/pkgs/container/kasas
- **Documentation**: https://paulmeier.github.io/kasas/

## Installation

### Via Community Applications (recommended)

To add this repository as a template source manually:

1. In Unraid, open the **Apps** tab and go to **Settings**.
2. Under **Template Repositories**, add:
   ```
   https://github.com/paulmeier/kasas-unraid
   ```
3. Click **Check for Updates**, then search for **kasas**.

### Manual installation

1. In Unraid, go to **Docker** → **Add Container**.
2. Click **Load template from URL** and paste:
   ```
   https://raw.githubusercontent.com/paulmeier/kasas-unraid/main/templates/kasas.xml
   ```
3. Click **Load**, fill in the fields, and click **Apply**.

## First run

### 1. Make the App Data directory writable

kasas runs as a **non-root user (UID 65532)**, so the mounted App Data directory must be writable by it. Before starting the container, open an Unraid terminal and run once:

```bash
mkdir -p /mnt/user/appdata/kasas
chown -R 65532:65532 /mnt/user/appdata/kasas
```

Without this, kasas cannot create its database and the container will fail its health check.

### 2. Provide a SimpleFIN credential

kasas needs a [SimpleFIN](https://www.simplefin.org/) bridge credential to sync transactions. You can either:

- Paste a one-time **SimpleFIN Setup Token** (or an already-claimed **Access URL**) into the template before starting, **or**
- Leave both blank and add the credential later from the dashboard **Settings** page (no restart needed).

The resolved access URL is saved to the secret store in your App Data volume, so the one-time token is only consumed once.

### 3. Open the dashboard

Browse to `http://YOUR-UNRAID-IP:8080`. The REST API is at `/api/v1`, the MCP server at `/mcp`, Prometheus metrics at `/metrics`, and health at `/healthz` and `/readyz`.

## Security

> [!IMPORTANT]
> By default kasas is **unauthenticated** — anyone who can reach the port can read your financial data and change settings.

Set the **Dashboard / API Token** field (or generate a token from the dashboard **Settings** page) to require `Authorization: Bearer <token>` on the REST API, the dashboard, and the MCP server, and keep the container on a trusted network. The health and metrics endpoints are never gated.

## Configuration

The template exposes the full kasas configuration surface as environment variables, each with a sensible default. The most common knobs are shown by default; switch the container to **Advanced View** to see the rest.

| Group | Settings |
| --- | --- |
| **Core** | WebUI / API / MCP port, App Data volume |
| **SimpleFIN** | Setup token, access URL |
| **Security** | Dashboard / API token |
| **Database** | Driver (sqlite / postgres), SQLite path, Postgres DSN |
| **Interfaces** | Enable MCP server, enable web dashboard |
| **Sync** | Enabled, interval, lookback days, run-on-start |
| **Events** | Enabled, event retention, history retention |
| **Webhooks** | Enabled, timeout, max attempts |
| **Plugins** | Enabled, directory, hook timeout, queue size |
| **Updates** | Update check, allow in-place update, repository |
| **Secrets / Vault** | Secrets file, Vault enable/address/token/mount/path |
| **Logging** | Level, format |

Every field maps to a `KASAS_*` environment variable. See the [Configuration reference](https://paulmeier.github.io/kasas/getting-started/configuration/) for the full meaning of each.

### Postgres backend

SQLite (the default) needs nothing extra. To use Postgres instead:

1. Run a separate Postgres container on Unraid (e.g. the official `postgres` Community Applications template).
2. Set **Database Driver** to `postgres`.
3. Set **Database DSN** to point at it, e.g. `postgres://user:pass@host:5432/kasas?sslmode=disable`.

### Updates

The in-app update checker and self-updater are **turned off** in this template, because Docker deployments upgrade by pulling a new image. Update kasas from the Unraid **Docker** tab the same way as any other container (**Check for Updates**, then apply).

## Tailscale

There are two ways to reach kasas over [Tailscale](https://tailscale.com/).

### Easiest: the Unraid host plugin

Install the **Tailscale** plugin on your Unraid host (Apps → search "Tailscale"). A container on the default **bridge** network is then reachable on the host's Tailscale IP at port `8080` automatically — no changes to this template are needed.

### Per-container node: Unraid's in-container integration

To give the kasas container its *own* Tailscale node (its own MagicDNS name, Serve / Funnel, etc.):

1. Open the kasas container in **Docker** and switch to **Advanced View**.
2. Toggle **Use Tailscale** on and fill in the standard Tailscale fields.
3. **Because kasas ships as a non-root (UID 65532) `scratch` image, you must also run it as root** for the integration's `tailscaled` to start — otherwise it fails with `ERROR: No root privileges!`. Add the following to **Extra Parameters**:
   ```
   --user 0:0
   ```
4. Apply.

The template pre-declares `CA_TS_FALLBACK_DIR=/data/tailscale`, so Tailscale's state (machine key, node info) lands inside the App Data volume at `/mnt/user/appdata/kasas/tailscale/` and survives container recreation and image updates.

> [!NOTE]
> Running the container as root is only required for the in-container integration. The default (and recommended) non-root posture is fine for the host-plugin approach above.

## Support

Open an issue: https://github.com/paulmeier/kasas-unraid/issues
