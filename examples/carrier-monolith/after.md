# carrier-monolith — After

> State after applying the consolidation.

---

## Structure

```
carrier-monolith/
├── AGENTS.md                          (164 lines — single source of truth)
├── CLAUDE.md                          (4 lines — redirect to AGENTS.md)
├── docs/
│   ├── code-best-practices.md         (code conventions with examples)
│   ├── test-conventions.md            (pytest patterns, mocking, fixtures)
│   ├── git-conventions.md             (commit format, types, prefixes)
│   ├── naming-conventions.md          (variables, exceptions, type hints)
│   ├── use-case-conventions.md        (execute(), transactions, signals)
│   └── validation-conventions.md      (serializer vs view vs use case)
└── .cursor/rules/                     (pointer to AGENTS.md)
```

## What Changed

### 1. CLAUDE.md Consolidated to 4-Line Redirect

The 155-line standalone `CLAUDE.md` was replaced with:

```markdown
# CLAUDE.md

All project conventions, architecture, and coding standards are in [AGENTS.md](./AGENTS.md).
This file exists for compatibility with tools that read `CLAUDE.md` (e.g., Claude Code).
```

All unique content from `CLAUDE.md` was merged into `AGENTS.md` and `docs/` files.

### 2. AGENTS.md Refined to 164 Lines

The `AGENTS.md` was streamlined:
- Navigation contract preserved (Section 0)
- Duplicate content removed (already in `CLAUDE.md`)
- Detailed rules moved to `docs/` with links
- Condensed conventions to one-liners where possible

### 3. Single Source of Truth Established

Before: Two files with overlapping content → agent reads both, encounters partial conflicts.
After: One file (`AGENTS.md`) with links to `docs/` → agent has clear, non-contradictory guidance.

### 4. docs/ Files as Tier 2 Reference

All 6 docs/ files serve as detailed reference:
- `code-best-practices.md` — enums, logging, celery, migrations, early returns, imports
- `test-conventions.md` — pytest naming, mocking best practices, fixtures, serializer testing
- `git-conventions.md` — commit format with types and ticket prefixes
- `naming-conventions.md` — variables, exceptions, abbreviations, type hints
- `use-case-conventions.md` — execute(), no nesting, atomic transactions, signals
- `validation-conventions.md` — serializer vs view vs use case separation

---

## Key Metrics

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| AGENTS.md lines | ~200 | 164 | -18% |
| CLAUDE.md lines | ~155 | 4 | **-97%** |
| Overlapping content | ~75 lines | 0 | **Eliminated** |
| Sources of truth | 2 | 1 | **Single source** |
| docs/ files | 6 | 6 | No change |
| Net line reduction | — | -75 | Overlap removed |
| Navigation contract | Partial | Complete | Unified |

## Outcome

- **Single source of truth** — all AI tools read the same conventions
- **Zero overlap** — no contradictory guidance between files
- **75 lines of duplication eliminated** — less maintenance burden
- **CLAUDE.md is now a stable pointer** — never needs updating when conventions change
- **docs/ files unchanged** — detailed reference material preserved
