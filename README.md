# spec-marketplace

A personal [Claude Code](https://claude.com/claude-code) marketplace hosting the **`spec`** plugin — a portable, vault-backed spec workflow you can drop into any project.

`spec` turns ideas into implementation through three skills, routes every artifact into an Obsidian vault, and pins plan execution to the right model via dedicated agents. Everything is generic and config-driven: per-project settings live in a small JSON block in each project's (uncommitted) `CLAUDE.local.md`, so nothing from this bundle is committed to your project repos.

## What's in the box

- **Skills** — `spec:brainstorm` → `spec:plan` → `spec:execute`, wrappers over `superpowers` + `grill-with-docs` (+ your project's design skill) that route output to the vault.
- **Agents** — `spec:plan-executor` (Sonnet), `spec:plan-executor-heavy` (Opus), `spec:plan-reviewer` (Opus, read-only) for cheap, reviewed plan execution.
- **Command** — `/spec:setup` configures a project and scaffolds its vault.
- **Vault template** — a full Obsidian vault skeleton (Daily, References, Roadmap, Scratch, Projects + a project subtree), every folder carrying an `_index.md` that documents its purpose and the frontmatter to use.

## Prerequisites

`spec` *depends on* (but does not bundle) these — install/connect them separately:

| Dependency | Why |
|---|---|
| [`superpowers`](https://github.com/obra/superpowers) plugin | `brainstorming`, `writing-plans`, `subagent-driven-development` |
| `grill-with-docs` skill | domain-glossary + ADR grilling during brainstorm |
| `obsidian` MCP | reading/writing the vault |
| `playwright` MCP | used by typical design skills (visual review) |
| a project **design skill** (optional, project-owned) | visuals during brainstorm; named in config |

## Install

```sh
claude plugin marketplace add <this-repo>     # e.g. youruser/spec-marketplace
claude plugin install spec@spec-marketplace
```

## Per-project setup

In a project repo:

```
/spec:setup
```

It derives the project name from the repo, proposes a vault location (`~/Documents/Vault`), asks for the vault name, discovers any design skill, writes the config into `CLAUDE.local.md` (kept out of git via `.git/info/exclude` — never editing the committed `.gitignore`), and scaffolds the vault. It is **idempotent**: existing files are skipped, never overwritten.

The config block it writes:

```json
{
  "project": "your-repo",
  "vault": { "name": "YourVault", "root": "/Users/you/Documents/Vault", "subpath": "Projects/YourVault" },
  "designSkill": "your-design-skill"
}
```

Then: `/spec:brainstorm` → `/spec:plan` → `/spec:execute`.

## The vault model

A **vault = one project** (a group of related repos/microservices). Different projects use different vaults; you switch vaults by hand in Obsidian. The Obsidian MCP always talks to whichever vault is *open*, so the skills **guard** every write — they read the project `_index.md`'s `vault:` marker and refuse to write if the wrong vault is open, telling you to switch. Several repos in one project all point their config at the same vault + subpath, so a feature's spec lives once and is implemented across them.

## Global rules (your machine, not this bundle)

Cross-cutting personal rules (e.g. attribution policy, Markdown formatting) belong in `~/.claude/CLAUDE.md` and apply to all your work — `spec` doesn't manage them. Set them up once per machine.
