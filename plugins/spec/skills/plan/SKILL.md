---
name: plan
description: Use when creating, writing, or drafting an implementation plan for a feature, or when the user invokes /spec:plan. Saves the plan to the project's Obsidian vault and embeds the pinned-agent execution policy.
---

# spec:plan

Wrapper over the standard planning workflow that adds two rules: plans live in the project's Obsidian vault, and every plan embeds the pinned-agent execution policy so it can be executed cheaply later. Generic and config-driven.

## Project config (read this first)

Read the `spec` config — the fenced ```json under `## spec configuration` in `CLAUDE.local.md`. It provides `project`, `vault.name`, `vault.root`, `vault.subpath`, and optional `designSkill`. **If absent, STOP** and tell the user to run `/spec:setup`. `{subpath}` = `vault.subpath`, `{root}` = `vault.root`.

## Vault protocol (applies to every vault write)

1. **Guard the active vault.** Before writing, `mcp__obsidian__vault_read` `{subpath}/_index.md` and confirm its `vault:` frontmatter equals `vault.name`. Mismatch or missing → **STOP**: "Open the **<vault.name>** vault in Obsidian, then say continue."
2. **Respect `_index.md`.** Before creating the plan note, read `{subpath}/Plans/_index.md` and copy its frontmatter template into the new plan — do not hardcode the frontmatter here.
3. **Large files.** For plans the REST API truncates (~3000+ lines), use the `Write` tool against `{root}/{subpath}/Plans/<file>.md`; otherwise use `mcp__obsidian__vault_write`.

## Workflow

**REQUIRED SUB-SKILL:** Use `superpowers:writing-plans` for plan structure, task granularity, TDD steps, no-placeholder discipline, and self-review. This skill only overrides *where the plan goes* and *what it must contain*.

1. **Gather inputs from the vault first.** Read the spec, the relevant `{subpath}/Contexts/` glossary, and any related `{subpath}/ADRs/` via the obsidian MCP (`mcp__obsidian__vault_read`; use `vault_get_document_map` then read by heading for large notes). Map the affected code (e.g. with the Explore agent) before writing. **If the spec has a `## Design` section** (specs from `spec:brainstorm` do), open that design and embed a **Design reference** block in the plan repeating the design reference verbatim, so the executor builds against the approved visuals instead of guessing.

2. **Save to the vault, not the repo.** Write to `{subpath}/Plans/YYYY-MM-DD-<feature>.md`. Never write to `docs/superpowers/plans/`.

3. **Plan frontmatter:** copy the template from `{subpath}/Plans/_index.md` (the folder's `_index.md` is the source of truth for the field set).

4. **MANDATORY: embed the Execution model policy section** (template below) near the top, right after the Tech Stack header. Decide which tasks are cross-cutting/**heavy** — large refactors, orchestration/engine code, deletions that cascade across files, hard debugging — and list them under `plan-executor-heavy`; everything else goes under `plan-executor`.

## Execution model policy — paste into every plan, fill in the task lists

````markdown
## Execution model policy (enforce on a fresh session)

Run **subagent-driven**: the orchestrator reads this table and dispatches each task to a **pinned subagent** via the Agent tool's `subagent_type`. The agent's frontmatter fixes its model + effort, so the model is enforced by *which agent runs*, not by remembering to switch. One fresh subagent per task also clears context between tasks.

| Tasks | `subagent_type` | Pins |
|---|---|---|
| <routine task numbers> | `plan-executor` | Sonnet, effort `high` |
| <cross-cutting task numbers> | `plan-executor-heavy` | Opus, effort `medium` |
| A task stuck on a red test | re-dispatch to `plan-executor-heavy` | Opus (`/effort xhigh` only to debug) |

After each executor returns, the orchestrator dispatches `plan-reviewer` (Opus, effort `high`, read-only) on the task's diff — Sonnet writes, Opus checks, catching missed detail cheaply. On **CHANGES-NEEDED**, re-dispatch the **same** executor with the fix list, then review again; on **APPROVE**, surface to the user for sign-off. Dispatch by **explicit** `subagent_type` (description matching is assistive only). The orchestrator stays on Sonnet or Opus-`low`; never leave `xhigh`/`max` as a standing default.
````

The agents `plan-executor`, `plan-executor-heavy`, and `plan-reviewer` ship with this plugin. If your harness can't see them, the plan can't execute as written — install/enable the `spec` plugin before relying on this section.

## Executing it later

To run the finished plan in a fresh session, use the **`spec:execute`** skill.
