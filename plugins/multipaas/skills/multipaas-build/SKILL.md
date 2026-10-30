---
name: multipaas-build
description: Build an app on a MultiPaaS application's generated GraphQL API, pinned to one API version, by running the multipaas CLI — set up multipaas.json, pull the schema for codegen, and read and write test records. Use when the user wants to build, scaffold or change an app on their MultiPaaS model, call its API, create test data, set up a background job that acts as the app (app identities, client-credentials tokens) or work with app-wide data.
allowed-tools: Bash(multipaas *)
---

# Build an app on a pinned API version

The `multipaas` skill covers installing the CLI, signing in and the exit codes. The `multipaas-model` skill changes the model. The `multipaas-sign-in` skill adds end-user sign-in. This skill sets up the app's project and its calls to the generated API.

These commands need CLI 0.5.0 or later. If `multipaas --version` reports an older version, ask the user to run `HOMEBREW_GITHUB_API_TOKEN="$(gh auth token)" brew update`, then, as a separate command, `HOMEBREW_GITHUB_API_TOKEN="$(gh auth token)" brew upgrade multipaas`.

Creating an application (`apps create`) needs CLI 0.8.0 or later. Upgrade the same way.

`records --app-wide` needs CLI 0.10.0 or later. Upgrade the same way.

`envs` and `--env` need CLI 0.11.0 or later. Upgrade the same way.

## Set up the project

Run these in the app's own directory, which is where `multipaas.json` lives.

1. Find the application: `multipaas apps list --json`. If the list is `[]` or none fits, offer to create one, and on a yes run `multipaas apps create --name NAME --json`. Its `application.id` is the `APP_ID` from here on, and the new application has no app-tenant yet. If it exits 1 with `UNAUTHORIZED`, tell the user to ask a Client manager in their organization to create it or give them permission. For `CLIENT_REQUIRED`, see the `multipaas` skill. If there is still no application, stop here: the later steps need an `APP_ID`. Pick an environment to test in: `multipaas envs list --app APP_ID --json`. Each environment keeps its own app-tenants and app-wide records. Recommend a non-default environment for test data, and when only the default exists, offer to create one with `multipaas envs create --app APP_ID --name test --json`, whose `environment.id` is the `ENV_ID` from here on. It needs permission to manage the application. If it exits 1 with `UNAUTHORIZED`, tell the user to ask someone who manages the application. Find an app-tenant to test in: `multipaas tenants list --app APP_ID --env ENV_ID --json`. Ask the user which one if more than one fits. If the list is `[]`, that environment has no app-tenant yet. Offer to create one to test in, and on a yes run `multipaas tenants create --app APP_ID --env ENV_ID --name NAME --json`. It prints `{"appTenant": {"id", "name"}}`, and its `id` is the `APP_TENANT_ID` for step 2. It needs permission to manage the application. If it exits 1 with `UNAUTHORIZED`, tell the user to ask someone who manages the application.
2. `multipaas init --app APP_ID --tenant APP_TENANT_ID --json` writes `multipaas.json` and pins the newest API version. It prints:
   `{"config": {"applicationId", "testAppTenantId", "apiVersion", "oauthClientId"?}, "endpoints": {"graphql", "authorize", "token", "revoke"}}`
   Commit `multipaas.json`. Put the endpoints in the app's config. Never write a MultiPaaS host from memory.
3. `multipaas schema pull --json` writes the pinned version's schema to `multipaas.graphql` and prints `{"apiVersion", "path"}`. Point the app's GraphQL codegen at that file, and commit it with the generated code.

In this directory, `--app` defaults to the application in `multipaas.json`, and `--tenant` to its test app-tenant when `--app` is that same application. `apiVersion` is `""` when the application has no model yet. Model it first, then run `multipaas pin --json`.

## Calling the generated API

- Every request is `POST` to `endpoints.graphql`, with the headers `Authorization: Bearer <an access token>` and `X-MultiPaaS-App-Api-Version: <apiVersion from multipaas.json>`.
- There is no default version. A request without the header is refused with HTTP 400, and an unknown version with 404.
- The access token comes from end-user sign-in (see the `multipaas-sign-in` skill), or for a background job from a client-credentials request (see below). The developer's own CLI sign-in never goes into the app.
- For each entity, the schema has a query for one record by id, a paginated list, and delete and restore mutations. Create is left out when a required field without a default isn't exposed, and update is left out when none of the entity's fields are exposed. `me` is a top-level query. The schema itself is the reference: read `multipaas.graphql` rather than guessing names.
- A list takes `page: {page, pageSize}`. `page` counts from 0, and `pageSize` defaults to 20 and can be at most 100: a larger one is refused, not reduced. It returns `{items, pageInfo: {page, pageSize, hasMore}}`, with live records only, newest first. Fetch the next page while `hasMore` is true.
- A top-level list can't be filtered. To show one record's children, such as a cookbook's recipes, give the parent entity a `related` field over the child's reference field (see the `multipaas-model` skill), and query it on the parent: `cookbook(id: …) { recipes { items { … } } }`. It takes the same `page` argument. Don't fetch every record and filter in the app.
- A reference field is set through its `…Id` input. Reading it returns the record it points at, even when that record is retired, or `null` when there is none or the end user can't read it.
- A refused request carries `errors[0].extensions.code`. For `VALIDATION`, `errors[0].extensions.fieldErrors` is `[{"field", "rule", "message"}]`, where `field` is the input's GraphQL name: `…Id` for a reference, `page` or `pageSize` for a list's paging, and empty when the field can't be named.
- The CLI doesn't run queries against the app's API. To see what records the app-tenant holds, use `multipaas records list` (below).
- An enum field is a GraphQL enum named for its enum type. Send and read its values by their GraphQL names, which `multipaas.graphql` lists.
- An app pinned to a version built after app-wide records shipped (its schema has an `enum RecordScope`) also gets these. Each record type has `appWide: Boolean!`. A list takes `scope: ALL | TENANT | APP_WIDE`, and the default is `ALL`: the app-tenant's own records plus the app-wide records the caller's roles let them view. `createdBy` has `kind: PERSON | APP`. A user's `account` is `null` under an app-wide token, when the user is an app identity (under any token, `me` included), and when the user is the author of an app-wide record, so never read through `account` without a null check. A version pinned before that has none of these, and the `multipaas-upgrade` skill moves the app forward.

## Background jobs that act as the app

A job that runs without an end user, such as one that fills a shared word list, signs in as an **app identity**: the app's own identity, which holds roles like a person does.

1. Create one: `multipaas app-identity create --name "JOB NAME" --json`. It prints `{"appIdentity": {"id", …}, "secret"}`: `appIdentity.id` is the `CLIENT_ID` and `secret` is the `CLIENT_SECRET`. The secret is shown once. Store it as a secret in the job's environment, never in the repo.
2. Give it roles on its own page in the web app, which has a Roles section for the whole application and for each app-tenant. An app-wide token (step 3) needs a role held across the application. An app-tenant token reads app-wide records with a role allowing `record.app_wide.view` at that app-tenant. Writing app-wide records takes an app-wide token and a role held across the application with `record.app_wide.create` or `record.app_wide.edit` on the entity; given at an app-tenant, those permissions grant nothing. Reading or writing one app-tenant's data needs a role at that app-tenant or across the application.
3. Get a token: `POST` to `endpoints.token` from `multipaas init --json` with `grant_type=client_credentials`, identifying the client either by the form fields `client_id` and `client_secret` or by HTTP Basic carrying the id and secret together, not both, plus `app_tenant=APP_TENANT_ID` for one app-tenant's data, or no `app_tenant` for the application's app-wide data. The answer has `access_token` and `expires_in` and no refresh token, so ask again when it expires.

   ```
   printf 'user = "%s:%s"\n' "$CLIENT_ID" "$CLIENT_SECRET" | curl -s -K - -X POST "$TOKEN_ENDPOINT" -d grant_type=client_credentials -d app_tenant="$APP_TENANT_ID"
   ```

   That asks for one app-tenant's token. For an app-wide token, drop `-d app_tenant="$APP_TENANT_ID"`.

   In the job's own code, send the secret from its HTTP client (Basic auth or the request body), never through a shell command line.

   `invalid_target` means the identity can't act in the scope asked for. Without `app_tenant`: it holds no role across the application. With `app_tenant`: it holds no role at that app-tenant or across the application, or the id isn't an active app-tenant of this application.
4. Call the generated API as in the section above, with that token. The scope follows the token. A tenantless token reads and writes only app-wide records, and it needs an API version built after app-wide records shipped: an older one answers `403 insufficient_scope`. An app-tenant token writes that app-tenant's records, and reads them plus the app-wide records its roles let it view.
5. Rotate the secret: `multipaas app-identity secret add CLIENT_ID --json`, deploy the new one, then revoke the old one: find its `id` in `multipaas app-identity secret list CLIENT_ID --json` (under `secrets`, told apart by `lastFour` or `createdAt`) and run `multipaas app-identity secret revoke SECRET_ID --json` with it.

## Moving to a newer API version

A model change mints a new API version. The app stays on its pin until it moves, so a change never breaks it. To see what changed or to move the app, use the `multipaas-upgrade` skill.

## Test data

The `records` commands read and write the test app-tenant's records, or with `--app-wide` the application's app-wide records, as the developer, with their own permissions in the web app. They go through MultiPaaS's own API, not the app's generated one, so the data is keyed by **field id** (from `multipaas model show --json`), not by field name.

| Command | Does |
|---|---|
| `multipaas records list --entity ENTITY_ID --json` | `[{"id", "appWide", "data", "createdAt", "updatedAt"}]`, newest first |
| `multipaas records get --entity ENTITY_ID --id RECORD_ID --json` | `{"record": {"id", "appWide", "data", "createdAt", "updatedAt"}}`; a record that doesn't exist is a refusal |
| `multipaas records list --entity ENTITY_ID --parent-field FIELD_ID --parent-record RECORD_ID --json` | The child records of that parent, newest first: the records whose parent reference field (`isParent` in `model show`) holds `--parent-record`. Give both flags or neither |
| `multipaas records create --entity ENTITY_ID --data '{"FIELD_ID": value}' --json` | `{"record": {...}}` |
| `multipaas records update --entity ENTITY_ID --id RECORD_ID [--data '{...}'] [--clear FIELD_ID,FIELD_ID] --json` | Sets the fields in `--data`, clears those in `--clear`, and keeps the rest |
| `multipaas records retire --entity ENTITY_ID --id RECORD_ID --json` | Retires the record |
| `multipaas records list --app-wide --entity ENTITY_ID --json` | The application's app-wide records. Add `--app-wide` to any command above, and `create`, `get`, `update` and `retire` act on app-wide records too. It replaces the default test app-tenant, and the CLI refuses it together with `--tenant`; each record carries `"appWide"` |

Without `--app-wide`, `records list` returns only the test app-tenant's own records. An app-wide record is visible to every app-tenant in its environment whose roles allow `record.app_wide.view`.

App-wide records belong to the environment the command runs in. `records --app-wide` acts in the application's default environment unless it is given `--env ENV_ID`. `records` takes `--env` only with `--app-wide`; the CLI refuses it together with `--tenant`, because a tenant already belongs to one environment. The `--env` must be one of `--app`'s environments. An app-wide token (a job's) always works in the default environment.

## When MultiPaaS refuses

A refusal exits 1 with `{"error": {"code", "message"}}`. Tell the user the message as it is.

| Code | Meaning |
|---|---|
| `NOT_INITIALIZED` | No `multipaas.json` here. Run `init` |
| `NOT_PINNED` | `multipaas.json` pins no version of this application. Run `pin` |
| `NO_API_VERSION` | The application has no model yet. Model it first |
| `NOT_FOUND` | One of the ids the refused command was given doesn't exist or isn't reachable by the user, whatever the message says (for example "no application X that you can reach", "appTenant not found"). Check each id against the list for its kind, and fix it before retrying. Application: `multipaas apps list`. API version: `multipaas versions list`. App-tenant (`--tenant`, or `testAppTenantId` in `multipaas.json`): `multipaas tenants list --app APP_ID`; if it isn't there, tell the user to ask for access. Only if the user names an app-tenant they want instead, correct `multipaas.json` to that one by running `multipaas init --app APP_ID --tenant APP_TENANT_ID --api-version API_VERSION --json` with their chosen id, taking `API_VERSION` from the `apiVersion` already in `multipaas.json` so the pin stays where it is, and leaving `--api-version` out only when that value is empty. If `init` refuses that version, tell the user, and never retry without the flag. Never pick an app-tenant yourself to get past it. Record: first check the command's `--entity` id against `multipaas model show --json`, then its record id with `multipaas records list --entity ENTITY_ID`, adding `--app-wide` if the refused command used it, because without it the list shows only app-tenant records. If the entity isn't in `model show`, follow the `ENTITY_NOT_FOUND` row below, and an empty list for it doesn't mean the record is missing. App identity (`app-identity secret add` or `list`): `multipaas app-identity list`. Secret (`app-identity secret revoke`): `multipaas app-identity secret list CLIENT_ID`, and never treat an old secret as revoked until `secret revoke` succeeds. Client (`apps create --client`): use one of the clients the `CLIENT_REQUIRED` refusal names; with no client at all ("no client you can create applications in"), tell the user to ask a Client manager in their organization to create the application or give them permission. With `--app-wide`, "no application-wide access" means the user holds no role across the whole application: ask them to get one in the web app, or use `--tenant` instead. Never retry by creating records elsewhere to get past it |
| `ENTITY_NOT_FOUND` | The `--entity` id on a `records` command isn't an entity of this application. Take the id of the entity the user meant from `multipaas model show --json` and run the command again with only that id. If `model show` no longer lists that entity, tell the user it was retired, and never write to a different entity |
| `UNAUTHORIZED` | The user lacks the permission in the web app, for example `record.create`, or the app-tenant isn't one they can reach |
| `VALIDATION` | A record's data breaks the model. The message names the fields |
| `CLI_ERROR` | A bad flag, for example `--data` that isn't JSON, or `--app` missing outside a directory with `multipaas.json`. Fix it and run the command again |
