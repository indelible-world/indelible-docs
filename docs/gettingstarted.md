---
title: Getting Started
icon: lucide/play
---

# Getting Started with the Indelible Standard

## Introduction

Indelible lets you establish verifiable, on-chain proof of authorship for digital content. The core of the [Indelible Standard](standard/index.md) is the **attestation**, a cryptographic record that ties a piece of content to an on-chain "Authority," or news organization, at a specific point in time, secured by the [TAANQ](standard/taanq/index.md) smart contract.

There are two ways to start creating attestations, depending on how much control you want over the process:

**[Indelible API](#using-the-indelible-api)** (fastest & free for pilot partners) 

: A hosted service handles CID computation and on-chain transactions for you, no blockchain required.

**[Doing it yourself](#doing-it-yourself)** (fully independent) 
   
: Integrate directly with the TAANQ smart contract using the `indelible` npm package for full control.

Once you've created attestations, you can [verify content](standard/taanq/quote-verification.md) against them and [embed attestations on your site](standard/html.md).

## Using the Indelible API
Publishers can use the [Indelible Publisher API](api/index.md) without running any blockchain infrastructure themselves. You send it an API key and some content; it computes the content's IPFS CID, queues the [commit/reveal](standard/taanq/commit-reveal.md) transactions, and signs them with a secure platform key to which your organization has [delegated](standard/taanq/delegations.md).


## Doing It Yourself

Use the [`indelible`](https://github.com/indelible-world/indelible-module) npm package to interact with the TAANQ smart contract directly from your own code. This is the best option if you want to build your own custom interface, integrate with other protocols, or have full control over the attestation process.

## Integrating with HTMLData

Tell browser-based verifiers that your content has been certified with [IndelibleTAANQ](standard/taanq/index.md) using our custom [HTMLData tags](standard/html.md). 

## Next steps

Contact us at [indelible.world](https://indelible.world/#contact) for more information.