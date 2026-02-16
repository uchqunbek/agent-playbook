# AI Agent Guide

> Best practices for structuring project documentation so AI coding agents produce high-quality, consistent code.

---

## Why This Exists

AI coding agents (Claude Code, Cursor, GitHub Copilot) read your project documentation **before** writing a single line of code. The structure, size, and clarity of that documentation directly determines output quality.

Bad docs → bad code. No docs → the agent guesses. Monolithic docs → the agent wastes its context window and misses the rules that matter.

This guide provides a **proven, copy-paste-ready system** for organizing AI agent documentation across any tech stack.

---

## The Problem

| Scenario | What Happens |
|----------|-------------|
| **No docs** | Agent guesses conventions, invents patterns, produces inconsistent code |
| **Monolithic docs** (500+ lines) | Agent loads everything into context, runs out of space for actual code, misses key rules buried on line 400 |
| **Overlapping files** (CLAUDE.md + AGENTS.md with duplicate content) | Agent encounters contradictions, picks one randomly |
| **Scattered rules** (5+ files, no navigation) | Agent reads first file, misses the rest |

---

## The Solution: Layered Documentation

A 3-tier model that balances completeness with context window efficiency:

```
┌─────────────────────────────────────────┐
│  Tier 1: AGENTS.md (~200-250 lines)     │  ← Agent reads FIRST
│  Architecture, key rules, workflow,     │     Always loaded into context
│  navigation contract with links         │
├─────────────────────────────────────────┤
│  Tier 2: docs/ directory                │  ← Agent reads ON DEMAND
│  Detailed conventions, full code        │     Only when working on
│  examples, deep reference material      │     specific topics
├─────────────────────────────────────────┤
│  Tier 3: Source code                    │  ← Agent discovers via SEARCH
│  Actual patterns in the codebase        │     Grep, glob, file reads
│  The ultimate source of truth           │
└─────────────────────────────────────────┘
```

**Tier 1 — `AGENTS.md`** (always loaded, ~200-250 lines):
- Project overview, prerequisites, key commands
- Directory structure and architecture diagram
- Condensed conventions (key rules only, no code examples)
- Navigation contract pointing to Tier 2
- Agent workflow steps

**Tier 2 — `docs/` directory** (loaded on demand):
- Full code examples (good and bad)
- Detailed rules with explanations
- Domain-specific conventions
- Each file: 100-300 lines, focused on one topic

**Tier 3 — Source code** (searched, never pre-loaded):
- Real implementations agents discover via grep/glob
- The authoritative reference for "how is this actually done"

---

## 5 Core Principles

### 1. Single Source of Truth

Every rule lives in **exactly one place**. If `AGENTS.md` says "use `@SpringTransactional`" and `docs/code-best-practices.md` has the full explanation with examples, the rule is defined once with a clear link between them.

### 2. CLAUDE.md as Redirect

`CLAUDE.md` is a **4-line pointer** to `AGENTS.md`. No duplicated content, no drift.

```markdown
# CLAUDE.md

All project conventions, architecture, and coding standards are in [AGENTS.md](./AGENTS.md).
This file exists for compatibility with tools that read `CLAUDE.md` (e.g., Claude Code).
```

### 3. Navigation Contracts

The first section (Section 0) of `AGENTS.md` tells agents **exactly what to read and when**. This prevents agents from reading everything or missing critical docs.

```markdown
## 0. Navigation Contract

When starting a task, traverse documentation in this order:

1. **This file** (`AGENTS.md`) — workflow, architecture, key rules
2. **`docs/` files** — detailed conventions (only when you need code examples)
```

### 4. Context Window Budget

Keep `AGENTS.md` **under 250 lines**. This leaves room in the agent's context for:
- The actual code being modified
- Search results and file reads
- Conversation history
- Generated output

Use tables instead of prose, one-liners instead of paragraphs, links instead of inline content.

### 5. Tool Integration

Every AI tool has its own config file. Each one should **point to AGENTS.md**, not duplicate content:

| Tool | Config File | Content |
|------|-------------|---------|
| Claude Code | `CLAUDE.md` | 4-line redirect to AGENTS.md |
| Cursor | `.cursor/rules/project-rules.mdc` | Pointer to AGENTS.md + docs/ |
| GitHub Copilot | `.github/copilot-instructions.md` | Pointer to AGENTS.md + docs/ |

---

## Quick Start

1. **Pick your stack template** from [`templates/`](./templates/)
2. **Copy the files** into your repository root
3. **Fill in `<!-- CUSTOMIZE -->` placeholders** with your project-specific details
4. **Set up tool pointers** from [`templates/common/`](./templates/common/)
5. **Validate** — run through the [migration checklist](./guide/migration-checklist.md)

---

## Templates

| Stack | Template | Based On |
|-------|----------|----------|
| Java / Spring Boot | [`templates/java-spring-boot/`](./templates/java-spring-boot/) | shipper-tms-backend |
| Python / Django | [`templates/python-django/`](./templates/python-django/) | carrier-monolith |
| TypeScript / Frontend | [`templates/typescript-frontend/`](./templates/typescript-frontend/) | New template |
| Common (all stacks) | [`templates/common/`](./templates/common/) | Universal pointers |

---

## Deep-Dive Guides

| Guide | What It Covers |
|-------|---------------|
| [Layered Documentation](./guide/layered-documentation.md) | The 3-tier model explained in detail |
| [Navigation Contracts](./guide/navigation-contracts.md) | Teaching agents to read selectively |
| [Context Window Optimization](./guide/context-window-optimization.md) | Line budgets and condensing techniques |
| [Tool Integration](./guide/tool-integration.md) | CLAUDE.md, Cursor, Copilot, MCP setup |
| [Migration Checklist](./guide/migration-checklist.md) | Step-by-step adoption playbook |

---

## Real-World Examples

See how these principles were applied to actual SuperDispatch repositories:

| Repository | Before → After |
|------------|---------------|
| shipper-tms-backend (Java/Spring Boot) | [Example](./examples/shipper-tms-backend/) — 632-line monolith → 235-line layered + docs/ |
| carrier-monolith (Python/Django) | [Example](./examples/carrier-monolith/) — Overlapping files → consolidated single source |

---

## Chat Prompt

Paste this into Claude Code, Cursor, or Copilot Chat when working on a new repository:

> Set up AI agent documentation for this repository following the guide at
> `https://github.com/superdispatch/ai-agent-guide`. Read the README and the
> matching stack template, then create AGENTS.md, CLAUDE.md, and docs/ for
> this project.

---

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for how to improve these templates and guides.
