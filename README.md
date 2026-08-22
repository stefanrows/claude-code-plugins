# stefanrows-plugins

A [Claude Code](https://docs.claude.com/en/docs/claude-code) plugin marketplace by [Stefan Rows](https://github.com/stefanrows).

## Install the marketplace

In Claude Code, run:

```
/plugin marketplace add stefanrows/claude-code-plugins
```

Then install any plugin from the list:

```
/plugin install merge-to-main-plugin@stefanrows-plugins
```

To refresh after I publish updates:

```
/plugin marketplace update stefanrows-plugins
```

## Plugins

### merge-to-main-plugin

A repeatable, safe workflow for landing changes on `main`: pre-merge lint/test/build checks, Conventional Commits, doc updates staged alongside code, explicit user confirmation before merge, and post-deploy monitoring.

By default it always asks for confirmation before merging.

> [!WARNING]
> **Pre-authorized merges (v1.2.0+):** if your request contains an explicit waiver — **"merge without asking"**, **"no confirmation"**, or **"ship now"** — the skill skips the confirmation gate and merges to `main` immediately after checks pass. Only the *ask* is skipped: lint/test/build, project-specific rules in your `CLAUDE.md`, and pre-merge blockers still run, and any failure, caveated check, or unexpected file in the diff cancels the waiver and asks anyway. If you don't want this behavior, just never use those phrases — a plain "merge to main" or "ship this" always gets the confirmation prompt.

**Unity support (v1.3.0+):** detected automatically via `ProjectSettings/ProjectVersion.txt`. The merge gate is asset integrity (missing `.meta` files, conflict markers in scene/prefab YAML, committed `Library/`/generated paths, large binaries outside Git LFS) plus a clean compile and green EditMode tests via Unity batchmode. When the local Editor can't run (e.g. from WSL2, or the project is locked open elsewhere), the skill skips the local run, says so explicitly, and treats CI (GameCI / Unity Build Automation) as the gate instead. See `plugins/merge-to-main-plugin/skills/merge-to-main/references/unity.md` for the full details.

Invoke the skill from Claude Code:

```
/merge-to-main-plugin:merge-to-main
```

Or just ask Claude to "merge to main", "ship", "land", or "release" — the skill is wired to those triggers.

## Repository layout

```
.
├── .claude-plugin/
│   └── marketplace.json      # marketplace catalog
└── plugins/
    └── merge-to-main-plugin/
        ├── .claude-plugin/
        │   └── plugin.json   # plugin manifest
        └── skills/
            └── merge-to-main/
                ├── SKILL.md
                └── references/
                    └── unity.md   # Unity-specific detail, read on detection
```

See the [plugin marketplace docs](https://code.claude.com/docs/en/plugin-marketplaces) for the full schema and hosting details.

## License

MIT
