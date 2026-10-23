# NIST Post-Quantum Standards

In August 2024 NIST published its first three post-quantum cryptography standards. They are the most likely building blocks for any blockchain migration, because wallets, hardware vendors and libraries will implement them first. This chapter summarises each standard, its sizes and its fit for blockchains.

## The standards

| Standard | Name | Based on | Type | Hardness assumption |
|---|---|---|---|---|
| FIPS 203 | ML-KEM | CRYSTALS-Kyber | key encapsulation | module lattices (Module-LWE) |
| FIPS 204 | ML-DSA | CRYSTALS-Dilithium | signature | module lattices |
| FIPS 205 | SLH-DSA | SPHINCS+ | signature | hash functions only, stateless |
| FIPS 206 (draft at the time of writing) | FN-DSA | Falcon | signature | NTRU lattices |

In March 2025 NIST also selected HQC, a code-based KEM, as a backup to ML-KEM, with a separate standard to follow. Separately, NIST SP 800-208 (2020) already approves the **stateful** hash-based signatures XMSS and LMS for specific uses. Check NIST's current pages for FN-DSA's status before relying on it.

Blockchains need **signatures** for transactions and consensus. KEMs matter for encrypted channels and encrypted data (chapter 6).

## Sizes

Sizes below are from the standards and submission documents (bytes).

| Scheme | Public key | Signature | Notes |
|---|---|---|---|
| ECDSA secp256k1 (today) | 33 (compressed) | ~71–72 (DER) or 64 | baseline |
| Schnorr BIP 340 (today) | 32 | 64 | baseline |
| ML-DSA-44 | 1,312 | 2,420 | NIST category 2 |
| ML-DSA-65 | 1,952 | 3,309 | category 3 |
| ML-DSA-87 | 2,592 | 4,627 | category 5 |
| FN-DSA / Falcon-512 | 897 | ~666 | category 1; compact but complex signing |
| FN-DSA / Falcon-1024 | 1,793 | ~1,280 | category 5 |
| SLH-DSA-128s | 32 | 7,856 | small, slow signing |
| SLH-DSA-128f | 32 | 17,088 | faster signing, larger signature |
| SLH-DSA-256s | 64 | 29,792 | category 5 |

Signatures grow by one to three orders of magnitude. That is the central engineering problem of blockchain migration (chapter 3).

## Choosing among them

**ML-DSA** is the general-purpose default: fast, simple to implement in constant time, moderate size. Most vendors will support it first.

**FN-DSA (Falcon)** has the smallest combined key and signature among the lattice options, which is attractive for block space. But signing requires careful floating-point Gaussian sampling, which has been a source of side-channel concerns, and hardware wallets may find it harder to implement safely. Verification is simple.

**SLH-DSA** relies only on hash functions, the most conservative assumption. Public keys are tiny but signatures are large and signing is slow. It is a strong candidate for rarely used, high-value keys (cold storage, emergency recovery) where size matters less than confidence.

**Stateful hash-based schemes (XMSS, LMS)** give small signatures for hash-based security, but the signer must never reuse a one-time key state. Restoring a wallet from backup and signing again can reuse state and leak the key. That makes them risky for ordinary wallets, though they may suit tightly controlled signers.

## Hybrid signatures

During transition, many systems combine a classical and a post-quantum signature, requiring both to verify. The combination stays secure if either scheme holds. The cost is size and complexity, and careful binding so an attacker cannot strip one signature and reuse the other. The IETF has ongoing work on composite signatures; check its drafts for current constructions.

## Implementation maturity

- **liboqs** (Open Quantum Safe) implements the NIST algorithms and others for experimentation.
- **OpenSSL 3.5** (2025) added ML-KEM, ML-DSA and SLH-DSA.
- **BoringSSL**, **AWS-LC** and others support ML-KEM, mainly for TLS.
- Hardware wallets and HSMs are adding support progressively; check vendor documentation for production status.

## Worked example

> **Execution status:** not executed in this book's CI. Commands, code and expected results below are written from documentation, not observed output; verify locally.

Generate and verify ML-DSA and SLH-DSA signatures with OpenSSL 3.5 or later, and compare sizes.

1. Check your version: `openssl version` (3.5.0 or later), and list algorithms with `openssl list -signature-algorithms | grep -i -E 'ml-dsa|slh-dsa'`.
2. Generate keys and sign a 32-byte message:

```bash
head -c 32 /dev/urandom > msg.bin
for alg in ML-DSA-44 ML-DSA-65 SLH-DSA-SHA2-128s; do
  openssl genpkey -algorithm $alg -out $alg.key
  openssl pkey -in $alg.key -pubout -out $alg.pub
  openssl pkeyutl -sign -rawin -in msg.bin -inkey $alg.key -out $alg.sig
  openssl pkeyutl -verify -rawin -in msg.bin -pubin -inkey $alg.pub -sigfile $alg.sig
  echo "$alg sig bytes: $(wc -c < $alg.sig)"
done
```

3. Compare the printed sizes with the table above. Time each signing with `time` to see SLH-DSA's slower signing.
4. Corrupt one byte of a signature and confirm verification fails.
5. Alternatively, use `liboqs-python` to run the same comparison including Falcon.

## Common pitfalls

- Treating draft standards (FN-DSA at the time of writing) as final.
- Choosing stateful hash-based signatures for wallets restored from backup.
- Comparing schemes by signature size only, ignoring public-key size and signing complexity.
- Hybrid constructions that do not bind both signatures to the same message and context.
- Assuming library support implies hardware-wallet support.

## Exercises

1. Build a table of total bytes (public key + signature) per transaction input for each scheme, and the multiplier versus Schnorr.
2. Implement a toy hybrid signature (Ed25519 plus ML-DSA-44) in Python with liboqs, binding both to a domain-separated message, and write tests that reject either signature alone.
3. Read FIPS 204's section on randomised versus deterministic signing and summarise the trade-off for hardware wallets.

## Further reading

- FIPS 203, 204, 205 (August 2024): <https://csrc.nist.gov/projects/post-quantum-cryptography>
- NIST SP 800-208, Recommendation for Stateful Hash-Based Signature Schemes (2020)
- Open Quantum Safe (liboqs): <https://openquantumsafe.org>
- OpenSSL 3.5 release notes: <https://www.openssl.org/news/>
- Falcon specification: <https://falcon-sign.info>
