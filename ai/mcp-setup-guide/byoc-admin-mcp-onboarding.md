# BYOC Admin MCP — Onboarding Guide

Connect your Claude agent to the CelerData BYOC Admin API to query clusters
directly — list clusters, pull metrics, search FE/BE logs, run diagnostic commands,
and more.

## What it is
A local MCP server (Node.js) that wraps the BYOC Admin API
(`byoc-admin.celerdata.com`). You run it locally via Claude Desktop; it logs in
automatically using your BYOC credentials (`BYOC_USER` + `BYOC_PASSWORD`) and
refreshes the session on its own — no manual cookie needed.

Once connected, your agent has tools like `list_clusters`, `describe_cluster`,
`query_metrics`, `search_sr_log`, `run_starrocks_command`, `run_linux_cmd`,
`get_query_info`, `be_cn_memory_analyze`, `fe_jstack`, `get_dmesg`, and more.

## Prerequisites
1. A **BYOC Admin account** — your login for `byoc-admin.celerdata.com`.
2. **Node.js** installed.
3. The **`byoc-admin-mcp` repo** cloned locally (ask the team for access).
4. If your network can't reach `byoc-admin.celerdata.com` directly, connect the
   **VPN** first.

## Setup

**1. Build the server**
```bash
cd ~/Documents/development/celerdata/repos/byoc-admin-mcp
npm install
npm run build
```

**2. Add it as a local MCP server in Claude Desktop**
Settings → Developer → **Local MCP servers** → **Edit Config**, and add:
```json
"byoc-admin": {
  "command": "node",
  "args": [
    "/Users/<you>/Documents/development/celerdata/repos/byoc-admin-mcp/dist/index.js"
  ],
  "env": {
    "BYOC_USER": "<your-byoc-admin-username>",
    "BYOC_PASSWORD": "<your-byoc-admin-password>"
  }
}
```
Replace `<you>` with your home folder and fill in your own BYOC credentials.

**3. Restart Claude Desktop.** The `byoc-admin` server should show **running**.

## Verify
Run `list_clusters` in your agent. If it returns clusters, you're connected. The
server logs in with your `BYOC_USER`/`BYOC_PASSWORD` and refreshes automatically —
no cookie to paste.

## Notes
- Auth is fully automatic via `BYOC_USER` + `BYOC_PASSWORD`. (A legacy `BYOC_COOKIE`
  env var or the `set_credentials` tool still work as fallbacks, but you shouldn't
  need them.)
- `search_sr_log` responses are capped (~140KB, no pagination), so you pull dense
  slices around a time window rather than a full log dump.
- Keep your credentials private — they live in your local config, not in the repo.
