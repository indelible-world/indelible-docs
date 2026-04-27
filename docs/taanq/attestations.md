---
title: Attestations
icon: lucide/check
---
# TAANQ Attestations

TAANQ Attestations are on-chain records that confirm the authorship of a specific piece of content at a particular point in time.

Each attestation links a content identifier (IPFS CID) to an authority address, recording when that authority acknowledged authorship of that content. Attestations are immutable once created, though they can be revoked within a 7-day window.

---

## Data Model

```solidity
struct Attestation {
    bytes32 ipfsHash;        // IPFS CID of the attested content
    bytes32 qvHash;          // Quote-verification hash
    bytes32 parentIpfsHash;  // Optional: CID of a prior version (for edits)
    address authority;       // Address that made the attestation
    uint256 timestamp;       // Block timestamp when the attestation was created
    uint256 revokedAt;       // Block timestamp of revocation; 0 if not revoked
}
```

Attestations are stored in a public array on the contract. The array is pre-initialized with a placeholder at index `0` so that all real attestations start at index `1`. An index of `0` therefore means "no attestation found."

### Fields

| Field | Type | Description |
|---|---|---|
| `ipfsHash` | `bytes32` | The IPFS CID of the attested content. |
| `qvHash` | `bytes32` | A hash used for native quote verification. |
| `parentIpfsHash` | `bytes32` | Optional CID of the content's prior version, indicating this attestation is an edit. Leave as `bytes32(0)` if not applicable. |
| `authority` | `address` | The Ethereum address that authoritatively attested to the content. This may differ from `msg.sender` when a [delegation](delegations.md) is active. |
| `timestamp` | `uint256` | The `block.timestamp` at the time the attestation was created. |
| `revokedAt` | `uint256` | The `block.timestamp` when the attestation was revoked. A value of `0` means the attestation is still active. |

---

## Lookups

Two mappings allow efficient on-chain lookups without iterating the full attestation array.

```solidity
// All attestation indices for a given IPFS CID
mapping(bytes32 => uint256[]) public cidToAttestationIndices;

// The single attestation index for a given CID + authority pair
mapping(bytes32 => mapping(address => uint256)) public cidAndAddressToAttestationIndices;
```

- Use `cidToAttestationIndices[ipfsHash]` to retrieve an array that contains all attestation indices for a piece of content. You will need to provide an index in this list to get a specific attestation index, and then use that index to look up the attestation in the main `attestations` array.
- Use `cidAndAddressToAttestationIndices[ipfsHash][authority]` to look up whether a specific authority has attested to a specific piece of content, and retrieve that attestation's index directly.

---

## Creating an Attestation

Creating an attestation uses the [commit/reveal](commit-reveal.md) scheme to prevent front-running. The full flow is:

1. **Commit** — Call `commit(saltedHash)` where `saltedHash = keccak256(abi.encodePacked(ipfsHash, bytes32(bytes20(msg.sender)), salt))`.
2. **Wait** — Wait at least **60 seconds** after the commit is mined.
3. **Reveal** — Call `reveal(...)` with the original values to create the attestation.

### `reveal()`

```solidity
function reveal(
    bytes32 saltedHash,
    bytes32 salt,
    bytes32 ipfsHash,
    bytes32 qvHash,
    bytes32 parentIpfsHash,
    address authority
) external
```

| Parameter | Description |
|---|---|
| `saltedHash` | The hash submitted during `commit()`. |
| `salt` | The secret value used to construct `saltedHash`. |
| `ipfsHash` | IPFS CID of the content being attested. |
| `qvHash` | Quote-verification hash. |
| `parentIpfsHash` | CID of a prior version, or `bytes32(0)` if this is not an edit. |
| `authority` | The authority address for the attestation. Must be `msg.sender`, or `msg.sender` must be a [delegate](delegations.md) of `authority`. |

**Validation:**

- The salted hash must match `keccak256(abi.encodePacked(ipfsHash, bytes32(bytes20(msg.sender)), salt))`.
- The commit must exist and be at least 60 seconds old.
- If `authority != msg.sender`, a valid [delegation](delegations.md) must exist.
- An attestation for the same `ipfsHash` + `authority` pair must not already exist.

On success, the commit is deleted (returning a gas refund) and the attestation is appended to the `attestations` array.

---

## Revoking an Attestation

```solidity
function revokeAttestation(uint256 attestationId) external
```

An attestation can be revoked by its authority address or a current [delegate](delegations.md) of that authority. Revocation sets `revokedAt` to `block.timestamp`.

**Constraints:**

- The attestation must have been created within the last **7 days** (`DEFAULT_REVOCABLE_WINDOW = 604800` seconds).
- The attestation must not already be revoked.

After the 7-day window closes, an attestation becomes permanent and cannot be revoked.

!!! warning
    Revoking an attestation does not delete it from the blockchain. The attestation record remains publicly accessible, but its `revokedAt` timestamp indicates that it has been revoked. This design ensures transparency while allowing authors to disavow attestations signed by insecure keys within a reasonable timeframe. **A revoked attestation can not be re-attested by the same authority.** If an author wants to re-attest the same content after revocation, they must create a new attestation with a different IPFS CID (e.g., by making a minor edit to the content).


---

## Constants

| Constant | Value | Description |
|---|---|---|
| `DEFAULT_REVOCABLE_WINDOW` | `604800` | Seconds after creation during which an attestation may be revoked (7 days). |
| `DEFAULT_REVEAL_TIMER` | `60` | Minimum seconds that must pass between a `commit()` and its corresponding `reveal()`. |