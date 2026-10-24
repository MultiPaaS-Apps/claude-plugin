---
name: multipaas-model
description: Change a MultiPaaS application's model — declare, change and retire entities, enums, fields (including enum and reference fields) and unique constraints — by running the multipaas CLI. Use when the user wants to model data, add or change an entity, enum or field, or asks what their MultiPaaS model looks like.
allowed-tools: Bash(multipaas *)
---

# Change the model

The `multipaas` skill covers installing the CLI, signing in and the exit codes. Follow it for those. This skill is for the model commands.

These commands need CLI 0.5.0 or later. If `multipaas --version` reports an older version, ask the user to run `HOMEBREW_GITHUB_API_TOKEN="$(gh auth token)" brew update`, then, as a separate command, `HOMEBREW_GITHUB_API_TOKEN="$(gh auth token)" brew upgrade multipaas`.

Every change acts on the live model of one application, as the signed-in developer, with exactly their permissions in the MultiPaaS web app. A change takes effect immediately and mints a new API version. There is no publish step.

## Before changing anything

1. Find the application: `multipaas apps list --json`. Ask the user which one if more than one fits. If none fits, offer to create one, and on a yes run `multipaas apps create --name NAME --json` and use its `application.id`. It needs CLI 0.8.0 or later: check `multipaas --version` first, and if it is older, ask the user to upgrade as above before running it. If it exits 1 with `UNAUTHORIZED`, the user can't create applications in that client: tell them to ask a Client manager in their organization to create it or give them permission. For `CLIENT_REQUIRED`, see the `multipaas` skill. If there is still no application, stop here: the later steps need an `APP_ID`.
2. Read its model: `multipaas model show --app APP_ID --json`. Use the ids it prints. Never guess an id.
3. Say what you are about to change, in a sentence, before running a command that retires something.

If `multipaas model show` exits 1 with `CLI_ERROR` and an `unknown command` message, the CLI is older than 0.2.0. Ask the user to run `HOMEBREW_GITHUB_API_TOKEN="$(gh auth token)" brew update`, then, as a separate command, `HOMEBREW_GITHUB_API_TOKEN="$(gh auth token)" brew upgrade multipaas`.

## Commands

All take `--app APP_ID --json`. On exit 0, a change prints `{"entity" | "enumType" | "field" | "constraint": {...}, "apiVersion": "ID"}`.

| Command | Does |
|---|---|
| `multipaas model show` | `{"apiVersion": "ID", "entities": [{"id", "name", "graphqlTypeName", "graphqlPluralName", "managed", "fields": [...], "constraints": [...]}], "enums": [{"id", "name", "graphqlTypeName", "values": [{"value", "graphqlName"}]}]}`. `apiVersion` is `""` before the first change |
| `multipaas entity create --name NAME [--type-name T] [--plural-name P]` | Declare an entity |
| `multipaas entity update --id ENTITY_ID [--name] [--type-name] [--plural-name] [--managed[=false]]` | Rename an entity or change whether it is managed |
| `multipaas entity retire --id ENTITY_ID` | Retire an entity |
| `multipaas enum create --name NAME --values a,b=API_NAME [--type-name T]` | Declare an enum. Each value is `VALUE` or `VALUE=API_NAME`, split on the last `=`, so a value that contains `=` needs a trailing `=API_NAME`; the API name defaults to SCREAMING_SNAKE_CASE of the value |
| `multipaas enum update --id ENUM_ID [--name] [--type-name] [--values a,b=API_NAME]` | Change an enum. `--values` takes the same `VALUE` or `VALUE=API_NAME` form and replaces the whole list, so a value not given is removed (refused while a record or a field default holds it), and an API name not given goes back to its default |
| `multipaas enum retire --id ENUM_ID` | Retire an enum |
| `multipaas field add --entity ENTITY_ID --type TYPE [--name] [--api-name] [--required] [--enum-type ENUM_ID] [--references ENTITY_ID] [--related FIELD_ID] [--parent] [--lifecycle]` | Add a field |
| `multipaas field update --id FIELD_ID [--type] [--name] [--api-name] [--required[=false]] [--enum-type ENUM_ID] [--references] [--related] [--parent[=false]]` | Change a field. Only the flags given change |
| `multipaas field retire --id FIELD_ID` | Retire a field |
| `multipaas constraint add --entity ENTITY_ID --fields FIELD_ID,FIELD_ID` | Make those fields unique together |
| `multipaas constraint retire --id CONSTRAINT_ID` | Retire a constraint |

`TYPE` is one of `text`, `number`, `boolean`, `timestamp`, `enum` (with `--enum-type`, an enum's id from `model show`; create the enum first with `enum create`, and one enum can be shared by several fields), `reference` (with `--references`, the target entity's id) or `related` (with `--related`, the id of a reference field on another entity that points here).

A `related` field is how a record lists the records that reference it: to list a cookbook's recipes, add a `related` field to Cookbook with `--related` set to Recipe's reference field to Cookbook. It adds no data. The generated API pages through those records on the parent.

## After a change

Tell the user what changed and the `apiVersion` it printed. That is the API version the model is at after the change. An app the user builds pins a version, so a change never breaks an app already built on an earlier one.

Changing a field's `--api-name` renames it in the API, in the new version only. An app pinned to an earlier version keeps the old name, and its records carry over, because records are keyed by field id. Changing only `--name` leaves the API as it is.

Exit 0 with `"apiVersion": ""` means the change was made but its version could not be read. Run `multipaas model show --app APP_ID --json` to get the current version.

## When MultiPaaS refuses

A change MultiPaaS refuses exits 1 with `{"error": {"code", "message"}}` and a code other than `CLI_ERROR`, and nothing changed. Tell the user the message as it is. It is the platform's own reason. Don't retry the same change. Suggest what would make it allowed, where the code says:

| Code | Meaning |
|---|---|
| `UNAUTHORIZED` | The user may not change this application's model: they lack `model.edit` on the application. Making a field a parent, or changing what a parent field references (its `--references`, or a `--type` that is not `reference`), also needs `access.manage.application`: `field add --parent`, and any `field update` that leaves the field a parent while giving it a new parent link. Both are the same as in the web app. Only someone who manages the application's access can grant it |
| `NOT_FOUND` | No application with that id that the user can reach, or no entity, field or constraint with the id given to `--id` |
| `ENTITY_NOT_FOUND` | `--entity` or `--references` names no entity in this application |
| `MODEL_GENERATION_FAILED` | The platform could not generate the API for the changed model, so the change was not made. Tell the user the message |
| `DUPLICATE_ENTITY_NAME`, `DUPLICATE_FIELD_NAME`, `DUPLICATE_CONSTRAINT` | That name, or that set of fields, already exists |
| `UNSAFE_FIELD_CHANGE` | The change would rewrite existing records. Add a new field instead |
| `ENTITY_IN_USE`, `FIELD_IN_CONSTRAINT`, `REFERENCE_HAS_RELATED_FIELD` | Something else depends on it. The message names what to retire first |
| `EXISTING_RECORDS_VIOLATE_CONSTRAINT` | Existing records already break the uniqueness |
| `VALIDATION` | An input is invalid. The message names the fields |

`CLI_ERROR` with a message such as `Unknown field type` is a bad flag value: fix it and run the command again.
