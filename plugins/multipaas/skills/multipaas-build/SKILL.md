---
name: multipaas-build
description: Build an app on a MultiPaaS application's generated GraphQL API, pinned to one API version, by running the multipaas CLI — set up multipaas.json, pull the schema for codegen, and read and write test records. Use when the user wants to build, scaffold or change an app on their MultiPaaS model, call its API, or create test data.
allowed-tools: Bash(multipaas *)
---

# Build an app on a pinned API version

The `multipaas` skill covers installing the CLI, signing in and the exit codes. The `multipaas-model` skill changes the model. The `multipaas-sign-in` skill adds end-user sign-in. This skill sets up the app's project and its calls to the generated API.

These commands need CLI 0.5.0 or later. If `multipaas --version` reports an older version, ask the user to run `HOMEBREW_GITHUB_API_TOKEN="$(gh auth token)" brew update`, then, as a separate command, `HOMEBREW_GITHUB_API_TOKEN="$(gh auth token)" brew upgrade multipaas`.

## Set up the project

Run these in the app's own directory, which is where `multipaas.json` lives.

1. Find the application: `multipaas apps list --json`. Find an app-tenant to test in: `multipaas tenants list --app APP_ID --json`. Ask the user which one if more than one fits. If the list is `[]`, the application has no app-tenant yet. Offer to create one to test in, and on a yes run `multipaas tenants create --app APP_ID --name NAME --json`. It prints `{"appTenant": {"id", "name"}}`, and its `id` is the `APP_TENANT_ID` for step 2. It needs permission to manage the application. If it exits 1 with `UNAUTHORIZED`, tell the user to ask someone who manages the application.
2. `multipaas init --app APP_ID --tenant APP_TENANT_ID --json` writes `multipaas.json` and pins the newest API version. It prints:
   `{"config": {"applicationId", "testAppTenantId", "apiVersion", "oauthClientId"?}, "endpoints": {"graphql", "authorize", "token", "revoke"}}`
   Commit `multipaas.json`. Put the endpoints in the app's config. Never write a MultiPaaS host from memory.
3. `multipaas schema pull --json` writes the pinned version's schema to `multipaas.graphql` and prints `{"apiVersion", "path"}`. Point the app's GraphQL codegen at that file, and commit it with the generated code.

In this directory, `--app` defaults to the application in `multipaas.json`, and `--tenant` to its test app-tenant when `--app` is that same application. `apiVersion` is `""` when the application has no model yet. Model it first, then run `multipaas pin --json`.

## Calling the generated API

- Every request is `POST` to `endpoints.graphql`, with the headers `Authorization: Bearer <the end user's access token>` and `X-MultiPaaS-App-Api-Version: <apiVersion from multipaas.json>`.
- There is no default version. A request without the header is refused with HTTP 400, and an unknown version with 404.
- The access token comes from end-user sign-in: see the `multipaas-sign-in` skill. The developer's own CLI sign-in never goes into the app.
- For each entity, the schema has a query for one record by id, a paginated list, and delete and restore mutations. Create is left out when a required field without a default isn't exposed, and update is left out when none of the entity's fields are exposed. `me` is a top-level query. The schema itself is the reference: read `multipaas.graphql` rather than guessing names.
- A list takes `page: {page, pageSize}`. `page` counts from 0, and `pageSize` defaults to 20 and can be at most 100: a larger one is refused, not reduced. It returns `{items, pageInfo: {page, pageSize, hasMore}}`, with live records only, newest first. Fetch the next page while `hasMore` is true.
- A top-level list can't be filtered. To show one record's children, such as a cookbook's recipes, give the parent entity a `related` field over the child's reference field (see the `multipaas-model` skill), and query it on the parent: `cookbook(id: …) { recipes { items { … } } }`. It takes the same `page` argument. Don't fetch every record and filter in the app.
- A reference field is set through its `…Id` input. Reading it returns the record it points at, even when that record is retired, or `null` when there is none or the end user can't read it.
- A refused request carries `errors[0].extensions.code`. For `VALIDATION`, `errors[0].extensions.fieldErrors` is `[{"field", "rule", "message"}]`, where `field` is the input's GraphQL name: `…Id` for a reference, `page` or `pageSize` for a list's paging, and empty when the field can't be named.
- The CLI doesn't run queries against the app's API. To see what records the app-tenant holds, use `multipaas records list` (below).
- An enum field is a GraphQL enum named for its enum type. Send and read its values by their GraphQL names, which `multipaas.graphql` lists.

## Moving to a newer API version

A model change mints a new API version. The app stays on its pin until it moves, so a change never breaks it. To see what changed or to move the app, use the `multipaas-upgrade` skill.

## Test data

The `records` commands read and write the test app-tenant's records as the developer, with their own permissions in the web app. They go through MultiPaaS's own API, not the app's generated one, so the data is keyed by **field id** (from `multipaas model show --json`), not by field name.

| Command | Does |
|---|---|
| `multipaas records list --entity ENTITY_ID --json` | `[{"id", "data", "createdAt", "updatedAt"}]`, newest first |
| `multipaas records create --entity ENTITY_ID --data '{"FIELD_ID": value}' --json` | `{"record": {...}}` |
| `multipaas records update --entity ENTITY_ID --id RECORD_ID [--data '{...}'] [--clear FIELD_ID,FIELD_ID] --json` | Sets the fields in `--data`, clears those in `--clear`, and keeps the rest |
| `multipaas records retire --entity ENTITY_ID --id RECORD_ID --json` | Retires the record |

## When MultiPaaS refuses

A refusal exits 1 with `{"error": {"code", "message"}}`. Tell the user the message as it is.

| Code | Meaning |
|---|---|
| `NOT_INITIALIZED` | No `multipaas.json` here. Run `init` |
| `NOT_PINNED` | `multipaas.json` pins no version of this application. Run `pin` |
| `NO_API_VERSION` | The application has no model yet. Model it first |
| `NOT_FOUND` | No such application, API version or record, or an app-tenant the user's access doesn't include |
| `UNAUTHORIZED` | The user lacks the permission in the web app, for example `record.create`, or the app-tenant isn't one they can reach |
| `VALIDATION` | A record's data breaks the model. The message names the fields |
| `CLI_ERROR` | A bad flag, for example `--data` that isn't JSON, or `--app` missing outside a directory with `multipaas.json`. Fix it and run the command again |
