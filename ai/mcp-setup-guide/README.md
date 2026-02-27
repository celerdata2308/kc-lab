# CelerData MCP Setup Guide

This directory contains the configuration and tools for connecting Claude Desktop to CelerData's internal systems via MCP (Model Context Protocol) servers.

## Overview

| MCP Server | Type | Description | Prerequisites |
|---|---|---|---|
| `feishu-mcp` | Remote (Railway) | Feishu/Lark docs, wiki, chat, messaging | None |
| `google-docs` | Remote (Railway) | Google Docs, Sheets, Drive | None |
| `starrocks` | Remote (Railway) | StarRocks clusters (shared, managed by team) | Bearer token |
| `byoc-admin` | Local (Node.js) | BYOC cluster management, diagnostics, metrics | BYOC session cookie |
| `starrocks-private` | Local (Python/uvx) | Private StarRocks cluster (private-metrics-pipeline) | Proton VPN + SSH tunnel |

## Configuration

All MCP servers are configured in `claude_desktop_config.json`, which is symlinked (alias) to:

```
~/Library/Application Support/Claude/claude_desktop_config.json
```

Editing either location updates both.

---

## feishu-mcp

Connects Claude to Feishu/Lark for searching and reading documents, wiki pages, chat messages, and sending messages.

**Type:** Remote (Railway)

**Setup:** No local setup needed. The MCP runs on Railway and is shared by the team.

**Config:**
```json
"feishu-mcp": {
  "command": "npx",
  "args": [
    "mcp-remote",
    "https://feishu-mcp-production.up.railway.app/sse"
  ]
}
```

**Available tools:** Search docs, get documents, search/read wiki, list/read chats, send messages, manage folders, export documents.

---

## google-docs

Connects Claude to Google Docs, Sheets, and Drive for reading, creating, and editing documents and spreadsheets.

**Type:** Remote (Railway)

**Setup:** No local setup needed. OAuth authentication is handled by the remote server.

**Config:**
```json
"google-docs": {
  "command": "npx",
  "args": [
    "mcp-remote",
    "https://google-docs-mcp-production-b4cc.up.railway.app/sse"
  ]
}
```

**Available tools:** Read/write/create Google Docs, read/write spreadsheets, manage Drive files and folders, comments, formatting.

---

## starrocks

Connects Claude to shared StarRocks clusters for SQL queries, database exploration, and data visualization.

**Type:** Remote (Railway, managed by team)

**Setup:** Requires a Bearer token for authentication. Contact the team admin if you need a new token.

**Config:**
```json
"starrocks": {
  "command": "npx",
  "args": [
    "mcp-remote",
    "https://romantic-rebirth-production.up.railway.app/mcp",
    "--header",
    "Authorization:Bearer <your-token>"
  ]
}
```

**Available clusters:** Run `list_clusters` in Claude to see available clusters, then `use_cluster` to switch.

**Available tools:** `read_query`, `write_query`, `analyze_query`, `table_overview`, `db_summary`, `query_and_plotly_chart`.

---

## byoc-admin

Connects Claude to the CelerData BYOC Admin API for cluster management, diagnostics, and monitoring.

**Type:** Local (Node.js)

**Setup:**
1. Build the project:
   ```bash
   cd ~/Documents/development/celerdata/byoc-admin-mcp
   npm install
   npm run build
   ```
2. Get your BYOC session cookie from [byoc-admin.celerdata.com](https://byoc-admin.celerdata.com) (browser DevTools → Cookies → `SRSAASSESSION`).
3. Update `BYOC_COOKIE` in the config.

**Config:**
```json
"byoc-admin": {
  "command": "node",
  "args": [
    "/Users/<you>/Documents/development/celerdata/byoc-admin-mcp/dist/index.js"
  ],
  "env": {
    "BYOC_COOKIE": "<your-SRSAASSESSION-cookie>"
  }
}
```

**Note:** The session cookie expires periodically. If tools start failing with auth errors, grab a fresh cookie from the browser.

**Available tools:** See [byoc-admin-mcp/README.md](./byoc-admin-mcp/README.md) for the full tool list.

---

## starrocks-private

Connects Claude directly to the **private-metrics-pipeline** StarRocks cluster via SSH tunnel. This cluster is on a private endpoint and not accessible through the shared StarRocks MCP.

**Type:** Local (Python/uvx via [mcp-server-starrocks](https://github.com/StarRocks/mcp-server-starrocks))

### Prerequisites

1. **Proton VPN** — must be connected to reach the jump server
2. **SSH key** — `byoc-user` key file (stored in `~/.ssh/byoc-user`)
3. **Python uv/uvx** — install with `brew install uv`

### Connection Flow

```
┌──────────┐     VPN      ┌──────────────┐   SSH tunnel   ┌─────────────────────────┐
│  Your Mac │────────────▶│ Jump Server   │──────────────▶│ private-metrics-pipeline │
│ :9030     │             │ 52.40.248.112 │               │ (StarRocks FE)           │
└──────────┘              └──────────────┘               └─────────────────────────┘
     ▲
     │ localhost:9030
     │
┌──────────────────┐
│ starrocks-private │
│ MCP (uvx)        │
└──────────────────┘
```

### Setup (One-time)

1. **Install uv:**
   ```bash
   brew install uv
   ```

2. **Place the SSH key:**
   ```bash
   cp ~/Documents/development/celerdata/secrets/byoc-user ~/.ssh/byoc-user
   chmod 600 ~/.ssh/byoc-user
   ```

3. **Install the LaunchAgent** (for easy tunnel management):
   ```bash
   cp ~/Documents/development/celerdata/secrets/com.celerdata.sr-tunnel.plist ~/Library/LaunchAgents/
   launchctl load ~/Library/LaunchAgents/com.celerdata.sr-tunnel.plist
   ```

4. **Add the shell alias** (optional but recommended):
   ```bash
   echo 'alias sr-tunnel="~/Documents/development/celerdata/secrets/sr-tunnel.sh"' >> ~/.zshrc
   source ~/.zshrc
   ```

5. **Add the MCP config** to `claude_desktop_config.json`:
   ```json
   "starrocks-private": {
     "command": "uvx",
     "args": ["mcp-server-starrocks"],
     "env": {
       "STARROCKS_HOST": "127.0.0.1",
       "STARROCKS_PORT": "9030",
       "STARROCKS_USER": "<your-user>",
       "STARROCKS_PASSWORD": "<your-password>"
     }
   }
   ```

6. **Restart Claude Desktop.**

### Daily Usage

```bash
# 1. Connect Proton VPN

# 2. Start the tunnel
sr-tunnel              # ✅ Tunnel connected! (localhost:9030)

# 3. Use Claude to query the cluster
#    (Claude will use starrocks-private MCP automatically)

# 4. When done
sr-tunnel stop         # 🔌 Tunnel stopped

# Check status anytime
sr-tunnel status       # ✅ Tunnel is running / ❌ Tunnel is not running
```

### Manual SSH Tunnel (without sr-tunnel script)

If you prefer to run the tunnel manually:
```bash
ssh -i ~/.ssh/byoc-user \
    -L 9030:1cogri9tn-internal.cloud-app.celerdata.com:9030 \
    -N \
    ec2-user@52.40.248.112
```

### Cluster Info

| Field | Value |
|---|---|
| Cluster name | private-metrics-pipeline |
| Region | us-west-2 |
| Version | 3.5.10-ee |
| FE node | 1x m6i.xlarge |
| BE node | 1x m6i.xlarge |
| Databases | alerts, byoc, metrics, sys |

### Troubleshooting

| Problem | Solution |
|---|---|
| `sr-tunnel` fails | Check Proton VPN is connected |
| `launchctl start` fails with "Operation not permitted" | SSH key must be in `~/.ssh/`, not `~/Documents/` (macOS privacy restriction) |
| MCP shows "Can't connect to MySQL server" | Tunnel is not running — run `sr-tunnel` first |
| MCP works in Claude Desktop but not Cowork | Cowork's sandbox cannot access localhost tunnel; use Claude Desktop (non-Cowork) instead |

---

## File Structure

```
celerdata/
├── README.md                          ← This file
├── claude_desktop_config.json         ← MCP config (alias to Claude Desktop config)
├── byoc-admin-mcp/                    ← BYOC Admin MCP server source
│   ├── README.md
│   ├── dist/index.js
│   └── ...
├── feishu-mcp/                        ← Feishu MCP related files
├── secrets/
│   ├── .env                           ← Database credentials
│   ├── byoc-user                      ← SSH key (copy to ~/.ssh/)
│   ├── sr-tunnel.sh                   ← Tunnel start/stop/status script
│   ├── com.celerdata.sr-tunnel.plist  ← macOS LaunchAgent for tunnel
│   ├── credentials.json               ← Google OAuth credentials
│   └── google_token.pickle            ← Google OAuth token
└── metrics_pipeline/                  ← Metrics pipeline related files
```
