# Example agent

The code below is the shape of a complete agent in TypeScript. It comes from the BubbleBridge test agent. Adapt the framework calls to the project.

## The client

```ts
import { BubbleBridge, pkce, ApiError, type Scope, type TokenResponse } from '@bubblebridge/sdk';

const SCOPES: Scope[] = ['google.gmail.readonly', 'google.calendar.events'];
const COOKIE = 'bb_connect';

const bb = new BubbleBridge({
  clientId: process.env.BUBBLEBRIDGE_CLIENT_ID!,
  clientSecret: process.env.BUBBLEBRIDGE_CLIENT_SECRET!,
});
```

## The connect route

```ts
app.get('/connect', async (req, res) => {
  const redirectUri = `https://${req.hostname}/callback`;
  const { state, codeVerifier, codeChallenge } = await pkce();
  res.cookie(COOKIE, `${state}.${codeVerifier}`, { httpOnly: true, secure: true, sameSite: 'lax', maxAge: 600_000 });
  res.redirect(bb.oauth.authorizeUrl({ scopes: SCOPES, redirectUri, state, codeChallenge }));
});
```

## The callback route

```ts
app.get('/callback', async (req, res) => {
  const redirectUri = `https://${req.hostname}/callback`;
  const [expectedState, codeVerifier] = String(req.cookies[COOKIE] ?? '').split('.');
  const { code, state } = req.query;
  if (!code || !state || !codeVerifier || state !== expectedState) {
    return res.status(400).send('The state does not match. Click Add to BubbleBridge again.');
  }
  const tokens = await bb.oauth.exchangeCode({ code: String(code), codeVerifier, redirectUri });
  await putUser(tokens.user_id, tokens);
  res.clearCookie(COOKIE);
  res.send('Connected.');
});
```

## The token store

One record per user. Replace with the database of the project.

```ts
interface StoredUser {
  userId: string;
  tokens: TokenResponse;
}

async function putUser(userId: string, tokens: TokenResponse): Promise<void> { /* write */ }
async function getUser(userId: string): Promise<StoredUser | undefined> { /* read */ }
async function deleteUser(userId: string): Promise<void> { /* delete */ }
async function listUsers(): Promise<StoredUser[]> { /* list */ }
```

## The user client

```ts
function clientFor({ userId, tokens }: StoredUser) {
  return bb.forUser(
    { ...tokens, expires_in: 0 },
    { onTokensUpdated: (fresh) => void putUser(userId, fresh).catch(console.error) },
  );
}
```

## A job over all users

```ts
const SKIP_CODES = ['connection_reauth_required', 'agent_not_enabled'];

for (const stored of await listUsers()) {
  try {
    const user = clientFor(stored);
    const list = (await user.google.gmail.listMessages('newer_than:1d', 20)) as { messages?: { id: string }[] };
    for (const { id } of list.messages ?? []) {
      const message = await user.google.request<{ snippet?: string }>(
        'GET',
        `/gmail/v1/users/me/messages/${id}?format=metadata&metadataHeaders=From&metadataHeaders=Subject`,
      );
      // use message.snippet
    }
    await user.google.calendar.createEvent('primary', {
      summary: 'Inbox digest',
      description: 'Summary text',
      start: new Date().toISOString(),
      end: new Date(Date.now() + 15 * 60_000).toISOString(),
    });
  } catch (error) {
    if (error instanceof ApiError && error.body.code === 'unauthorized') {
      await deleteUser(stored.userId);
    } else if (error instanceof ApiError && SKIP_CODES.includes(error.body.code)) {
      console.log(`user ${stored.userId} skipped: ${error.body.code}`);
    } else {
      console.log(`user ${stored.userId} failed: ${String(error)}`);
    }
  }
}
```

## A page with the button

```html
<h1>My agent</h1>
<p>The agent needs read access to your Gmail inbox and write access to your calendar events.</p>
<a href="/connect">Add to BubbleBridge</a>
```
