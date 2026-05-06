---
title: JavaScript Library
icon: lucide/package
---

# `indelible` JavaScript Library

The `indelible` npm package is a framework-agnostic JavaScript client for the Indelible Standard. It handles all on-chain reads and writes against the TAANQ and ENS contracts, as well as the off-chain cryptographic operations (hashing, Merkle proofs, CID encoding) needed to participate in the protocol.

Use this library from a browser app, Node.js script, or any other JavaScript runtime.

---

## Installation

```bash
npm install indelible viem
```

[viem](https://viem.sh) is a peer dependency. Bring your own version (`^2.0.0`). The library does not bundle wallet discovery (EIP-6963) or chain-switching — those concerns live in the host app.

---

## Setting up clients

Every function that reads from or writes to the chain takes a viem `PublicClient`, and write functions additionally take a `WalletClient`. Create them once and pass them through:

```js
import { createPublicClient, createWalletClient, http, custom } from 'viem';
import { sepolia } from 'viem/chains';

// Read-only — works in Node.js or the browser
const publicClient = createPublicClient({
    chain: sepolia,
    transport: http(),
});

// Browser — requires window.ethereum (or any EIP-1193 provider)
const walletClient = createWalletClient({
    account,          // viem Account object
    chain: sepolia,
    transport: custom(window.ethereum),
});
```

The library is chain-agnostic. Any chain where the TAANQ contract is deployed will work; swap the chain and transport as needed.

---

## Modules

The package is split into focused entry points so you can import only what you need:

| Entry point | Contents |
|---|---|
| `indelible/verify` | Read-only functions: CID verification, attestation lookups, quote proof verification |
| `indelible/publish` | Write functions: commit/reveal, revocation, delegation, ENS binding, quote proofs |
| `indelible/utils` | Cryptographic utilities: hashing, CID encoding, Merkle tree building |
| `indelible/constants` | Contract addresses, protocol constants, result codes |
| `indelible/chains` | Chain registry, RPC URLs, and viem client construction |
| `indelible/abi/taanq` | TAANQ contract ABI (JSON) |
| `indelible/abi/ens` | Indelible ENS contract ABI (JSON) |

---

## Quick examples

### Verify a CID

```js
import { verifyCid } from 'indelible/verify';

const result = await verifyCid(publicClient, 'bafkrei...');
console.log(result.headline, result.details);
```

### Attest content (commit + reveal)

```js
import { commitAttestation, revealAttestation } from 'indelible/publish';

const { pendingCommit } = await commitAttestation({
    walletClient,
    publicClient,
    content: 'Hello, world.',
    account,
});

// Persist pendingCommit (e.g. localStorage) — wait ≥ 60 s before revealing
const { attestationIndex } = await revealAttestation({
    walletClient,
    publicClient,
    pendingCommit,
    account,
});
```

### Generate and verify a quote proof

```js
import { proveQuote } from 'indelible/publish';
import { verifyQuoteProof } from 'indelible/verify';

// Generate
const { proofJson } = await proveQuote({ articleText, quote, authority, publicClient });

// Verify
const { verification, quoteText, allProofsValid } = await verifyQuoteProof(publicClient, proofJson);
```

---

See the [Verify](verify.md), [Publish](publish.md), [Utilities & Constants](utils.md), and [Chains](chains.md) pages for full API reference.
