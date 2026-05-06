---
title: Utilities & Constants
icon: lucide/wrench
---

# `indelible/utils` & `indelible/constants`

Off-chain cryptographic utilities and protocol constants used throughout the library. These functions have no network dependencies and run synchronously unless otherwise noted.

---

## Constants — `indelible/constants`

```js
import {
    TAANQ_ADDRESS,
    ENS_INDELIBLE_ADDRESS,
    ENS_REGISTRY_ADDRESS,
    MERKLE_SPLIT,
    RESULT_CODE,
} from 'indelible/constants';
```

### Contract addresses

| Constant | Type | Description |
|---|---|---|
| `TAANQ_ADDRESS` | `string` | Deployed address of the `IndelibleTAANQ` contract. |
| `ENS_INDELIBLE_ADDRESS` | `string` | Deployed address of the Indelible ENS binding contract. |
| `ENS_REGISTRY_ADDRESS` | `string` | Address of the canonical ENS registry. |

Pass these directly to viem's `readContract` / `writeContract` when building custom integrations on top of the ABIs.

---

### `MERKLE_SPLIT`

```js
MERKLE_SPLIT // 46
```

The fixed character length of each chunk when splitting text for a Merkle quote proof tree. All chunks except the last will be exactly `MERKLE_SPLIT` characters. The `buildTree` utility uses this value automatically.

---

### `RESULT_CODE`

```js
RESULT_CODE.NOT_FOUND   // No attestation found for the given CID
RESULT_CODE.VERIFIED    // Attestation found and is not revoked
RESULT_CODE.UNVERIFIED  // Content hash does not match any attestation
RESULT_CODE.REVOKED     // Attestation was found but has been revoked
RESULT_CODE.WARNING     // Attestation found but requires additional scrutiny
```

Compare `VerificationResult.code` against these values for machine-readable status checks:

```js
import { RESULT_CODE } from 'indelible/constants';
import { verifyCid } from 'indelible/verify';

const result = await verifyCid(publicClient, cid);

if (result.code === RESULT_CODE.VERIFIED) {
    // Show a verified badge
} else if (result.code === RESULT_CODE.REVOKED) {
    // Warn the user
}
```

---

### `RESULT_CODE_INFO`

Metadata object for each result code: a human-readable label, a CSS class hint, and a numeric priority.

```js
import { RESULT_CODE_INFO } from 'indelible/constants';

RESULT_CODE_INFO[RESULT_CODE.VERIFIED]
// { label: 'Verified', cssClass: 'result-verified', priority: 4 }

RESULT_CODE_INFO[RESULT_CODE.REVOKED]
// { label: 'Revoked', cssClass: 'result-revoked', priority: 1 }
```

| Property | Type | Description |
|---|---|---|
| `label` | `string` | Short human-readable status name suitable for display. |
| `cssClass` | `string` | CSS class hint (e.g. `'result-verified'`) for styling result badges. |
| `priority` | `number` | Lower number = higher urgency. Used to pick the most important code when a result carries multiple codes. |

Priority values: `UNVERIFIED` (0) > `REVOKED` (1) > `NOT_FOUND` (2) > `WARNING` (3) > `VERIFIED` (4).

---

### `RESULT_CODE_PRIORITY`

The result codes sorted by ascending priority — i.e. most important first. Derived automatically from `RESULT_CODE_INFO`.

```js
import { RESULT_CODE_PRIORITY } from 'indelible/constants';

RESULT_CODE_PRIORITY // [2, 3, 0, 4, 1] — [UNVERIFIED, REVOKED, NOT_FOUND, WARNING, VERIFIED]
```

**Type:** `number[]`

Use this array together with `getPrimaryResultCode` when you need to reduce a multi-code result to a single display state.

---

### `getPrimaryResultCode(codes)`

Picks the highest-priority (most important) result code from an array of codes.

```js
import { getPrimaryResultCode, RESULT_CODE } from 'indelible/constants';

getPrimaryResultCode([RESULT_CODE.VERIFIED, RESULT_CODE.WARNING]);
// RESULT_CODE.WARNING  (warning outranks verified)

getPrimaryResultCode([RESULT_CODE.UNVERIFIED, RESULT_CODE.REVOKED]);
// RESULT_CODE.UNVERIFIED  (unverified has highest priority)
```

| Parameter | Type | Description |
|---|---|---|
| `codes` | `number[]` | An array of `RESULT_CODE` values, e.g. from a multi-attestation result. |

**Returns:** `number | undefined` — the highest-priority code, or `undefined` if the array is empty.

---

### `getResultCodeCssClass(code)`

Returns the CSS class string for a given result code.

```js
import { getResultCodeCssClass, RESULT_CODE } from 'indelible/constants';

getResultCodeCssClass(RESULT_CODE.VERIFIED);   // 'result-verified'
getResultCodeCssClass(RESULT_CODE.REVOKED);    // 'result-revoked'
getResultCodeCssClass(RESULT_CODE.NOT_FOUND);  // 'result-not-found'
```

| Parameter | Type | Description |
|---|---|---|
| `code` | `number` | A `RESULT_CODE` value. |

**Returns:** `string | undefined`

Equivalent to `RESULT_CODE_INFO[code]?.cssClass`. Useful when applying dynamic classes to result badge elements.

---

## Utilities — `indelible/utils`

```js
import {
    hashContent,
    hexHashContent,
    createRawCIDv1,
    getCIDFromHash,
    getCIDFromRawDigest,
    decodeCidToIpfsHash,
    buildTree,
    dnsEncodeName,
    prettifyTimestamp,
    downloadJson,
} from 'indelible/utils';
```

---

### Hashing

#### `hashContent(data)`

Returns the SHA-256 digest of the input as a `Uint8Array`.

```js
const digest = hashContent('Hello, world.');
// Uint8Array(32) [ ... ]
```

| Parameter | Type | Description |
|---|---|---|
| `data` | `string \| Uint8Array` | The content to hash. |

**Returns:** `Uint8Array`

---

#### `hexHashContent(data)`

Returns the SHA-256 digest of the input as a lowercase hex string (no `0x` prefix).

```js
const hex = hexHashContent('Hello, world.');
// 'b94d27b9934d3e08a52e52d7da7dabfac484efe04294e576...'
```

| Parameter | Type | Description |
|---|---|---|
| `data` | `string \| Uint8Array` | The content to hash. |

**Returns:** `string`

---

### CID encoding

#### `createRawCIDv1(data)`

Computes the base32-encoded CIDv1 of the given content — the same identifier that IPFS would assign if you added the content with raw (`raw`) codec.

```js
const cid = createRawCIDv1('Hello, world.');
// 'bafkrei...'
```

| Parameter | Type | Description |
|---|---|---|
| `data` | `string \| Uint8Array` | The content to create a CID for. |

**Returns:** `string`

This is the CID you pass to `commitAttestation` (via the `content` parameter, which the library hashes internally) and to all verification lookups.

---

#### `getCIDFromHash(hash)`

Converts a SHA-256 hex string into a base32 CIDv1 string.

```js
const cid = getCIDFromHash(hexHash);
```

| Parameter | Type | Description |
|---|---|---|
| `hash` | `string` | A 64-character lowercase hex SHA-256 digest. |

**Returns:** `string`

---

#### `getCIDFromRawDigest(bytes)`

Converts a raw 32-byte SHA-256 digest into a base32 CIDv1 string.

```js
const cid = getCIDFromRawDigest(digest);
```

| Parameter | Type | Description |
|---|---|---|
| `bytes` | `Uint8Array` | A 32-byte SHA-256 digest. |

**Returns:** `string`

---

#### `decodeCidToIpfsHash(cidStr)`

Extracts the `bytes32` hex value from a CIDv1 string. This is the format the TAANQ contract stores and expects for `ipfsHash`.

```js
const ipfsHash = decodeCidToIpfsHash('bafkrei...');
// '0x...' — 32-byte hex string
```

| Parameter | Type | Description |
|---|---|---|
| `cidStr` | `string` | A base32 CIDv1 string. |

**Returns:** `string` — a `0x`-prefixed 32-byte hex string.

Use this when calling viem's `readContract` or `writeContract` directly with the [contract ABIs](index.md#modules) and you need to convert a human-readable CID into the on-chain format.

---

### Merkle trees

#### `buildTree(text)`

Splits `text` into fixed-size chunks of [`MERKLE_SPLIT`](#merkle_split) characters and returns an OpenZeppelin `StandardMerkleTree` instance. The tree's root is used as the `qvHash` in an attestation.

```js
const tree = buildTree(articleText);
const qvHash = tree.root; // bytes32 hex — store in the attestation
```

| Parameter | Type | Description |
|---|---|---|
| `text` | `string` | The full text to build a Merkle tree from. |

**Returns:** `StandardMerkleTree` (from `@openzeppelin/merkle-tree`)

Each leaf is a `['string', 'string']` tuple of `[index, chunkText]`. This matches the format expected by `verifyQuoteProof` and the [quote verification](../standard/taanq/quote-verification.md) protocol.

---

### ENS helpers

#### `dnsEncodeName(name)`

Encodes an ENS name into DNS wire format (length-prefixed labels). Required when calling the ENS registry and resolver contracts directly.

```js
const encoded = dnsEncodeName('alice.eth');
// Uint8Array [ 5, 97, 108, 105, 99, 101, 3, 101, 116, 104, 0 ]
```

| Parameter | Type | Description |
|---|---|---|
| `name` | `string` | A dot-separated ENS name (e.g. `"alice.eth"`). |

**Returns:** `Uint8Array`

---

### Display helpers

#### `prettifyTimestamp(ts)`

Formats a Unix timestamp (seconds) into a human-readable local date/time string.

```js
prettifyTimestamp(1714000000n);
// e.g. 'April 25, 2024, 9:46 AM'
```

| Parameter | Type | Description |
|---|---|---|
| `ts` | `number \| bigint` | A Unix timestamp in seconds, as returned by the contract. |

**Returns:** `string`

Formatting follows the runtime's locale. Suitable for display in UIs but not for machine processing.

---

#### `downloadJson(data, filename)`

Triggers a browser file download for a JSON-serializable object. **Browser only** — throws in Node.js environments.

```js
downloadJson(proofJson, 'quote-proof.json');
```

| Parameter | Type | Description |
|---|---|---|
| `data` | `object` | The object to serialize as JSON. |
| `filename` | `string` | The suggested download filename. |

Also exported from `indelible/publish` for convenience.
