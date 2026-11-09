---
name: multipaas-model
description: Change a MultiPaaS application's model — declare, change and retire entities, enums, fields (including enum and reference fields) and unique constraints — by running the multipaas CLI. Use when the user wants to model data, add or change an entity, enum or field, or asks what their MultiPaaS model looks like.
allowed-tools: Bash(multipaas *)
---

# Change the model

The `multipaas` skill covers installing the CLI, signing in and the exit codes. Follow it for those. This skill is for the model commands.

These commands need CLI 0.5.0 or later. If `multipaas --version` reports an older version, ask the user to run `brew update`, then, as a separate command, `brew upgrade multipaas`.

Every change acts on the live model of one application, as the signed-in developer, with exactly their permissions in the MultiPaaS web app. A change takes effect immediately and mints a new API version. There is no publish step.

## Before changing anything

1. Find the application: `multipaas apps list --json`. Ask the user which one if more than one fits. If none fits, offer to create one, and on a yes run `multipaas apps create --name NAME --json` and use its `application.id`. It needs CLI 0.8.0 or later: check `multipaas --version` first, and if it is older, ask the user to upgrade as above before running it. If it exits 1 with `UNAUTHORIZED`, the user can't create applications in that client: tell them to ask a Client manager in their organization to create it or give them permission. For `CLIENT_REQUIRED`, see the `multipaas` skill. If there is still no application, stop here: the later steps need an `APP_ID`.
2. Read its model: `multipaas model show --app APP_ID --json`. Use the ids it prints. Never guess an id.
3. Say what you are about to change, in a sentence, before running a command that retires something.

If `multipaas model show` exits 1 with `CLI_ERROR` and an `unknown command` message, the CLI is older than 0.2.0. Ask the user to run `brew update`, then, as a separate command, `brew upgrade multipaas`.

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
| `multipaas field add --entity ENTITY_ID --type TYPE [--name] [--api-name] [--required] [--default VALUE] [--not-null] [--enum-type ENUM_ID] [--references ENTITY_ID] [--related FIELD_ID] [--parent] [--max-file-bytes N] [--lifecycle]` | Add a field |
| `multipaas field update --id FIELD_ID [--type] [--name] [--api-name] [--required[=false]] [--default VALUE \| --clear-default] [--not-null \| --nullable] [--enum-type ENUM_ID] [--references] [--related] [--parent[=false]] [--max-file-bytes N]` | Change a field. Only the flags given change |
| `multipaas field retire --id FIELD_ID` | Retire a field |
| `multipaas constraint add --entity ENTITY_ID --fields FIELD_ID,FIELD_ID` | Make those fields unique together |
| `multipaas constraint retire --id CONSTRAINT_ID` | Retire a constraint |

Every entity starts managed: each record is its own unit of access, and its creator manages it. Turn managed off (`entity update --managed=false`) only for an entity that has a required parent field (a reference field with `--parent` and `--required`, from `field add` or `field update`); its records then follow the parent record's access, including records other users added. On an unmanaged entity `record.edit_all` is enough to delete records, and sharing one takes managing its parent. Creating a child, or moving it to another parent, takes edit access to that parent. A child follows its parent only while the parent is a live record of the same app-tenant with access of its own (a managed record, or one that follows a parent in turn). Otherwise (no parent value, a retired parent, an app-wide parent, or a parent with no access of its own) not even its creator can read it, and only roles that read every record of its entity or of an entity above it in the parent chain can.

`TYPE` is one of `text`, `number`, `boolean`, `timestamp`, `enum` (with `--enum-type`, an enum's id from `model show`; create the enum first with `enum create`, and one enum can be shared by several fields), `reference` (with `--references`, the target entity's id), `related` (with `--related`, the id of a reference field on another entity that points here) or `file` (a file on each record, with an optional `--max-file-bytes` for its largest size in whole bytes, up to 512 MiB, which is the default; `file`, and `--max-file-bytes` on `field add` and `field update`, need CLI 1.1.0 or later: check `multipaas --version` first, and if it is older, ask the user to upgrade as above). A file field can be required, but it has no default and can't be not null. The `multipaas-build` skill says how an app uploads and reads a file.

A field can have a default: the value a record gets when it is written without one. Set it with `--default VALUE` on `field add` or `field update`, and remove it with `field update --clear-default`, which a not-null field refuses with VALIDATION "a not-null field needs a default value": give `--clear-default --nullable` in one `field update` to do both. Write the value as the field's type: a number for a number field, `true` or `false` for a boolean field, one of the enum's values for an enum field, an ISO timestamp such as `2026-01-01T00:00:00Z` for a timestamp field, a record id for a reference field, and the text itself for a text field. A file or related field has no default, and the CLI refuses `--default` for one. `--not-null` makes the field always hold a value. It needs a default, given with `--default` or already set, and turning it on writes the default onto every existing record that has no value. `field update --nullable` turns it off. A reference, related or file field can't be not null. `model show` lists each field's default and whether it is not null. `--default`, `--clear-default`, `--not-null` and `--nullable` need CLI 1.4.0 or later: check `multipaas --version` first, and if it is older, ask the user to upgrade as above.

A `related` field is how a record lists the records that reference it: to list a cookbook's recipes, add a `related` field to Cookbook with `--related` set to Recipe's reference field to Cookbook. It adds no data. The generated API pages through those records on the parent.

## After a change

Tell the user what changed and the `apiVersion` it printed. That is the API version the model is at after the change. An app the user builds pins a version, so a change never breaks an app already built on an earlier one.

Changing a field's `--api-name` renames it in the API, in the new version only. An app pinned to an earlier version keeps the old name, and its records carry over, because records are keyed by field id. Changing only `--name` leaves the API as it is.

Exit 0 with `"apiVersion": ""` means the change was made but its version could not be read. Run `multipaas model show --app APP_ID --json` to get the current version.

## When MultiPaaS refuses

A change MultiPaaS refuses exits 1 with `{"error": {"code", "message"}}` and a code other than `CLI_ERROR`, and nothing changed. Tell the user the message as it is. It is the platform's own reason. Don't retry the same change. Suggest what would make it allowed, where the code says:

| Code | Meaning |
|---|---|
| `UNAUTHORIZED` | The user may not change this application's model: they lack `model.edit` on the application. Making a field a parent, or changing what a parent field references (its `--references`, or a `--type` that is not `reference`), also needs `access.manage.application`: `field add --parent`, and any `field update` that leaves the field a parent while giving it a new parent link. Both are the same as in the web app. The CLI checks `model.edit` and `access.manage.application` in the default environment. Only someone who manages the application's access can grant it |
| `NOT_FOUND` | No application with that id that the user can reach, or no entity, field or constraint with the id given to `--id` |
| `ENTITY_NOT_FOUND` | `--entity` or `--references` names no entity in this application |
| `MODEL_GENERATION_FAILED` | The platform could not generate the API for the changed model, so the change was not made. Tell the user the message |
| `DUPLICATE_ENTITY_NAME`, `DUPLICATE_FIELD_NAME`, `DUPLICATE_CONSTRAINT` | That name, or that set of fields, already exists |
| `UNSAFE_FIELD_CHANGE` | The change would rewrite existing records. Add a new field instead |
| `ENTITY_IN_USE`, `FIELD_IN_CONSTRAINT`, `REFERENCE_HAS_RELATED_FIELD` | Something else depends on it. The message names what to retire first |
| `EXISTING_RECORDS_VIOLATE_CONSTRAINT` | Existing records already break the uniqueness. Uniqueness holds within one app-tenant and among one environment's app-wide records, but adding a constraint checks every live environment. Follow "A uniqueness refusal" below |
| `VALIDATION` | An input is invalid. The message names the fields |

`CLI_ERROR` with a message such as `Unknown field type` is a bad flag value: fix it and run the command again.

### A uniqueness refusal

`constraint add` has no `--env` and acts in the application's default environment. The error names the duplicate records only when they are in that environment and the user can read them.

1. Records named: ask the user which of them to change or remove, do it, then run `constraint add` again. The message can end "records in another environment of this application also share values" (do step 2 after this one) or "records in other app-tenants also share values" (step 3 for those).
2. The message says records are in another environment of this application, alone or after named ones: that can be the environment `multipaas.json` pins. Listing by environment needs CLI 0.13.0 or later: check `multipaas --version` first, and if it is older, ask the user to upgrade as at the top. Take each `ENV_ID` from `multipaas envs list --app APP_ID --json`. Find the records with `multipaas records list --app APP_ID --entity ENTITY_ID --app-wide --env ENV_ID --json`, and for each app-tenant of that environment (`multipaas tenants list --app APP_ID --env ENV_ID --json`) with `--tenant APP_TENANT_ID` in place of `--app-wide --env ENV_ID`. Ask the user which to change or remove, do it, then run `constraint add` again.
3. The message is only "existing records share values for the constrained fields", or says "in other app-tenants", or `UNAUTHORIZED` stops the listing, or step 2 finds none of the duplicates: the user can't read those records, and the bare message can also mean they can read app-tenant records but not app-wide ones. Tell them to ask someone who can to clear the duplicates, then run `constraint add` again.
