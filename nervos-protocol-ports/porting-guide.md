# Nervos Network — Embedded C++ Port Opportunities
*Research: 2026-03-02 | Focus: ESP32 / SBC embedded deployment*

---

## TL;DR Priority Stack

| Priority | Repo | What it is | Port effort | Value |
|---|---|---|---|---|
| 🔥 HIGH | `molecule` | Wire serialization for all CKB data | Low — pure logic | Essential for any on-chain tx construction |
| 🔥 HIGH | `golomb-coded-set` | BIP158 compact block filters | Low — pure math | Light client tx detection without full node |
| 🔥 HIGH | `sparse-merkle-tree` | SMT proofs for state verification | Medium | Verify CKB state on-device |
| 🟡 MED | `merkle-mountain-range` | MMR for header chain proofs | Medium | Trustless header verification |
| 🟡 MED | `ckb-auth` (C already!) | Multi-chain auth (BTC/ETH/CKB sigs) | Already C — wrap it | Sign CKB txs with Bitcoin/ETH keys |
| 🟡 MED | `ckb-c-stdlib` | CKB data structures + Blake2b/Keccak | Already C — use directly | Core primitives, zero port work |
| 🟢 LOW | `fiber-sphinx` | Onion routing for Fiber payments | High — complex crypto | Privacy for ESP32 payment nodes |
| 🟢 LOW | `tentacle` | P2P networking (yamux multiplexed) | High — async Rust | Full P2P node on SBC (not ESP32) |
| 🟢 LOW | `overlord` | BFT consensus protocol | Very high | Future: embedded validator |
| 🔵 LATER | `ckb-vm` | RISC-V VM | Extreme | Run CKB scripts on-device |

---

## 1. 🔥 Molecule — Wire Format Serialization

**Repo:** `nervosnetwork/molecule`  
**Language:** Rust (+ C bindings in `bindings/c/`)  
**What it is:** CKB's canonical serialization format. Every transaction, cell, script, and header on CKB is encoded in Molecule format. It's like Protocol Buffers but simpler — fixed-layout structs with deterministic encoding.

**Why it matters for embedded:**
Every time we construct or parse a CKB transaction on-device (light client, POS terminal, hardware wallet) we need Molecule. Currently our light client uses hand-rolled byte manipulation. A proper Molecule C++ port would:
- Let us construct valid transactions entirely on-device
- Parse incoming headers/cells without relying on JSON-RPC
- Enable hardware wallet signing (sign raw Molecule-encoded tx)

**Port assessment:**
- Pure logic, no I/O — perfect for embedded
- Molecule already has a C binding in the repo (`bindings/c/`) — we could wrap that directly
- The generated code is just struct layout + reader/builder macros — very portable
- ESP32 ram budget: a typical tx is ~500 bytes, well within PSRAM range
- `molecule` compiler generates C from `.mol` schema files — we'd run it once, commit the generated headers

**What we'd build:** `WyMolecule.h` — generated C++ wrappers around Molecule-encoded CKB types (Transaction, RawTransaction, CellInput, CellOutput, Script, Header). Drop-in for anywhere we currently do manual byte packing.

---

## 2. 🔥 Golomb-Coded Set (BIP158 Compact Block Filters)

**Repo:** `nervosnetwork/golomb-coded-set`  
**Language:** Rust (adapted from rust-bitcoin)  
**What it is:** BIP158 is the standard for compact block filters — a probabilistic data structure that lets a light client ask "does this block contain transactions relevant to my address?" without downloading the full block. CKB light client uses this.

**Why it matters for embedded:**
This is the missing piece for truly autonomous ESP32 light nodes. Right now our light client asks the full node to filter for us (trusts the node). With a GCS implementation on-device:
- ESP32 downloads block filter (~1KB vs ~50KB full block)
- Tests against own address locally
- Only downloads full block if filter matches
- Zero trust, works over any connection

**Port assessment:**
- Pure math — Golomb-Rice coding, hash function, bit manipulation
- No heap allocation required if we use fixed-size filter
- The Rust code is ~400 lines — straightforward port
- Bitcoin already has this in C (libbitcoinkernel) — reference available
- ESP32 time budget: filter check is microseconds

**What we'd build:** `WyGCS.h` — BIP158 GCS encode/decode/match. Feed it a filter and an address, get back a bool. Slot into the light client's filter-checking phase.

---

## 3. 🔥 Sparse Merkle Tree

**Repo:** `nervosnetwork/sparse-merkle-tree`  
**Language:** Rust (`no_std` supported!)  
**What it is:** An optimized sparse merkle tree (SMT) used by CKB for state proofs. `no_std` support means it was literally designed for embedded/constrained environments.

**Why it matters for embedded:**
CKB's layer 2 protocols (Fiber scripts, XUDT, RGB++) use SMT proofs to verify state without a full node. An ESP32 with SMT support could:
- Verify a Fiber payment channel state proof on-device
- Verify XUDT (stablecoin) balances from a compact proof
- Act as a trustless payment terminal — verify receipt proofs locally

**Port assessment:**
- `no_std` Rust → already targets embedded, port to C++ is mechanical
- Core logic: hash left+right nodes up the tree, compare root
- Already has C bindings in `c/` subdirectory
- Memory: proof size is O(log N) = 256 bytes for 256-bit tree
- CKB uses Blake2b as the hash — we already have that

**What we'd build:** `WySMT.h` — verify SMT inclusion/exclusion proofs. Primary use: verify Fiber channel state and XUDT balances on-device.

---

## 4. 🟡 Merkle Mountain Range

**Repo:** `nervosnetwork/merkle-mountain-range`  
**Language:** Rust  
**What it is:** An append-only authenticated data structure used in CKB for the header chain. An MMR proof lets you prove that a block header is part of the canonical chain without downloading all headers.

**Why it matters for embedded:**
The light client uses MMR to verify the header chain. Currently we trust the connected node to give us valid headers. With on-device MMR verification:
- Verify any historical CKB header is genuine
- Trustless header chain sync — can detect dishonest nodes
- Checkpoint-based sync: download one MMR proof, verify thousands of headers

**Port assessment:**
- Medium complexity — tree structure with bagging peaks algorithm
- No I/O, pure computation
- Memory scales with chain height: ~O(log N) nodes to keep in RAM
- Need persistent storage for accumulated peaks (NVMe on SBC, PSRAM on ESP32)

**What we'd build:** `WyMMR.h` — verify MMR inclusion proofs, accumulate new peaks as headers arrive. Plug into light client's header verification phase.

---

## 5. 🟡 ckb-auth — Already C, Use Directly

**Repo:** `nervosnetwork/ckb-auth`  
**Language:** C (the important parts are already C in `c/` directory)  
**What it is:** A library supporting authentication via multiple blockchain key formats — Bitcoin secp256k1, Ethereum keccak+secp256k1, CKB native, Cardano Ed25519, Solana, Ripple, and more.

**Why it matters for embedded:**
This is huge. It means an ESP32 payment device could:
- **Accept payment to a Bitcoin address** — user signs with their Bitcoin wallet
- **Accept payment to an Ethereum address** — MetaMask, hardware wallets
- **No CKB wallet required** by the payer

Phill's POS system becomes cross-chain natively — QR shows a CKB address, user pays with any of 12 blockchain wallets.

**Port assessment:**
- Already C — `c/auth.c` and `c/ckb_auth.h` are the core
- Depends on secp256k1 (already in our stack via trezor-crypto)
- Keccak256 already in ckb-c-stdlib
- Ed25519 already in trezor-crypto
- We essentially just need to wire the auth.c dispatch logic into our C++ layer
- **Lowest effort, highest impact** item on this list

**What we'd build:** `WyAuth.h` — wraps ckb-auth's C implementation, exposes `verifySignature(pubkeyHash, message, signature, authType)` where authType is Bitcoin/Ethereum/CKB/etc. Drop into POS system and hardware wallet.

---

## 6. 🟡 ckb-c-stdlib — Already C, Already Useful

**Repo:** `nervosnetwork/ckb-c-stdlib`  
**Language:** C  
**What it is:** CKB's official C standard library for script development. Contains:
- Blake2b implementation (`blake2b.h`)
- Blake3 (`blake3.h`)
- Keccak256 (`ckb_keccak256.h`)
- Molecule C bindings (`molecule/`)
- Type ID utilities
- CKB data structure definitions

**Port assessment:**
- Zero port work — it's already C header-only
- We already use some of this (Blake2b from trezor-crypto)
- The Molecule C bindings here are the same ones referenced above
- Just add as a submodule to CKB-ESP32 and include

**What we'd build:** Add as submodule to `wyltek-embedded-builder`, expose via `WyCKBStdlib.h` shim for Arduino compatibility.

---

## 7. 🟢 fiber-sphinx — Onion Routing

**Repo:** `nervosnetwork/fiber-sphinx`  
**Language:** Rust  
**What it is:** Sphinx onion packet construction/processing for Fiber Network. Same onion routing used by Lightning Network — wraps each payment hop in encrypted layers so no single node knows both sender and recipient.

**Why it matters for embedded:**
An ESP32 Fiber node with Sphinx support would:
- Route payments with full privacy (no hop knows the full path)
- Be a proper Fiber network participant, not just an endpoint
- Enable private micropayment terminals

**Port assessment:**
- Hard — ChaCha20-Poly1305 + ECDH key agreement + multi-layer encryption
- ChaCha20 exists in mbedTLS (already on ESP32) — primitive is available
- The protocol logic itself is ~600 lines of Rust
- More SBC territory than ESP32 (we already run Fiber on N100/ckbnode)
- **Good candidate for a future SBC library, lower priority for ESP32**

---

## 8. 🟢 tentacle — P2P Networking

**Repo:** `nervosnetwork/tentacle`  
**Language:** Rust (async, tokio-based)  
**What it is:** CKB's custom P2P networking stack — yamux stream multiplexing over TCP, with pluggable security (secio/noise) and custom protocol mounting. This is how CKB nodes talk to each other.

**Why it matters for embedded:**
With tentacle, an SBC could speak the native CKB P2P protocol — no JSON-RPC dependency, direct protocol participation. An ESP32 could potentially speak a subset (light client protocol only).

**Port assessment:**
- Very hard for full port — async Rust with tokio, not portable
- For SBC (N100, OPi5+): could compile Rust binary natively — no port needed
- For ESP32: subset port possible (just the light client protocol messages) but significant work
- Lower priority — our current HTTP/JSON-RPC approach works fine for now

---

## 9. 🔵 ckb-vm — RISC-V Interpreter

**Repo:** `nervosnetwork/ckb-vm`  
**Language:** Rust  
**What it is:** CKB's RISC-V virtual machine — runs CKB lock/type scripts. Every on-chain computation happens in this VM.

**Why it matters (someday):**
Running CKB-VM on an ESP32 or SBC would mean you could execute CKB scripts locally — verify transactions completely independently, without trusting any node. True trustless verification.

**Port assessment:**
- Extreme effort — full RISC-V interpreter
- ESP32-P4 is powerful enough (hardware SHA, 32MB PSRAM) but RAM budget is tight
- There's a prior art: `ckb-vm-aot` (AOT compilation) — might be more feasible
- Realistic on N100/OPi5+, exotic on ESP32
- **Long-term moonshot — log it and revisit**

---

## Bonus: Non-Nervos but Relevant

### `faster-hex` (nervosnetwork fork)
Fast hex encoding/decoding. We do a lot of hex in CKB. Already in our stack via trezor, but this has SIMD-accelerated paths useful on N100/Ryzen builds.

### Bitcoin BIP158 Reference (rust-bitcoin)
The GCS above was adapted from rust-bitcoin. The original C implementation in Bitcoin Core is also portable and well-tested — alternative source if the Nervos port has gaps.

### PTLC vs HTLC (Fiber uses PTLC)
Fiber uses Point Time Lock Contracts (Schnorr/adaptor signatures) instead of Hash Time Lock Contracts. More private than Lightning. Adaptor signature primitives would be a valuable addition to `CKB-ESP32` — enables offline payment channel settlement proofs.

---

## Recommended Build Order

1. **Now (slot into CKB-ESP32):**
   - `ckb-c-stdlib` as submodule (zero work, immediate primitives)
   - `WyAuth.h` wrapping ckb-auth C (low effort, cross-chain POS)
   - `WyMolecule.h` — run moleculec, commit generated headers

2. **Next sprint:**
   - `WyGCS.h` — GCS filter matching (completes trustless light client)
   - `WySMT.h` — SMT proof verification (enables Fiber state on-device)

3. **Later:**
   - `WyMMR.h` — trustless header chain (full no-trust light client)
   - Sphinx (SBC only, Fiber privacy)

4. **Moonshot:**
   - ckb-vm port to ESP32-P4

---

## Files
- This document: `research/nervos-embedded-opportunities.md`
- Related: `research/bitchat-integration.md`
