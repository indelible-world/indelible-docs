---
title: Verify
icon: lucide/shield-check
---

# `indelible/verify`

Read-only functions for fetching and verifying attestations and quote proofs. None of these functions submit transactions.

```js
import { verifyCid, verifyQuoteProof, getAttestationByIndex } from 'indelible/verify';
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
| `proofData` | `object` | A parsed [quote proof JSON](../standard/taanq/quote-verification.md#the-proof-file) object. |

**Returns:** `Promise<{ verification: VerificationResult, quoteText: string, allProofsValid: boolean }>`

| Return field | Type | Description |
|---|---|---|
| `verification` | `VerificationResult` | The result of verifying the attestation referenced in the proof. |
| `quoteText` | `string` | The full quote text reconstructed by joining the ordered proof chunks. |
| `allProofsValid` | `boolean` | `true` if every Merkle proof in the file validates against the on-chain `qvHash`. |

```js
import { verifyQuoteProof } from 'indelible/verify';

const proofData = JSON.parse(await file.text());
const { verification, quoteText, allProofsValid } = await verifyQuoteProof(publicClient, proofData);

if (allProofsValid && verification.code === RESULT_CODE.VERIFIED) {
    console.log('Verified quote:', quoteText);
}
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
```

| Property | Type | Description |
|---|---|---|
| `ipfsHash` | `string` | `bytes32` hex representation of the content's IPFS CID. |
| `qvHash` | `string` | Merkle root used for [quote verification](../standard/taanq/quote-verification.md). Zero (`0x000...`) if not used. |
| `parentIpfsHash` | `string` | CID of a prior version when this attestation represents an edit. Zero if not an edit. |
| `authority` | `string` | The Ethereum address credited with authorship. |
| `timestamp` | `bigint` | Unix timestamp (seconds) of the block in which the attestation was created. |
| `revokedAt` | `bigint` | Unix timestamp of revocation, or `0n` if the attestation is still active. |
