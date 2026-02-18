# Recommended Tools

> Curated plugins and skills for engineers, designers, and PMs using Claude Code.

1. [How to Install](#how-to-install)
2. [Essential Plugins (All Roles)](#essential-plugins-all-roles)
3. [Plugins by Role](#plugins-by-role)
4. [Quick Install Cheatsheet](#quick-install-cheatsheet)
5. [Discovery Resources](#discovery-resources)

---

## How to Install

**Plugins** bundle slash commands, subagents, skills, MCP servers, and hooks into a single package. **Skills** are instruction files that teach Claude how to approach tasks (TDD, debugging, etc.).

| Action | Command |
|--------|---------|
| Install from official directory | `/plugin install <name>@claude-plugin-directory` |
| Add a community marketplace | `/plugin marketplace add <org/repo>` |
| Install from a marketplace | `/plugin install <name>@<marketplace>` |
| Browse installed plugins | `/plugin menu` |

---

## Essential Plugins (All Roles)

Every Super Dispatch team member should install these:

| Plugin | Source | Install | What It Does |
|--------|--------|---------|--------------|
| superpowers | [obra/superpowers](https://github.com/obra/superpowers) | `/plugin marketplace add obra/superpowers` | Core dev workflows: brainstorming, TDD, debugging, planning, code review |
| context7 | Official | `/plugin install context7@claude-plugin-directory` | Up-to-date library docs — prevents hallucinated APIs |
| security-guidance | Official | `/plugin install security-guidance@claude-plugin-directory` | Pre-commit vulnerability scanning (XSS, injection, secrets) |
| commit-commands | Official | `/plugin install commit-commands@claude-plugin-directory` | Structured commit and branch workflows |

---

## Plugins by Role

### Engineers

**All engineers:**

| Plugin | Install | Purpose |
|--------|---------|---------|
| pr-review-toolkit | `/plugin install pr-review-toolkit@claude-plugin-directory` | Parallel PR review agents with confidence scoring |
| code-review | `/plugin install code-review@claude-plugin-directory` | Automated diff analysis before human review |
| playwright | `/plugin install playwright@claude-plugin-directory` | Browser testing via natural language |
| ralph-loop | `/plugin install ralph-loop@claude-plugin-directory` | Autonomous multi-task sessions (migrations, CRUD, test coverage) |

**Security skills** (marketplace: [trailofbits/skills](https://github.com/trailofbits/skills)):

| Skill | Purpose |
|-------|---------|
| static-analysis | CodeQL / Semgrep vulnerability scanning |
| differential-review | Security-focused PR diff review |
| insecure-defaults | Detect hardcoded secrets, weak crypto |

**Frontend engineers** — add:

| Plugin | Install | Purpose |
|--------|---------|---------|
| frontend-design | `/plugin install frontend-design@claude-plugin-directory` | Production-grade UI generation with intentional design choices |
| figma | `/plugin install figma@claude-plugin-directory` | Design-to-code from Figma files — tokens, spacing, components |
| chrome-devtools-mcp | Community | Live browser debugging with network/console/performance access |

### Designers

| Plugin | Install | Purpose |
|--------|---------|---------|
| frontend-design | `/plugin install frontend-design@claude-plugin-directory` | Generate UI components from descriptions |
| figma | `/plugin install figma@claude-plugin-directory` | Extract design tokens, screenshots, component metadata |
| ui-designer | daymade marketplace | Extract design systems from screenshots/mockups |

Cross-reference: [MCP setup guide](./mcp-setup.md) for Figma MCP server configuration.

### Product Managers

| Plugin | Install | Purpose |
|--------|---------|---------|
| ppt-creator | daymade marketplace | Slide deck generation with data visualization |
| deep-research | daymade marketplace | Comprehensive research workflows |
| competitors-analysis | daymade marketplace | Competitive research and analysis |
| meeting-minutes-taker | daymade marketplace | Meeting documentation from notes |
| internal-comms | anthropics marketplace | Status reports, newsletters, FAQs |

Cross-reference: [MCP setup guide](./mcp-setup.md) for Jira + Notion server configuration.

---

## Quick Install Cheatsheet

```bash
# === Everyone ===
/plugin install context7@claude-plugin-directory
/plugin install security-guidance@claude-plugin-directory
/plugin install commit-commands@claude-plugin-directory
/plugin marketplace add obra/superpowers

# === Engineers ===
/plugin install pr-review-toolkit@claude-plugin-directory
/plugin install code-review@claude-plugin-directory
/plugin install playwright@claude-plugin-directory
/plugin marketplace add trailofbits/skills

# === Frontend Engineers (add to above) ===
/plugin install frontend-design@claude-plugin-directory
/plugin install figma@claude-plugin-directory

# === Designers ===
/plugin install frontend-design@claude-plugin-directory
/plugin install figma@claude-plugin-directory
/plugin marketplace add daymade/claude-code-skills

# === Product Managers ===
/plugin marketplace add daymade/claude-code-skills
/plugin marketplace add anthropics/skills
```

---

## Discovery Resources

- [anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official) — official Anthropic directory
- [awesome-claude-code-plugins](https://github.com/ccplugins/awesome-claude-code-plugins) — curated community list
- [awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills) — skill catalog
- `/plugin menu` — browse installed marketplaces interactively
