# Resources

## Standards and guidance
- [NIST Post-Quantum Cryptography project](https://csrc.nist.gov/projects/post-quantum-cryptography): FIPS 203 (ML-KEM), FIPS 204 (ML-DSA), FIPS 205 (SLH-DSA), FN-DSA (Falcon) draft
- NIST SP 800-208, stateful hash-based signatures (XMSS, LMS), 2020
- NIST IR 8547 (initial public draft, 2024), Transition to Post-Quantum Cryptography Standards
- NSA CNSA 2.0 advisory (2022)
- BIP 360 draft, [BIPs repository](https://github.com/bitcoin/bips); [EIP-2028 and EIP-7623](https://eips.ethereum.org)

## Papers and posts
- Shor (1994); Grover (1996)
- Roetteler, Naehrig, Svore, Lauter, "Quantum resource estimates for computing elliptic curve discrete logarithms" (2017)
- Gidney and Ekerå, "How to factor 2048 bit RSA integers in 8 hours using 20 million noisy qubits" (2019)
- Gidney, "How to factor 2048 bit RSA integers with less than a million noisy qubits" (2025)
- Webber et al., "The impact of hardware specifications on reaching quantum advantage in the fault tolerant regime" (2022)
- Lamport, "Constructing Digital Signatures from a One Way Function" (1979)
- Vitalik Buterin, "How to hard-fork to save most users' funds in a quantum emergency", [ethresear.ch](https://ethresear.ch) (2024)
- Deloitte, "Quantum computers and the Bitcoin blockchain"

## Tools
- [Open Quantum Safe / liboqs](https://openquantumsafe.org)
- OpenSSL 3.5+ (ML-KEM, ML-DSA, SLH-DSA, hybrid TLS groups)
- [Falcon reference](https://falcon-sign.info)
- [Bitcoin Core](https://bitcoincore.org) regtest, [Foundry](https://book.getfoundry.sh)

## Related books in this library
- **zk-circuit-security**, **rollup-sequencer-prover-security**, **account-abstraction-security**, **onchain-malware-analysis**
