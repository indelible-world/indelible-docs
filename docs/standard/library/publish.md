---
title: Publish
icon: lucide/send
---

# `indelible/publish`

Functions for writing to the TAANQ and ENS contracts, plus off-chain operations like quote proof generation and JSON download.

```js
import { commitAttestation, revealAttestation, proveQuote } from 'indelible/publish';
```

All write functions require both a `walletClient` (for signing and sending transactions) and a `publicClient` (for reading contract state). See the [setup guide](index.md#setting-up-clients).

---

## Attestations

### `commitAttestation`

Submits the first step of the [commit/reveal](../standard/taanq/commit-reveal.md) flow. Computes a salted hash of the content and sends a `commit()` transaction to the contract.

```js
const { pendingCommit } = await commitAttestation({
    walletClient,
    publicClient,
    content,
    account,
    authority?,
    parentIpfsHash?,
});
```

| Parameter | Type | Description |
|---|---|---|
| `walletClient` | `WalletClient` | A viem wallet client with a connected account. |
| `publicClient` | `PublicClient` | A viem public client. |
| `content` | `string` | The raw text content to attest. The library computes the IPFS CID and `qvHash` from this string. |
| `account` | `Account` | The viem account that will sign the transaction. |
| `authority` | `string` _(optional)_ | The authority address to credit. Defaults to `account.address`. Set this when using a [delegate](../standard/taanq/delegations.md) wallet. |
| `parentIpfsHash` | `string` _(optional)_ | The IPFS CID of a prior version if this attestation is an edit. |

**Returns:** `Promise<{ pendingCommit: PendingCommit }>`

!!! warning "Persist `pendingCommit`"
    The `pendingCommit` object contains everything needed to complete the reveal. Store it durably (e.g. `localStorage`) before the page unloads — if it is lost, the commit cannot be revealed and the gas spent is wasted.

    ```js
    localStorage.setItem('pendingCommit', JSON.stringify(pendingCommit));
    ```

---

### `revealAttestation`

Submits the second step of the commit/reveal flow. Must be called at least **60 seconds** after the commit transaction is mined.

```js
const { attestationIndex } = await revealAttestation({
    walletClient,
    publicClient,
    pendingCommit,
    account,
});
```

| Parameter | Type | Description |
|---|---|---|
| `walletClient` | `WalletClient` | A viem wallet client. |
| `publicClient` | `PublicClient` | A viem public client. |
| `pendingCommit` | `PendingCommit` | The object returned by `commitAttestation`. |
| `account` | `Account` | The viem account that signed the commit. Must match the address used during commit. |

**Returns:** `Promise<{ attestationIndex: bigint }>`

The returned `attestationIndex` is the permanent on-chain index of the new attestation. Store it alongside the IPFS CID and authority address to form a complete [attestation reference](../standard/index.md#attestations).

---

### `revokeAttestation`

Revokes an existing attestation. Only callable by the attestation's authority address or an active delegate, and only within the 7-day revocation window.

```js
await revokeAttestation({
    walletClient,
    publicClient,
    attestationId,
    account,
});
```

| Parameter | Type | Description |
|---|---|---|
| `walletClient` | `WalletClient` | A viem wallet client. |
| `publicClient` | `PublicClient` | A viem public client. |
| `attestationId` | `number \| bigint` | The on-chain index of the attestation to revoke. |
| `account` | `Account` | The authority or delegate account. |

**Returns:** `Promise<void>`

---

### `getExistingAttestationIndex`

Checks whether an attestation already exists for a given IPFS hash and authority. Returns the attestation index, or `0n` if none exists. Useful before committing to avoid a duplicate.

```js
const index = await getExistingAttestationIndex({ publicClient, ipfsHash, authority });
```

| Parameter | Type | Description |
|---|---|---|
| `publicClient` | `PublicClient` | A viem public client. |
| `ipfsHash` | `string` | A `bytes32` hex IPFS hash (use [`decodeCidToIpfsHash`](utils.md#decodecidtoipfshash) to convert from a CID string). |
| `authority` | `string` | The Ethereum address of the authority. |

**Returns:** `Promise<bigint>`

---

## Delegations

### `delegate`

Authorizes a delegate address to create and revoke attestations on behalf of `account`.

```js
await delegate({
    walletClient,
    publicClient,
    delegateAddress,
    account,
});
```

| Parameter | Type | Description |
|---|---|---|
| `walletClient` | `WalletClient` | A viem wallet client. |
| `publicClient` | `PublicClient` | A viem public client. |
| `delegateAddress` | `string` | The Ethereum address to authorize as delegate. |
| `account` | `Account` | The authority account setting the delegation. |

**Returns:** `Promise<void>`

Calling this when a delegation already exists overwrites the previous delegate immediately. See [Delegations](../standard/taanq/delegations.md) for the full model.

---

### `revokeDelegation`

Removes the caller's active delegation.

```js
await revokeDelegation({ walletClient, publicClient, account });
```

| Parameter | Type | Description |
|---|---|---|
| `walletClient` | `WalletClient` | A viem wallet client. |
| `publicClient` | `PublicClient` | A viem public client. |
| `account` | `Account` | The authority account whose delegation will be revoked. |

**Returns:** `Promise<void>`

---

### `getDelegation`

Reads the current delegation for an authority address.

```js
const delegation = await getDelegation({ publicClient, authority });
// { delegateAddress: '0x...', timestamp: 1234567890n }
// or { delegateAddress: '0x000...', timestamp: 0n } if no delegation exists
```

| Parameter | Type | Description |
|---|---|---|
| `publicClient` | `PublicClient` | A viem public client. |
| `authority` | `string` | The Ethereum address of the authority. |

**Returns:** `Promise<{ delegateAddress: string, timestamp: bigint }>`

A `delegateAddress` of `0x0000000000000000000000000000000000000000` (the zero address) means no delegation is currently set.

---

## Quote proofs

### `proveQuote`

Generates a [quote proof JSON](../standard/taanq/quote-verification.md#the-proof-file) for a passage of text quoted from an attested article. The proof can be embedded in HTML or distributed as a standalone file.

```js
const { proofJson, onChain } = await proveQuote({
    articleText,
    quote,
    authority,
    publicClient?,
    chainId?,
});
```

| Parameter | Type | Description |
|---|---|---|
| `articleText` | `string` | The full text of the source article (the document being quoted from). |
| `quote` | `string` | The exact passage being quoted. Must appear within `articleText`. |
| `authority` | `string` | The Ethereum address that attested the source article. |
| `publicClient` | `PublicClient` _(optional)_ | When provided, the library looks up the on-chain attestation and includes `chainId` and `attestationIndex` in the proof. |
| `chainId` | `number` _(optional)_ | Explicit chain ID. Used when `publicClient` is not provided. |

**Returns:** `Promise<{ proofJson: object, onChain: boolean }>`

| Return field | Type | Description |
|---|---|---|
| `proofJson` | `object` | The complete proof JSON ready to serialize and distribute. |
| `onChain` | `boolean` | `true` if an on-chain attestation was found and embedded in the proof. |

```js
import { proveQuote, downloadJson } from 'indelible/publish';

const { proofJson } = await proveQuote({
    articleText,
    quote,
    authority: '0xD2f2...',
    publicClient,
});

// In the browser — triggers a file download
downloadJson(proofJson, 'quote-proof.json');
```

---

## ENS

### `registerEnsBinding`

Registers a binding between an ENS name and an Ethereum address on the Indelible ENS contract.

```js
await registerEnsBinding({
    walletClient,
    publicClient,
    ensName,
    account,
});
```

| Parameter | Type | Description |
|---|---|---|
| `walletClient` | `WalletClient` | A viem wallet client. |
| `publicClient` | `PublicClient` | A viem public client. |
| `ensName` | `string` | The ENS name to bind (e.g. `"alice.eth"`). |
| `account` | `Account` | The account registering the binding. Must own or control the ENS name. |

**Returns:** `Promise<void>`

See [ENS Integration](../standard/integrations/ens.md) for the full integration model.

---

## Helpers

### `generateSalt`

Generates a cryptographically random 32-byte salt suitable for use in the commit/reveal flow. Called automatically by `commitAttestation`; exposed for advanced use cases.

```js
const salt = generateSalt();
// Uint8Array(32) [ ... ]
```

---

### `buildSaltedHash`

Computes the `saltedHash` value required for the `commit()` contract call.

```js
const saltedHash = buildSaltedHash(ipfsHash, address, salt);
```

| Parameter | Type | Description |
|---|---|---|
| `ipfsHash` | `string` | The `bytes32` hex IPFS hash of the content. |
| `address` | `string` | The Ethereum address of the sender (`msg.sender` in the contract). |
| `salt` | `Uint8Array` | A 32-byte random salt, e.g. from `generateSalt()`. |

**Returns:** `string` — a `bytes32` hex hash.

---

### `downloadJson`

Triggers a browser file download for a JSON object. Browser-only convenience; in Node.js, use `JSON.stringify()` and write to disk directly.

```js
downloadJson(proofJson, 'quote-proof.json');
```

| Parameter | Type | Description |
|---|---|---|
| `data` | `object` | The data to serialize as JSON and download. |
| `filename` | `string` | The suggested filename for the download. |
