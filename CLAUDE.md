# Plugin releases

Every change to a plugin's files (skills, commands, hooks, etc.) must bump that
plugin's `version` in both `.claude-plugin/plugin.json` and
`.codex-plugin/plugin.json`, in the same commit/PR as the change. Keep the
corresponding version in `.claude-plugin/marketplace.json` in sync. At minimum
make a patch bump (e.g. `1.4.0` -> `1.4.1`).

Why: `claude plugin update <name>` compares the `version` string, not git
content or commit SHA. If the version isn't bumped, installed users' `plugin
update` reports "already at the latest version" even though the underlying
files changed on the marketplace branch — the fix never reaches them.

Codex reads the plugin version from `.codex-plugin/plugin.json`; its marketplace
entry does not duplicate the version.
