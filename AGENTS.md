# Plugin releases

Every change to a plugin's files (skills, commands, hooks, manifests, and so on)
must bump that plugin's version in both `.codex-plugin/plugin.json` and
`.claude-plugin/plugin.json` in the same commit or pull request. At minimum,
make a patch bump (for example, `1.4.0` to `1.4.1`).

Keep the corresponding version in `.claude-plugin/marketplace.json` in sync.
The Codex marketplace does not duplicate plugin versions; it reads the version
from the plugin manifest.
