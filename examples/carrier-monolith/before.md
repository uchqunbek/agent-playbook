# carrier-monolith — Before

> State before the documentation consolidation.

---

## Structure

```
carrier-monolith/
├── AGENTS.md     (~200 lines — agent-focused, with navigation contract)
├── CLAUDE.md     (~155 lines — overlapping content with AGENTS.md)
└── docs/         (existed but referenced inconsistently)
```

## Problems

### 1. Two Files with Overlapping Content

Both `AGENTS.md` and `CLAUDE.md` contained:
- Project overview and architecture
- Directory structure
- Key commands
- Code conventions summaries
- Test conventions summaries
- Commit format

Approximately **75 lines of content** were duplicated between the two files, creating two sources of truth.

### 2. CLAUDE.md as a Standalone Document

Instead of redirecting to `AGENTS.md`, `CLAUDE.md` was a full 155-line document with:
- Quick reference commands
- Repository overview
- Full architecture section with code examples
- Convention summaries

This meant agents using Claude Code got one set of instructions, while agents reading `AGENTS.md` got a partially different set.

### 3. Inconsistent Depth

`AGENTS.md` had more detailed sections for some topics (Pub/Sub, Scripts, Principles) while `CLAUDE.md` had more condensed but different summaries. An agent reading both files would encounter different levels of detail and potentially conflicting guidance.

---

## Key Metrics

| Metric | Value |
|--------|-------|
| AGENTS.md line count | ~200 |
| CLAUDE.md line count | ~155 |
| Overlapping content | ~75 lines |
| docs/ files | 6 (code, test, git, naming, use-case, validation) |
| Sources of truth | 2 (AGENTS.md + CLAUDE.md) |
| Navigation contract | Present in AGENTS.md, absent in CLAUDE.md |
