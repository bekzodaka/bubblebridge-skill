# BubbleBridge onboarding for <agent name>

Date: <YYYY-MM-DD>
Status: Draft, waiting for review

## 1. Summary

<What the agent does today. How it talks to Google today. What changes.>

## 2. Inventory

| File | Line | What it does | Google API | Google scope |
|---|---|---|---|---|
| | | | | |

## 3. Mapping

| Old call | BubbleBridge endpoint | BubbleBridge scope | SDK call |
|---|---|---|---|
| | | | |

### Calls without a BubbleBridge endpoint

| Old call | Proposal |
|---|---|
| | Keep on the old path / Remove / Ask BubbleBridge to add it |

## 4. Scopes to request

- `google.…`

## 5. New routes

| Route | Job |
|---|---|
| `GET /connect` | Makes the PKCE pair, stores it for ten minutes, redirects to the consent page |
| `GET /callback` | Checks the state, exchanges the code, stores the token pair under `user_id` |

Redirect URI: `https://<host>/callback`

## 6. Token storage

- Where the token pair of each user lives: <table or collection>
- Encryption at rest: <how>
- Map from the user identity of the agent to the BubbleBridge `user_id`: <how>

## 7. Call changes

| File | Old call | New call |
|---|---|---|
| | | |

## 8. Removals

- The Google OAuth client and its credentials: <files>
- The old consent routes: <files>
- The old token store: <table or files>
- Dependencies to remove: <packages>

## 9. Configuration

| Name | Where it lives |
|---|---|
| `BUBBLEBRIDGE_CLIENT_ID` | |
| `BUBBLEBRIDGE_CLIENT_SECRET` | |

## 10. Open questions

1. 
