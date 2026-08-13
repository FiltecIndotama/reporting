# Installing the ERPNext Connector in Claude Code

This guide covers connecting the **ERPnext filtec** MCP connector to Claude Code.

The connector is [**Frappe Assistant Core**](https://github.com/buildswithpaul/Frappe_Assistant_Core) (FAC) — a Frappe app that turns your ERPNext site into a remote MCP server. Claude talks to it over Streamable HTTP with OAuth 2.0 + PKCE, so Claude never sees your ERPNext password, and every tool call is executed **as you**, respecting your existing Frappe roles and DocType permissions.

It exposes 24 tools: document CRUD, search, reports, workflows/approvals, dashboards, and analytics (`run_python_code`, `run_database_query`).

---

## Before you start

| Requirement | Detail |
|---|---|
| Claude Code | v2.x or later (`claude --version`) |
| ERPNext site | Reachable over **HTTPS** (OAuth is rejected over plain HTTP except `localhost`) |
| Your ERPNext user | Must have the `Assistant User` or `Assistant Admin` role |
| Server-side app | `frappe_assistant_core` installed on the site |

If someone else already installed FAC on the Filtec site, skip to [Part 2](#part-2--add-the-connector-to-claude-code-cli). You only need the endpoint URL.

---

## Part 1 — Server-side setup (one-time, ERPNext administrator)

Skip this entire section if the connector is already live on your site.

### Step 1.1 — Install the app

**Frappe Cloud:** Dashboard → your site → **Apps** tab → find *Frappe Assistant Core* → **Install**.

**Self-hosted bench:**

```bash
cd /path/to/frappe-bench
bench get-app https://github.com/buildswithpaul/Frappe_Assistant_Core
bench --site <your-site> install-app frappe_assistant_core
bench --site <your-site> migrate
```

### Step 1.2 — Enable the assistant

```bash
bench --site <your-site> set-config assistant_enabled 1
bench restart
```

### Step 1.3 — Turn on OAuth discovery

In ERPNext Desk, search for **Assistant Core Settings** → **OAuth** tab, and enable all three:

- ✅ Show Authorization Server Metadata
- ✅ **Enable Dynamic Client Registration** ← Claude Code cannot connect without this
- ✅ Show Protected Resource Metadata

Leave **Allowed Public Client Origins** blank. Claude Code registers itself dynamically and does not need a pre-registered origin. (Only add entries here if you also use MCP Inspector — `http://localhost:6274` — or a custom web app.)

Click **Save**.

### Step 1.4 — Grant user access

For each person who will use the connector, open their **User** record and:

1. Add the role **`Assistant User`** (or `Assistant Admin` for full administrative tools).
2. Tick the **Assistant Enabled** checkbox.

Permissions are *additive on top of* their normal ERPNext permissions, never a bypass. If a user can't read a DocType in Desk, they can't read it through Claude either.

### Step 1.5 — Copy the MCP endpoint URL

In Desk, go to **FAC Admin** and copy the **MCP Endpoint URL**. It follows this pattern:

```
https://<your-erpnext-site>/api/method/frappe_assistant_core.api.fac_endpoint.handle_mcp
```

> Use the URL shown in FAC Admin verbatim — don't hand-assemble it. Note the endpoint is the `handle_mcp` method path, **not** your site root.

Sanity-check that OAuth discovery is publicly reachable before moving on:

```bash
curl -s https://<your-erpnext-site>/.well-known/openid-configuration | head -20
```

You should get JSON containing `authorization_endpoint` and `token_endpoint`. An HTML login page or a 404 means Step 1.3 didn't take effect.

---

## Part 2 — Add the connector to Claude Code (CLI)

This is the terminal / desktop version of Claude Code.

### Step 2.1 — Register the server

```bash
claude mcp add --transport http erpnext-filtec \
  https://<your-erpnext-site>/api/method/frappe_assistant_core.api.fac_endpoint.handle_mcp
```

`--transport http` is required. Without it Claude Code defaults to `stdio` and will try to execute the URL as a local command.

**Choosing a scope** with `-s`:

| Scope | Flag | Stored in | Use when |
|---|---|---|---|
| Local *(default)* | `-s local` | `~/.claude.json`, keyed to the current project | Just you, just this project |
| User | `-s user` | `~/.claude.json`, global | You want ERPNext available in **every** project — usually the right choice |
| Project | `-s project` | `.mcp.json` in the repo, **committed to git** | The whole team should get it automatically |

For personal use across all your work:

```bash
claude mcp add --transport http -s user erpnext-filtec \
  https://<your-erpnext-site>/api/method/frappe_assistant_core.api.fac_endpoint.handle_mcp
```

For team-wide setup, use `-s project` and commit the generated `.mcp.json`. It contains only the URL — no secrets — and each teammate authenticates individually:

```json
{
  "mcpServers": {
    "erpnext-filtec": {
      "type": "http",
      "url": "https://<your-erpnext-site>/api/method/frappe_assistant_core.api.fac_endpoint.handle_mcp"
    }
  }
}
```

### Step 2.2 — Authenticate

Start Claude Code and run the MCP command:

```bash
claude
```

```
/mcp
```

You'll see `erpnext-filtec` listed, most likely as **needs authentication**. Select it and choose **Authenticate**. Your browser opens the ERPNext login page.

1. Sign in with your normal ERPNext credentials.
2. Click **Allow** on the authorization screen.
3. The browser redirects back and reports success; return to your terminal.

The status should now read **connected**. Tokens are stored by Claude Code and refreshed automatically — you won't repeat this on every session.

> **If your ERPNext OAuth client requires a fixed redirect URI**, re-add the server with a pinned callback port and register `http://localhost:<port>/callback` on the ERPNext side:
> ```bash
> claude mcp add --transport http --callback-port 8765 erpnext-filtec <url>
> ```
> With Dynamic Client Registration enabled (Step 1.3) this is not needed.

### Step 2.3 — Verify

```
/mcp
```

Select `erpnext-filtec` → **View tools**. You should see all 24:

- **Documents** — `get_document`, `list_documents`, `create_document`, `update_document`, `delete_document`, `submit_document`
- **Search** — `search`, `search_documents`, `search_doctype`, `search_link`, `fetch`
- **Reports & workflows** — `report_list`, `report_requirements`, `generate_report`, `get_pending_approvals`, `run_workflow`
- **Analytics & files** — `run_python_code`, `run_database_query`, `analyze_business_data`, `extract_file_content`
- **Dashboards** — `create_dashboard`, `create_dashboard_chart`, `list_user_dashboards`
- **Schema** — `get_doctype_info`

Then try a real read to confirm the permission chain works end to end:

> List the 5 most recent Sales Orders with their customer and grand total.

---

## Part 3 — Claude Code on the web and mobile

Claude Code on the web (claude.ai/code) and the mobile app **do not read your local `.mcp.json` or `~/.claude.json`**. Connectors there come from your claude.ai account settings, so this is a separate, second setup — doing Part 2 does not cover it.

### Step 3.1 — Add the custom connector

1. Go to **claude.ai → Settings → Connectors**.
2. Click **Add custom connector**.
3. Paste the same MCP Endpoint URL from Step 1.5.
4. Give it a name (e.g. `ERPnext filtec`) and click **Add**.
5. Click **Connect** and complete the same ERPNext OAuth login.

### Step 3.2 — Enable it for the session

Adding a connector is not the same as enabling it. In the chat's connector settings, toggle **ERPnext filtec** on — this is the screen showing each tool with an **Always / Ask** setting.

> ⚠️ **Heads-up on your current account:** you have **two** connectors both named "ERPnext filtec". One is connected and active; the other is unauthenticated and switched off. That duplicate is the usual cause of "the connector is there but Claude says it has no tools" — you're looking at the settings for one entry while the session uses the other. Delete the dead one in Settings → Connectors to avoid confusion.

---

## Part 4 — Permissions and safety

The per-tool **Always / Ask** setting controls whether Claude runs a tool without stopping to ask you.

Four tools change or expose data in ways that are hard to undo, and are worth leaving on **Ask** rather than **Always**:

| Tool | Why |
|---|---|
| `delete_document` | Permanently removes ERPNext records |
| `run_python_code` | Arbitrary server-side execution in the Frappe context |
| `run_database_query` | Direct database access, beyond ordinary DocType-level guardrails |
| `submit_document` | Submitted documents are immutable — reversing means a cancel + amend cycle |

In the CLI, the equivalent is a `permissions.allow` list in `.claude/settings.json` — allowlist the read-only tools and let the rest prompt:

```json
{
  "permissions": {
    "allow": [
      "mcp__erpnext-filtec__get_document",
      "mcp__erpnext-filtec__list_documents",
      "mcp__erpnext-filtec__search_documents",
      "mcp__erpnext-filtec__get_doctype_info",
      "mcp__erpnext-filtec__report_list",
      "mcp__erpnext-filtec__generate_report"
    ]
  }
}
```

The MCP tool naming pattern is `mcp__<server-name>__<tool-name>`, so these strings must match the name you chose in Step 2.1.

Remember: FAC never elevates privileges. The worst a tool can do is whatever your own ERPNext account is already allowed to do — so scope the ERPNext role itself if you want a hard ceiling.

---

## Part 5 — Troubleshooting

| Symptom | Cause & fix |
|---|---|
| `claude mcp add` succeeds but the server shows **failed** | Missing `--transport http` — it was added as a stdio command. Remove and re-add: `claude mcp remove erpnext-filtec` |
| Browser shows **"Dynamic client registration is disabled"** | Step 1.3 — tick *Enable Dynamic Client Registration* in Assistant Core Settings |
| **HTTPS requirement error** during OAuth | The endpoint must be `https://` (or `http://localhost:`). Fix the site's TLS/reverse proxy |
| OAuth completes but tools list is **empty** | Your user lacks the `Assistant User` role or has *Assistant Enabled* unticked (Step 1.4) |
| **CORS error** in the browser during auth | Add your client origin to *Allowed Public Client Origins* |
| `/.well-known/openid-configuration` returns 404 or HTML | *Show Authorization Server Metadata* is off, or a reverse proxy is intercepting `/.well-known/`. Confirm with the `curl` in Step 1.5 |
| Tools return **"not permitted"** on specific DocTypes | Expected — that's your normal ERPNext permission, not a connector fault. Fix it via Role Permission Manager |
| Connection worked, then stopped | Token expired and refresh failed. Run `/mcp` → select the server → **Authenticate** again |

Useful commands:

```bash
claude mcp list                 # all configured servers and their status
claude mcp get erpnext-filtec   # full config for one server
claude mcp remove erpnext-filtec
```

---

## Quick reference

```bash
# 1. Server side (once, as ERPNext admin)
bench get-app https://github.com/buildswithpaul/Frappe_Assistant_Core
bench --site <your-site> install-app frappe_assistant_core
bench --site <your-site> migrate
bench --site <your-site> set-config assistant_enabled 1
bench restart
# → Assistant Core Settings → OAuth tab → enable all three checkboxes
# → User record → add "Assistant User" role + tick "Assistant Enabled"

# 2. Claude Code CLI
claude mcp add --transport http -s user erpnext-filtec \
  https://<your-erpnext-site>/api/method/frappe_assistant_core.api.fac_endpoint.handle_mcp
claude
/mcp          # → select erpnext-filtec → Authenticate → log in via browser

# 3. Claude Code on web/mobile
# claude.ai → Settings → Connectors → Add custom connector → same URL → Connect
```

---

## Sources

- [Frappe Assistant Core — GitHub](https://github.com/buildswithpaul/Frappe_Assistant_Core)
- [Getting Started guide](https://github.com/buildswithpaul/Frappe_Assistant_Core/blob/main/docs/getting-started/GETTING_STARTED.md)
- [OAuth Quick Start](https://github.com/buildswithpaul/Frappe_Assistant_Core/blob/main/docs/getting-started/oauth/oauth_quick_start.md)
- [Claude Code MCP documentation](https://code.claude.com/docs/en/mcp)
