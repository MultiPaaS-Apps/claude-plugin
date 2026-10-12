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
- A client is public: it has no secret, so never invent one.

## The flow

Use the `endpoints` that `multipaas init --json` printed, and `oauthClientId` from `multipaas.json`. Prefer the app framework's own OAuth/PKCE library, when it has one, over hand-written crypto.

1. **Authorize.** Send the browser to `endpoints.authorize` with these parameters:
   - `client_id`
   - `redirect_uri`: exactly one of the registered ones
   - `response_type=code`
   - `code_challenge`: base64url SHA-256 of a random verifier
   - `code_challenge_method=S256`
   - `state`: random, and checked on return

   There is no `scope` parameter. The user picks the app-tenant they are signing in to on the consent page.
2. **Exchange.** The callback receives `code` and `state`. Within 60 seconds, `POST` form-encoded to `endpoints.token` with `grant_type=authorization_code`, `client_id`, `code`, `redirect_uri` and `code_verifier`. The answer is `{"access_token", "refresh_token", "token_type": "Bearer", "expires_in": 3600}`.
3. **Call the API** with `Authorization: Bearer <access_token>`, as the `multipaas-build` skill describes.
4. **Refresh** before the access token expires: `POST` form-encoded to `endpoints.token` with `grant_type=refresh_token`, `client_id` and `refresh_token`.
   - Each refresh returns a new pair. Store both, and never use a refresh token twice: reusing one revokes the whole sign-in.
   - A sign-in lasts at most 7 days from the first sign-in, and then the user signs in again.
5. **Sign out:** `POST` form-encoded to `endpoints.revoke` with `client_id` and `token` (the refresh token).

A token request the server refuses answers `{"error", "error_description"}`. An `invalid_grant` on refresh means the sign-in is over: send the user through step 1 again.
