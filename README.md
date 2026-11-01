# MultiPaaS for Claude Code

The `multipaas` plugin teaches Claude Code to drive the
[multipaas CLI](https://github.com/MultiPaaS-Apps/cli). Install the CLI first.

In Claude Code:

```text
/plugin marketplace add MultiPaaS-Apps/claude-plugin
/plugin install multipaas@multipaas
```

## Releases

Releases are cut by release-please. Merging a `feat:` or `fix:` PR to `main` opens or updates a
release PR with the changelog, and merging that PR sets the version in `plugin.json` and tags `vX.Y.Z`.
A feature PR leaves the version alone.
