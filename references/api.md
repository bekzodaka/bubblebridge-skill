---
title: API reference
summary: The HTTP routes that an agent calls. The SDK wraps each one.
---

The API host is `https://api.bubblebridge.io`. The consent page is on the site host `https://bubblebridge.io`. Every error has the shape in [Errors](/docs/errors).

## GET /authorize

The consent page. Send the browser here. It is a page on the site host, not an API route.

| Query parameter | Value |
|---|---|
| `client_id` | The client ID of the agent. |
| `redirect_uri` | One of the redirect URIs of the agent, letter for letter. |
| `scope` | Scope names separated by spaces. |
| `state` | A random value. The callback gets it back. |
| `code_challenge` | The S256 challenge of the PKCE verifier. |

After Allow the browser opens `<redirect_uri>?code=<code>&state=<state>`. The code works one time and for ten minutes.

## POST /oauth/token

Exchanges a code or a refresh token for a token pair. The body is `application/json` or `application/x-www-form-urlencoded`. The response is JSON.

Code grant:

```json
{
  "grant_type": "authorization_code",
  "code": "...",
  "code_verifier": "...",
  "redirect_uri": "https://agent.example.com/callback",
  "client_id": "...",
  "client_secret": "..."
}
```

Refresh grant:

```json
{
  "grant_type": "refresh_token",
  "refresh_token": "...",
  "client_id": "...",
  "client_secret": "..."
}
```

Response:

```json
{
  "access_token": "eyJ...",
  "refresh_token": "...",
  "expires_in": 3600,
  "scope": "google.gmail.readonly google.calendar.events",
  "user_id": "01JC..."
}
```

Each refresh returns a new refresh token and ends the old one. A second use of an old refresh token revokes the grant of the user.

## POST /oauth/revoke

Ends the refresh token and every token of the same family. Same encodings as the token route.

```json
{ "token": "<refresh token>", "client_id": "...", "client_secret": "..." }
```

## GET /.well-known/jwks.json

The RSA public key that signs the access tokens. The access token is a JWT with RS256. Its `exp` claim is one hour after issue.

## GET /v1/connections

Lists the connections of the user that are enabled for the agent.

```text
Authorization: Bearer <access token>
```

```json
{
  "connections": [
    { "id": "01JC...", "provider": "google", "account_email": "bob@gmail.com", "scopes": ["google.gmail.readonly"], "status": "active" }
  ]
}
```

`status` is `active` or `reauth_required`.

## ANY /v1/providers/google/{path}

The proxy. `path` is a Google API path from [Scopes and endpoints](/docs/scopes-and-endpoints) with its query string. Example: `GET /v1/providers/google/calendar/v3/calendars/primary/events?timeMin=...`.

| Header | Value |
|---|---|
| `Authorization` | `Bearer <access token>` |
| `Content-Type` | `application/json` for a body. |
| `X-BubbleBridge-Connection` | Optional. The connection ID if the user has more than one connection that matches the scope. |

The proxy returns the Google status and the Google body as is.

## OpenAPI

The SDK exports the generated types `paths`, `operations`, and `components` of the OpenAPI document of the API.
