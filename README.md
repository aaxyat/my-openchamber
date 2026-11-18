# my-openchamber

Portable copy of my OpenChamber + OpenCode setup — UI settings, MCP servers, skills and extensions — so a new machine gets the same working environment with one command.

## What travels

| Path in repo | Comes from | Notes |
|---|---|---|
| `config/openchamber/settings.json` | `~/.config/openchamber/settings.json` | machine-local keys stripped (see below) |
| `config/openchamber/preferences.json` | `~/.config/openchamber/preferences.json` | UI preferences |
| `config/openchamber/plugin-switches.json` | `~/.config/openchamber/plugin-switches.json` | enabled/disabled plugins |
| `config/opencode/opencode.json` | `~/.config/opencode/opencode.json` | MCP servers, plugins, compaction, websearch |
| `config/cortexkit/magic-context.jsonc` | `~/.config/cortexkit/magic-context.jsonc` | Magic Context historian/dreamer models |
| `manifests/skills.json` | `~/.agents/skills` | 43 skills pinned to 3 upstream packs by commit |
| `manifests/extensions.json` | `~/.config/openchamber/extensions.json` | extension repos pinned by commit, plus capability grants |

Skills and extensions are **pinned, not vendored**: the repo stores repo URL + commit, and `apply` clones exactly that. Small repo, reproducible result.

## What never travels

`capture` copies an explicit allowlist, so these stay on the machine that owns them:

- `relay-identity.json` — private signing key
- `remote-clients.json`, `client-pairing-sessions.json` — pairing tokens
- `git-credential-endpoint.json` — per-install secret
- `~/.config/opencode/service.json` — OpenCode server password
- `~/.mcp-auth/`, `~/.dsh/.credentials.yaml` — provider credentials
- `~/.config/opencode/stitch-api-key` — Stitch MCP key; `opencode.json` reaches it as `{file:~/.config/opencode/stitch-api-key}` so the secret never enters this repo
- `~/.config/opencode/9router-api-key` — 9router provider key, referenced the same way; `capture` refuses to write a literal `apiKey`/`token`/`password` and tells you to move it to a file
- `chats/`, `memory.json` (unless `--memory`), `message-search.sqlite*` — session data
- `settings.json` keys: `projects`, `activeProjectId`, `lastDirectory`, `homeDirectory`, `desktopLocalPort`, `desktopWindowState`, `desktopHosts`, `securityScopedBookmarks`, `permissionAutoAccept`, `ghCliActive`, `sttLocalModel`

Provider auth is per machine on purpose: run `opencode auth login` (or import) after `apply`.

## New machine

```bash
git clone https://github.com/aaxyat/my-openchamber.git ~/Documents/Projects/my-openchamber
cd ~/Documents/Projects/my-openchamber
./bin/oc-sync apply --dry-run     # see exactly what would change
./bin/oc-sync apply
```

Then quit and reopen OpenChamber, and authenticate providers.

`apply` backs up every file it overwrites as `<name>.bak-<timestamp>` next to the original, so a bad run is one `mv` away from undone. On a machine that already has OpenChamber settings, `apply` merges: portable keys are overwritten, machine-local ones (`projects`, `lastDirectory`, window state, ports) are kept.

## After changing settings on a machine

```bash
./bin/oc-sync capture     # this machine -> repo
git diff                  # check what moved
git commit -am "settings: ..."
git push
```

`./bin/oc-sync status` shows drift between the repo and the current machine (files, skill links, pack commits) without writing anything.

## Prerequisites on a target machine

- `git`, `python3`
- `node` — resolved as `$NODE`, then `PATH`, then `~/.dsh/.../node/bin/node`, then `~/.local/bin/node`
- MCP dependencies under `${AGENT_TOOLS}`:
  - `playwright-mcp/` — `@playwright/mcp` + browsers
  - `cloudflare-mcp-oauth/` — `mcp-remote` bridge
- Cloudflare MCP auth copied to `~/.mcp-auth`
- Stitch MCP key at `~/.config/opencode/stitch-api-key`, mode `600`, no trailing newline (create it at stitch.withgoogle.com → Stitch settings → API Keys). Without it `stitch` still reports connected — `/mcp` initialize needs no auth — but every tool call fails with "missing required authentication credential"
- 9router key at `~/.config/opencode/9router-api-key`, no trailing newline; `opencode.json` reads it through `{file:...}`

## Environment overrides

| Variable | Default | Purpose |
|---|---|---|
| `OPENCHAMBER_CONFIG_DIR` | `~/.config/openchamber` | OpenChamber config root |
| `OPENCODE_CONFIG_DIR` | `~/.config/opencode` | OpenCode config root |
| `CORTEXKIT_CONFIG_DIR` | `~/.config/cortexkit` | Magic Context config root |
| `SKILLS_LINK_DIR` | `~/.agents/skills` | where skill symlinks are created |
| `SKILL_PACKS_DIR` | `${AGENT_TOOLS}/skill-packs` | where skill packs are cloned |
| `AGENT_TOOLS` | `/opt/agents/tools` if present, else `~/.local/share/agent-tools` | MCP dependency root |
| `NODE` | auto-detected | node binary for the local MCP servers |

## Notes

- Quit OpenChamber before `apply` — a running instance rewrites `settings.json` on exit.
- Extensions are cloned at pinned commits. If one needs a build step, run `bun install && bun run build` in its directory afterwards.
- `~/.config/opencode/AGENTS.md` and `~/.agents/{AGENTS,RULES}.md` are symlinks into the `my-dsh` repo, not this one.
- `apply --memory` is not a flag; capture memory with `./bin/oc-sync capture --memory` when you want agent memory to travel too.
