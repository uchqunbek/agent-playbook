# Migration Checklist

> Step-by-step playbook for adopting layered AI agent documentation in your repository.

---

## Phase 1: Audit Existing Documentation

- [ ] List all documentation files in the repository (CLAUDE.md, AGENTS.md, README.md, CONTRIBUTING.md, docs/, etc.)
- [ ] Count lines in each file
- [ ] Identify overlapping content between files
- [ ] Identify which content is AI-agent-relevant vs. human-developer-relevant
- [ ] Note any tool-specific config files (.cursor/, .github/copilot-instructions.md)

**Output:** A list of files, line counts, and overlap areas.

---

## Phase 2: Create `docs/` Directory

- [ ] Create `docs/` directory in repository root
- [ ] Extract code conventions into `docs/code-best-practices.md`
  - All code examples, formatting rules, naming conventions
  - Anti-patterns with "do" and "don't" examples
- [ ] Extract test conventions into `docs/test-conventions.md`
  - Test naming patterns, base classes, helper services
  - Mocking rules, fixture patterns, anti-patterns
- [ ] Extract git conventions into `docs/git-conventions.md`
  - Commit message format, branch naming
  - Good and bad examples from recent history
- [ ] Extract domain-specific docs (if needed)
  - `naming-conventions.md`, `use-case-conventions.md`, `validation-conventions.md`

**Output:** `docs/` directory with 3-6 focused files.

---

## Phase 3: Build `AGENTS.md`

- [ ] Start from the stack template in this guide (or build from scratch)
- [ ] Write Section 0: Navigation Contract
- [ ] Write Section 1: Project Overview (name, prerequisites, key commands, directory structure, architecture)
- [ ] Write Section 2: Code Conventions — **key rules only**, one-liner per rule, link to docs/
- [ ] Write Section 3: Testing Rules — **key rules only**, test infra table, link to docs/
- [ ] Write domain sections (4, 5, ...) as needed — messaging, migrations, etc.
- [ ] Write Agent Workflow section — step-by-step process
- [ ] Write Git Conventions section — format + link to docs/
- [ ] Add Quick Links table at the bottom
- [ ] **Verify under 250 lines**

**Output:** `AGENTS.md` at 200-250 lines with links to all docs/ files.

---

## Phase 4: Set Up Tool Redirects

- [ ] Create or update `CLAUDE.md` — 4-line redirect to AGENTS.md
- [ ] Create `.cursor/rules/project-rules.mdc` — pointer to AGENTS.md + docs/
- [ ] Create `.github/copilot-instructions.md` — pointer to AGENTS.md + docs/
- [ ] Delete any duplicated content from old tool configs

**Output:** All tool config files point to AGENTS.md as single source.

---

## Phase 5: Validate

- [ ] All markdown links resolve to existing files
- [ ] `AGENTS.md` is under 250 lines (`wc -l AGENTS.md`)
- [ ] No content is duplicated between AGENTS.md and docs/
- [ ] No content is duplicated between CLAUDE.md and AGENTS.md
- [ ] Every `docs/` file is linked from AGENTS.md
- [ ] `<!-- CUSTOMIZE -->` placeholders (if using template) are all filled in
- [ ] An AI agent following the navigation contract can find any convention in 2 hops
- [ ] Test with an actual AI agent — give it a task and observe what it reads

**Output:** Validated, working layered documentation.

---

## Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Leaving orphan content in CLAUDE.md | Replace with 4-line redirect |
| AGENTS.md grows past 250 lines over time | Move new content to docs/, keep AGENTS.md as summary |
| docs/ files grow too large | Split by topic, not by chronology |
| Navigation contract points to non-existent files | Validate links after every structural change |

---

## Timeline Estimate

| Phase | Effort |
|-------|--------|
| Phase 1: Audit | 30 minutes |
| Phase 2: Create docs/ | 1-2 hours |
| Phase 3: Build AGENTS.md | 1-2 hours |
| Phase 4: Tool redirects | 15 minutes |
| Phase 5: Validate | 30 minutes |
| **Total** | **3-4 hours** |
