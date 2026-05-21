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
                └── SKILL.md
```

See the [plugin marketplace docs](https://code.claude.com/docs/en/plugin-marketplaces) for the full schema and hosting details.

## License

MIT
