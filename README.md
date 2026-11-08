# BubbleBridge skill

One skill for your coding agent. It moves an existing AI agent onto [BubbleBridge](https://bubblebridge.io): it finds every Google call, writes a design document for your review, and then implements the change with the BubbleBridge SDK.

## Install

```sh
npx skills add bekzodaka/bubblebridge-skill
```

Or clone it into the skills folder of your coding agent.

```sh
git clone https://github.com/bekzodaka/bubblebridge-skill .claude/skills/bubblebridge
```

## Run

Open your coding agent in the repository of your agent and send one prompt.

```text
Onboard this agent to BubbleBridge.
```

## What it does

1. Scans the codebase for Google API calls, OAuth clients, token stores, and credentials.
2. Maps each call to a BubbleBridge scope and endpoint. Flags the calls that BubbleBridge does not serve yet.
3. Writes `docs/bubblebridge-onboarding.md` and stops for your review.
4. Implements the plan: the SDK, the connect and callback routes, the token store, the call changes, and the removals.
5. Hands you a checklist to register the agent and test the flow.

## Files

| File | Content |
|---|---|
| `SKILL.md` | The skill. |
| `references/endpoints.md` | The scopes, the five endpoints, the limits, and the error codes. |
| `references/sdk.md` | The SDK reference. |
| `references/api.md` | The HTTP API, for a codebase that is not JavaScript or TypeScript. |
| `references/example-agent.md` | The shape of a complete agent. |
| `references/design-doc-template.md` | The template for the design document. |

The full docs are at [bubblebridge.io/docs](https://bubblebridge.io/docs).

## License

MIT
