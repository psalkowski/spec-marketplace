# spec-marketplace — archived

> [!WARNING]
> **This repository is archived and no longer maintained.**
>
> The **`spec`** plugin now lives in [**psalkowski/claude-marketplace**](https://github.com/psalkowski/claude-marketplace).

## Migration

Install `spec` from its new home:

```sh
/plugin marketplace add psalkowski/claude-marketplace
/plugin install spec
```

If you previously added this marketplace, remove it and add the new one:

```sh
/plugin marketplace remove spec-marketplace
/plugin marketplace add psalkowski/claude-marketplace
```

The plugin itself is unchanged — the same `spec:plan` / `spec:execute` skills, agents,
`/spec:setup` command, and vault templates moved over wholesale.

---

A personal [Claude Code](https://claude.com/claude-code) marketplace that hosted the **`spec`** plugin — a portable, vault-backed spec workflow you can drop into any project. See the [new repository](https://github.com/psalkowski/claude-marketplace) for the maintained version.
