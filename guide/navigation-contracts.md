# Navigation Contracts

> Teaching AI agents to read documentation selectively instead of loading everything.

---

## What Is a Navigation Contract?

A navigation contract is a section at the **top** of `AGENTS.md` (Section 0) that tells agents:
1. **What to read** — which files, in which order
2. **When to stop** — conditions for not reading further
3. **When to go deeper** — triggers for loading Tier 2 docs

Without a navigation contract, agents either read everything (wasting context) or read nothing beyond the first file (missing rules).

---

## Simple Style (Recommended Default)

Best for most projects. Two-step traversal:

```markdown
## 0. Navigation Contract

When starting a task, traverse documentation in this order:

1. **This file** (`AGENTS.md`) — workflow, architecture, key rules
2. **`docs/` files** — detailed conventions (only when you need code examples)
```

**When to use:** Projects with 1-3 docs/ files and a straightforward AGENTS.md.

---

## Granular Style

For larger projects with many docs/ files. Adds explicit stop conditions and ordering:

```markdown
## 0. Navigation Contract

Agents MUST traverse context in this order:

1) **Explicit links in this file** →
2) **`core` context set** (code practices, tests, toggles) →
3) **`pubsub`** (messaging patterns) →
4) **`scripts`** (data migration) →
5) Fallback to `README.md`

Agents MUST stop after `limits.max_files` or when `max_tokens_hint` is approached.
Prefer `.md` before `.py` unless a code path is explicitly referenced.
```

**When to use:** Projects with 5+ docs/ files, multiple domains, or agents that consistently over-read.

---

## Key Design Decisions

### Section 0 Placement

Always make the navigation contract **Section 0** — the very first heading after any introductory line. Agents read files top-to-bottom and may stop partway through if the context window fills up. The navigation contract must be encountered first.

### Stop Conditions

Tell agents when to stop reading additional files:

| Stop Condition | Example |
|----------------|---------|
| **File count** | "Stop after reading 3 docs/ files" |
| **Token hint** | "Stop when max_tokens_hint is approached" |
| **Relevance** | "Only read docs/ files relevant to your current task" |
| **Explicit** | "Read test-conventions.md only when writing tests" |

### Quick Links Table

End `AGENTS.md` with a quick links table that reinforces the navigation contract:

```markdown
## Quick Links

| Document | Content |
|----------|---------|
| [docs/code-best-practices.md](docs/code-best-practices.md) | Code style, all code examples |
| [docs/test-conventions.md](docs/test-conventions.md) | Test base classes, helpers, naming |
| [docs/git-conventions.md](docs/git-conventions.md) | Commit format, branch naming |
```

---

## Testing Your Navigation Contract

Verify your contract works by checking these scenarios:

| Scenario | Agent Should |
|----------|-------------|
| New feature implementation | Read AGENTS.md → code-best-practices.md → test-conventions.md |
| Bug fix with test | Read AGENTS.md → test-conventions.md |
| Documentation-only change | Read AGENTS.md → git-conventions.md |
| Commit message formatting | Read AGENTS.md only (Section 7) |

If agents consistently read files they don't need, make your stop conditions more explicit.
If agents miss rules, check that the relevant link appears in AGENTS.md.

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Navigation contract buried at line 100+ | Move to Section 0, before any content |
| No links to docs/ files | Add explicit links after every condensed rule |
| Too many files listed in contract | Group into 2-3 "context sets" |
| No stop condition | Add "only when you need code examples" or file count limit |
| README.md listed as primary | README is for humans; AGENTS.md is for agents |
