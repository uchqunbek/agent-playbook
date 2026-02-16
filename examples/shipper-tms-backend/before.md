# shipper-tms-backend — Before

> State before the layered documentation optimization.

---

## Structure

```
shipper-tms-backend/
├── AGENTS.md     (632 lines — monolithic, everything in one file)
├── CLAUDE.md     (duplicate content from AGENTS.md)
└── docs/         (did not exist)
```

## Problems

### 1. Monolithic AGENTS.md (632 lines)

Everything was in a single file:
- Project overview and architecture
- Full code examples with 20+ line Java snippets
- Complete test conventions with all base class examples
- Git conventions with full example lists
- Detailed domain rules (messaging, migrations)

At 632 lines, this consumed a significant portion of the agent's context window before any code was loaded.

### 2. Full Code Examples in AGENTS.md

Code examples that belonged in reference docs were inline:

```markdown
## Code Conventions
...
(20-line Java code example for formatting)
...
(15-line Java code example for streams)
...
(10-line Java code example for transactions)
```

These examples are valuable for reference but don't need to be loaded for every task.

### 3. No Navigation Contract

No guidance for agents on what to read and when. Agents loaded the entire 632-line file for every task, regardless of whether they needed test conventions, git rules, or migration patterns.

### 4. CLAUDE.md Duplication

`CLAUDE.md` contained a subset of the same content as `AGENTS.md`, creating two sources of truth that could drift apart over time.

---

## Key Metrics

| Metric | Value |
|--------|-------|
| AGENTS.md line count | 632 |
| CLAUDE.md line count | ~50 (duplicate content) |
| docs/ directory | Did not exist |
| Total files for agents | 2 (with overlap) |
| Navigation contract | None |
| Code examples in AGENTS.md | ~200 lines of Java code |
