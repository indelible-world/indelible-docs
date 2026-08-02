---
title: Errors
icon: lucide/triangle-alert
---
# Errors

Errors are returned as JSON with a descriptive `error` field and an appropriate HTTP status code.

```json
{ "error": "invalid or missing API key" }
```

## HTTP Status Codes

| Status | Meaning |
|---|---|
| `400` | The request body failed validation, or no `chainId` could be resolved. |
| `401` | The API key is missing, invalid, or revoked. |
| `404` | The requested attestation doesn't exist, or doesn't belong to your organization. |
| `409` | Your organization has no active, verified delegation for the requested chain. |
| `202` / `200` | Success — see [Attestations](attestations.md). |

## Validation Errors (`400`)

Returned when the request body doesn't match the expected shape:

```json
{
  "error": "invalid request",
  "issues": [
    { "path": ["url"], "message": "Invalid url" }
  ]
}
```

`issues` follows [Zod](https://zod.dev)'s issue format — check `path` for the offending field.

A `400` is also returned if `chainId` is omitted and your organization has no default chain configured:

```json
{ "error": "no chainId given and no default configured" }
```

## No Active Delegation (`409`)

```json
{ "error": "no active delegation for chain 11155111", "chainId": 11155111 }
```

Every attestation is submitted by a platform key acting as your authority's delegate. This error means your organization hasn't yet completed the on-chain [delegation](../standard/taanq/delegations.md) ceremony for the requested chain, or the delegation was revoked. Contact your Indelible representative to complete onboarding for that chain - see [Before You Start](index.md#before-you-start).

## Attestation Failures (`status: "failed"`)

Some failures happen asynchronously, after a `202` has already been returned — the transaction is submitted but reverts on-chain. These surface as `status: "failed"` with a message in `error` when you poll [`GET /v1/attestations/:id`](attestations.md#get-an-attestation), rather than as an HTTP error response. Common causes:

- The delegation was revoked on-chain between request time and submission.
- Internal server errors
- Fluctuating gas fees
- The commit/reveal delay window was violated (rare — the worker enforces the ~60 second wait automatically).

Failed attestations are not retried automatically. Submit a new request with the same `content` to try again once the underlying issue (e.g. delegation) is resolved.
