# Plugin releases

Every change to a plugin's files (skills, commands, hooks, etc.) must bump that
plugin's `version` in its `.claude-plugin/plugin.json`, in the same commit/PR
as the change. At minimum a patch bump (e.g. `1.3.0` -> `1.3.1`).

Why: `claude plugin update <name>` compares the `version` string, not git
content or commit SHA. If the version isn't bumped, installed users' `plugin
update` reports "already at the latest version" even though the underlying
files changed on the marketplace branch — the fix never reaches them.
