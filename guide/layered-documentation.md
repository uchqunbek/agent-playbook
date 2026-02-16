# Layered Documentation

> Why flat docs fail and how the 3-tier pyramid solves it.

---

## Why Flat Documentation Fails

A single monolithic file creates these problems:

| Problem | Impact |
|---------|--------|
| **Context window waste** | 500+ lines of docs leaves less room for code, search results, and conversation |
| **Signal-to-noise ratio** | Agents treat all lines equally — detailed code examples compete with critical architecture rules |
| **Maintenance burden** | Every edit risks breaking something; contributors avoid touching the file |
| **No selective loading** | Agent reads everything even when working on a narrow task |

---

## The 3-Tier Pyramid

```
        ┌──────────────┐
        │   Tier 1     │  AGENTS.md (~200-250 lines)
        │  "What to    │  Summaries, architecture, workflow,
        │   know"      │  navigation contract
        ├──────────────┤
        │   Tier 2     │  docs/ directory (100-300 lines each)
        │  "How to     │  Full code examples, detailed rules,
        │   do it"     │  domain-specific conventions
        ├──────────────┤
        │   Tier 3     │  Source code (unlimited)
        │  "How it's   │  Real implementations the agent
        │   done"      │  discovers via search
        └──────────────┘
```

### Tier 1: `AGENTS.md`

**Always loaded** into the agent's context. Budget: **200-250 lines**.

Contains:
- Navigation contract (Section 0)
- Project overview, prerequisites, key commands
- Directory structure with short descriptions
- Architecture diagram (text-based)
- Key conventions **without code examples** — one-line rules only
- Agent workflow steps
- Quick links table pointing to Tier 2

Does NOT contain:
- Full code examples (use links to docs/)
- Verbose explanations
- Domain deep-dives
- Tool configuration

### Tier 2: `docs/` Directory

**Loaded on demand** when the agent needs detailed guidance for a specific topic.

Standard files every project should have:

| File | Content |
|------|---------|
| `code-best-practices.md` | Formatting, naming, patterns, anti-patterns with code examples |
| `test-conventions.md` | Test naming, structure, base classes, helpers, anti-patterns |
| `git-conventions.md` | Commit format, branch naming, examples |

Optional domain-specific files (add as needed):
- `naming-conventions.md` — detailed naming patterns
- `use-case-conventions.md` — business logic patterns
- `validation-conventions.md` — validation layer separation
- `messaging-conventions.md` — event/queue patterns

### Tier 3: Source Code

**Never pre-loaded.** Agents discover patterns by searching the codebase when they need a concrete example. This is the ultimate source of truth — docs describe intent, code shows reality.

---

## What Goes Where

| Content Type | Tier | Example |
|-------------|------|---------|
| "Use `@SpringTransactional`" | Tier 1 (AGENTS.md) | One-line rule |
| Code example of `@SpringTransactional` usage | Tier 2 (docs/) | 10-line snippet |
| Actual `@SpringTransactional` in production code | Tier 3 (source) | Agent finds via grep |
| Project architecture diagram | Tier 1 (AGENTS.md) | 5-line ASCII diagram |
| Full API layer documentation | Tier 2 (docs/) | Detailed guide |
| Test naming convention | Tier 1 (AGENTS.md) | Format string only |
| Test examples with all base classes | Tier 2 (docs/) | Full examples |

---

## Line Budgets

| File | Target | Max |
|------|--------|-----|
| `AGENTS.md` | 200 | 250 |
| `docs/*.md` (each) | 150 | 300 |
| Total Tier 2 | 500 | 1000 |

These budgets ensure agents always have room in their context window for the actual work.

---

## Cross-Referencing

Link from Tier 1 to Tier 2 using this pattern:

```markdown
## 2. Code Conventions (Key Rules)

- 120-char line limit (`.editorconfig`)
- DI via constructor: `@RequiredArgsConstructor` + `final` fields
- Use `@SpringTransactional` (not plain `@Transactional`)

**Full rules with code examples:** [docs/code-best-practices.md](docs/code-best-practices.md)
```

This gives agents:
1. The rule itself (enough for most tasks)
2. A clear path to detailed examples (when they need more)

---

## Measuring Success

After applying the layered model, check:

- [ ] `AGENTS.md` is under 250 lines
- [ ] Every rule in `AGENTS.md` has a corresponding section in `docs/`
- [ ] No content is duplicated between Tier 1 and Tier 2
- [ ] Agents following the navigation contract can find any rule in 2 hops
- [ ] Total Tier 2 content is organized by topic, not by chronology
