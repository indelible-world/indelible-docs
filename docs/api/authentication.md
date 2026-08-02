---
title: Authentication
icon: lucide/key-round
---
# Authentication

The Certify API authenticates requests with a bearer API key. Every `/v1/*` route requires one.

```
Authorization: Bearer ind_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Keys are prefixed `ind_live_` and are issued per publisher. The raw key is shown only once and cannot be retrieved again. If it's lost, let us know and we'll issue a new one.

!!! danger "Treat API keys as secrets"
    An API key can create attestations attributed to your organization's authority address. Store it in a secrets manager or environment variable, never in client-side code or version control.

## Example Request

```bash
curl https://api.publishers.indelible.world/v1/attestations \
  -H "Authorization: Bearer ind_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example-news.com/articles/some-story",
    "content": "The full plain-text body of the article..."
  }'
```

## Invalid or Missing Keys

Requests without a valid, unrevoked key receive:

```json
{ "error": "invalid or missing API key" }
```

with HTTP status `401`. See [Errors](errors.md) for the full list of error responses.

## Rotating a Key

Issuing a new key does not automatically revoke old ones - an organization can hold multiple active keys (e.g. one per integration). To retire a key, contact your Indelible representative to have it revoked; revoked keys are rejected immediately regardless of remaining validity.
