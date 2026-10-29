---
name: multipaas-feedback
description: Report a MultiPaaS bug or missing capability to MultiPaaS for the user, by running the multipaas CLI — draft the report with its context, ask once unless the user has said to always send, and follow up on reports already sent. Use when MultiPaaS itself is at fault or can't do what the user needs, when the user asks to report a MultiPaaS problem or request a MultiPaaS feature, or asks what became of their MultiPaaS feedback.
allowed-tools: Bash(multipaas *)
---

# Report a MultiPaaS problem

The `multipaas` skill covers installing the CLI, signing in and the exit codes. This skill sends MultiPaaS a report when MultiPaaS itself is in the user's way, so the problem reaches the people who can fix it instead of being worked around in silence.

This needs CLI 0.9.0 or later. If `multipaas --version` reports an older version, ask the user to run `HOMEBREW_GITHUB_API_TOKEN="$(gh auth token)" brew update`, then, as a separate command, `HOMEBREW_GITHUB_API_TOKEN="$(gh auth token)" brew upgrade multipaas`.

## When to report

Report when you have evidence that MultiPaaS is at fault, or that it can't do what the user needs:

- **bug**, something doesn't work as expected: a response or error that contradicts these skills or the application's model, a server error, or a `CLI_ERROR` that keeps happening once you have ruled out the flags and the network.
- **feature**, something the user would like to see: the work needs a command, field or API that MultiPaaS doesn't have, and stalls or needs a workaround without it.

Never report a mistake in the user's own code, a wrong flag you passed, or a refusal MultiPaaS makes on purpose, such as `UNAUTHORIZED` for a permission the user lacks. When you can't tell whether a problem is MultiPaaS's, tell the user what you found instead of reporting it.

The user can also ask you to report something or request a feature. Then report it as they describe it.

A report doesn't fix anything. After sending one, or after the user declines, carry on with the user's work, and tell them about any workaround.

## What goes in a report

- `--kind`: `bug` or `feature`.
- `--summary`: one line naming the problem, at most 200 characters.
- The details: what the user was doing, the command or operation, the exact error code and message, what was expected instead, and, for a bug, the steps that reproduce it. At most 10,000 characters.

Never include secrets, tokens or credentials, and never include the contents of the user's records. Name entities and fields, and describe data by its shape, not its values.

The CLI adds who is sending and its own version. In a directory with `multipaas.json`, it also adds that application and the API version it is pinned to. Choose what the report is about:

- **The project's application:** run it in the directory with `multipaas.json`, with no flag.
- **Another application:** `--app APP_ID`.
- **MultiPaaS in general,** such as signing in, the CLI itself, or managing the client's users: `--general`.

## Ask once, unless the user said always send

1. Run `multipaas feedback settings --json`. It prints `{"autoSend": BOOL}`.
2. If `autoSend` is `false`, show the user the drafted report: its kind, summary, details, and what it is about. Ask once whether to send it, then go with their answer. If they decline, don't send it, and don't ask about that problem again in this session. Never hold up the user's work waiting for an answer.
3. If the user says to always send, or that you needn't ask, run `multipaas feedback settings --auto-send on --json`, then send. If they later want to be asked again, run it with `--auto-send off`.
4. If `autoSend` is `true`, send without asking. Then tell the user what you sent and its `id`.

What the user says in the session wins for that report. If they tell you not to send something, don't send it, whatever the setting says.

## Send

Always pass `--via plugin` and `--json`. Put the summary in single quotes, writing each `'` in it as `'\''`. Pass the details on standard input through the quoted heredoc below, so nothing in them is expanded. No line of the details may be exactly the heredoc's end marker:

```bash
multipaas feedback send --kind bug --summary 'SUMMARY' --via plugin --json --details-file - <<'MULTIPAAS_FEEDBACK_END'
DETAILS
MULTIPAAS_FEEDBACK_END
```

Add `--app APP_ID` or `--general` when the report isn't about the project's application. On exit 0 it prints `{"feedback": {"id", "kind", "status", "summary", "details", "source", ...}}` with `status` `received` and `source` `plugin`. Tell the user the `id`.

## Follow up

| Command | stdout on exit 0 |
|---|---|
| `multipaas feedback list --json` | `[{"id", "kind", "status", "summary", "details", "createdAt", "updatedAt", ...}]`, the user's own reports, newest first. `summary` is `null` when the report has none: its title is then the first line of `details` |
| `multipaas feedback show --id ID --json` | `{"feedback": {..., "history": [{"status", "note", "changedAt"}]}}`, with the history oldest first. A `note` is what MultiPaaS told the user about that change |

A report's `status` is `received`, `under_review`, `planned`, `fixed` or `declined`.

## When MultiPaaS refuses

A refusal exits 1 with `{"error": {"code", "message"}}`. Tell the user the message as it is.

| Code | Meaning |
|---|---|
| `VALIDATION` | The summary or details are too long, or the details are empty. `error.fieldErrors` names each field, with a `field`, `rule` and `message`. Fix the draft and send it again |
| `RATE_LIMITED` | The user has already sent 20 reports in the past hour, so no more are accepted for now. Don't retry now. Tell the user, and offer to send it later |
| `NOT_FOUND` | From `send`: the user can't reach the application the report is about, the one in `multipaas.json` or `--app`'s. Send it again with `--general`, which files the same report without the application, so it isn't lost. Use `--app` for a different application only when the user names it. From `show`: no report with that id belongs to the user |
| `CLI_ERROR` | A bad flag, such as a `--kind` other than `bug` or `feature`, or `--app` together with `--general`: fix it and run the command again. Otherwise the machine is offline or the server failed: tell the user, keep the draft, and send it once they say the problem is resolved |
