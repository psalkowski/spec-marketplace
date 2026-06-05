---
name: execute
description: Use when executing, running, or implementing an implementation plan stored in the project's Obsidian vault, or when the user invokes /spec:execute. Pins each task to the right model via dedicated subagents and gates every task behind an Opus review.
---

# spec:execute

Wrapper over the standard plan-execution loop that pins each task to the right model via dedicated subagents and gates every task behind an Opus review. Generic and config-driven.

## Project config (read this first)

Read the `spec` config — the fenced ```json under `## spec configuration` in `CLAUDE.local.md`. It provides `project`, `vault.name`, `vault.root`, `vault.subpath`. **If absent, STOP** and tell the user to run `/spec:setup`. `{subpath}` = `vault.subpath`.

## Workflow

**REQUIRED SUB-SKILL:** Use `superpowers:subagent-driven-development` for the per-task dispatch, checkpoints, and overall loop. This skill only overrides *which subagent* runs each task and *who reviews*.

1. **Load the plan from the vault.** Before reading, confirm the active vault: `mcp__obsidian__vault_read` `{subpath}/_index.md` and check its `vault:` frontmatter equals `vault.name`; mismatch → **STOP** and ask the user to open the **<vault.name>** vault in Obsidian. Then read `{subpath}/Plans/<plan>.md` (`vault_get_document_map`, then `vault_read` by heading for large plans). Read its **Execution model policy** table — that table is the routing source of truth.

2. **Dispatch each task by explicit `subagent_type`.** Use `spec:plan-executor` for routine tasks and `spec:plan-executor-heavy` for the tasks the table marks cross-cutting. Plugin agents are namespaced — the bare names (`plan-executor`, …) do not resolve. If the plan's table predates the namespacing and lists bare names, map them to the `spec:`-prefixed ones. Pass the task's full text (files, steps, code, commands) in the dispatch prompt. Do NOT auto-select an agent by its description.

3. **Review gate after every task.** When the executor returns, dispatch `spec:plan-reviewer` (read-only, Opus) on the task's diff. On **CHANGES-NEEDED**, re-dispatch the **same** executor with the reviewer's fix list, then review again. On **APPROVE**, surface the result to the user.

4. **Honour stop gates.** If the plan says "stop and ask" before a task, stop and ask before dispatching it.

5. **Keep the orchestrator cheap.** The main session only reads the plan, dispatches, and relays reviews — stay on Sonnet (or Opus-`low`). Never set `xhigh`/`max` as a standing default; bump effort only to debug a stuck test, then drop back.

## Requires

The agents `spec:plan-executor`, `spec:plan-executor-heavy`, and `spec:plan-reviewer` (they ship with the `spec` plugin, namespaced under it) and the `obsidian` MCP. If the agents aren't visible to your harness, enable the `spec` plugin before running.
