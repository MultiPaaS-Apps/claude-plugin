---
name: multipaas-sign-in
description: Add end-user sign-in to an app built on a MultiPaaS application — register its OAuth client with the multipaas CLI and implement the authorization-code flow with PKCE against MultiPaaS. Use when the user wants their app's users to sign in, needs an access token for the generated API, or wants to change the app's redirect URLs.
allowed-tools: Bash(multipaas *)
---

# End-user sign-in

An app's end users sign in to MultiPaaS through the app's own OAuth client. The app gets an access token for the generated API. The `multipaas-build` skill covers the project and the API calls.

These commands need CLI 0.3.0 or later, and the `application.manage_oauth` permission on the application. Without the permission, they're refused with `UNAUTHORIZED`, the same as in the web app. Only someone who manages the application's access can grant it.

## Register the client

| Command | Does |
|---|---|
| `multipaas oauth-client list --json` | `[{"id", "name", "redirectUris", "createdAt"}]` |
| `multipaas oauth-client create --name NAME --redirect-uri URL [--redirect-uri URL] --json` | `{"oauthClient": {...}}`. In a directory with `multipaas.json`, its id is saved there as `oauthClientId` |
| `multipaas oauth-client update [--id ID] [--name NAME] [--redirect-uri URL ...] --json` | Changes only what is given. `--redirect-uri` replaces the whole list. `--id` defaults to `oauthClientId` when `multipaas.json` is for the same application |

- A redirect URI must be `https`, or `http` on `localhost` or `127.0.0.1`.
- A client takes 1 to 10 redirect URIs.
- Register the URL the app actually receives the callback at, such as the dev server's, and the deployed one.
- A client registered here is public: it has no secret, so never invent one. A background job uses an app identity's confidential client instead (see `multipaas-build`).

## Signing in to a test environment

Choosing an environment needs CLI CLI_FLOOR_TBD or later. Check `multipaas --version` first, and if it is older, ask the user to upgrade.

An application keeps its test data in separate environments, and a sign-in is to one of them. To sign in to another than the default, run `multipaas init --app APP_ID --env ENV_ID --tenant APP_TENANT_ID --json`, with the environment id from `multipaas envs list --app APP_ID --json` and an app-tenant of that environment. It saves `environmentId` in `multipaas.json`, and the authorize request passes it as `environment`. The same client and redirect URIs serve every environment, so there is nothing to register again. `init` always saves the environment of the app-tenant it pins, with or without `--env`, so an app whose test app-tenant is in another environment signs in to that environment. A `multipaas.json` written by a CLI older than CLI_FLOOR_TBD has no `environmentId`, and its app signs in to the default environment until `init` is run again with the newer CLI.

## App identities

An app identity belongs to the environment it was created in: `multipaas app-identity create --env ENV_ID --name NAME --json`, or the environment in `multipaas.json` when `--env` is left out; both need CLI CLI_FLOOR_TBD or later. Its client-credentials tokens are for that environment only, and an `app_tenant` from another environment is refused. `multipaas-build` covers creating identities and getting their tokens.

## The flow

Use the `endpoints` that `multipaas init --json` printed, and `oauthClientId` from `multipaas.json`. Prefer the app framework's own OAuth/PKCE library, when it has one, over hand-written crypto.

1. **Authorize.** Send the browser to `endpoints.authorize` with these parameters:
   - `client_id`
   - `redirect_uri`: exactly one of the registered ones
   - `response_type=code`
   - `code_challenge`: base64url SHA-256 of a random verifier
   - `code_challenge_method=S256`
   - `state`: random, and checked on return
   - `environment`: the `environmentId` from `multipaas.json`, when it has one. Leave it out for the default environment. An environment that isn't one of the application's, or has been removed, is refused with `invalid_request`.

   There is no `scope` parameter. The user picks the app-tenant they are signing in to on the consent page. The consent page lists only the chosen environment's app-tenants, and the tokens are for that environment.
2. **Exchange.** The callback receives `code` and `state`. Within 60 seconds, `POST` form-encoded to `endpoints.token` with `grant_type=authorization_code`, `client_id`, `code`, `redirect_uri` and `code_verifier`. The answer is `{"access_token", "refresh_token", "token_type": "Bearer", "expires_in": 3600}`.
3. **Call the API** with `Authorization: Bearer <access_token>`, as the `multipaas-build` skill describes.
4. **Refresh** before the access token expires: `POST` form-encoded to `endpoints.token` with `grant_type=refresh_token`, `client_id` and `refresh_token`.
   - Each refresh returns a new pair. Store both, and never use a refresh token twice: reusing one revokes the whole sign-in.
   - A sign-in lasts at most 7 days from the first sign-in, and then the user signs in again.
5. **Sign out:** `POST` form-encoded to `endpoints.revoke` with `client_id` and `token` (the refresh token).

A token request the server refuses answers `{"error", "error_description"}`. An `invalid_grant` on refresh means the sign-in is over: send the user through step 1 again.
