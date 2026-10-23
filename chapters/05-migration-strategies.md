# Migration Strategies

Migrating a blockchain to post-quantum signatures is part cryptography and part coordination. Users must move funds, wallets and hardware must support new schemes, and the protocol must decide what to do about coins whose owners never act. This chapter compares the main strategies.

## Strategy overview

| Strategy | Mechanism | Needs protocol change? | Who must act |
|---|---|---|---|
| New PQ output or account type | users move funds to PQ-protected outputs | yes (soft fork or hard fork) | every user, before a CRQC |
| Smart account validators | accounts swap signature validator to a PQ scheme | no on chains with programmable accounts | users of smart accounts |
| Hidden PQ fallback | PQ key committed now in a hidden script branch or module, used later | depends | users, early |
| Commit-delay-reveal migration | prove ownership by committing before revealing the key | yes | users, after a trigger |
| Seed-preimage proofs | prove knowledge of the wallet seed (hash preimage) with a ZK proof | yes, emergency hard fork | users with HD wallets |
| Sunset of legacy signatures | after a deadline, legacy signatures become invalid | yes | everyone; lost coins frozen |

## Account abstraction as a migration path

On chains with programmable accounts, a smart account can change its validation logic without moving funds. A user could add a PQ validator module (for example one verifying a hash-based or lattice signature) and then disable the ECDSA owner. This works today in principle, with high gas costs (chapter 3). It does not help EOAs unless they first move to a smart account, and EIP-7702 delegated EOAs still keep the original key as a valid signer unless the protocol changes.

## Hidden fallback paths

Users can prepare now by committing to a PQ key without using it:

- On Bitcoin, a Taproot script tree could contain a leaf with a PQ spending condition if the scripting language supported PQ verification. Today it does not, which is why proposals such as BIP 360 exist.
- On smart accounts, a recovery module can hold the hash of a PQ public key, to be revealed and activated later.

Since the commitment is a hash, it stays hidden from a quantum attacker until revealed.

## Commit-delay-reveal

For coins whose key is hidden behind a hash, a CRQC attacker can still race the spending transaction in the mempool. A commit-delay-reveal scheme defends against this: the owner first publishes a commitment (a hash binding the coin, the new destination and a secret) and waits for it to be buried under enough blocks; then reveals the spend. An attacker who derives the key from the reveal is too late, because the protocol honours the earliest commitment. Variants have been discussed on the bitcoin-dev mailing list over several years. They need consensus support and add delay to migration spends.

## Emergency hard fork with seed proofs

Vitalik Buterin's 2024 ethresear.ch post "How to hard-fork to save most users' funds in a quantum emergency" outlines a plan for Ethereum: roll back blocks after a detected quantum attack, disable EOA signatures, and let users recover by proving with a STARK that they know the BIP 32 seed (a hash preimage) behind their key. Because hash functions remain secure, the proof is quantum-safe. Users with keys not derived from a seed, or who lost the seed, would not be covered.

## Sunsetting legacy signatures

Some Bitcoin developers have proposed phased plans in which, after notice periods, spending to vulnerable output types is disallowed and later legacy signatures themselves become invalid, freezing coins that have not migrated. A draft by Jameson Lopp and co-authors in 2025 is one example. The argument for freezing is that otherwise the first CRQC owner takes the coins; the argument against is that it confiscates from owners who did not act. Whatever the outcome, chains should decide in advance rather than during an emergency.

## Validators and consensus

PoS chains must also migrate validator keys. Options include adding a PQ withdrawal credential (so stake can be recovered even if signing keys are compromised), hash-based signatures with STARK aggregation for votes, and a planned consensus upgrade. Withdrawal credentials committed as hashes today give a useful safety net.

## Worked example

> **Execution status:** not executed in this book's CI. Commands, code and expected results below are written from documentation, not observed output; verify locally.

Implement a Lamport one-time signature fallback for a smart account in Foundry, as a teaching model of a hash-based PQ path. Lamport signatures are too large for production but simple to verify.

1. Generate a key in Python: 256 pairs of 32-byte secrets; the public key is their Keccak-256 hashes. Commit to `keccak256(abi.encodePacked(pub))` in the account.

```python
import os
from eth_hash.auto import keccak
sk = [[os.urandom(32), os.urandom(32)] for _ in range(256)]
pk = [[keccak(a), keccak(b)] for a, b in sk]
def sign(msg32: bytes):
    bits = int.from_bytes(msg32, "big")
    return [sk[i][(bits >> (255 - i)) & 1] for i in range(256)]
```

2. Solidity verifier:

```solidity
function lamportVerify(bytes32 m, bytes32[2][256] calldata pub, bytes32[256] calldata sig)
    internal pure returns (bool)
{
    for (uint256 i; i < 256; i++) {
        uint256 bit = (uint256(m) >> (255 - i)) & 1;
        if (keccak256(abi.encodePacked(sig[i])) != pub[i][bit]) return false;
    }
    return true;
}
```

3. The account stores only the commitment. A `migrate(newOwner, pub, sig)` function checks `keccak256(abi.encodePacked(pub)) == commitment`, verifies the signature over `keccak256(abi.encode(address(this), block.chainid, newOwner))`, disables ECDSA, and sets the commitment to zero so the one-time key cannot be reused.
4. Write Foundry tests: valid migration succeeds; a second use fails; a wrong message fails. Run `forge test --gas-report` and record the gas (it will be high).

## Common pitfalls

- Assuming smart accounts are PQ-safe while ECDSA owners or EIP-7702 keys remain valid.
- Reusing one-time hash-based keys.
- Revealing a fallback PQ key early, which gives no harm but loses the option of rotating to a better scheme.
- Plans that ignore lost coins until an emergency.
- Migration designs without user-facing wallet support.

## Exercises

1. Complete the Lamport fallback and its tests, and add a check that the old ECDSA path is disabled after migration.
2. Write a design note for a commit-delay-reveal migration on a UTXO chain: commitment contents, delay, and how conflicts are resolved.
3. Draft a one-page policy proposal for handling lost exposed coins, presenting both freeze and no-freeze positions fairly.

## Further reading

- Vitalik Buterin, "How to hard-fork to save most users' funds in a quantum emergency", ethresear.ch (2024)
- Lamport, "Constructing Digital Signatures from a One Way Function" (1979)
- BIP 360 draft and bitcoin-dev discussions on quantum migration
- Lopp et al., post-quantum migration and legacy signature sunset draft (2025)
- See **account-abstraction-security**
