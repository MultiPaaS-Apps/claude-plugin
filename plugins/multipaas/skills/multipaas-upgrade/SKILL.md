---
name: multipaas-upgrade
description: Move an app built on MultiPaaS to a newer API version, on request, by running the multipaas CLI — show what changed between the app's pinned version and the newest (breaking, dangerous and added), then, when the user asks, pin it, pull the schema, regenerate the app's code and fix what the change broke. Use when the user asks what changed in their MultiPaaS API, whether their app is behind, or to upgrade it.
allowed-tools: Bash(multipaas *)
---

# Move an app to a newer API version

The `multipaas-build` skill sets up the project and covers calling the generated API. This skill moves an app that is already pinned. The app keeps working on its pin, so never move it unless the user asks.

This needs CLI 0.4.0 or later. If `multipaas --version` reports an older version, ask the user to run `HOMEBREW_GITHUB_API_TOKEN="$(gh auth token)" brew update`, then, as a separate command, `HOMEBREW_GITHUB_API_TOKEN="$(gh auth token)" brew upgrade multipaas`.

## See what changed

Run this in the app's directory, where `multipaas.json` lives.

`multipaas versions diff --json` compares the pinned version with the newest. To compare two others, pass `--from ID` and `--to ID`, with IDs from `multipaas versions list --json`. It prints:
`{"from", "to", "breaking": [{"type", "description"}], "dangerous": [...], "added": [...]}`

- **breaking:** a change that can fail an app written against `from`, for example a removed field or a changed type. Search the app's code for each type and field named.
- **dangerous:** a change that keeps the app's requests valid but can change what they do, for example a new optional input field or a changed default.
- **added:** new types and fields. They don't affect requests the app already sends. These are what the app can now use.

Tell the user what would break in their app, by naming the places in their code, then what to watch, then what's new. When all three lists are empty, no change graphql-js reports affects the app's existing requests, so there is nothing to change in the app.

## Upgrade, when the user asks

1. `multipaas pin --api-version TO --json`, with `TO` the `to` from the diff, so the app moves to the version whose changes were reviewed. It prints the new `multipaas.json`.
2. `multipaas schema pull --json` writes the new version's schema to the app's schema file.
3. Regenerate the app's code with its own codegen. Then fix each breaking change the diff named, and whatever the compiler or the type checker reports.
4. Check that the app sends the new `apiVersion` from `multipaas.json` in `X-MultiPaaS-App-Api-Version`.
5. Run the app's build and tests. Tell the user what changed and what you fixed.

If the user wants to stay on the pin, change nothing.

## When MultiPaaS refuses

A refusal exits 1 with `{"error": {"code", "message"}}`. Tell the user the message as it is.

| Code | Meaning |
|---|---|
| `NOT_PINNED` | `multipaas.json` pins no version of this application. Set the project up with the `multipaas-build` skill, or pass `--from` |
| `NO_API_VERSION` | The application has no model yet |
| `NOT_FOUND` | No such API version of this application |
| `UNAUTHORIZED` | The user has no permission in this application |
| `CLI_ERROR` | A bad flag, or `--app` missing outside a directory with `multipaas.json`. Fix it and run the command again |
