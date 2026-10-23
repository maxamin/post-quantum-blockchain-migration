# Harvest Now, Decrypt Later and a Migration Checklist

Signatures only need to be quantum-safe from the moment a CRQC exists. Encryption is different: data encrypted today with classical key exchange can be recorded now and decrypted later. This chapter covers where that matters for blockchain projects and ends with a migration readiness checklist.

## Harvest now, decrypt later

An adversary records encrypted traffic or ciphertexts today and stores them until a quantum computer can break the key exchange (ECDH, RSA). Anything whose confidentiality must last beyond that point is at risk now.

| Data | Where it lives | Lifetime of sensitivity |
|---|---|---|
| Encrypted wallet backups in cloud storage | password-based (symmetric) or key-wrapped with ECC | as long as funds sit in the wallet |
| TLS traffic between wallets, RPC providers and custody APIs | ECDHE key exchange | depends on what passes: API keys, seeds during setup, PII |
| MPC protocol messages | often encrypted with ECC-based channels | as long as shares remain valid |
| Encrypted mempools and threshold-encrypted transactions | pairing or ECC-based threshold encryption | short (until block inclusion), so lower risk |
| Shielded transaction data (privacy coins, private payments) | ECC-based note encryption | effectively permanent privacy expectation |
| Validator and node p2p traffic | ECC-based handshakes | mostly low, metadata only |

Two observations. Symmetric encryption with 256-bit keys (AES-256, ChaCha20) is considered safe against Grover, so password-encrypted backups depend mainly on password strength and KDF, not on quantum threat. Privacy systems are hit hardest: if a chain's shielded notes are encrypted to ECC keys, a future CRQC could reveal historical transaction details, even though funds remain safe if spend authorisation migrates.

## Hybrid key exchange today

Hybrid key exchange combines a classical and a PQ KEM so the session stays secure if either holds. `X25519MLKEM768` is the hybrid group being deployed in TLS 1.3; major browsers enabled hybrid ML-KEM key exchange by default in 2024–2025, and OpenSSL 3.5 supports it. For blockchain infrastructure, enabling it on RPC endpoints, custody APIs and internal service meshes is a cheap, high-value step.

## Proof systems

Pairing-based SNARKs (Groth16, PLONK with KZG commitments) rely on elliptic-curve assumptions that Shor breaks. A CRQC could forge proofs, which for a rollup or bridge means forging state transitions. Hash-based STARKs and FRI-based systems are considered plausibly post-quantum. Projects using pairing-based proofs should know which components depend on them and have a plan; see **zk-circuit-security** and **rollup-sequencer-prover-security**.

## Migration readiness checklist

```mermaid
flowchart LR
    I[Inventory] --> P[Prioritise by exposure and lifetime]
    P --> Q[Quick wins: hybrid TLS, KDFs]
    Q --> D[Design: PQ signatures, fallbacks]
    D --> T[Test on devnets]
    T --> C[Communicate and coordinate]
```

### Inventory

- List every place your system uses ECDSA, Schnorr, EdDSA, BLS, ECDH, RSA, or pairing-based proofs.
- Classify each by function: authorisation (signatures), confidentiality (encryption), integrity of computation (proofs).
- Record key exposure (chapter 4) for funds you control.

### Prioritise

- Confidentiality with long lifetimes first (HNDL).
- Large exposed balances next.
- Consensus and proof systems need long lead times; start design work early.

### Quick wins

- Enable hybrid ML-KEM key exchange on TLS endpoints you operate.
- Use 256-bit symmetric keys and memory-hard KDFs for backups.
- Stop address reuse; avoid sharing xpubs unnecessarily.
- Move treasury funds from exposed outputs to fresh hashed outputs.

### Design

- For smart accounts: plan a PQ validator module and test its gas.
- For UTXO funds: track PQ output proposals and prepare commitments where possible.
- For validators: prefer withdrawal credentials that do not depend on ECC alone.
- For crypto-agility: avoid hard-coding one signature scheme into formats and contracts; include algorithm identifiers.

### Test and coordinate

- Run PQ signing and verification in CI on devnets.
- Track hardware wallet and HSM vendor roadmaps.
- Publish a migration plan for users with clear dates and instructions.

## Worked example

> **Execution status:** not executed in this book's CI. Commands, code and expected results below are written from documentation, not observed output; verify locally.

Test hybrid key exchange locally with OpenSSL 3.5 or later.

1. Create a self-signed certificate and run a local server:

```bash
openssl req -x509 -newkey ec -pkeyopt ec_paramgen_curve:P-256 -nodes \
  -keyout key.pem -out cert.pem -days 1 -subj "/CN=localhost"
openssl s_server -accept 4433 -cert cert.pem -key key.pem -groups X25519MLKEM768:X25519 -www
```

2. Connect with a client that offers the hybrid group:

```bash
openssl s_client -connect localhost:4433 -groups X25519MLKEM768 </dev/null 2>&1 | grep -i -E "group|temp key"
```

3. Look for the negotiated group in the output. Then connect with `-groups X25519` only and compare.
4. Capture both handshakes with `tcpdump` or Wireshark and compare ClientHello sizes; the hybrid key share is over a kilobyte larger.
5. Repeat against your own staging RPC endpoint (not someone else's server) to see what it negotiates today.

## Common pitfalls

- Assuming signature migration also fixes confidentiality.
- Worrying about AES-256 while ignoring ECC-based key exchange.
- Forgetting proof systems in the inventory.
- No crypto-agility: formats that cannot name a new algorithm.
- Leaving migration plans until a credible quantum announcement.

## Exercises

1. Produce a cryptographic inventory for a project you know, classifying each use by function and HNDL risk.
2. Enable hybrid key exchange on a local nginx or Caddy instance built against a supporting TLS library and verify it with `openssl s_client`.
3. Write a one-page migration plan with dated milestones for a hypothetical smart-account wallet.

## Further reading

- NIST post-quantum cryptography project: <https://csrc.nist.gov/projects/post-quantum-cryptography>
- NIST IR 8547 (initial public draft, 2024)
- NSA, Commercial National Security Algorithm Suite 2.0 (CNSA 2.0) advisory (2022)
- IETF TLS working group drafts on hybrid ML-KEM key exchange
- Open Quantum Safe: <https://openquantumsafe.org>
- See **zk-circuit-security** and **rollup-sequencer-prover-security**
