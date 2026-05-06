---
title: What is TAANQ
icon: lucide/newspaper
---
# Timestamped Authorship Attestation with Native Quote Verification

**TAANQ** is the smart contract protocol that powers on-chain authorship attestations for indelible.world. It provides a trustless, censorship-resistant way to establish that a specific piece of content existed, who authored it, and when — without relying on any centralized authority.

---

## What TAANQ Does

At its core, TAANQ lets an Ethereum address (an *authority*) attest that they authored a specific piece of content at a specific time. The content is identified by its IPFS CID — a hash derived from the content itself, meaning the CID is unique to that exact content and cannot be forged.

A TAANQ attestation records:

- **What** was authored — identified by an IPFS CID (`ipfsHash`)
- **Who** authored it — the authority's Ethereum address
- **When** it was attested — a block timestamp
- **Quote verification** — a secondary hash (`qvHash`) for verifying embedded quotes
- **Edit lineage** — an optional link to a prior version (`parentIpfsHash`)

Because this data lives on-chain, it cannot be altered, deleted, or censored after the fact.

---

## How It Works

The `IndelibleTAANQ` smart contract provides four core operations:

| Operation | Description |
|---|---|
| [Commit/Reveal](commit-reveal.md) | A two-step process for creating attestations that prevents front-running. |
| [Attestations](attestations.md) | The permanent on-chain records of authorship. |
| [Delegations](delegations.md) | Allows an authority to authorize a separate address to attest on its behalf. |
| [Revocation](attestations.md#revoking-an-attestation) | An attestation can be revoked by its authority within 7 days of creation. |

---

## Key Design Decisions

**Index starts at 1.** The attestation array is pre-seeded with a dummy entry at index `0`, so a lookup returning `0` unambiguously means "not found."

**Commit/reveal prevents front-running.** Without this scheme, a bad actor watching the mempool could copy an attestation transaction and submit it with a higher gas fee, stealing authorship credit. See [Commit/Reveal](commit-reveal.md) for details.

**7-day revocation window.** Attestations can be corrected or retracted shortly after creation, but become permanent after 7 days. This balances flexibility with the guarantee of long-term immutability.

**One delegate per authority.** Each authority address can designate exactly one delegate at a time, keeping the delegation model simple. The delegate can be changed or revoked at any time.

**Block timestamps, not block numbers.** Revocation windows and commit timers use `block.timestamp` rather than block numbers. This makes time constraints human-readable and portable across chains with different block times.