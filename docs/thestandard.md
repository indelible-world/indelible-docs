---
title: The Standard
icon: lucide/scroll
---

# The Indelible Standard

The Indelible Standard defines the best practices for publishing and verifying content using the Indelible Standard. By adhering to the Indelible Standard, creators can confidently publish tamper-proof content that is easily verifiable by anyone.

## Attestations

Attestations are the core building block of the Indelible Standard. An attestation is an on-chain cryptographic proof that a specific piece of content existed at a certain point in time and was authored by a particular entity (what we call an Authority). By creating and publishing attestations, creators can establish a verifiable record of their work that can be independently confirmed by anyone, without relying on any centralized authority (hence, the publisher is the Authority).

On-chain attestations are immutable and publicly accessible. The following is an example of a reference to an attestation using the Indelible Standard:
``` json
{
    "ipfsCid": "bafkreifzjut3te2nhyekklss27nh3k72ysco7y32koao5eei66wof36n5e",
    "chainId": 11155111,
    "authority": "0xD2f2c95962632B4742703CC058889c624380C748",
    "attestationIndex": 4
}
```



[Implement in HTML](html.md){ .md-button  d}