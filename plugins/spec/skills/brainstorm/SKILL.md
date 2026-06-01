---
name: brainstorm
description: Use when brainstorming, exploring, scoping, or specifying a new feature, page, component, or behaviour before implementation, or when the user invokes /spec:brainstorm. Routes every artifact to the project's Obsidian vault and runs grilling + the project's design skill.
---

# spec:brainstorm

Wrapper over the standard brainstorming workflow that forces the right sub-skills to actually run — `superpowers:brainstorming`, `grill-with-docs`, and (when configured) the project's **design skill** — and routes every artifact to the project's Obsidian vault. Generic and config-driven: it reads where the vault is, and which design skill (if any) to use, from this project's config.

## Project config (read this first)

Read the `spec` config — the fenced ```json under `## spec configuration` in `CLAUDE.local.md` (already in your context). It provides:

- `project` — repo/project slug
- `vault.name` — the Obsidian vault this project writes to
- `vault.root` — vault filesystem root (for the large-file `Write` fallback)
- `vault.subpath` — the project's space inside the vault (vault-relative, e.g. `Projects/YourProject`)
- `designSkill` — the skill to invoke for visuals, or absent if the project has none

**If the config block is missing, STOP** and tell the user to run `/spec:setup` first. Throughout this skill, `{subpath}` means `vault.subpath` and `{root}` means `vault.root`.

## Vault protocol (applies to every vault write)

1. **Guard the active vault.** The Obsidian MCP talks to whatever vault is *currently open*; it cannot switch. Before your first write, `mcp__obsidian__vault_read` `{subpath}/_index.md` and confirm its `vault:` frontmatter equals `vault.name`. If the file is missing or the name differs, **STOP**: "Open the **<vault.name>** vault in Obsidian, then say continue." Never write until it matches.
2. **Respect `_index.md` on every new note.** Before creating a note in any folder, read that folder's `_index.md`: follow its stated purpose (is this the right folder?) and **copy its "Frontmatter template (copy for a new X)" block** into the new note, filling the fields — never invent frontmatter. After creating a note in a folder whose `_index.md` keeps a list (Features, Contexts, ADRs), **add its entry to that list** per the folder's convention.
3. **Large files.** For notes the REST API truncates (~3000+ lines), use the `Write` tool against the absolute path `{root}/{subpath}/...`. Otherwise prefer `mcp__obsidian__vault_write` / `vault_patch`.

## The non-negotiable gate

You MUST **invoke** each sub-skill below with the Skill tool, in order. Mentioning a sub-skill, summarising it, "keeping it in mind", or "offering it as a next step" is a violation — invoke it. Create one TodoWrite item per step in the Sequence and complete them in order.

**Violating the letter of this sequence is violating its spirit.** In particular, do NOT let `brainstorming`'s own terminal step ("invoke writing-plans") fire — this wrapper redefines what happens after the dialogue.

## Sequence

1. **Invoke `superpowers:brainstorming`** for context exploration, one-at-a-time clarifying questions, 2-3 approaches, and a *proposed* design. STOP there: do NOT write the spec doc yet, do NOT write to `docs/superpowers/`, and do NOT transition to writing-plans. The spec and the plan hand-off happen later, in THIS skill.

2. **Invoke `grill-with-docs`** on the proposed design — always, not "if terms look unsettled". Its domain docs live in the vault, not the repo (there is no filesystem `CONTEXT.md` — do not create one): read `{subpath}/Contexts/_index.md` first, then only the relevant glossary note(s); resolve terms inline into `{subpath}/Contexts/<context-slug>.md` (following that folder's `_index.md` template, and adding the glossary to its Contexts list); write any ADR to `{subpath}/ADRs/NNNN-<slug>.md`, numbering by `vault_list`-ing the dir for the highest `NNNN`. Revise the proposed design with whatever the grilling surfaces.

3. **Invoke the project's design skill — only if `designSkill` is set.** When `config.designSkill` names a skill, invoke it **by that name** with the Skill tool once the brainstorm AND grill questions are all answered — REQUIRED for any feature with a UI surface (a page, panel, dialog, table, card, …). Get the user's visual approval and capture whatever **design reference** that skill reports back (e.g. a route to run + a source file path + the states shown). If `config.designSkill` is absent, **skip this step and say so explicitly**; also skip (and state why) for a pure-backend feature with no rendered surface.

4. **Write the spec to the vault** at `{subpath}/Specs/<topic>-design.md`, following `{subpath}/Specs/_index.md`'s frontmatter template. If step 3 produced a design reference, the spec MUST carry a `## Design` section repeating it verbatim so the plan and the implementer can find the approved visuals without guessing:
   ```markdown
   ## Design
   - <design reference exactly as the design skill reported it — route to run, source file, states shown>
   ```
   Then run brainstorming's spec self-review (placeholders, contradictions, scope, ambiguity — fix inline) and ask the user to review the written spec before continuing.

5. **Hand off to `spec:plan`** (NOT `superpowers:writing-plans`). The plan reads this spec and carries its `## Design` section forward so the executor builds against the approved visuals.

## Red flags — STOP, you're about to skip a step

| Rationalization | Reality |
|---|---|
| "Terms look settled, I'll skip grill-with-docs" | grill is mandatory — it is where terms get verified against the vault glossary, not assumed. Invoke it. |
| "designSkill is set but a mockup feels like overkill" | If it renders anything and a design skill is configured, invoke it — prose can't be reviewed visually. |
| "No designSkill, so I'll quietly move on" | Skipping is fine, but say so explicitly — don't leave the user guessing whether visuals were considered. |
| "The deliverable is just the spec" | The deliverable is spec + (if configured) an approved design + a sharpened glossary. |
| "I'll keep grill/design in mind and offer them as next steps" | Mentioning ≠ invoking. Use the Skill tool. |
| "Brainstorming says invoke writing-plans next" | This wrapper overrides that. grill and design run first; the plan skill is `spec:plan`. |

## Requires

`superpowers:brainstorming`, `grill-with-docs`, `spec:plan`, the `obsidian` MCP, and — if `config.designSkill` is set — that named skill (project-owned, not part of this bundle). If a required skill or the configured design skill is absent, tell the user before relying on the step that needs it.
