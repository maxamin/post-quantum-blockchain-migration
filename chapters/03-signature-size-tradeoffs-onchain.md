# Signature Size and Cost Trade-offs On-Chain

Post-quantum signatures are large. On a blockchain, every byte is replicated to every node, stored forever, and paid for in fees. This chapter quantifies the cost on Bitcoin and Ethereum and looks at techniques for reducing it.

## Bitcoin: block weight

Bitcoin limits blocks to 4 million weight units. Witness bytes count one unit each; non-witness bytes count four. A typical single-input Taproot key-path spend is around 57.5 virtual bytes of input including a 64-byte signature in the witness.

| Input signature scheme | Approx. witness bytes (pk + sig) | Inputs per block if witness dominates |
|---|---|---|
| Schnorr (today, key path) | 64 (key in output) | many thousands |
| Falcon-512 | ~1,560 | roughly 2,500 |
| ML-DSA-44 | ~3,730 | roughly 1,000 |
| SLH-DSA-128s | ~7,890 | roughly 500 |

These are rough upper bounds that ignore non-witness data, but they show the order of magnitude: transaction throughput could drop by a factor of 20 to 100 if every input carried a PQ signature. Proposals therefore aim to use PQ signatures only where necessary, for example as a fallback path in a script tree that is revealed only if needed.

On the Bitcoin side, BIP 360 (a draft first titled "Pay to Quantum Resistant Hash", whose design has evolved since) proposes a new output type for quantum-resistant spending paths. It is a draft, not activated; follow its current text and mailing-list discussion.

## Ethereum: calldata and verification gas

Ethereum charges for calldata (16 gas per non-zero byte under EIP-2028, with a floor price for data-heavy transactions introduced by EIP-7623) and for verification computation.

```python
def calldata_gas(n_bytes, nonzero_ratio=0.98):
    nz = int(n_bytes * nonzero_ratio)
    return nz * 16 + (n_bytes - nz) * 4

for name, size in [("ECDSA", 65), ("Falcon-512", 666 + 897),
                   ("ML-DSA-44", 2420 + 1312), ("SLH-DSA-128s", 7856 + 32)]:
    print(name, calldata_gas(size))
```

Verification cost depends on implementation. Solidity verifiers for lattice signatures are expensive because they need many modular multiplications over polynomials; hash-based verifiers need thousands of hash calls. Precompiles would cut costs drastically, and EIPs for post-quantum verification precompiles have been proposed. Until then, smart-account-based PQ signatures are feasible but costly.

Storing the public key once in the account (rather than sending it with each transaction) helps for accounts with large public keys. Committing to the public key's hash and passing it only when needed is another option.

## Reduction techniques

| Technique | Idea | Trade-off |
|---|---|---|
| Fallback paths | PQ key only in a hidden script branch or recovery module | no protection until activated; needs a migration trigger |
| Public key commitments | store a hash, reveal the key once | reveals key on first use |
| Signature aggregation | combine many signatures into one proof | no standard PQ aggregation; STARK-based aggregation is research |
| Proof-based compression | prove many PQ signatures valid in one STARK | prover cost, new trust in the proof system |
| Stateful hash-based schemes | small signatures (XMSS, LMS, Winternitz) | state management; dangerous for restored wallets |
| One-time keys | Lamport or Winternitz one-time signatures | each key signs once; fits UTXO models naturally |

Proof-based aggregation is attractive because STARKs rely only on hashes and so are themselves plausibly quantum-resistant. Ethereum researchers have discussed using STARKs to aggregate hash-based signatures for consensus. This is active research; treat specific performance claims cautiously.

## Consensus signatures

Proof-of-stake chains with large validator sets use BLS signatures precisely because they aggregate. No standardised PQ scheme offers comparable aggregation. Migrating consensus is therefore harder than migrating user transactions and may require redesigning how votes are collected and verified.

## Worked example

> **Execution status:** not executed in this book's CI. Commands, code and expected results below are written from documentation, not observed output; verify locally.

Compute and compare on-chain costs for each scheme using local tools.

1. Save the Python snippet above as `cost.py` and run `python3 cost.py`. Note the calldata gas for each scheme.
2. Extend it with Bitcoin: compute vbytes for a single-input, two-output transaction with each scheme's witness (`vbytes = non_witness + witness/4`), using 41 bytes per input non-witness, 43 per P2TR output and 10.5 for overhead as round figures.
3. Use Foundry to measure verification gas for a hash-based scheme. Write a Lamport one-time signature verifier (chapter 5 gives one) and run `forge test --gas-report` for a single verification.
4. Combine both: produce a table of total gas (calldata plus verification) per transaction for ECDSA and Lamport, and the multiplier.
5. Record which numbers you measured and which you estimated; keep the scripts so others can rerun them.

## Common pitfalls

- Comparing signature sizes without public-key sizes.
- Ignoring verification gas, which can dominate for Solidity verifiers.
- Assuming PQ aggregation exists in the way BLS aggregation does.
- Treating draft BIPs and EIPs as decided.
- Quoting throughput impacts without stating assumptions.

## Exercises

1. Extend `cost.py` to model a block where 10%, 50% and 100% of inputs use PQ signatures, and report throughput relative to today.
2. Estimate storage growth per year for a chain with fixed transaction count if all signatures moved to ML-DSA-44.
3. Survey current EIPs and Ethereum research posts on PQ signature precompiles or aggregation and summarise their status with dates.

## Further reading

- BIP 360 draft and discussion: <https://github.com/bitcoin/bips>
- EIP-2028 and EIP-7623 (calldata pricing): <https://eips.ethereum.org>
- Ethereum Research forum, post-quantum threads: <https://ethresear.ch>
- NIST SP 800-208 (XMSS, LMS)
- See **zk-circuit-security** for STARK and SNARK background
