# Cryptape Research Survey

*Research: 2026-03-02*

Cryptape is the core engineering and cryptography research team behind Nervos CKB. Their GitHub repos are the source of truth for what's possible on CKB — production contracts, PoCs, protocol research, and tooling that often predates official nervosnetwork repos.

Full org: https://github.com/cryptape

---

## Immediately Relevant to Embedded / Wyltek Projects

### quantum-resistant-lock-script — CRITICAL
Repo: https://github.com/cryptape/quantum-resistant-lock-script

A fully audited (ScaleBit) SPHINCS+ lock script on CKB. Three implementations: C, Rust, and Hybrid. **Deployed on mainnet.**

Performance (CKB-VM cycles):

| Parameter set | Signature size | sha2-simple (C) |
|---|---|---|
| 128s | 7,856 bytes | 11.5M cycles |
| 128f | 17,088 bytes | 32.2M cycles |
| 192s | 16,224 bytes | 17.6M cycles |
| 256s | 29,792 bytes | 25.7M cycles |

Why it matters for embedded:
- Our SPHINCS+ moonshot for ESP32-P4 can use the C implementation directly
- The on-chain contract is already live — no new lock script needed
- Hardware SHA accelerators on P4 target exactly the sha2 variants here
- Audited — no need to re-audit if we use the same library
- This effectively downgrades SPHINCS+ from "moonshot" to "Later" on our roadmap

Next step: Pull the C SPHINCS+ source, benchmark on ESP32-P4 with/without hardware SHA.

---

### ccs-suite — Chinese Commercial Cryptography for RISC-V
Repo: https://github.com/cryptape/ccs-suite

SM2 (elliptic curve signatures), SM3 (hash), SM4 (block cipher), SM9, ZUC — all targeted for RISC-V environments. Also see their GmSSL fork with CKB-VM adaptations.

Why it matters: mandatory in Chinese financial/government systems. RISC-V targeting means already optimised for constrained environments. Worth having in the embedded crypto toolkit.

---

### omnilock — The Universal Lock Script
Repo: https://github.com/cryptape/omnilock

A lock script supporting multiple auth methods: secp256k1 (CKB), Ethereum, Bitcoin, EOS, and more. Production-grade, RFC'd, on mainnet.

Key insight: This is WyAuth's on-chain counterpart. We don't need to write a custom lock script for multi-chain auth. WyAuth on the device + Omnilock on-chain = complete cross-chain signing. Our POS terminal generates an Omnilock address per invoice, accepts any supported sig type.

RFC: https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0042-omnilock/0042-omnilock.md

---

### ckb-transaction-cobuild-poc — EIP-712 Style Signing
Repo: https://github.com/cryptape/ckb-transaction-cobuild-poc

Transaction Co-Build Protocol (TCoB) — structured transaction signing where users see typed, human-readable data ("Pay 100 CKB to alice.ckb") instead of raw bytes. Simplifies witness layout, improves DApp/wallet interoperability.

Why it matters for embedded: Hardware wallets need to show users what they're signing. TCoB is the UX layer that makes this possible. Should be in WyMolecule scope — when we construct transactions on-device, format them as TCoB for display.

---

### nostr-binding — Nostr Events as CKB Assets
Repo: https://github.com/cryptape/nostr-binding

Bind any Nostr event to a CKB cell — making Nostr events ownable on-chain. Nostr keys (secp256k1, same as Bitcoin/CKB) become cell owners. Live on testnet and mainnet.

Key insight for BitChat: BitChat uses Ed25519 identity keys. If BitChat adopted Nostr-compatible secp256k1 keys (or a bridge key), BitChat mesh identities could own CKB cells via nostr-binding. Payments to a mesh chat identity, fully on-chain. An ESP32 BitChat node could receive CKB payments to its identity — no separate wallet required.

---

### ckb-bf-zkvm — ZK Proofs on CKB-VM
Repo: https://github.com/cryptape/ckb-bf-zkvm

BrainFuck zkVM using Halo2 (KZG), verified on CKB-VM. Proof size: ~4KB constant. Verification: ~130M cycles.

Significance: ZK verification is viable on CKB. The pattern (prove off-chain, verify on-chain) applies to:
- Gaming: provably fair randomness, private move proofs (ZK Kabletop)
- IoT: verifiable sensor data without revealing raw readings
- Privacy payments: prove you have enough balance without revealing it

---

### ckb-combine-lock-poc — Composable Lock Scripts
Repo: https://github.com/cryptape/ckb-combine-lock-poc

Combine multiple lock scripts: "2-of-3 multisig OR time-locked single sig OR oracle condition". Lock scripts become composable building blocks.

Embedded relevance:
- Hardware device = one key in a multisig scheme
- Time-locked fallback: if device goes offline 30 days, recovery key activates
- Oracle condition: unlock when sensor reading meets a threshold (IoT payment automation)

---

### rvv-encoder + rvv-prototype — RISC-V Vector Extensions
Repos: cryptape/rvv-encoder, cryptape/rvv-prototype

RISC-V V extension encoder and prototype. Feeds into CKB-VM gaining RVV (vector) support.

Embedded angle: ESP32-P4 is RISC-V with vector-capable extensions. If CKB-VM gains RVV, crypto operations on-chain get SIMD acceleration. The encoder is useful for writing optimised CKB scripts targeting vector hardware. Directly relevant to the CKB-VM on P4 moonshot.

---

### open-tx research — Partially Signed Transactions
Repo: https://github.com/cryptape/ckb-open-tx-research

"Open transactions" — partially signed CKB transactions completable by others. Like Bitcoin's PSBT. Enables DEX, atomic swaps, collaborative tx construction.

Embedded angle: An ESP32 generates a partial tx ("sell this sensor reading for 10 CKB") and broadcasts it. Any buyer completes the transaction. No server, no escrow, no trust. Foundation for trustless IoT commerce.

---

## Infrastructure Worth Knowing

| Repo | What it is |
|---|---|
| `ckb-script-templates` | Capsule contract templates — use for all new CKB scripts |
| `fiber-dashboard` | Fiber node dashboard — reference for our Fiber kiosk UI |
| `ckb-smt-tool` | SMT integration helpers — reference for WySMT |
| `ckb-discovery` | P2P peer discovery — reference for light client |
| `ckb-unisat-poc` | Bitcoin wallet (UniSat/OKX) on CKB — multi-chain auth PoC |
| `ckb-js-pokemon` | NFT + gaming on CKB — Kabletop-era reference |
| `hotstuff` | HotStuff BFT consensus (Rust) |
| `bft-rs` | BFT protocol library |

---

## Key Insights — Roadmap Impact

**SPHINCS+ is production-ready on CKB today.** The on-chain side is done, audited, deployed. We only need the ESP32-P4 signing side. Move from moonshot → Later.

**Omnilock is WyAuth's on-chain target.** No new lock script needed. WyAuth device library + Omnilock on-chain = complete multi-chain signing stack.

**TCoB is the UX layer for hardware wallets.** Add to WyMolecule scope — structured display data for embedded signing devices.

**Nostr-binding bridges mesh identity to on-chain assets.** BitChat + nostr-binding = mesh chat nodes that can own CKB. New item to add to roadmap.

**ZK verification proven viable (~130M cycles).** Gaming + IoT applications. ZK Kabletop (provably fair, private moves) becomes a concrete research direction.

**Open transactions enable trustless IoT commerce.** ESP32 generates sell offers, anyone completes them. No intermediary. New embedded use case.

---

## New Roadmap Items from This Research

Upgrade existing:
- WyAuth → target Omnilock on-chain (no new lock script)
- SPHINCS+ → move from Moonshot to Later (on-chain done)
- WyMolecule → include TCoB structured signing

New additions:
- Nostr-binding integration (BitChat/Nostr identity → CKB cell ownership)
- Open transactions on embedded (ESP32 partial tx generation)
- ZK gaming proofs (Kabletop + Halo2 zkVM pattern)
