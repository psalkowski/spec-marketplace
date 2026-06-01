# spec

Vault-backed spec workflow for Claude Code: **brainstorm → plan → execute**, routing every artifact into an Obsidian vault and pinning plan execution to the right model. Generic and config-driven — reads per-project settings from a JSON block in `CLAUDE.local.md`.

## Components

| Kind | Name | Role |
|---|---|---|
| skill | `spec:brainstorm` | `superpowers:brainstorming` + `grill-with-docs` + (configured) design skill → spec in the vault |
| skill | `spec:plan` | `superpowers:writing-plans` → plan in the vault, with the execution-model policy table |
| skill | `spec:execute` | `superpowers:subagent-driven-development` → dispatches the agents below, review-gated |
| agent | `plan-executor` | Sonnet, `high` — routine plan tasks |
| agent | `plan-executor-heavy` | Opus, `medium` — cross-cutting tasks |
| agent | `plan-reviewer` | Opus, `high`, read-only — reviews each task's diff |
| command | `/spec:setup` | writes config + scaffolds the vault (idempotent) |
| templates | `templates/vault`, `templates/project` | the Obsidian vault skeleton `/spec:setup` materializes |

## Configuration

`/spec:setup` writes this into `CLAUDE.local.md` under `## spec configuration`:

```json
{
  "project": "your-repo",
  "vault": { "name": "YourVault", "root": "/Users/you/Documents/Vault", "subpath": "Projects/YourVault" },
  "designSkill": "your-design-skill"
}
```

- `vault.subpath` is **required** — the project's space inside the vault.
- `designSkill` is **optional** — omit it and `spec:brainstorm` skips the design step (and says so).

## Conventions live in the vault, not the skills

Each vault folder's `_index.md` documents its purpose and the frontmatter template for notes in it. The skills **read the folder's `_index.md` before creating a note** and copy its template — so the note schema is owned by the vault and stays per-project customizable.

See the repo root `README.md` for prerequisites and install.
