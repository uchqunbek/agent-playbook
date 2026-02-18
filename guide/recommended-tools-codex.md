# Recommended Tools — Codex

> MCP servers and system instructions for engineers using OpenAI Codex CLI.
>
> **See also:** [Overview](./recommended-tools.md) | [Claude Code](./recommended-tools-claude-code.md) | [Cursor](./recommended-tools-cursor.md)

1. [Capabilities](#capabilities)
2. [MCP Servers](#mcp-servers)
   - [Recommended Servers by Role](#recommended-servers-by-role)
3. [System Instructions](#system-instructions)
   - [Per-session (CLI flag)](#per-session-cli-flag)
   - [Per-project (file)](#per-project-file)
   - [Per-user (config.toml)](#per-user-configtoml)
4. [Configuration Reference](#configuration-reference)

---

## Capabilities

Codex does not have a plugin or extension system. It supports two customization mechanisms:

| Mechanism | What It Does | How to Configure |
|-----------|-------------|-----------------|
| MCP servers | External tool integration (Jira, Notion, Figma, Context7) | `codex mcp add` or `config.toml` |
| System instructions | Custom rules and conventions for agent behavior | CLI flag or `instructions.md` file |

---

## MCP Servers

MCP is the primary way to extend Codex. See the [MCP setup guide](./mcp-setup.md) for full setup instructions covering:

- **Cloud servers** (OAuth): Atlassian/Jira, Notion, Figma, Sentry
- **Community servers** (no auth): Context7

### Recommended Servers by Role

| Role | Servers |
|------|---------|
| Everyone | Context7 (library docs) |
| Engineers | Context7, Sentry (error tracking) |
| Designers | Context7, Figma (design tokens) |
| PMs | Context7, Atlassian (Jira), Notion |

---

## System Instructions

System instructions tell Codex how to behave — equivalent to Claude Code skills or Cursor rules.

### Per-session (CLI flag)

```bash
codex --instructions "Follow AGENTS.md conventions. Use composition over inheritance."
```

### Per-project (file)

Create `instructions.md` in the project root. Codex reads it automatically on startup:

```markdown
# Instructions

Follow all conventions in AGENTS.md.
Use the project's existing test framework.
Never disable tests — fix them.
```

### Per-user (config.toml)

Add default instructions in `~/.codex/config.toml`:

```toml
model = "o4-mini"
instructions = "Always follow AGENTS.md. Use composition over inheritance."
```

---

## Configuration Reference

| Scope | Path | Use For |
|-------|------|---------|
| User (personal) | `~/.codex/config.toml` | Default model, personal instructions, OAuth MCP servers |
| Project (shared) | `.codex/config.toml` | Auth-free MCP servers shared with the team |
| Project instructions | `instructions.md` (repo root) | Project-specific agent behavior rules |
