---
name: multipaas
description: Sign in to MultiPaaS and look up the signed-in developer, their MultiPaaS applications, and an application's app-tenants, by running the multipaas CLI. Use when the user mentions MultiPaaS, their MultiPaaS apps or tenants, or signing in to MultiPaaS.
allowed-tools: Bash(multipaas *)
---

# MultiPaaS CLI

`multipaas` is the MultiPaaS command line. It acts as the signed-in developer, with exactly the permissions they have in the MultiPaaS web app.

Always pass `--json`. With it, every command prints exactly one JSON document on stdout. Always read the exit code.

## Install

Run `multipaas --version`. If the command is missing, ask the user to install it. While MultiPaaS is in private preview, this needs their GitHub access. Tapping clones a private git repository, so git needs their credentials first: `gh auth login`, then `gh auth setup-git` (or tap over SSH, as [the tap's README](https://github.com/MultiPaaS-Apps/homebrew-tap) describes). Then:

```bash
brew tap multipaas-apps/tap
HOMEBREW_GITHUB_API_TOKEN="$(gh auth token)" brew install multipaas-apps/tap/multipaas
```

## Which MultiPaaS

The CLI talks to `https://api.multipaas.com` unless the environment variable `MULTIPAAS_API_URL` is set. The user sets it themselves, to the URL they were given. Never make one up, and don't set it unless the user tells you the value. A sign-in is kept per URL, so after the user changes it they sign in again.

## Commands

| Command | stdout on exit 0 |
|---|---|
| `multipaas login --json` | `{"user": {"id": "ID", "email": "EMAIL", "name": "NAME"}}`; `name` is `null` when the user has none |
| `multipaas whoami --json` | The same, without signing in again |
| `multipaas apps list --json` | `[{"clientId": "ID", "clientName": "NAME", "id": "ID", "name": "NAME"}]` |
| `multipaas tenants list --app APP_ID --json` | `[{"id": "ID", "name": "NAME"}]`, the app-tenants of that application |
| `multipaas logout --json` | `{"signedOut": true}`, or `false` when the user was not signed in. On exit 0 the user is signed out. If the server could not confirm revoking the tokens, it exits 1 with code `REVOKE_FAILED`, and the user is still signed out on this machine |

`APP_ID` is an `id` from `apps list`.
To read or change an application's model, use the `multipaas-model` skill.

To build an app on an application's API, use the `multipaas-build` skill; for its end users' sign-in, the `multipaas-sign-in` skill.
To move an app to a newer API version, use the `multipaas-upgrade` skill.

## Signing in

`login` opens the user's browser and waits up to 5 minutes for them to approve "Allow MultiPaaS CLI?".

1. Run it in the background, because it blocks until the user approves.
2. Tell the user to approve in their browser. Show them the URL that `login` prints on stderr, in case no browser opened.
3. When it exits 0, carry on with what the user asked for.
4. If `login` itself exits 2 (`SIGN_IN_REFUSED` or `SIGN_IN_TIMEOUT`), the user declined or did not approve in time. Tell the user, and do not run `login` again unless they ask you to.

## Exit codes

| Exit | Meaning | What to do |
|---|---|---|
| 0 | Done | Use stdout |
| 1 | Refused or failed. stdout is `{"error": {"code": "CODE", "message": "MESSAGE"}}` | Tell the user the message and code. `REVOKE_FAILED` (from `logout`) means the user is signed out on this machine but the server did not confirm revoking the tokens: tell the user, and don't retry. `INVALID_API_URL` means `MULTIPAAS_API_URL` is set to a value the CLI refuses (it must be an `https` origin on an `api.` host): tell the user, and don't try other values. Any other code except `CLI_ERROR` is MultiPaaS refusing on purpose: don't retry the same command. `CLI_ERROR` is a failure on this machine or the network (offline, a server error, a bad flag): fix the flag if that is the cause, otherwise tell the user and retry once they say the problem is resolved |
| 2 | Not signed in (`NOT_SIGNED_IN`), or `login` was declined or timed out (`SIGN_IN_REFUSED`, `SIGN_IN_TIMEOUT`) | For `NOT_SIGNED_IN`, sign in (above), then run the original command again. For `SIGN_IN_REFUSED` or `SIGN_IN_TIMEOUT`, tell the user and stop: don't run `login` again on your own |
| 3 | This CLI is too old for MultiPaaS | Ask the user to run `HOMEBREW_GITHUB_API_TOKEN="$(gh auth token)" brew update`, then, as a separate command, `HOMEBREW_GITHUB_API_TOKEN="$(gh auth token)" brew upgrade multipaas`, then retry |
