---
name: multipaas-upgrade
description: Move an app built on MultiPaaS to a newer API version, on request, by running the multipaas CLI — show what changed between the app's pinned version and the newest (breaking, dangerous and added), then, when the user asks, pin it, pull the schema, regenerate the app's code and fix what the change broke. Use when the user asks what changed in their MultiPaaS API, whether their app is behind, or to upgrade it.
allowed-tools: Bash(multipaas *)
---

# Move an app to a newer API version

The `multipaas-build` skill sets up the project and covers calling the generated API. This skill moves an app that is already pinned. The app keeps working on its pin, so never move it unless the user asks.

This needs CLI 0.4.0 or later. If `multipaas --version` reports an older version, ask the user to run `brew update`, then, as a separate command, `brew upgrade multipaas`.

## See what changed

Run this in the app's directory, where `multipaas.json` lives.

`multipaas versions diff --json` compares the pinned version with the newest. To compare two others, pass `--from ID` and `--to ID`, with IDs from `multipaas versions list --json`. It prints:
`{"from", "to", "breaking": [{"type", "description"}], "dangerous": [...], "added": [...]}`

- **breaking:** a change that can fail an app written against `from`, for example a removed field or a changed type. Search the app's code for each type and field named.
- **dangerous:** a change that keeps the app's requests valid but can change what they do, for example a new optional input field or a changed default.
- **added:** new types and fields, such as a list field's `…ListFilter` input. They don't affect requests the app already sends. These are what the app can now use.

When **dangerous** includes an optional argument `where` or `orderBy` added to a field that returns a page, or **added** includes `totalCount` on a page type, search the app for code that fetches every page and then filters, sorts or counts in the app, and offer to move it into the query. When **dangerous** includes an optional field `is` added to a `…ReferenceFilter` input, or a field added to a `…Filter` input whose type is a `…RelatedFilter`, or **added** includes a `…RelatedFilter` type, search the app for code that fetches every page of one entity and then keeps records by what is or isn't related to them, or by a field of the record they reference, and offer to move it into one query. When **added** includes a `…ListFilter` input, search the app for code that fetches every page of an entity and then keeps records whose list field holds a value, and offer to move it into `where` with `has`, only where the app's own check is an exact, case-sensitive match on one element: `has` is, so a case-insensitive or substring check would change results. Code that filters only the page it fetched gets different pages once the filter moves into the query, so point that out rather than offering the move.

A renamed field shows up as its old name removed (**breaking**) and its new name added. It can land in all three lists: on its type, on the create and update inputs that set it, and, when the field can be cleared, on the `<Entity>Field` enum that the update input's `clear` list uses. Rename it in the app's code.

Tell the user what would break in their app, by naming the places in their code, then what to watch, then what's new. When all three lists are empty, no change graphql-js reports affects the app's existing requests, so there is nothing to change in the app.

Versions made before file fields let an entity or enum have the type name `File`, `FileUpload` or `HttpHeader`. Run `multipaas model show --json` and read each entity's and enum's `graphqlTypeName`. Enums are in that output only from CLI 0.5.0, so if `multipaas --version` reports an older version, ask the user to upgrade as at the top before you read it. When one of them has one of those type names, MultiPaaS refuses every model change that leaves it in place, so the newest version stays one without file fields. When the model has more than one of them, every change to one of them leaves another in place and is refused too, so don't offer a rename or retire: tell the user, and offer to report it with the `multipaas-feedback` skill. When it has exactly one, tell the user, and offer to rename it, or retire it if it's no longer needed and nothing uses it, with the `multipaas-model` skill, which also says what to do when that change is refused too. The change makes a new version, so diff it as above before moving the app.

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
