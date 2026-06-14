# claude-code-setup

A reusable, project-scoped Claude Code setup — **skills**, **MCP servers**, **hooks**, and
coding guidelines — that you can drop into any project. Paste-and-run friendly.

Repo: https://github.com/grknbyk/claude-code-setup

```bash
git clone https://github.com/grknbyk/claude-code-setup.git
```

Last updated: 2026-06-15

---

## 0. Prerequisites (runtimes)

| Tool | Why | Check |
|------|-----|-------|
| Node + npx | Node skills, `skills` CLI, npx-based MCP, impeccable CLI | `node -v && npx -v` |
| Python 3.11+ | Python MCP servers | `python --version` |
| uv / uvx | Runs the Python MCP servers | `uv --version && uvx --version` |
| git | `git` MCP server, cloning sources | `git --version` |

Install uv (gives you uvx): `powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"`

---

## 1. Skills  (`.claude/skills/`)

Project-scoped. Installed via the `skills` CLI
(`npx -y skills add <owner/repo> --skill <name> --agent claude-code`) unless noted.

### impeccable
- Source: https://github.com/pbakaus/impeccable.git
- Tutorial: https://impeccable.style/tutorials/getting-started/
- Install: `npx impeccable skills install`
- Ships a design-detector hook — enable it in §3.

### context-mode
- Source: https://github.com/mksglu/context-mode.git
- Install: `npx -y skills add mksglu/context-mode --skill context-mode --agent claude-code`
- Routes large tool outputs to subagents to save tokens.
- Needs an MCP server (the `ctx_*` tools) + routing hooks — fully set up project-level in §2a and §3.

### karpathy-guidelines
- Source: https://github.com/multica-ai/andrej-karpathy-skills.git
- Install: `npx -y skills add forrestchang/andrej-karpathy-skills --skill karpathy-guidelines --agent claude-code`
- Behavioral guardrails to cut common LLM coding mistakes. (MIT)
- **Also distilled into project-root `CLAUDE.md`** (loaded every session → always-on). Keep both:
  CLAUDE.md = always applied; the skill = on-demand full version.

### grill-me
- Source: https://github.com/mattpocock/skills/tree/main/skills/productivity/grill-me
- Install: `npx -y skills add mattpocock/skills --skill grill-me --agent claude-code`
- Stress-tests an existing plan (relentless one-at-a-time Q&A).

### caveman
- Source: https://github.com/mattpocock/skills/tree/main/skills/productivity/caveman
- Install: `npx -y skills add mattpocock/skills --skill caveman --agent claude-code`
- Ultra-compressed output mode (~75% fewer tokens).

### skill-creator
- Source: https://github.com/anthropics/skills.git
- Install: `npx -y skills add anthropics/skills --skill skill-creator --agent claude-code`
- Create / eval / optimize / package skills.

### handoff  (customized)
- Source: https://github.com/mattpocock/skills/tree/main/skills/productivity/handoff
- Install: `npx -y skills add mattpocock/skills --skill handoff --agent claude-code`
- Then edit the save line in `.claude/skills/handoff/SKILL.md`:

```diff
- Save to the temporary directory of the user's OS - not the current workspace.
+ Save to `.claude/handoffs/<YYYY-MM-DD-HHMMSS>-<slug>.md` in the current workspace
+ (create the directory if needed), and also overwrite `.claude/handoffs/latest.md`
+ with the same content.
```

Resume a new session with `@.claude/handoffs/latest.md`.

---

## 2. MCP servers  (`.mcp.json`, project root)

Project-scoped. Claude Code prompts to approve them on startup. Reload after changes.

### 2a. Reference servers
Source for all four: https://github.com/modelcontextprotocol/servers.git

`.mcp.json`:
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "."]
    },
    "fetch":  { "command": "uvx", "args": ["mcp-server-fetch"] },
    "git":    { "command": "uvx", "args": ["mcp-server-git"] },
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"],
      "env": { "MEMORY_FILE_PATH": ".claude/mcp-memory.json" }
    },
    "context-mode":    { "command": "npx", "args": ["-y", "context-mode"] },
    "playwright":      { "command": "npx", "args": ["-y", "@playwright/mcp"] },
    "chrome-devtools": { "command": "npx", "args": ["-y", "chrome-devtools-mcp"] }
  }
}
```

| Server | Runner | Detail |
|--------|--------|--------|
| filesystem | npx `@modelcontextprotocol/server-filesystem` | Read/write/search; access locked to the dir passed as arg. |
| fetch | uvx `mcp-server-fetch` | Fetch a URL → markdown. |
| git | uvx `mcp-server-git` | Read/search/manipulate a git repo (run `git init` first). |
| memory | npx `@modelcontextprotocol/server-memory` | Knowledge-graph memory → `.claude/mcp-memory.json`. |
| context-mode | npx `context-mode` | 11 `ctx_*` tools for the context-mode skill. Needs Node ≥22.5. See §3 for the routing hook. |
| playwright | npx `@playwright/mcp` | Cross-browser automation + test authoring (Chrome/Firefox/WebKit/Edge). Source: https://github.com/microsoft/playwright-mcp.git |
| chrome-devtools | npx `chrome-devtools-mcp` | Chrome-only debugging + perf (traces, Lighthouse, network). Source: https://github.com/ChromeDevTools/chrome-devtools-mcp.git |

### 2b. Disabling an MCP server (without deleting)
Add the server name to `disabledMcpjsonServers` in `.claude/settings.json` (shared) or
`.claude/settings.local.json` (private). Reload to apply.
```json
{ "disabledMcpjsonServers": ["chrome-devtools"] }
```
Other ways: `/mcp` (view/manage at runtime) · `claude mcp remove <name>` or edit `.mcp.json` (permanent).

### 2c. Browser MCPs — alternate install (plugin form)
The `.mcp.json` entries above are the project-scoped install. The plugin equivalents:
- Playwright: `/plugin marketplace add microsoft/playwright-cli`
- Chrome DevTools: `/plugin install chrome-devtools-mcp@chromedevtools-chrome-devtools-mcp`

---

## 3. Hooks  (`.claude/settings.json` — committed)

### Impeccable design detector hook
Runs the 41-rule deterministic detector (no LLM, no tokens) after Claude edits a UI
file (`.tsx .jsx .html .vue .svelte .astro .css .scss .sass .less .ts .js`) and surfaces
findings as a system reminder. Fires on agent Edit/Write/MultiEdit, not manual IDE edits.

Enable:
```bash
npx impeccable skills install
node .claude/skills/impeccable/scripts/hook-admin.mjs on
```
Manage:
```bash
/impeccable hooks status | off | ignore-rule <id> | ignore-file <glob>
```
Config: `.impeccable/config.json` (shared) + `.impeccable/config.local.json` (per-dev).

### Context-mode routing hooks (project-level, no plugin)
context-mode is three project-level parts:
1. **Skill** — `.claude/skills/context-mode/` (routing guidance).
2. **MCP server** — `.mcp.json` `context-mode` entry → the 11 `ctx_*` tools. Needs Node ≥22.5.
3. **Hooks** — in `.claude/settings.json` (committed, travels with the repo):
   - `SessionStart` → `npx -y context-mode hook claude-code sessionstart` (injects the routing rules into the session).
   - `PreToolUse` (matcher `Bash|WebFetch|Read|Grep|Agent|mcp__`) → `npx -y context-mode hook claude-code pretooluse` (real-time routing + subagent injection).

We deliberately avoid the plugin (`/plugin install …`) because it installs **globally**; the
`npx` hooks above keep everything project-level.

> Perf note: `PreToolUse` fires on most tool calls and shells out to `npx` each time, which
> adds latency. If it feels slow, delete the `PreToolUse` block — `SessionStart` alone still
> injects the routing rules. For zero npx overhead, `npm install context-mode` locally and
> point the hook command at `node ./node_modules/context-mode/...`.

---

## 4. Fresh-machine reproduction (order)

```bash
# 1. prerequisites (see §0): node, python, uv, git

# 2. skills
npx -y skills add mksglu/context-mode --skill context-mode --agent claude-code
npx -y skills add forrestchang/andrej-karpathy-skills --skill karpathy-guidelines --agent claude-code
npx -y skills add mattpocock/skills --skill grill-me --agent claude-code
npx -y skills add mattpocock/skills --skill caveman --agent claude-code
npx -y skills add anthropics/skills --skill skill-creator --agent claude-code
npx -y skills add mattpocock/skills --skill handoff --agent claude-code   # then apply the §1 diff
npx impeccable skills install
node .claude/skills/impeccable/scripts/hook-admin.mjs on

# 3. MCP servers — create .mcp.json (see §2a)
# 4. settings — create .claude/settings.json (committed) with (see §3):
#      - context-mode hooks: SessionStart + PreToolUse
#      - impeccable PostToolUse hook
#      - "disabledMcpjsonServers": ["chrome-devtools"]   (see §2b)
#    (note: `hook-admin.mjs on` writes the impeccable hook to settings.local.json instead;
#     move it into settings.json if you want it committed with the package)
# 5. CLAUDE.md — distill karpathy-guidelines into project-root CLAUDE.md (always-on, committed)

# 6. update impeccable later
npx impeccable skills update
```

### Required actions after setup
1. **Reload Claude Code** (restart the window/session) — hooks and `.mcp.json` changes only
   take effect on reload.
2. **Approve the MCP servers** when prompted on startup. Enabled now: filesystem, fetch, git,
   memory, context-mode, playwright. `chrome-devtools` is disabled by default — remove it from
   `disabledMcpjsonServers` (§2b) to use it.
3. **Verify**: `/mcp` lists the servers; editing a `.html`/`.css` file should trigger the
   impeccable detector; a new session should show context-mode routing.

---

## 5. Notes / gotchas
- **impeccable build mismatch**: the repo ships per-harness builds. Claude Code needs the
  `.claude/` build (scripts at `.claude/skills/impeccable/scripts/`). The `.agents/` (Codex)
  build has the wrong paths and its setup step fails on Claude Code.
- **`.impeccable/` lives at the project root** (not under `.claude/`) on purpose — harness-neutral
  config shared across Claude/Cursor/Codex. Don't move it; scripts resolve it from the project root.
- **MCP server code is cached, not vendored**: npx → `%LOCALAPPDATA%\npm-cache`, uvx →
  `%LOCALAPPDATA%\uv\cache`. Only `.mcp.json` + the memory file live in the project.

---

## 6. Git & portability

This `.claude/` setup is meant to be committed and cloned into other projects.

**Committed (travels with the package):**
- `.claude/settings.json` — hooks + `disabledMcpjsonServers`
- `.claude/skills/**` — all skills
- `.mcp.json` — MCP servers (paths are project-root-relative, so they work in any project)
- `.impeccable/config.json` — shared hook config (enabled, ignore rules)
- `CLAUDE.md` (project root) — always-on coding guidelines (karpathy distillation)
- `AGENTS.md` + `project/AGENTS.md` — DOX documentation tree (see §7)
- `project/` — your application code & files
- `README.md`, `skills-lock.json`

**Git-ignored (per-machine / regenerated — see `.gitignore`):**
- `.claude/settings.local.json` — personal/machine overrides
- `.impeccable/config.local.json` — per-dev hook consent
- `.claude/mcp-memory.json`, `.impeccable/hook.cache.json` — runtime data
- `node_modules/`

**The `.json` / `.local.json` pattern:** the `*.json` file carries shared config and is
committed; the matching `*.local.json` holds per-developer/machine state, is git-ignored, and
regenerates automatically. The `.local.json` files are never part of the package — you can
delete them locally and they come back when needed.

**Why relative paths in `.mcp.json`:** `.mcp.json` does NOT expand variables (no
`${CLAUDE_PROJECT_DIR}`). Claude Code launches MCP servers with the project root as the working
directory, so `.` (filesystem) and `.claude/mcp-memory.json` (memory) resolve correctly in any
project. Verify with `/mcp` after the first reload.

**Reuse in another project:**
```bash
git clone <your-repo> newproj && cd newproj
# reload Claude Code → approve MCP servers. That's it.
```

---

## 7. Docs: DOX (`AGENTS.md`)

[DOX](https://github.com/agent0ai/dox) is a lightweight documentation framework — a hierarchy of
`AGENTS.md` files that any agent (Claude Code, Codex, etc.) reads before editing and updates after.
No install; just `AGENTS.md` files committed in the tree.

- `AGENTS.md` (root) — project-wide rail + the Child DOX Index.
- `project/AGENTS.md` — child contract for the codebase under `project/`.

Workflow: read the `AGENTS.md` chain from root → target **before** editing; update the nearest
owning `AGENTS.md` **after** meaningful changes. To grow the tree as the codebase expands, tell the
agent: `Initialize DOX tree for this project now.`
