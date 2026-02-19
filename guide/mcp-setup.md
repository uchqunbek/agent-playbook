# MCP Server Setup

> Setting up Jira, Notion, Figma, Sentry, and Context7 for Claude Code, Cursor, and Codex.

1. [Scope & Concepts](#scope--concepts)
2. [Cloud MCP Servers (OAuth)](#cloud-mcp-servers-oauth)
   - [Claude Code](#claude-code)
   - [Cursor](#cursor)
   - [Codex](#codex)
3. [Community / Utility Servers](#community--utility-servers)
   - [Context7](#context7)
4. [Project-Scoped Config](#project-scoped-config)
5. [After Setup](#after-setup)
   - [Restart or Reload](#1-restart-or-reload)
   - [Authorize](#2-authorize-oauth-servers-only)
   - [Confirm Connection](#3-confirm-connection)
   - [Removing a Server](#4-removing-a-server)
---

## Scope & Concepts

This guide is for **engineers setting up their local environment**. Do not copy it into a project's `docs/` directory.

| | User scope (personal) | Project scope (shared) |
|-|----------------------|----------------------|
| **Claude Code** | `~/.claude.json` (via `claude mcp add --scope user`) | `.mcp.json` |
| **Cursor** | `~/.cursor/mcp.json` (or Settings UI) | `.cursor/mcp.json` |
| **Codex** | `~/.codex/config.toml` | `.codex/config.toml` |

**Rule of thumb:** OAuth / API tokens → user scope. Auth-free tools → project scope.

---

## Cloud MCP Servers (OAuth)

All four servers below require OAuth via a browser popup. Use **user scope**.

| Server | URL | Notes |
|--------|-----|-------|
| Atlassian | `https://mcp.atlassian.com/v1/mcp` | Covers both Jira and Confluence |
| Notion | `https://mcp.notion.com/mcp` | Only pages you have access to |
| Figma | `https://mcp.figma.com/mcp` | Design tokens, screenshots, metadata |
| Sentry | `https://mcp.sentry.dev/mcp` | `search_issues`, `get_issue_details` |

### Claude Code

One command per server — OAuth opens in browser automatically:

```bash
claude mcp add atlassian --transport http --url https://mcp.atlassian.com/v1/mcp --scope user
claude mcp add notion    --transport http --url https://mcp.notion.com/mcp --scope user
claude mcp add figma     --transport http --url https://mcp.figma.com/mcp --scope user
claude mcp add sentry    --transport http --url https://mcp.sentry.dev/mcp --scope user
```

### Cursor

Add to `~/.cursor/mcp.json` (user scope). Cursor handles OAuth automatically:

```json
{
  "mcpServers": {
    "atlassian": { "url": "https://mcp.atlassian.com/v1/mcp" },
    "notion":    { "url": "https://mcp.notion.com/mcp" },
    "figma":     { "url": "https://mcp.figma.com/mcp" },
    "sentry":    { "url": "https://mcp.sentry.dev/mcp" }
  }
}
```

### Codex

Add to `~/.codex/config.toml`. SSE endpoints need the `mcp-remote` bridge — repeat for each server URL from the table above:

```toml
[mcp_servers.atlassian]
command = "npx"
args = ["-y", "mcp-remote", "https://mcp.atlassian.com/v1/mcp"]
```

---

## Community / Utility Servers

### Context7

Fetches up-to-date library docs so the agent does not hallucinate APIs. No auth required.

**Claude Code:**
```bash
claude mcp add context7 -- npx -y @upstash/context7-mcp@latest
```

**Cursor** (`~/.cursor/mcp.json` or `.cursor/mcp.json`):
```json
{ "mcpServers": { "context7": { "command": "npx", "args": ["-y", "@upstash/context7-mcp@latest"] } } }
```

**Codex** (`~/.codex/config.toml` or `.codex/config.toml`):
```toml
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp@latest"]
```

---

## Project-Scoped Config

For auth-free servers shared across the team, commit a config file to the repo root. Anyone who clones gets the server automatically.

**Claude Code** (`.mcp.json`) / **Cursor** (`.cursor/mcp.json`) — same format:
```json
{ "mcpServers": { "context7": { "command": "npx", "args": ["-y", "@upstash/context7-mcp@latest"] } } }
```

**Codex** (`.codex/config.toml`):
```toml
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp@latest"]
```

**Warning:** Never commit tokens, API keys, or secrets in these files. OAuth servers must use user scope.

---

## After Setup

### 1. Restart or Reload

| Tool | What to do |
|------|-----------|
| **Claude Code** | Nothing — `claude mcp add` registers immediately |
| **Cursor** | Reload window (`Cmd+Shift+P` → "Reload Window"), or restart Cursor |
| **Codex** | Start a new session — config is read on startup |

### 2. Authorize (OAuth Servers Only)

On first use of an OAuth server, a browser popup opens. Approve the requested permissions and select your organization/workspace. The token is stored locally and refreshed automatically. If the popup does not appear, check that the URL is correct, your browser allows popups, and you are logged into the service.

### 3. Confirm Connection

| Tool | How to check |
|------|-------------|
| **Claude Code** | `claude mcp list` — server should appear with status |
| **Cursor** | Settings → MCP — green dot means connected |
| **Codex** | Type `/mcp` in the TUI — lists active servers |

If a server shows as disconnected, restart the tool and try again. For stdio servers, ensure `npx`/`node` is on your `PATH`.

### 4. Removing a Server

| Tool | How to remove |
|------|--------------|
| **Claude Code** | `claude mcp remove <name> --scope user` |
| **Cursor** | Delete the entry from the JSON config file |
| **Codex** | Delete the `[mcp_servers.<name>]` block from TOML |
