# shipper-tms-backend — After

> State after applying the layered documentation model.

---

## Structure

```
shipper-tms-backend/
├── AGENTS.md                      (235 lines — architecture, key rules, navigation)
├── CLAUDE.md                      (4 lines — redirect to AGENTS.md)
├── docs/
│   ├── code-best-practices.md     (295 lines — full code examples)
│   ├── test-conventions.md        (287 lines — test base classes, helpers, patterns)
│   └── git-conventions.md         (52 lines — commit format, branch naming)
```

## What Changed

### 1. AGENTS.md Condensed to 235 Lines (63% reduction)

The 632-line monolithic file was restructured into a focused entry point:

- **Section 0: Navigation Contract** — tells agents what to read and when
- **Sections 1-7: Key rules only** — one-liner per convention, no code examples
- **Links to docs/** — every section ends with "Full rules: [docs/file.md]"
- **Quick Links table** — reinforces navigation at the bottom

Example of condensing (testing section):

```markdown
<!-- Before: ~80 lines with full code examples -->
<!-- After: 25 lines of key rules + link -->

## 3. Testing Rules (Key Rules)
- **Naming:** `{method}_{condition(s)}__{expected_result}` — all lowercase, double underscore
- **Structure:** Given-When-Then with section comments
- **Integration tests:** extend `AbstractServiceTest`, use real repositories, use `FAKER`
- **DO NOT** use `@SpringTransactional` on tests
- **DO NOT** mock repositories in integration tests
**Full rules with code examples:** [docs/test-conventions.md](docs/test-conventions.md)
```

### 2. docs/ Directory Created with 3 Files

All detailed content moved to focused reference files:
- `code-best-practices.md` (295 lines) — every code example, formatting rule, anti-pattern
- `test-conventions.md` (287 lines) — all test base classes, helpers, naming, full examples
- `git-conventions.md` (52 lines) — commit format with real examples from git log

### 3. CLAUDE.md Reduced to 4-Line Redirect

```markdown
# CLAUDE.md

All project conventions, architecture, and coding standards are in [AGENTS.md](./AGENTS.md).
This file exists for compatibility with tools that read `CLAUDE.md` (e.g., Claude Code).
```

---

## Key Metrics

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| AGENTS.md lines | 632 | 235 | **-63%** |
| CLAUDE.md lines | ~50 | 4 | **-92%** |
| docs/ files | 0 | 3 | +3 files |
| Total documentation lines | ~682 | 873 | +28% (more content, better organized) |
| Entry point (context loaded first) | 632 lines | 235 lines | **-63%** |
| Content preserved | — | 100% | No content lost |
| Navigation contract | None | Section 0 | Added |

## Outcome

- Agents load **63% less documentation** on every task
- Detailed examples are still available via links (loaded on demand)
- No content was lost — it was reorganized, not removed
- Single source of truth for all AI tools (CLAUDE.md → AGENTS.md)
