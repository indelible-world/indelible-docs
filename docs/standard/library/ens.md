---
title: ENS
icon: simple/ens
---

# `indelible/ens`

Read-only functions for querying ENS name bindings stored on the Indelible ENS contract. None of these functions submit transactions — use [`indelible/publish`](publish.md#registerensbinding) for write operations.

```js
import {
    getBindingsByAddress,
    getBindingByName,
    getBindingByNode,
    resolveIndelibleAddress,
    decodeDnsName,
    EnsVerification,
} from 'indelible/ens';
```

See [ENS Integration](../integrations/ens.md) for a description of how bindings work on-chain.

---

## `EnsVerification`

A JavaScript class representing a single ENS-to-address verification record stored on-chain.

```js
verification.authority        // `0x…` address — the Indelible authority bound to this name
verification.node             // `0x…` bytes32 — ENS namehash of the bound name
verification.dnsName          // Uint8Array | `0x…` — DNS-wire-format encoded name
verification.startTimestamp   // number — Unix timestamp (seconds) when the binding became active
verification.endTimestamp     // number — Unix timestamp when revoked; 0 if still active
verification.index            // number — position in the on-chain `verifications` array
```

| Property | Type | Description |
|---|---|---|
| `authority` | `` `0x${string}` `` | Ethereum address that owns this binding. |
| `node` | `` `0x${string}` `` | ENS namehash of the bound name. |
| `dnsName` | `Uint8Array \| string` | DNS-wire-format encoded name as stored on-chain. |
| `startTimestamp` | `number` | Unix timestamp (seconds) when the binding became active. |
| `endTimestamp` | `number` | Unix timestamp when revoked, or `0` if the binding is still active. |
| `index` | `number` | Position in the on-chain `verifications` array. |

### Computed properties

**`verification.isActive`** — `boolean`

`true` if the binding is currently active (i.e. `startTimestamp` has passed and `endTimestamp` is either `0` or in the future).

**`verification.name`** — `string`

The human-readable ENS name decoded from `dnsName` (e.g. `"alice.eth"`). Returns `"(unknown)"` if the bytes cannot be decoded.

### `verification.isActiveAt(timestamp)`

Returns `true` if the binding was active at the given Unix timestamp (seconds). Useful for time-based verification of historical records.

```js
const wasActive = verification.isActiveAt(Math.floor(someDate / 1000));
```

---

## `getBindingsByAddress`

Fetches all verification records ever bound to an Ethereum address, including revoked ones.

```js
const bindings = await getBindingsByAddress(publicClient, address, opts?);
```

| Parameter | Type | Description |
|---|---|---|
| `publicClient` | `PublicClient` | A viem public client connected to the correct chain. |
| `address` | `` `0x${string}` `` | The Ethereum address to look up. |
| `opts.ensIndelibleAddress` | `` `0x${string}` `` _(optional)_ | Override the default `ENS_INDELIBLE_ADDRESS` contract address. |

**Returns:** `Promise<EnsVerification[]>`

The returned array may contain both active and revoked bindings. Filter by `binding.isActive` to get only current bindings.

```js
import { getBindingsByAddress } from 'indelible/ens';

const bindings = await getBindingsByAddress(publicClient, '0xabc...');
const active = bindings.filter(b => b.isActive);
console.log(active.map(b => b.name));
// ['alice.eth']
```

---

## `getBindingByName`

Fetches the verification record for a given ENS name. The name is normalised before hashing.

```js
const binding = await getBindingByName(publicClient, ensName, opts?);
```

| Parameter | Type | Description |
|---|---|---|
| `publicClient` | `PublicClient` | A viem public client. |
| `ensName` | `string` | Human-readable ENS name (e.g. `"alice.eth"`). |
| `opts.ensIndelibleAddress` | `` `0x${string}` `` _(optional)_ | Override the default contract address. |

**Returns:** `Promise<EnsVerification | null>`

Returns `null` if no binding exists for the name.

```js
import { getBindingByName } from 'indelible/ens';

const binding = await getBindingByName(publicClient, 'alice.eth');
if (binding?.isActive) {
    console.log('Bound to:', binding.authority);
}
```

---

## `getBindingByNode`

Fetches the verification record for a pre-computed ENS namehash. Use this when you already have the node and want to skip the normalisation + hashing step.

```js
const binding = await getBindingByNode(publicClient, node, opts?);
```

| Parameter | Type | Description |
|---|---|---|
| `publicClient` | `PublicClient` | A viem public client. |
| `node` | `` `0x${string}` `` | ENS namehash (bytes32 hex). |
| `opts.ensIndelibleAddress` | `` `0x${string}` `` _(optional)_ | Override the default contract address. |

**Returns:** `Promise<EnsVerification | null>`

Returns `null` if no binding exists for the node.

---

## `resolveIndelibleAddress`

Resolves the Ethereum address that an ENS node's `indelible-address` text record points to, as returned directly by the Indelible ENS contract.

```js
const address = await resolveIndelibleAddress(publicClient, node, opts?);
```

| Parameter | Type | Description |
|---|---|---|
| `publicClient` | `PublicClient` | A viem public client. |
| `node` | `` `0x${string}` `` | ENS namehash (bytes32 hex). |
| `opts.ensIndelibleAddress` | `` `0x${string}` `` _(optional)_ | Override the default contract address. |

**Returns:** `Promise<\`0x${string}\` | null>`

Returns `null` if the address is unresolvable or the zero address.

---

## `decodeDnsName`

Decodes a DNS-wire-format encoded name (as stored on-chain) to a dot-separated string.

```js
const name = decodeDnsName(hexOrBytes);
```

| Parameter | Type | Description |
|---|---|---|
| `hexOrBytes` | `` `0x${string}` \| Uint8Array `` | DNS-wire-format bytes as a hex string or `Uint8Array`. |

**Returns:** `string` — the decoded name (e.g. `"alice.eth"`), or `"(unknown)"` on failure.

This is a pure synchronous utility — no network calls are made.

---

## Low-level contract reads

These functions map directly to individual contract storage reads. Prefer the high-level helpers above unless you need fine-grained control.

### `getVerification`

Reads a single `EnsVerification` record by its index in the on-chain `verifications` array.

```js
const record = await getVerification(publicClient, index, opts?);
```

| Parameter | Type | Description |
|---|---|---|
| `publicClient` | `PublicClient` | A viem public client. |
| `index` | `number \| bigint` | Index in the `verifications` array. |
| `opts.ensIndelibleAddress` | `` `0x${string}` `` _(optional)_ | Override the default contract address. |

**Returns:** `Promise<EnsVerification | null>` — `null` if the call reverts (e.g. out-of-bounds index).

### `getAddrToBindings`

Reads a single entry from the `addrToBindings[address]` array at slot `i`. Returns `0` if empty.

```js
const bindingIndex = await getAddrToBindings(publicClient, address, i, opts?);
```

**Returns:** `Promise<number>`

### `getNodeToBinding`

Reads the current verification index for an ENS node from `nodeToBinding[node]`. Returns `0` if no binding exists.

```js
const bindingIndex = await getNodeToBinding(publicClient, node, opts?);
```

**Returns:** `Promise<number>`
