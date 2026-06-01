---
type: note
---
# Plans

Implementation plans from `superpowers:writing-plans`. Each plan is bite-sized tasks (TDD, frequent commits, checkbox tracking) and embeds an **Execution model policy** table for `spec:execute`.

Filename format: `YYYY-MM-DD-<feature-name>.md`

## Frontmatter template (copy for a new plan)

```yaml
---
type: plan
status: draft                       # draft | active | done | superseded
date: YYYY-MM-DD
feature: "[[<feature-slug>]]"       # Feature hub; null for infra/tooling plans
implements: "[[<spec-note-name>]]"  # the spec this plan implements (exact note name)
contexts: ["[[<context-slug>]]"]    # glossaries this plan relies on; [] if none
---
```

`implements` points to the spec's exact note name — specs use a `-design` suffix so the link stays unambiguous.
