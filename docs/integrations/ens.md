---
title: ENS
icon: simple/ens
---

# ENS Integration

The Indelible ENS integration allows creators to publicly link an [ENS name](https://ens.domains/) to their Indelible authority address. This creates a human-readable, on-chain binding between an ENS name and an Indelible authority, making it easier for anyone to verify who is behind a given address.

## How It Works

The `IndelibleENS` contract reads the `indelible-address` text record from an ENS name's resolver. To register a binding, the value of that text record must match the address of the caller. This ensures only the ENS name's owner can create (or update) a binding for their name.

Bindings are stored as `EnsVerification` records on-chain. Each record contains the authority address, the ENS node (namehash), the DNS-encoded name, and timestamps for when the binding was created and (if applicable) revoked.

### Setting Up Your ENS Text Record

Before registering a binding, set the `indelible-address` text record on your ENS name to your Indelible authority address (a standard `0x`-prefixed hex address). This can be done through the [ENS Manager](https://app.ens.domains/) or any compatible ENS client.

## Registering a Binding

Call `registerEnsBinding` with the DNS-encoded form of your ENS name. The contract will:

1. Compute the namehash of the provided DNS name.
2. Resolve the `indelible-address` text record from the ENS registry.
3. Verify that the resolved address matches `msg.sender`.
4. If a previous binding exists for that name and has not been revoked, revoke it by recording an `endTimestamp`.
5. Create a new `EnsVerification` record and index it by both node and sender address.

```solidity
function registerEnsBinding(bytes calldata dnsName) external
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `dnsName` | `bytes` | DNS-encoded ENS name (e.g., the wire-format encoding of `alice.eth`) |

!!! tip "Updating Your Binding"
    To revoke a binding, update the ENS name's `indelible-address` record to a different address or remove it entirely, and then call `registerEnsBinding` again with the same name. The contract will detect the change and mark the previous binding as revoked.

!!! note "Check Your records"
    The `indelible-address` text record on the ENS name must be set to your address before calling this function, or the transaction will revert with `"Record invalid"`.

## Removing a Binding

Anyone can call `removeEnsBinding` to revoke a binding that is no longer valid — i.e., where the ENS name's `indelible-address` record no longer points to the registered authority. Indelible runs an automated checker that monitors bindings and calls this function whenever a record becomes stale.

```solidity
function removeEnsBinding(bytes32 node) external
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `node` | `bytes32` | The namehash of the ENS name whose binding should be removed |

The function reverts if the binding does not exist, if the ENS record still matches the registered authority (binding is still valid), or if the binding has already been revoked.

!!! info "Permanence"
    Removing a binding does not delete the verification record from the blockchain. Instead, it sets the `endTimestamp` to indicate that the binding is no longer active. This allows anyone to query the full history of ENS bindings for an address, including when they were created and revoked. Verification records search for valid bindings by checking for records that were active at the time of attestation.

??? note "Auto-Revocation"
    Indelible's backend periodically checks all active ENS bindings to ensure they remain valid. If it detects that an ENS name's `indelible-address` record has changed or been removed, it automatically calls `removeEnsBinding` to revoke the corresponding binding on-chain. This ensures that the on-chain records accurately reflect the current state of ENS ownership. Anyone can run this script independently to verify the integrity of ENS bindings. See [the Indelible Recordwatcher](https://github.com/indelible-world/indelible-ens-recordwatcher) for details.

## Verification Records

Each binding is stored as an `EnsVerification` struct:

```solidity
struct EnsVerification {
    address authority;       // The Indelible address bound to this ENS name
    bytes32 node;            // Namehash of the ENS name
    bytes dnsName;           // DNS-encoded ENS name
    uint256 startTimestamp;  // Block timestamp when the binding was created
    uint256 endTimestamp;    // Block timestamp when revoked; 0 if still active
}
```

All verifications are stored in the `verifications` array (index 0 is a non-functional placeholder — real bindings start at index 1). Two mappings provide efficient lookups:

- `nodeToBinding[node]` — returns the current verification index for an ENS node.
- `addrToBindings[address]` — returns all verification indices ever associated with an address, including revoked ones. This allows time-based verification: you can confirm that a given address controlled a particular ENS name during a specific time window.

## Contract Address

The `IndelibleENS` contract uses the canonical ENS registry at `0x00000000000C2E074eC69A0dFb2997BA6C7d2e1e`, which is the same across Ethereum mainnet and major testnets.