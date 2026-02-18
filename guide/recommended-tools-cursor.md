# Recommended Tools — Cursor

> VS Code extensions for engineers, designers, and PMs using Cursor.
>
> **See also:** [Overview](./recommended-tools.md) | [Claude Code](./recommended-tools-claude-code.md) | [Codex](./recommended-tools-codex.md)

1. [How to Install](#how-to-install)
2. [Essential Extensions (All Roles)](#essential-extensions-all-roles)
3. [Extensions by Role](#extensions-by-role)
   - [Engineers](#engineers)
   - [Designers](#designers)
   - [Product Managers](#product-managers)
4. [Quick Install Cheatsheet](#quick-install-cheatsheet)

---

## How to Install

Cursor is a VS Code fork — all VS Code extensions work. MCP servers are configured separately (see [MCP setup guide](./mcp-setup.md)). Custom rules go in `.cursor/rules/` (see [tool integration guide](./tool-integration.md)).

| Action | How |
|--------|-----|
| Install via UI | `Cmd+Shift+X` → search → Install |
| Install via CLI | `cursor --install-extension <publisher.id>` |
| Manage extensions | `Cmd+Shift+X` → Installed tab |

---

## Essential Extensions (All Roles)

| Extension | ID | What It Does |
|-----------|----|--------------|
| ESLint | `dbaeumer.vscode-eslint` | Lint JS/TS on save — catches errors before commit |
| GitLens | `eamodio.gitlens` | Inline blame, file history, branch compare |
| Error Lens | `usernamehw.errorlens` | Inline error/warning highlights — no need to hover |
| GitHub Pull Requests | `github.vscode-pull-request-github` | Review and manage PRs without leaving the editor |

---

## Extensions by Role

### Engineers

**All engineers:**

| Extension | ID | Purpose |
|-----------|----|---------|
| Import Cost | `wix.vscode-import-cost` | Show bundle size of imports inline |
| REST Client | `humao.rest-client` | Send HTTP requests from `.http` files |
| Docker | `ms-azuretools.vscode-docker` | Dockerfile syntax, image management, compose support |

**Frontend engineers** — add:

| Extension | ID | Purpose |
|-----------|----|---------|
| Tailwind CSS IntelliSense | `bradlc.vscode-tailwindcss` | Autocomplete, linting, hover preview for Tailwind classes |
| CSS Peek | `pranaygp.vscode-css-peek` | Go-to-definition for CSS classes in HTML/JSX |
| Auto Rename Tag | `formulahendry.auto-rename-tag` | Rename paired HTML/JSX tags simultaneously |

### Designers

| Extension | ID | Purpose |
|-----------|----|---------|
| Color Highlight | `naumovs.color-highlight` | Visualize color values inline |
| SVG Preview | `SimonSiefke.svg-preview` | Preview SVG files in the editor |
| Figma for VS Code | `figma.figma-vscode-extension` | Inspect Figma designs alongside code |

### Product Managers

| Extension | ID | Purpose |
|-----------|----|---------|
| Markdown All in One | `yzhang.markdown-all-in-one` | TOC generation, formatting shortcuts, preview |
| Markdown Preview Enhanced | `shd101wyy.markdown-preview-enhanced` | Rich preview with diagrams and math |

---

## Quick Install Cheatsheet

```bash
# === Everyone ===
cursor --install-extension dbaeumer.vscode-eslint
cursor --install-extension eamodio.gitlens
cursor --install-extension usernamehw.errorlens
cursor --install-extension github.vscode-pull-request-github

# === Engineers ===
cursor --install-extension wix.vscode-import-cost
cursor --install-extension humao.rest-client
cursor --install-extension ms-azuretools.vscode-docker

# === Frontend Engineers (add to above) ===
cursor --install-extension bradlc.vscode-tailwindcss
cursor --install-extension pranaygp.vscode-css-peek
cursor --install-extension formulahendry.auto-rename-tag

# === Designers ===
cursor --install-extension naumovs.color-highlight
cursor --install-extension SimonSiefke.svg-preview
cursor --install-extension figma.figma-vscode-extension

# === Product Managers ===
cursor --install-extension yzhang.markdown-all-in-one
cursor --install-extension shd101wyy.markdown-preview-enhanced
```
