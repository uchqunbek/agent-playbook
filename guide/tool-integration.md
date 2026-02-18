# Tool Integration

> How to set up CLAUDE.md, Cursor, GitHub Copilot, and MCP tools to use your AGENTS.md.

---

## The Pointer Pattern

Every AI tool has its own config file. Instead of writing tool-specific documentation, each config file **points to `AGENTS.md`** as the single source of truth.

```
CLAUDE.md ──────────────→ AGENTS.md ←──── .cursor/rules/project-rules.mdc
                              ↑
.github/copilot-instructions.md ──┘
```

This means:
- **One place to update** when conventions change
- **Zero content drift** between tools
- **Consistent agent behavior** regardless of which tool is used

---

## Claude Code (`CLAUDE.md`)

Claude Code auto-loads `CLAUDE.md` from the repository root.

**File:** `CLAUDE.md` (repository root)

```markdown
# CLAUDE.md

All project conventions, architecture, and coding standards are in [AGENTS.md](./AGENTS.md).
This file exists for compatibility with tools that read `CLAUDE.md` (e.g., Claude Code).
```

That's it. Four lines. Claude Code follows the link and loads `AGENTS.md`.

---

## Cursor (`.cursor/rules/`)

Cursor loads MDC rule files from `.cursor/rules/`.

**File:** `.cursor/rules/project-rules.mdc`

```markdown
---
description: Project coding standards and conventions
alwaysApply: true
---

# Project Rules

All project conventions, architecture, and coding standards are defined in:

- **[AGENTS.md](../../AGENTS.md)** — Architecture, workflow, and key rules
- **[docs/](../../docs/)** — Detailed conventions with code examples

When working on this project, always follow the conventions in AGENTS.md.
Refer to the docs/ directory for detailed code examples and patterns.
```

**Key settings:**
- `alwaysApply: true` — loaded for every conversation
- `description` — helps Cursor decide when to apply the rule

---

## GitHub Copilot (`.github/copilot-instructions.md`)

GitHub Copilot loads instructions from `.github/copilot-instructions.md`.

**File:** `.github/copilot-instructions.md`

```markdown
# Copilot Instructions

All project conventions, architecture, and coding standards are defined in:

- **[AGENTS.md](../AGENTS.md)** — Architecture, workflow, and key rules
- **[docs/](../docs/)** — Detailed conventions with code examples

Follow the conventions in AGENTS.md for all code generation.
Refer to the docs/ directory when you need detailed code examples.
```

---

## File Placement Summary

```
your-repo/
├── AGENTS.md                          # Source of truth (all tools read this)
├── CLAUDE.md                          # Claude Code pointer → AGENTS.md
├── docs/
│   ├── code-best-practices.md         # Detailed code conventions
│   ├── test-conventions.md            # Detailed test conventions
│   └── git-conventions.md             # Detailed git conventions
├── .cursor/
│   └── rules/
│       └── project-rules.mdc          # Cursor pointer → AGENTS.md
└── .github/
    └── copilot-instructions.md        # Copilot pointer → AGENTS.md
```

---

## MCP (Model Context Protocol)

MCP servers give AI agents access to external tools (Jira, Sentry, Figma, etc.).
For setup instructions, see **[MCP Server Setup](./mcp-setup.md)**.

---

## Adding a New Tool

When a new AI coding tool emerges:

1. Check what config file it reads (documentation or README)
2. Create a pointer file in the expected location
3. Point it to `AGENTS.md` and `docs/`
4. Update this guide

The pattern is always the same: **pointer → AGENTS.md → docs/**.
