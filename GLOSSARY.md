# Glossary

- **CRQC:** cryptographically relevant quantum computer, able to run Shor's algorithm at useful scale.
- **Shor's algorithm:** quantum algorithm that solves factoring and discrete logarithms, breaking RSA and elliptic-curve cryptography.
- **Grover's algorithm:** quantum search with quadratic speedup; weakens but does not break hashes and symmetric ciphers.
- **ML-KEM (FIPS 203):** lattice-based key encapsulation mechanism, from Kyber.
- **ML-DSA (FIPS 204):** lattice-based signature scheme, from Dilithium.
- **SLH-DSA (FIPS 205):** stateless hash-based signature scheme, from SPHINCS+.
- **FN-DSA:** Falcon-based lattice signature, standardisation in progress.
- **Stateful hash-based signature:** XMSS or LMS; secure only if one-time key state is never reused.
- **Exposed public key:** a public key visible on-chain, allowing a CRQC to attack it offline.
- **Harvest now, decrypt later (HNDL):** recording ciphertext today to decrypt once quantum attacks are possible.
- **Hybrid scheme:** combination of classical and post-quantum primitives so security holds if either does.
- **Commit-delay-reveal:** migration technique that binds a spend to an earlier commitment to defeat mempool key-recovery races.
- **Crypto-agility:** ability to change algorithms without redesigning formats and protocols.
- **Mosca's inequality:** if security lifetime plus migration time exceeds time to a CRQC, action is already overdue.
