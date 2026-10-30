---
name: multipaas-import
description: Load an existing app's data into a MultiPaaS application — turn an export from the old database into JSON-lines files, then upload, check and run the import — by running the multipaas CLI. Use when the user wants to import, migrate or bootstrap data from another database into MultiPaaS.
allowed-tools: Bash(multipaas *)
---

# Import data

The `multipaas` skill covers installing the CLI, signing in and the exit codes. Follow it for those. This skill is for the import commands.

These commands need CLI 0.7.0 or later. If `multipaas --version` reports an older version, ask the user to run `HOMEBREW_GITHUB_API_TOKEN="$(gh auth token)" brew update`, then, as a separate command, `HOMEBREW_GITHUB_API_TOKEN="$(gh auth token)" brew upgrade multipaas`.

`import delete` and `import status --include-deleted` need CLI 0.9.0 or later. Upgrade the same way.

`--env` on the import commands needs CLI 0.11.0 or later. Upgrade the same way.

Every import command takes `--env ENV_ID`, an `id` from `multipaas envs list --app APP_ID --json`. An import belongs to one environment and writes its app-tenants and records there; without `--env` it uses the application's default environment. Import test data into a non-default environment first (`multipaas envs create --app APP_ID --name test --json` makes one), and give every command for that import the same `--env`.

An import needs permission to manage the application's access (`access.manage.application`). It writes everything or nothing.

## 1. Know the target

1. Find the application: `multipaas apps list --json`. Ask the user which one if more than one fits. If none fits, offer to create one, and on a yes run `multipaas apps create --name NAME --json` and use its `application.id`. It needs CLI 0.8.0 or later: check `multipaas --version` first, and if it is older, ask the user to upgrade as above before running it. If it exits 1 with `UNAUTHORIZED`, the user can't create applications in that client: tell them to ask a Client manager in their organization to create it or give them permission. For `CLIENT_REQUIRED`, see the `multipaas` skill. If there is still no application, stop here: the later steps need an `APP_ID`.
2. Read its model: `multipaas model show --app APP_ID --json`. Every record you import must fit an entity there. If the model is missing an entity or field the old data needs, change the model first with the `multipaas-model` skill.

## 2. Write the files

Write one JSON-lines file per kind into one directory, one JSON object per line, from the user's old database. Ask the user how to reach that data, and write the export with their tools. Never invent rows.

| File | Each line |
|---|---|
| `tenants.jsonl` | `{"id": "OLD_ID", "name": "NAME"}`, one per app-tenant (an account, team or organisation in the old app) |
| `users.jsonl` | `{"id": "OLD_ID", "email": "EMAIL", "name": "NAME"}`. `name` is optional |
| `memberships.jsonl` | `{"user": "OLD_USER_ID", "tenant": "OLD_TENANT_ID", "role": "ROLE NAME"}`. The role must be one the application declares |
| `<entity>.jsonl` | `{"id": "OLD_ID", "tenant": "OLD_TENANT_ID", "createdBy": "OLD_USER_ID", "createdAt": "ISO 8601", "updatedAt": "ISO 8601", "deletedAt": null, "fields": {"GRAPHQL_FIELD_NAME": VALUE}}`. Name the file after the entity's name, type name or plural name from `model show` |

- Ids are the old database's ids. A reference field's value is the old id of the row it points at; MultiPaaS assigns new ids to tenants and records and remaps references. A user whose email already has an account is linked to that account, which keeps its id and name. Rows can be in any order.
- An `id` is at most 1,024 bytes in UTF-8, with no control characters or unpaired surrogates.
- `fields` uses each field's `graphqlFieldName` from `model show`. An enum value is the value, not its API name. `createdBy`, the timestamps and `deletedAt` are optional; a non-null `deletedAt` imports the record as deleted.
- An earlier import's rows can be referenced by their old ids, so tenants and users can be imported first and records later.

## 3. Import

| Command | Does |
|---|---|
| `multipaas import create --app APP_ID --dir DIR --json` | Creates an import and uploads every `.jsonl` in `DIR`. Prints `{"import": {...}}` with its `id`. If any file can't be mapped or used, nothing is created; see the refusal table below |
| `multipaas import add --app APP_ID --import IMPORT_ID --kind tenants\|users\|memberships\|records [--entity ENTITY_ID] --file PATH --json` | Adds one file. `records` needs `--entity`, an entity id from `model show` |
| `multipaas import check --app APP_ID --import IMPORT_ID --json` | Checks every row, waits, and prints the import |
| `multipaas import run --app APP_ID --import IMPORT_ID --json` | Writes everything in one go, only after a clean check, waits, and prints the import |
| `multipaas import status --app APP_ID --import IMPORT_ID [--include-deleted] --json` | Prints the import. A deleted import is `NOT_FOUND` unless `--include-deleted` is given; then it has `deletedAt` set |
| `multipaas import delete --app APP_ID --import IMPORT_ID --json` | Marks the import deleted and prints it with `deletedAt` set. Its files, its problems and any data it imported stay. Refused with `DATA_IMPORT_STATE` while it is checking or importing |

The import is `{"id", "status", "files": [...], "problemCount", "problems": [{"file", "line", "sourceId", "field", "code", "message"}], "counts": [{"key", "count"}], ...}`. `check` and `run` exit 0 when they finish, whatever the result, so read `status`.

In the table below, `import check`, `import run` and `import status` mean those commands from the table above with the same `--app`, `--import` and `--json`, and `import create --dir` means the create command with the same `--app`, `--dir` and `--json`; it makes a new import, with a new id:

| `status` | Next |
|---|---|
| `checked` | Tell the user the counts, then ask before running `import run`. Running writes the data |
| `check-failed` | Nothing was written. Read each problem's `code`. A problem that names a file is a data problem, with or without a line: fix the data or the model, then run `import create --dir` again for a new import. With a line, fix that row. `invalid-row` with "The entity for this file no longer exists" has no line: the entity was retired, probably on purpose, so ask the user: either leave that file out and run `import create --dir` again, or declare an entity with that name again with the `multipaas-model` skill (a new entity, which `create --dir` matches by name; declare it so the file's name without `.jsonl` is its name, its type name (`--type-name`) or its plural name (`--plural-name`), and add the fields the rows use with `field add --api-name` set to each key in `fields`) and make a new import. `file-checksum-mismatch` has no line: the stored upload differs from what was registered, so create the import again. `check-error` and `check-timed-out` name no file: run `import check` again on the same import, and tell the user if it fails again. `check-unauthorized` means the user can no longer manage the application, or lacks read access to records the import refers to: tell them, and don't retry |
| `imported` | Done. Tell the user the counts, and that imported users are not emailed: they join their tenants at their next sign-in within 7 days of the run. After that they must be invited. A wrong imported invitation can be deleted from the console's Members screen, under Invitations, before the person signs in; they then do not join |
| `import-failed` | Nothing was written. Read each problem's `code`. A problem that names a file is a data problem, with or without a line: fix the data or the model, then run `import create --dir` again, because the files of an import that failed can't be changed. With a line, fix that row. `invalid-row` with "The entity for this file no longer exists" has no line: the entity was retired, probably on purpose, so ask the user: either leave that file out and run `import create --dir` again, or declare an entity with that name again with the `multipaas-model` skill (a new entity, which `create --dir` matches by name; declare it so the file's name without `.jsonl` is its name, its type name (`--type-name`) or its plural name (`--plural-name`), and add the fields the rows use with `field add --api-name` set to each key in `fields`) and make a new import. `file-checksum-mismatch` has no line: the stored upload differs from what was registered, so create the import again. `import-error` and `import-timed-out` name no file and leave nothing to fix: run `import check`, then `import run`, again on the same import, and tell the user if it fails again. `import-unauthorized` means the user can no longer manage the application, or lacks read access to records the import refers to: tell them, and don't retry. `application-inactive` means the application was deactivated: tell the user, and don't retry |
| `checking`, `importing` | Still running. Run `import status` again in a minute; `check` and `run` stop waiting after 20 minutes with `IMPORT_STILL_RUNNING` |

Once a replacement for a `check-failed` or `import-failed` import has been made with `import create --dir`, tell the user the failed one can now be deleted, and on a yes run `import delete` with the failed import's id, not the new one. Deleting only hides it: `import status --include-deleted` still shows it with its files and problems. Don't delete an `imported` import to undo it: the data it wrote stays.

Common problem codes: `invalid-json`, `missing-id`, `missing-field`, `invalid-value`, `unknown-field` (not a GraphQL field name of that entity), `broken-reference` (no row with that old id), `cross-tenant-reference`, `unknown-tenant`, `unknown-user`, `unknown-role`, `duplicate-source-id`, `duplicate-unique-key`, `already-imported` (an earlier import already has that id), `id-too-long`, `id-invalid-character`.

## When MultiPaaS refuses

A refused command exits 1 with `{"error": {"code", "message"}}`. Tell the user the message. Don't retry the same command.

| Code | Meaning |
|---|---|
| `UNAUTHORIZED` | The user cannot manage this application's access |
| `IMPORT_FILE_UNMATCHED`, `IMPORT_FILE_AMBIGUOUS`, `IMPORT_DIR_EMPTY` | `create --dir` could not map the directory; the message names the files. A `tenants`, `users` or `memberships` file that also matches an entity is `IMPORT_FILE_AMBIGUOUS`: add it with `import add` (`--kind tenants\|users\|memberships`, or `--kind records --entity ENTITY_ID`). Nothing was created |
| `IMPORT_FILE_INVALID` | `create --dir` found a file that cannot be read, is empty or is over 512 MiB; the message names the files. Nothing was created |
| `UPLOAD_REFUSED` | The upload failed. If the message says the file was removed from the import, add it again. If it says the file is still listed, it cannot be added again: run `import create --dir` for a new import |
| `DUPLICATE_DATA_IMPORT_FILE` | The import already has a file of that kind (and entity), or, from `create --dir`, two files in the directory map to the same kind or entity and nothing was created. The message names the files |
| `INVALID_DATA_IMPORT_FILE` | `--entity` is missing for `records`, given for another kind, or not an entity of this application |
| `INVALID_FILE_UPLOAD` | The file is empty, over 512 MiB, or its name is too long |
| `DATA_IMPORT_STATE` | The import can't do that now (for example running before a clean check, adding a file after a run, or deleting it while it is checking or importing). Run `import status` |
| `DATA_IMPORT_EMPTY` | The import has no files |
| `FILE_UPLOAD_NOT_VERIFIED` | An upload did not complete and its file is still on the import, so it cannot be added again. Run `import create --dir` for a new import |
| `PROBLEM_REPORT_UNAVAILABLE` | The full problem report could not be downloaded. Run `import status` again |
| `IMPORT_STILL_RUNNING` | Still running after 20 minutes. Run `import status` later |
| `NOT_FOUND` | No application or import with that id that the user can reach. A deleted import is `NOT_FOUND` too, unless `import status` is given `--include-deleted` |
