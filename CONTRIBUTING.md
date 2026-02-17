# Contributing

Thank you for helping improve the Agent Playbook. This document explains how to contribute effectively.

---

## What to Contribute

| Contribution Type | Where It Goes |
|-------------------|---------------|
| New stack template | `templates/<stack-name>/` |
| Improvements to existing templates | `templates/<stack-name>/` |
| New deep-dive guide | `guide/<topic>.md` |
| Real-world migration example | `examples/<repo-name>/` |
| Bug fixes (broken links, typos) | Wherever the issue is |

---

## Template Guidelines

### Structure

Every stack template **must** include:
- `AGENTS.md` — under 250 lines, with navigation contract
- `docs/code-best-practices.md` — detailed code conventions with examples
- `docs/test-conventions.md` — testing rules with examples
- `docs/git-conventions.md` — commit and branch conventions

### Placeholders

Use `<!-- CUSTOMIZE: description -->` for project-specific content:

```markdown
<!-- CUSTOMIZE: Replace with your project's tech stack and version -->
- **Java 21** (toolchain enforced)
- **Gradle 9.x** (use `./gradlew` wrapper)
- **Spring Boot 3.x**
```

Placeholders must be:
- **HTML comments** (invisible in rendered markdown, visible in source)
- **Descriptive** — explain what to replace and why
- **Positioned inline** — immediately before or around the content to customize

### Line Budgets

| File | Max Lines |
|------|-----------|
| `AGENTS.md` | 250 |
| `docs/*.md` | 300 |
| `guide/*.md` | 150 |

---

## Guide Guidelines

Files in `guide/` should be:
- **Stack-agnostic** — no Java/Python/TypeScript-specific content
- **Actionable** — explain the "how", not just the "what"
- **Concise** — 100-150 lines, use tables and bullet points
- **Self-contained** — each guide covers one topic completely

---

## Examples Guidelines

Before/after examples should include:
- **Metrics** — line counts, file counts, concrete improvements
- **What changed** — specific structural changes made
- **Why** — the problems that motivated the change
- **Excerpts** — representative snippets (not full file copies)

---

## Pull Request Process

1. Fork the repository
2. Create a branch: `feature/add-<stack>-template` or `fix/<description>`
3. Make your changes following the guidelines above
4. Verify:
   - All markdown links resolve to existing files
   - `AGENTS.md` templates are under 250 lines
   - `<!-- CUSTOMIZE -->` placeholders are present where needed
5. Open a PR with a clear description of what changed and why

---

## Validation

Before submitting, run the same checks CI runs:

```bash
# Check AGENTS.md line counts
wc -l templates/*/AGENTS.md

# Verify all internal links (install markdown-link-check)
find . -name "*.md" -exec markdown-link-check {} \;

# Check for placeholder presence in templates
grep -r "CUSTOMIZE" templates/
```
