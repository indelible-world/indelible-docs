---
title: Chains
icon: lucide/link
---

# `indelible/chains`

Chain registry and viem client construction helpers for verifier UIs. The Indelible Protocol contracts are deployed at the same address on every supported chain, so a verifier typically just needs to pick which chain to connect to — often driven by a `chainId` embedded in a proof file.

```js
import {
    CHAINS,
    DEFAULT_RPC_URLS,
    PUBLIC_RPC_URLS,
    CHAIN_DISPLAY_NAMES,
    DEFAULT_ALCHEMY_KEY,
    getChainKeyById,
    getChainById,
    createIndelibleClient,
} from 'indelible/chains';
```

---

## Constants

### `CHAINS`

Map of chain keys to viem `Chain` objects for every chain where the Indelible contracts are deployed.

```js
CHAINS.ethereum   // mainnet
CHAINS.arbitrum   // Arbitrum One
CHAINS.base       // Base
CHAINS.sepolia    // Sepolia testnet
```

**Type:** `Record<string, Chain>`

---

### `DEFAULT_RPC_URLS`

Default (Alchemy) RPC URL for each supported chain key. These are the primary transports used by `createIndelibleClient`.

```js
DEFAULT_RPC_URLS.ethereum  // 'https://eth-mainnet.g.alchemy.com/v2/...'
DEFAULT_RPC_URLS.base      // 'https://base-mainnet.g.alchemy.com/v2/...'
```

**Type:** `Record<string, string>`

---

### `PUBLIC_RPC_URLS`

Public, CORS-permissive fallback RPC URLs — used as the secondary transport when the default provider is unavailable (e.g. when the page is served from an IPFS gateway).

```js
PUBLIC_RPC_URLS.ethereum  // 'https://cloudflare-eth.com'
PUBLIC_RPC_URLS.base      // 'https://mainnet.base.org'
```

**Type:** `Record<string, string>`

---

### `CHAIN_DISPLAY_NAMES`

Human-readable display names keyed by numeric chain ID.

```js
CHAIN_DISPLAY_NAMES[1]    // 'Ethereum'
CHAIN_DISPLAY_NAMES[8453] // 'Base'
```

**Type:** `Record<number, string>`

---

### `DEFAULT_ALCHEMY_KEY`

The Alchemy API key used to build `DEFAULT_RPC_URLS`. It is restricted to the Indelible contract addresses. Exposed for informational purposes; use a `customRpcUrl` in `createIndelibleClient` if you want to supply your own provider.

**Type:** `string`

---

## Functions

### `getChainKeyById(chainId)`

Looks up the chain key (e.g. `'ethereum'`) for a numeric chain ID.

```js
const chainKey = getChainKeyById(8453);
// 'base'
```

| Parameter | Type | Description |
|---|---|---|
| `chainId` | `number` | The numeric EVM chain ID to look up. |

**Returns:** `string | undefined` — the chain key, or `undefined` if the chain is not supported.

Useful when you have a `chainId` from a proof file and need to pass a chain key to `createIndelibleClient` or populate a chain selector UI.

---

### `getChainById(chainId)`

Looks up the viem `Chain` object for a numeric chain ID.

```js
const chain = getChainById(1);
// mainnet Chain object
```

| Parameter | Type | Description |
|---|---|---|
| `chainId` | `number` | The numeric EVM chain ID to look up. |

**Returns:** `Chain | undefined`

---

### `createIndelibleClient(chainOrKey, customRpcUrl?)`

Builds a viem `PublicClient` for an Indelible-supported chain, with sensible transport defaults.

```js
const client = createIndelibleClient('base');
const client = createIndelibleClient('ethereum', 'https://my-own-rpc.example.com');
```

| Parameter | Type | Description |
|---|---|---|
| `chainOrKey` | `string \| number \| Chain` | The chain to connect to. Accepts a chain key (`'ethereum'`), a numeric chain ID (`1`), or a viem `Chain` object directly. Unknown chains fall back to Sepolia. |
| `customRpcUrl` | `string` _(optional)_ | When provided, used as the sole RPC transport instead of the Alchemy/public fallback pair. |

**Returns:** `PublicClient`

When `customRpcUrl` is omitted, a `fallback` transport is built from `DEFAULT_RPC_URLS[chainKey]` first, then `PUBLIC_RPC_URLS[chainKey]`. This ensures the client remains functional even when Alchemy is unavailable (e.g. on IPFS-hosted pages).

```js
import { createIndelibleClient, getChainKeyById } from 'indelible/chains';
import { verifyQuoteProof } from 'indelible/verify';

// Auto-detect chain from a proof file's embedded chainId
const chainKey = getChainKeyById(proofData.chainId);
const client = createIndelibleClient(chainKey ?? 'sepolia');

const { verification, quoteText, allProofsValid } = await verifyQuoteProof(client, proofData);
```
