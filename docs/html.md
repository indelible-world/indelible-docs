---
title: HTMLData Standard
icon: lucide/compass
---

# Indelible's HTMLData Standard

Use the Indelible HTMLData Standard to embed structured quote and article authorship attestation metadata directly into your web pages.

At the top level of your article, whether it be an `<article>`, `<main>`, or even `<body>` tag, you must include the **`data-indelible="{}"`**[^1] attribute to indicate that the content within is part of an attestation.

[^1]: The `data-indelible` attribute must contain a JSON object as specified in [the Indelible Standard](thestandard.md#attestations).

## To include or not to include

Indelible's HTMLData Standard is designed to be flexible, so you can choose to mark text as part of an attestation inclusively or exclusively.

### Inclusive Marking
By marking specific elements (e.g., `<p>`, `<h1>`, `<blockquote>`, etc.) with the **`data-indelible-include`** attribute, you indicate that only those marked elements are part of the attestation. This allows you to selectively attest specific sections of your content, such as quotes or key paragraphs, while leaving other parts of the article, like advertisements and captions, outside the attestation.
!!! note
    When using inclusive marking, any content that is not explicitly marked with `data-indelible-include` will be excluded from the attestation. This means that if you forget to mark a section of your article, it will not be included in the attestation, which could lead to incomplete attestations.

### Exclusive Marking
By marking a parent element (e.g., `<article>`, `<main>`, or `<body>`) with the **`data-indelible`** attribute without manually specifying any `data-indelible-include` tags, you indicate that all content within that element is part of the attestation. This is the most common approach and is ideal for articles where the entire content is being attested. To exclude specific elements from the attestation, you can use the **`data-indelible-exclude`** attribute on those elements.

---

## Embedding Quotes

When your article quotes another attested work, you can embed the full cryptographic proof of that quote directly in the HTML using the **`data-indelible-quote`** attribute. Its value is the complete [quote proof JSON](taanq/quote-verification.md#the-proof-file) for the quoted passage — allowing any reader or tool to verify the quote on-chain without needing a separate file.

### The `data-indelible-quote` Attribute

Add `data-indelible-quote` to any element that contains a quote (typically a `<blockquote>`, `<q>`, or `<p>`). Its value is a JSON object with the following fields:

| Field | Type | Description |
|---|---|---|
| `ipfsCid` | `string` | The IPFS CID of the source article being quoted. |
| `authority` | `string` | The Ethereum address of the authority who attested the source article. |
| `chainId` | `number` | The chain ID of the network where the source attestation lives. |
| `attestationIndex` | `number` | The on-chain index of the source attestation. |
| `proof` | `array` | Merkle proof entries for the text chunks that make up this quote. Each entry has a `value` (`[index, chunkText]`) and a `proof` (array of sibling hashes). |

The `proof` array contains one entry per chunk of the quoted text. Chunks are sorted by `value[0]` (the index) and joined to reconstruct the full quote. See [Quote Verification](taanq/quote-verification.md) for how the Merkle tree is built and how proofs are verified.

```html
<blockquote
  data-indelible-quote='{
    "ipfsCid": "bafkreibv5ztp3bnp227ftsllzsi2q52qihnvlzqkv3sklklbosmtltg52m",
    "authority": "0xD77d245c4Fd75adb4eadFC4147469257061f06c1",
    "chainId": 11155111,
    "attestationIndex": 4,
    "proof": [
      {
        "value": ["40", "orridor through which a fifth of the world'\''s o"],
        "proof": [
          "0x68ee0e41adef1af681cdff20d9b4347acd58d8bed57ef0c1b7643c8e53c5a605",
          "0xf62d940314d0f1c38fec0fd4cde019b7e27d872734bdd0169c4c9b38a0500473",
          "0x434243c2b409f736ea2c09a8faa3ef717636d2624ffff6c29aa22650639425f3",
          "0xb3176681bc8ebcb4e79787a5a696304170e742bd962edd226a7be9e5877da06a",
          "0x2e7ea38575a882343ea2976c8430e99b95055a2bceba5fb662f282dcadf6d7a7",
          "0x1e0d0e6a974e0172e167da4d1afb1178513919b07bbc24f886d15918e77eba65"
        ]
      },
      {
        "value": ["41", "il and gas traveled before the war. On Sunday,"],
        "proof": [
          "0x099a98b82cbd8ec2b424230a24341a78b6995cc506743e637a9f5dcf4d9bf643",
          "0xb3b99a8f23d3b16338c6b71e5597ab9c019b01cd8f5a2db13e1075595a260747",
          "0x002468c15ffaf3a750a873adfd25c5a8ac57c8638f88f1f87db84129c5f97066",
          "0xdb03bb228b23ec63a8ed83efd00582621a399f062dc37d4b14a330b39c7825d7",
          "0xd7142eaa356344a94ca83fa839417ba049e59f0a7d51448f3079a1cdd0f72b7c",
          "0x1e0d0e6a974e0172e167da4d1afb1178513919b07bbc24f886d15918e77eba65"
        ]
      }
    ]
  }'
>
  …orridor through which a fifth of the world's oil and gas traveled before the war. On Sunday,…
</blockquote>
```

!!! tip "Proof files and inline attributes share the same format"
    The JSON you put in `data-indelible-quote` is structurally identical to the standalone [quote proof file](taanq/quote-verification.md#the-proof-file). You can generate one and use it both ways — embed it inline in the HTML and distribute it as a downloadable file for the Indelible verifier.

### Quoted Elements and Inclusion/Exclusion

Elements with `data-indelible-quote` follow the same inclusion/exclusion rules as all other elements:

- In **exclusive mode**, they are included in the attestation by default unless wrapped in a `data-indelible-exclude` element.
- In **inclusive mode**, they must be explicitly marked with `data-indelible-include` to be part of the attestation.

Quotes that are excluded from the attestation are not included in the Merkle tree and will not appear in the proof file.