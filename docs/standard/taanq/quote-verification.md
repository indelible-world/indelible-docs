---
title: Quote Verification
icon: lucide/quote
---
# Native Quote Verification

TAANQ's native quote verification lets you prove that specific text was committed to on-chain by an attesting authority. It works by splitting quoted content into **chunks**, building a [Merkle tree](https://docs.openzeppelin.com/contracts/5.x/api/utils#MerkleProof) from those chunks using OpenZeppelin's `StandardMerkleTree`, and storing the **Merkle root** as the `qvHash` field of the attestation. Anyone can then verify any chunk of the quoted text against the on-chain root using a distributed proof file.

---

## The `qvHash`

Every TAANQ attestation includes an optional `qvHash` field:

```solidity
struct Attestation {
    bytes32 ipfsHash;        // IPFS CID of the attested content
    bytes32 qvHash;          // Merkle root of quote chunks (zero if unused)
    ...
}
```

If your content has no verifiable quotes, set `qvHash` to `bytes32(0)`.

If your content **does** include verifiable quotes, `qvHash` is the **Merkle root** of a tree whose leaves are the individual text chunks that make up the quoted content.

---

## Building the Merkle Tree

The tree is built with OpenZeppelin's [`StandardMerkleTree`](https://github.com/OpenZeppelin/merkle-tree), using leaves of type `['string', 'string']`. Each leaf is a pair:

```
[index, chunkText]
```

- **`index`** — a string representation of the chunk's position (e.g. `"0"`, `"1"`, `"2"`, …), used to reconstruct the original order during verification.
- **`chunkText`** — the exact text of that chunk.

```js
import { StandardMerkleTree } from '@openzeppelin/merkle-tree';

const leaves = chunks.map((chunk, i) => [String(i), chunk]);
const tree = StandardMerkleTree.of(leaves, ['string', 'string']);
const qvHash = tree.root; // store this as qvHash in the attestation
```

The `tree.root` is the value submitted as `qvHash` during the `commit()`/`reveal()` flow and stored permanently on-chain.

---

## The Proof File

To allow others to verify quotes, the attesting author distributes a **proof file** — a JSON document containing the Merkle proofs for each chunk. The proof file has the following structure:

```json
{
  "ipfsCid": "bafkreifzjut3te2nhyekklss27nh3k72ysco7y32koao5eei66wof36n5e",
  "authority": "0xD2f2c95962632B4742703CC058889c624380C748",
  "proof": [
    {
      "value": ["0", "The revolution will not be televised."],
      "proof": ["0xabc...", "0xdef..."]
    },
    {
      "value": ["1", " It was written in the stars."],
      "proof": ["0x123...", "0x456..."]
    }
  ]
}
```

| Field | Description |
|---|---|
| `ipfsCid` | The IPFS CID of the attesting article (the document that contains the quotes). |
| `authority` | The Ethereum address of the authority who attested that article. |
| `proof` | An array of Merkle proof objects, one per chunk. |
| `proof[].value` | A `[index, chunkText]` tuple — the leaf value. |
| `proof[].proof` | The Merkle sibling hashes needed to verify this leaf against the root. |

---

## Verification Flow

1. **Load the proof file** — obtain the JSON proof file for the article.
2. **Fetch the attestation** — use `ipfsCid` and `authority` from the proof file to look up the attestation on the `IndelibleTAANQ` contract. Read its `qvHash` as the Merkle root.
3. **Verify each chunk** — for each item in `proof`, call:
   ```js
   const valid = StandardMerkleTree.verify(
       merkleRoot,
       ['string', 'string'],
       item.value,   // [index, chunkText]
       item.proof,
   );
   ```
   If any proof is invalid, the quoted content cannot be verified.
4. **Reconstruct the quote** — sort chunks by `Number(item.value[0])` and join `item.value[1]` to recover the full quoted text.

If every proof passes, the exact reconstructed text is confirmed to have been committed on-chain by the attesting authority.
!!! note
    Quote verification proves that the exact text of the quote was committed on-chain, but it does not prove that the quote was actually present in the original article with that specific IPFS CID, only that the Authority says it did. To verify that, you would also need to obtain a copy of the original article (e.g., from IPFS using the `ipfsCid`) and confirm that the quoted text appears there.
---

## No Quotes

If the document contains no verifiable quotes, pass `bytes32(0)` as `qvHash` in the `reveal()` call. A zero `qvHash` is valid and simply indicates that no quote commitment was made.

---

## Segmented Quotes (Ellipses & Brackets)

A quote is often displayed with an ellipsis (`...`, `…`) or a bracketed editorial insertion (e.g. `[sic]`) where part of the original text has been omitted or altered for readability. The proof file format doesn't change to support this — it still only ever contains proofs for literal chunks of the source article. Instead:

- The **prover** (`proveQuote`) locates each literal segment on either side of a gap marker in order within the article, and includes proofs for the chunks covering every segment.
- The **verifier** (`verifyQuoteProof` / `quoteMatchesProvenText`) can optionally check a displayed quote (including its `...`/`[...]` markers) against the reconstructed proven text, in one of two modes:
    - **Hard** (default) — the quote must appear verbatim as a contiguous substring of the proven text. No breaking changes to prior behavior.
    - **Soft** — the quote's literal segments (split on ellipsis/bracket markers) must each appear in the proven text, in the same order. Text inside brackets is discarded and not checked at all.

This keeps the on-chain and proof-file format identical for contiguous and segmented quotes alike — segmentation is purely a client-side convenience for handling how quotes are conventionally displayed.
