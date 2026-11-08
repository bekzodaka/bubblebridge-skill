---
name: bubblebridge
description: Onboard an AI agent to BubbleBridge. Use when the user asks to onboard, migrate, or connect an agent to BubbleBridge, or to replace the agent's own Google OAuth client and its Gmail or Google Calendar calls with one BubbleBridge token.
---

# Onboard an agent to BubbleBridge

BubbleBridge is a proxy between users and AI agents. A user connects Google once on BubbleBridge. An agent gets one BubbleBridge token pair per user and calls Gmail and Google Calendar through the proxy. The agent never holds a Google credential.

This skill moves an existing agent onto BubbleBridge in five steps. Steps 1 to 3 change no code. Step 3 ends with a design document that the user reviews. Do not start step 4 before the user approves the document.

Read `references/endpoints.md` before step 2. Read `references/sdk.md` and `references/example-agent.md` before step 4. Read `references/api.md` if the codebase is not JavaScript or TypeScript.

## Step 1. Scan the codebase

Find every place where the agent touches Google. Search for these and record each hit with its file and line.

| Look for | Examples |
|---|---|
| Google client libraries | `googleapis`, `google-auth-library`, `@googleapis/gmail`, `@googleapis/calendar`, `google-api-python-client`, `google-auth`, `golang.org/x/oauth2/google` |
| Direct HTTP calls | `gmail.googleapis.com`, `www.googleapis.com/gmail`, `www.googleapis.com/calendar`, `oauth2.googleapis.com/token`, `accounts.google.com/o/oauth2` |
| OAuth client credentials | `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `client_secret.json`, `credentials.json` |
| Token storage | `refresh_token`, `access_token`, `token.json`, a users or accounts table with Google tokens |
| Consent routes | A route that redirects to Google, and the callback that exchanges the code |
| Scopes | `gmail.send`, `gmail.readonly`, `gmail.modify`, `calendar`, `calendar.events`, `calendar.readonly` |

Also record how the agent identifies a user today: a user ID, an email, or a session. BubbleBridge gives each user a `user_id`. The design document maps the two.

Write the result as an inventory table: file, line, what it does, which Google API, which Google scope.

## Step 2. Map each call

For each Google call in the inventory, find the row in `references/endpoints.md` that serves it.

- Write the BubbleBridge scope and the SDK function for each call that has a row.
- Flag each call that has no row. BubbleBridge serves five Google endpoints today. A flagged call stays on the old path or goes away. The user decides in the review.
- Collect the set of BubbleBridge scopes that the agent needs. Ask for no scope that no call uses.

Two common cases:

- Reading one message by ID uses `user.google.request('GET', '/gmail/v1/users/me/messages/<id>?format=metadata...')`. There is no helper, but the endpoint is on the list.
- A `gmail.modify` call, such as a label change or a draft, has no row. Flag it.

## Step 3. Write the design document

Write the document to `docs/bubblebridge-onboarding.md` in the repository of the user. Use `references/design-doc-template.md`. Fill every section. Keep each sentence short.

The document has:

1. The inventory from step 1.
2. The mapping from step 2, with the flagged calls.
3. The scopes to request.
4. The new routes: the connect route and the callback route, with their paths and the redirect URI.
5. The token storage: where the token pair of each user lives, how it is encrypted, and how the agent maps its user identity to the BubbleBridge `user_id`.
6. The call changes: each old call and the SDK call that replaces it.
7. The removals: the Google OAuth client, its credentials, and the old token store, after the new path works.
8. The configuration: `BUBBLEBRIDGE_CLIENT_ID` and `BUBBLEBRIDGE_CLIENT_SECRET`, and where they live. Never in git.
9. The open questions.

Stop here. Show the user the path of the document and the open questions. Wait for the approval. Change the document if the user asks.

## Step 4. Implement the plan

Do these in order. Run the tests and the linter of the project after each item.

1. Install the SDK with `npm install @bubblebridge/sdk`. The SDK needs Node.js 20 or later. For another language, build a small client from `references/api.md`.
2. Add the configuration. Read `BUBBLEBRIDGE_CLIENT_ID` and `BUBBLEBRIDGE_CLIENT_SECRET` from the environment or the secret store of the project. Add the two names to the example environment file of the project with empty values.
3. Create one `BubbleBridge` client for the agent. Create it once and share it.
4. Add the connect route. It calls `pkce`, stores `state` and `codeVerifier` for ten minutes in an `HttpOnly` cookie or a server session, and redirects to `bb.oauth.authorizeUrl(...)` with the scopes from the design document.
5. Add the callback route. It checks `state`, calls `bb.oauth.exchangeCode(...)`, and stores the whole token object under `tokens.user_id`. It clears the cookie.
6. Add the token store. One record per `user_id` with the token pair. Encrypt the refresh token at rest if the project encrypts other secrets. Add a `putUser`, a `getUser`, and a `deleteUser` function.
7. Add a function that builds the `UserClient` for one user. It loads the pair, sets `expires_in: 0`, and passes `onTokensUpdated` so each rotated pair is stored at once. Use one `UserClient` for all calls of one user in one run.
8. Replace each Google call with the SDK call from the mapping. Keep the shape of the data that the rest of the code reads. The proxy returns the Google response as is.
9. Add the error handling. On `unauthorized` delete the stored pair of the user. On `connection_reauth_required` and `agent_not_enabled` skip the user and try on the next run. Log every other `ApiError` with its `request_id`.
10. Add an Add to BubbleBridge button or link to the connect route on the page where users start the agent.
11. Remove the Google OAuth client, its credentials, the old consent routes, and the old token store. Remove the Google client libraries from the dependencies if nothing else uses them.
12. Update the README of the project: how a user connects, and the two environment variables.

Rules for the code:

- Never write a client secret, a refresh token, or an access token into git, a log line, or an error message.
- Set `expires_in: 0` when the pair comes from storage. The SDK counts `expires_in` from the moment of the call.
- Store every rotated pair. A reused old refresh token revokes the grant of the user.
- Do not call Google directly for a call that is on the list. Every call goes through the proxy.
- Keep the change small. Do not refactor code that the plan does not name.

## Step 5. Hand off

Give the user this checklist. Mark the items that you did.

1. Open `https://developers.bubblebridge.io`, sign in with Google, and send the onboarding form. Enter the invite code from the email.
2. Click New agent. Fill the slug, the name, the description, the homepage URL, the redirect URIs from the design document, and the scopes from the design document.
3. Copy the client ID and the client secret into the secret store named in the design document.
4. Deploy the agent.
5. Open `https://bubblebridge.io`, sign in, connect Google with the scopes of the agent, open the agent page, and click Add to BubbleBridge. Click Allow.
6. Run the agent for your user. Confirm the result in Gmail or Calendar.
7. Remove the agent on the settings page of BubbleBridge. Confirm that the next run gets `unauthorized` and deletes the stored pair.

The full docs are at `https://bubblebridge.io/docs`.
