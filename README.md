# MultiPaaS for Claude Code

The `multipaas` plugin teaches Claude Code to drive the
[multipaas CLI](https://github.com/MultiPaaS-Apps/cli). Install the CLI first.

In Claude Code:

```text
/plugin marketplace add MultiPaaS-Apps/claude-plugin
/plugin install multipaas@multipaas
```

While this repository is private, Claude Code clones it with your own git credentials. Install the
[GitHub CLI](https://cli.github.com) (`gh`), run `gh auth login`, then `gh auth setup-git`.

## Releases

Releases are cut by release-please. Merging a `feat:` or `fix:` PR to `main` opens or updates a
release PR with the changelog, and merging that PR sets the version in `plugin.json` and tags `vX.Y.Z`.
A feature PR leaves the version alone.
