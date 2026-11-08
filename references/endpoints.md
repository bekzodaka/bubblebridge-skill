---
title: Scopes and endpoints
summary: The four scopes your agent can ask for and the five Google endpoints the proxy serves.
---

## Scopes

Your agent asks for BubbleBridge scope names. BubbleBridge maps each one to one Google scope. The consent page shows each scope to the user. Ask only for the scopes that your agent uses.

| Scope | Google scope | Allows |
|---|---|---|
| `google.gmail.send` | `gmail.send` | Send an email from the account of the user. |
| `google.gmail.readonly` | `gmail.readonly` | List and read the messages of the user. |
| `google.calendar.events` | `calendar.events` | Create events on a calendar of the user. |
| `google.calendar.readonly` | `calendar.readonly` | List the events of a calendar of the user. |

## Endpoints

The proxy accepts only these five Google endpoints. Every other path returns `endpoint_not_allowed`. A path segment that starts with `:` matches any one value.

| Method | Provider path | Scope | SDK function |
|---|---|---|---|
| `POST` | `/gmail/v1/users/me/messages/send` | `google.gmail.send` | `user.google.gmail.send` |
| `GET` | `/gmail/v1/users/me/messages` | `google.gmail.readonly` | `user.google.gmail.listMessages` |
| `GET` | `/gmail/v1/users/me/messages/:id` | `google.gmail.readonly` | `user.google.request` |
| `GET` | `/calendar/v3/calendars/:cal/events` | `google.calendar.readonly` | `user.google.calendar.listEvents` |
| `POST` | `/calendar/v3/calendars/:cal/events` | `google.calendar.events` | `user.google.calendar.createEvent` |

The user ID in a Gmail path must be `me`. The query string passes through to Google.

## Limits

| Limit | Value | Error |
|---|---|---|
| Calls per user per agent | 60 in one minute | `rate_limited` |
| Request body | 6 MB | `payload_too_large` |
| Response body | 6 MB | `payload_too_large` |
| Upstream timeout | 25 seconds | `upstream_timeout` |

## Need another endpoint?

Tell us on the [onboard page](/contact) which Google path your agent calls. We add endpoints by request.

# Errors

