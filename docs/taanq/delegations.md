---
title: Delegations
icon: lucide/vote
---
# Authority Address Delegations

Delegations allow an authority address to authorize a separate address, a *delegate,* to act on its behalf. This is useful when an authority wants to use a hot wallet for day-to-day operations while keeping the primary authority key in cold storage. Indelible.world's publisher verification system uses this functionality to attest on behalf of news organizations while keeping the keys safe, even if it were to be hacked.

Each authority can have at most one active delegate at a time.

---

## Data Model

```solidity
struct Delegation {
    address delegateAddress; // The address authorized to act on behalf of the authority
    uint256 timestamp;       // Block timestamp when the delegation was created
}

mapping(address => Delegation) public delegations;
```

The `delegations` mapping is keyed by the **authority address**. To check whether an address has a delegate, look up `delegations[authorityAddress].delegateAddress`.

---

## Setting a Delegate

```solidity
function delegate(address delegateAddress) external
```

Calling `delegate()` sets the caller's active delegate. If a delegation already exists, it is overwritten with the new address and an updated timestamp.

| Parameter | Description |
|---|---|
| `delegateAddress` | The address to authorize as a delegate for `msg.sender`. |

**Effect:** `delegations[msg.sender]` is set to a new `Delegation` struct with `delegateAddress` and the current `block.timestamp`.

---

## Revoking a Delegation

```solidity
function revokeDelegation() public
```

Calling `revokeDelegation()` removes the caller's active delegation entirely by deleting the entry from the mapping. Because each authority holds only one delegation at a time, deletion has the same effect as setting a `revokedAt` time — the delegate can no longer act on the authority's behalf immediately.

---

## How Delegations Affect Attestations

When creating an [attestation](attestations.md) via `reveal()`, the `authority` parameter specifies whose authority is being claimed. If `authority != msg.sender`, the contract checks that:

```solidity
delegations[authority].delegateAddress == msg.sender
```

If this check fails, the transaction reverts with `"Delegation invalid"`.

The same check applies when revoking an attestation — the caller must either be the attestation's authority directly, or an active delegate of that authority.

---

## One Delegation at a Time

Currently the contract enforces a single active delegate per authority. Setting a new delegate immediately replaces the previous one. There is no delegation history stored on-chain — if you need to audit past delegations, query historical transaction logs off-chain.