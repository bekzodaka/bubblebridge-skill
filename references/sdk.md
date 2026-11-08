---
title: SDK reference
summary: Every class and function of @bubblebridge/sdk.
---

```sh
npm install @bubblebridge/sdk
```

The package is TypeScript with no runtime dependency. It uses the global `fetch`, `URLSearchParams`, `TextEncoder`, and `crypto`. It runs on Node.js 20 and later and in the browser. The license is MIT.

## BubbleBridge

One client for one agent. It holds the client ID and the client secret.

```ts
new BubbleBridge({ clientId, clientSecret, baseUrl?, apiUrl? })
```

| Option | Default | Use |
|---|---|---|
| `clientId` | | The client ID of the agent. |
| `clientSecret` | | The client secret of the agent. Keep it on the server. |
| `baseUrl` | `https://bubblebridge.io` | The site host. The consent page lives here. |
| `apiUrl` | `https://api.bubblebridge.io` | The API host. |

| Member | Returns | Job |
|---|---|---|
| `oauth` | `Oauth` | The OAuth functions. |
| `forUser(tokens, options?)` | `UserClient` | A client for one user. `options.onTokensUpdated` runs after each token rotation. |

## pkce

```ts
const { state, codeVerifier, codeChallenge } = await pkce();
```

Returns a random `state` value of 16 bytes, a random PKCE verifier of 32 bytes, and the S256 challenge of the verifier, all as base64url. It is async because it uses `crypto.subtle`.

## Oauth

| Function | Returns | Job |
|---|---|---|
| `authorizeUrl({ scopes, redirectUri, state, codeChallenge })` | `string` | The URL of the consent page. Send the browser there. |
| `exchangeCode({ code, codeVerifier, redirectUri })` | `Promise<TokenResponse>` | Exchanges the one-time code for the first token pair. |
| `refresh(refreshToken)` | `Promise<TokenResponse>` | Exchanges a refresh token for a new pair. `UserClient` calls it for you. |

## TokenResponse

```ts
interface TokenResponse {
  access_token: string;
  refresh_token: string;
  expires_in: number;
  scope: string;
  user_id: string;
}
```

Store the whole object under `user_id`. `scope` is a space-separated list of scope names.

## UserClient

One client for one user. `forUser` returns it. It refreshes the pair when the access token has less than 60 seconds left, runs one refresh at a time, and calls `onTokensUpdated` after each rotation.

| Member | Returns | Job |
|---|---|---|
| `google` | `Google` | The Google provider. |
| `accessToken()` | `Promise<string>` | A valid access token. It refreshes first if needed. |
| `connections()` | `Promise<ConnectionList>` | The connections that the user enabled for the agent. |
| `send(method, pathAndQuery, body?, connectionId?)` | `Promise<T>` | One call to any API path with the bearer token. |

## Google

| Member | Returns | Job |
|---|---|---|
| `gmail` | `Gmail` | The Gmail helpers. |
| `calendar` | `Calendar` | The Calendar helpers. |
| `request(method, path, body?, connectionId?)` | `Promise<T>` | One call to any Google path on the allowlist. `path` starts at `/gmail/...` or `/calendar/...` and holds its query string. `connectionId` picks one of several connections. |

## Gmail

| Function | Job |
|---|---|
| `send({ to, subject, text })` | Sends one plain text email from the account of the user. Returns the Gmail message resource. |
| `listMessages(query, maxResults)` | Lists the message IDs that match a Gmail search query, such as `newer_than:1d`. |

## Calendar

| Function | Job |
|---|---|
| `listEvents(calendarId, timeMin)` | Lists the single events that start at or after `timeMin`, in start order. `timeMin` is RFC 3339. |
| `createEvent(calendarId, { summary, description?, start, end })` | Creates one event. `start` and `end` are RFC 3339 date-times. |

Each helper returns the Google response as `unknown`. Define the fields that you read.

## Errors

| Class | Fields | When |
|---|---|---|
| `ApiError` | `status`, `body.code`, `body.message`, `body.request_id` | BubbleBridge returned an error. See [Errors](/docs/errors). |
| `UpstreamError` | `status`, `body` | Google returned an error through the proxy. `body` is the Google response text. |

A network failure throws the `fetch` error.

## Types

The package exports `Scope`, `TokenResponse`, `Connection`, `ConnectionList`, `ConnectionStatus`, `Method`, `SendEmail`, `NewEvent`, `Pkce`, and the generated `paths`, `operations`, and `components` of the OpenAPI document.

## Rust

A Rust SDK with the same functions exists in the same repository as crate `bubblebridge`. It is not on crates.io yet. Write to us on the [onboard page](/contact) if you need it.
