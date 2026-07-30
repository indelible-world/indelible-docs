---
title: Verify
icon: lucide/shield-check
---

# `indelible/verify`

Read-only functions for fetching and verifying attestations and quote proofs. None of these functions submit transactions.

```js
import { verifyCid, verifyQuoteProof, quoteMatchesProvenText, getAttestationByIndex } from 'indelible/verify';
```

---

## `verifyCid`

Looks up all attestations for a given IPFS CID and returns a summary of the verification result.

```js
const result = await verifyCid(publicClient, cid, authority?);
```

| Parameter | Type | Description |
|---|---|---|
| `publicClient` | `PublicClient` | A viem public client connected to the correct chain. |
| `cid` | `string` | The IPFS CIDv1 string to verify (e.g. `"bafkrei..."`). |
| `authority` | `string` _(optional)_ | An Ethereum address. When provided, the lookup is narrowed to attestations by that specific authority. |

**Returns:** `Promise<VerificationResult>`

When `authority` is omitted, the function finds all attestations for the CID and returns the most relevant result. Providing an authority address is more precise and avoids ambiguity when multiple parties have attested the same content.

---

## `VerificationResult`

The object returned by `verifyCid`.

```js
result.code       // RESULT_CODE value — the machine-readable status
result.headline   // string — short human-readable status (e.g. "Verified")
result.details    // string — longer explanation suitable for display
result.attestation // Attestation | null — the underlying attestation, if found
```

| Property | Type | Description |
|---|---|---|
| `code` | `RESULT_CODE` | One of `NOT_FOUND`, `VERIFIED`, `UNVERIFIED`, `REVOKED`, or `WARNING`. See [Constants](utils.md#result_code). |
| `headline` | `string` | A short label describing the result. |
| `details` | `string` | A human-readable explanation of the result. |
| `attestation` | `Attestation \| null` | The attestation associated with this result, or `null` if none was found. |

---

## `verifyQuoteProof`

Verifies a quote proof JSON file against the on-chain `qvHash` of the corresponding attestation.

```js
const { verification, quoteText, allProofsValid } = await verifyQuoteProof(publicClient, proofData);
```

| Parameter | Type | Description |
|---|---|---|
| `publicClient` | `PublicClient` | A viem public client. |
| `proofData` | `object` | A parsed [quote proof JSON](../taanq/quote-verification.md#the-proof-file) object. |
| `opts.quote` | `string` _(optional)_ | The quote as displayed to the user. When provided, the result includes `quoteMatches`, indicating whether this quote is covered by the proven text. |
| `opts.mode` | `'hard' \| 'soft'` _(optional)_ | Controls how `opts.quote` is matched against the proven text. Defaults to `'hard'`. See [`quoteMatchesProvenText`](#quotematchesproventext). |

**Returns:** `Promise<{ verification: VerificationResult, quoteText: string, allProofsValid: boolean, quoteMatches?: boolean }>`

| Return field | Type | Description |
|---|---|---|
| `verification` | `VerificationResult` | The result of verifying the attestation referenced in the proof. |
| `quoteText` | `string` | The full quote text reconstructed by joining the ordered proof chunks. |
| `allProofsValid` | `boolean` | `true` if every Merkle proof in the file validates against the on-chain `qvHash`. |
| `quoteMatches` | `boolean` _(only present if `opts.quote` was given)_ | Whether the displayed quote is covered by `quoteText`, per `opts.mode`. |

```js
import { verifyQuoteProof } from 'indelible/verify';

const proofData = JSON.parse(await file.text());
const { verification, quoteText, allProofsValid } = await verifyQuoteProof(publicClient, proofData);

if (allProofsValid && verification.code === RESULT_CODE.VERIFIED) {
    console.log('Verified quote:', quoteText);
}

// Also check a displayed (possibly segmented) quote against the proven text:
const { quoteMatches } = await verifyQuoteProof(publicClient, proofData, {
    quote: 'The revolution ... televised.',
    mode: 'soft',
});
```

---

## `quoteMatchesProvenText`

Checks whether a displayed quote is covered by proven text (the `quoteText` returned by `verifyQuoteProof`). Used internally by `verifyQuoteProof` when `opts.quote` is passed, and can also be called directly.

```js
const matches = quoteMatchesProvenText(quote, provenText, { mode? });
```

| Parameter | Type | Description |
|---|---|---|
| `quote` | `string` | The quote as displayed. May contain ellipses (`...`, `…`) or bracketed insertions (`[sic]`) when `mode` is `'soft'`. |
| `provenText` | `string` | The Merkle-proven text, e.g. `quoteText` from `verifyQuoteProof`. |
| `opts.mode` | `'hard' \| 'soft'` _(optional)_ | `'hard'` (default) requires `quote` to appear verbatim as a contiguous substring of `provenText` — this is the original behavior, with no breaking changes. `'soft'` splits `quote` on ellipsis/bracket gap markers and checks that each literal segment appears in `provenText`, **in the same order**. Text inside brackets is ignored entirely. |

**Returns:** `boolean`

```js
import { quoteMatchesProvenText } from 'indelible/verify';

quoteMatchesProvenText('The revolution will not be televised.', provenText);
// true — verbatim substring, works in either mode

quoteMatchesProvenText('The revolution ... televised.', provenText);
// false in 'hard' mode (default) — not a verbatim substring

quoteMatchesProvenText('The revolution ... televised.', provenText, { mode: 'soft' });
// true — both segments found in provenText, in order
```

---

## `getAttestationByIndex`

Fetches a single attestation by its on-chain index.

```js
const attestation = await getAttestationByIndex(publicClient, index);
```

| Parameter | Type | Description |
|---|---|---|
| `publicClient` | `PublicClient` | A viem public client. |
| `index` | `number \| bigint` | The attestation's index in the `attestations` array on the contract. |

**Returns:** `Promise<Attestation>`

Returns index `0` to check for "not found" — the contract pre-seeds the array with a placeholder at index `0`, so a real attestation always has an index ≥ 1.

---

## `cidToAttestationIndices`

Returns all attestation indices for a given IPFS CID, across all authorities.

```js
const indices = await cidToAttestationIndices(publicClient, cid);
// e.g. [1n, 7n, 42n]
```

| Parameter | Type | Description |
|---|---|---|
| `publicClient` | `PublicClient` | A viem public client. |
| `cid` | `string` | An IPFS CIDv1 string. |

**Returns:** `Promise<bigint[]>`

---

## `cidAndAddressToAttestationIndices`

Returns the single attestation index for a specific CID + authority pair. Returns `0n` if no attestation exists.

```js
const index = await cidAndAddressToAttestationIndices(publicClient, cid, authority);
```

| Parameter | Type | Description |
|---|---|---|
| `publicClient` | `PublicClient` | A viem public client. |
| `cid` | `string` | An IPFS CIDv1 string. |
| `authority` | `string` | An Ethereum address. |

**Returns:** `Promise<bigint>`

---

## `Attestation`

The raw attestation data returned from the contract, decoded into a JavaScript object.

```js
attestation.ipfsHash        // string — bytes32 hex of the IPFS CID
attestation.qvHash          // string — bytes32 hex Merkle root (zero if unused)
attestation.parentIpfsHash  // string — bytes32 hex of prior version CID (zero if none)
attestation.authority       // string — Ethereum address of the attesting authority
attestation.timestamp       // bigint — block.timestamp when the attestation was created
attestation.revokedAt       // bigint — block.timestamp of revocation; 0n if not revoked
attestation.childIpfsHash   // string — bytes32 hex of the newer version CID (zero if none)
```

| Property | Type | Description |
|---|---|---|
| `ipfsHash` | `string` | `bytes32` hex representation of the content's IPFS CID. |
| `qvHash` | `string` | Merkle root used for [quote verification](../taanq/quote-verification.md). Zero (`0x000...`) if not used. |
| `parentIpfsHash` | `string` | CID of a prior version when this attestation represents an edit. Zero if not an edit. |
| `authority` | `string` | The Ethereum address credited with authorship. |
| `timestamp` | `bigint` | Unix timestamp (seconds) of the block in which the attestation was created. |
| `revokedAt` | `bigint` | Unix timestamp of revocation, or `0n` if the attestation is still active. |
| `childIpfsHash` | `string` | `bytes32` hex CID of a newer version of the content, set by the publisher via [`setChildIpfsHash`](publish.md#setchildipfshash). Zero (`0x000...`) if no newer version has been signaled. |
