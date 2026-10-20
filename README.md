# @corellium/api-client

Typed TypeScript client for the Corellium REST API, generated from the Corellium
OpenAPI spec (API version 1.0.0).

## Installation

```bash
npm install @corellium/api-client
```

## Quick start

```ts
import { CorelliumClient } from '@corellium/api-client';

const client = new CorelliumClient({
  accessToken: process.env.CORELLIUM_API_TOKEN,
  // basePath defaults to https://app.corellium.com/api; set it for enterprise/on-prem
});

const instances = await client.instances.list();
console.log(instances);
```

## Authentication

- `accessToken` — a Corellium API token or JWT bearer. It may be a string or a
  (possibly async) function, which is re-evaluated per request (handy for refresh):

```ts
new CorelliumClient({ accessToken: () => tokenStore.get() });
```

- No token is required for public endpoints (health check, login, auth-provider
  metadata). If you only have credentials, log in first to obtain a token — note
  that `username` holds your account email:

```ts
const anon = new CorelliumClient({});
const { token } = await anon.auth.login({ username: 'me@example.com', password: '...' });
const client = new CorelliumClient({ accessToken: token });
```

## Namespaces & versioning

Operations are grouped into namespaces: `client.instances.*`, `client.projects.*`,
`client.snapshots.*`, and so on.

- Unversioned paths (`client.instances.list`) resolve to the **latest** version of
  each operation — a per-operation blend.
- Pinned paths (`client.v1.*`, `client.v2.*`, `client.v3.*`) bind to a specific
  version forever, for a contract that never moves.

```ts
await client.instances.list();    // latest
await client.v3.instances.list(); // pinned to v3
```

## Health check (no auth)

```ts
const client = new CorelliumClient({});
await client.status.ready(); // resolves when the API is ready, throws otherwise
```

## Upload progress and cancellation

Browser uploads can opt into progress reporting through the second per-call argument:

```ts
const controller = new AbortController();

const upload = client.agent.files.upload(
  { instanceId, filePath, body: file },
  {
    signal: controller.signal,
    onUploadProgress: ({ loaded, total, percent }) => {
      console.log({ loaded, total, percent });
    },
  },
);

// Reject the upload with AbortError:
const cancelUpload = () => controller.abort();
await upload;
```

Upload progress uses `XMLHttpRequest` because browser Fetch does not expose upload
progress. `total` and `percent` are omitted when the browser cannot determine the
upload size, and callbacks are transport events rather than guaranteed 0%/100% markers.
Calls without `onUploadProgress` continue to use Fetch. Progress requires a browser-like
XMLHttpRequest environment and cannot be combined with a configured custom `fetchApi`.
