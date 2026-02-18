# Recommended Tools

> Curated tools for engineers, designers, and PMs across Claude Code, Cursor, and Codex.
>
> **Jump to:** [Claude Code](./recommended-tools-claude-code.md) | [Cursor](./recommended-tools-cursor.md) | [Codex](./recommended-tools-codex.md)

1. [Tool Capabilities Overview](#tool-capabilities-overview)
2. [Recommendations by Role](#recommendations-by-role)
3. [Per-Tool Setup Guides](#per-tool-setup-guides)
4. [Discovery Resources](#discovery-resources)

---

## Tool Capabilities Overview

| Capability | Claude Code | Cursor | Codex |
|-----------|-------------|--------|-------|
| Plugins / Extensions | `/plugin install` | VS Code extensions | — |
| MCP servers | `claude mcp add` | `.cursor/mcp.json` | `codex mcp add` |
| Custom rules | Skills | `.cursor/rules/` | System instructions |
| Plugin marketplace | Yes | VS Code Marketplace | — |

For MCP server setup across all three tools, see the [MCP setup guide](./mcp-setup.md).

---

## Recommendations by Role

### Everyone (Essentials)

| Purpose | Claude Code | Cursor | Codex |
|---------|-------------|--------|-------|
| Library docs | context7 plugin | Context7 MCP | Context7 MCP |
| Security scanning | security-guidance plugin | ESLint security rules | — |
| Commit workflows | commit-commands plugin | GitLens extension | — |
| Dev workflows | superpowers plugin | — | — |
| Error visibility | — | Error Lens extension | — |
| PR management | — | GitHub Pull Requests extension | — |

### Engineers

| Purpose | Claude Code | Cursor | Codex |
|---------|-------------|--------|-------|
| Code review | pr-review-toolkit, code-review plugins | GitLens extension | — |
| Browser testing | playwright plugin | — | — |
| Autonomous tasks | ralph-loop plugin | — | — |
| Import analysis | — | Import Cost extension | — |
| API testing | — | REST Client extension | — |
| Container support | — | Docker extension | — |
| Error tracking | — | — | Sentry MCP |
| Security skills | trailofbits skills marketplace | ESLint security plugin | — |

### Frontend Engineers

| Purpose | Claude Code | Cursor | Codex |
|---------|-------------|--------|-------|
| UI generation | frontend-design plugin | — | — |
| Design-to-code | figma plugin | Figma for VS Code extension | Figma MCP |
| CSS tooling | — | Tailwind IntelliSense, CSS Peek | — |
| HTML/JSX editing | — | Auto Rename Tag extension | — |
| Browser debugging | chrome-devtools-mcp | — | — |

### Designers

| Purpose | Claude Code | Cursor | Codex |
|---------|-------------|--------|-------|
| UI components | frontend-design plugin | — | — |
| Figma integration | figma plugin | Figma for VS Code extension | Figma MCP |
| Design systems | ui-designer (daymade) | — | — |
| Color visualization | — | Color Highlight extension | — |
| SVG editing | — | SVG Preview extension | — |

### Product Managers

| Purpose | Claude Code | Cursor | Codex |
|---------|-------------|--------|-------|
| Presentations | ppt-creator (daymade) | — | — |
| Research | deep-research (daymade) | — | — |
| Competitor analysis | competitors-analysis (daymade) | — | — |
| Meeting notes | meeting-minutes-taker (daymade) | — | — |
| Internal comms | internal-comms (anthropics) | — | — |
| Markdown editing | — | Markdown All in One extension | — |
| Project tracking | — | — | Atlassian MCP |
| Documentation | — | — | Notion MCP |

---

## Per-Tool Setup Guides

| Tool | Guide | What It Covers |
|------|-------|---------------|
| Claude Code | [recommended-tools-claude-code.md](./recommended-tools-claude-code.md) | Plugins, skills, install commands |
| Cursor | [recommended-tools-cursor.md](./recommended-tools-cursor.md) | VS Code extensions by role |
| Codex | [recommended-tools-codex.md](./recommended-tools-codex.md) | MCP servers, system instructions |

---

## Discovery Resources

**Claude Code:**
- [anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official) — official Anthropic directory
- [awesome-claude-code-plugins](https://github.com/ccplugins/awesome-claude-code-plugins) — curated community list
- [awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills) — skill catalog
- `/plugin menu` — browse installed marketplaces interactively

**Cursor:**
- [VS Code Marketplace](https://marketplace.visualstudio.com/) — all VS Code extensions work in Cursor
- `Cmd+Shift+X` — browse and install from within the editor

**Codex:**
- [modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers) — official MCP server directory
- `codex mcp add` — add servers via CLI
