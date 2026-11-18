# Agent workspace

Durable agent artifacts for this machine, referenced by `~/.config/opencode/AGENTS.md` §7.

| Path | Holds |
|---|---|
| `bin` | scripts and helpers |
| `data` | non-regenerable outputs; deletions pass the Gate |
| `state` | checkpoints and locks |
| `logs` | run logs |
| `tmp` | scratch and manifests |

Owner-only permissions (`700`). No secrets.

The helper scripts named in `AGENTS.md` — `preflight`, `manifest`, `rm`, `gate-rm`, `undo-rm`, `git-snapshot`, `ssh-probe`, `ssh-verify`, `sshd-apply`, `retention` — are not installed yet. When one is missing, say so and do the step by hand; never claim its result.
