# AI Agent Workflow Guide

> Templates and guides for structuring AI agent documentation across any tech stack.
> For detailed theory and techniques, see the `guide/` directory.

---

## 0. Navigation Contract

When starting a task, traverse documentation in this order:

1. **This file** (`AGENTS.md`) — workflow, structure, key rules
2. **`guide/` files** — detailed theory and techniques (only when you need deep context)

---

## 1. Repository Overview

**What:** Copy-paste-ready templates + deep-dive guides for structuring `AGENTS.md`, `CLAUDE.md`, and `docs/` so AI coding agents produce high-quality, consistent code.

### Prerequisites

- Git
- Markdown editor
- (Optional) [markdown-link-check](https://github.com/tcort/markdown-link-check) for link validation

### Key Commands

```bash
wc -l templates/*/AGENTS.md          # Check AGENTS.md line counts
wc -l guide/*.md                     # Check guide line counts
grep -r "CUSTOMIZE" templates/       # Verify placeholders in templates
find . -name "*.md" -exec markdown-link-check {} \;  # Validate all links
```

### Directory Structure

```
agent-playbook/
├── templates/              # Stack-specific template sets
│   ├── common/             # Universal tool pointers (CLAUDE.md, Cursor, Copilot)
│   ├── java-spring-boot/   # Java / Spring Boot template
│   ├── typescript-frontend/ # TypeScript / Frontend template
│   ├── python-django/      # Python / Django template
│   ├── python-fastapi/     # Python / FastAPI template
│   ├── golang/             # Go template
│   └── cypress-e2e/        # Cypress E2E QA template
├── guide/                  # Deep-dive guides (Tier 2)
├── examples/               # Real-world before/after migrations
├── CONTRIBUTING.md         # Contribution guidelines
└── README.md               # Project introduction and quick start
```

---

## 2. Content Conventions (Key Rules)

| Rule | Detail |
|------|--------|
| Placeholder format | `<!-- CUSTOMIZE: description -->` — HTML comment, descriptive, inline |
| AGENTS.md line budget | **250 lines max** |
| docs/*.md line budget | **300 lines max** |
| guide/*.md line budget | **150 lines max** |
| Formatting preference | Tables over prose, one-liners over paragraphs |
| Single source of truth | No duplicated content between AGENTS.md and docs/ |
| CLAUDE.md | Always a 4-line redirect to AGENTS.md — never duplicate content |

**Full details:** [CONTRIBUTING.md](./CONTRIBUTING.md)

---

## 3. Template Structure (Key Rules)

Every stack template **must** include:

| File | Purpose |
|------|---------|
| `AGENTS.md` | Architecture, key rules, workflow — under 250 lines |
| `docs/code-best-practices.md` | Detailed code conventions with examples |
| `docs/test-conventions.md` | Testing rules with examples |
| `docs/git-conventions.md` | Commit and branch conventions |

### Section Numbering

Templates follow a consistent section order:

| Section | Content |
|---------|---------|
| 0 | Navigation Contract |
| 1 | Project Overview (prerequisites, commands, structure) |
| 2+ | Key Rules (code, testing, domain-specific) |
| N | Agent Workflow |
| N+1 | Git Conventions |
| Final | Quick Links table |

**Detailed rationale:** [guide/layered-documentation.md](./guide/layered-documentation.md)

---

## 4. Validation

Before submitting changes, verify:

| Check | Command / Method |
|-------|------------------|
| AGENTS.md under 250 lines | `wc -l templates/*/AGENTS.md` |
| Guide files under 150 lines | `wc -l guide/*.md` |
| All markdown links resolve | `find . -name "*.md" -exec markdown-link-check {} \;` |
| Placeholders present in templates | `grep -r "CUSTOMIZE" templates/` |
| No duplicated content | Manual review — AGENTS.md and docs/ must not repeat rules |

---

## 5. Agent Workflow

Follow this step-by-step process for every task:

### Step 1: Create Branch

- Branch from `main`
- Naming: `feature/add-<stack>-template` or `fix/<description>`

### Step 2: Read

- Read this file for project context
- Read relevant `guide/` files for theory and techniques

### Step 3: Study Existing Templates

- Find similar templates in `templates/`
- Identify common patterns (section structure, placeholders, formatting)

### Step 4: Plan

- Identify which files to create or modify
- Check that `<!-- CUSTOMIZE -->` placeholders are positioned correctly
- Verify line budgets before starting

### Step 5: Generate

- Follow existing template structure exactly
- Use the same section numbering and formatting conventions
- Include placeholders for all project-specific content

### Step 6: Validate

- Run the checks from Section 4
- Fix any line budget overages or broken links

### Step 7: Self-Review & Submit

- Run `git diff` and review every changed file
- Verify no unintended changes are included
- Commit with a descriptive message
- Open a PR with a clear description of what changed and why

---

## 6. Git Conventions

- **Branch from `main`** — never commit directly
- **Descriptive commit messages** — no ticket prefix required for this repo
- **PRs via fork + branch** — all changes merge through pull requests
- **Self-review required** — review all changes before creating a PR

**Full details:** [CONTRIBUTING.md](./CONTRIBUTING.md)

---

## Quick Links

| Document | Content |
|----------|---------|
| [guide/layered-documentation.md](./guide/layered-documentation.md) | The 3-tier documentation model |
| [guide/navigation-contracts.md](./guide/navigation-contracts.md) | Teaching agents to read selectively |
| [guide/context-window-optimization.md](./guide/context-window-optimization.md) | Line budgets and condensing techniques |
| [guide/tool-integration.md](./guide/tool-integration.md) | CLAUDE.md, Cursor, Copilot, MCP setup |
| [guide/mcp-setup.md](./guide/mcp-setup.md) | MCP server setup for engineers |
| [guide/migration-checklist.md](./guide/migration-checklist.md) | Step-by-step adoption playbook |
| [CONTRIBUTING.md](./CONTRIBUTING.md) | How to contribute templates and guides |
