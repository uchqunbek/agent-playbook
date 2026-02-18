# MCP Server Setup

> Setting up Jira, Notion, Figma, Sentry, and Context7 for Claude Code, Cursor, and Codex.

1. [Scope & Concepts](#scope--concepts)
2. [Cloud MCP Servers (OAuth)](#cloud-mcp-servers-oauth)
3. [Community / Utility Servers](#community--utility-servers)
4. [Project-Scoped Config](#project-scoped-config)
5. [Verifying & Managing](#verifying--managing)
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

| Server | URL |
|--------|-----|
| Atlassian (Jira + Confluence) | `https://mcp.atlassian.com/v1/sse` |
| Notion | `https://mcp.notion.so/sse` |
| Figma | `https://mcp.figma.com/sse` |
| Sentry | `https://mcp.sentry.dev/sse` |

### Claude Code

One command per server — OAuth opens in browser automatically:

```bash
claude mcp add atlassian --transport http --url https://mcp.atlassian.com/v1/sse --scope user
claude mcp add notion    --transport http --url https://mcp.notion.so/sse --scope user
claude mcp add figma     --transport http --url https://mcp.figma.com/sse --scope user
claude mcp add sentry    --transport http --url https://mcp.sentry.dev/sse --scope user
```

### Cursor

Add to `~/.cursor/mcp.json` (user scope). Cursor handles OAuth automatically:

```json
{
  "mcpServers": {
    "atlassian": { "url": "https://mcp.atlassian.com/v1/sse" },
    "notion":    { "url": "https://mcp.notion.so/sse" },
    "figma":     { "url": "https://mcp.figma.com/sse" },
    "sentry":    { "url": "https://mcp.sentry.dev/sse" }
  }
}
```

Or add them one by one via **Settings → Cursor Settings → MCP → Add New MCP Server**.

### Codex

Add to `~/.codex/config.toml`. SSE endpoints need the `mcp-remote` bridge:

```toml
[mcp_servers.atlassian]
command = "npx"
args = ["-y", "mcp-remote", "https://mcp.atlassian.com/v1/sse"]

[mcp_servers.notion]
command = "npx"
args = ["-y", "mcp-remote", "https://mcp.notion.so/sse"]

[mcp_servers.figma]
command = "npx"
args = ["-y", "mcp-remote", "https://mcp.figma.com/sse"]

[mcp_servers.sentry]
command = "npx"
args = ["-y", "mcp-remote", "https://mcp.sentry.dev/sse"]
```

### Server-Specific Notes

- **Atlassian** — one connection covers both Jira and Confluence. Multi-org users get prompted to pick one.
- **Notion** — only pages/databases you have access to are visible.
- **Figma** — useful for design tokens, screenshots, and component metadata.
- **Sentry** — use `search_issues` and `get_issue_details` to pull error context.

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

Slack and Datadog have no official MCP servers yet.

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

## Verifying & Managing

| Action | Claude Code | Cursor | Codex |
|--------|------------|--------|-------|
| List servers | `claude mcp list` | Settings → MCP | `/mcp` in TUI |
| Remove server | `claude mcp remove <name> --scope user` | Delete from JSON | Delete from TOML |

To expose MCP tools to agents, add a brief table to the project's `AGENTS.md` listing tool names and purposes — but keep setup instructions here, not in project docs.
