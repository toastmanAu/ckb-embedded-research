# Nervos Protocol Ports — Research

Analysis of which Nervos Network libraries are worth porting to C++ for embedded use, and what each port involves.

## The opportunity

Nervos has written a lot of excellent Rust (and some C) that implements CKB's core protocols. Most of it was never intended for embedded — but a surprising amount ports cleanly.

The goal: a suite of `Wy*` C++ header libraries that let an ESP32 or SBC participate in the CKB ecosystem without depending on any trusted node.

## Priority breakdown

See [`porting-guide.md`](./porting-guide.md) for the full analysis. Summary:

**Grab now (already C, near-zero work):**
- `ckb-c-stdlib` — Blake2b, Keccak256, Molecule C headers. Just add as submodule.
- `ckb-auth` — C implementation of 12-blockchain signature verification. Bitcoin keys, ETH keys, Cardano, Solana — all signing CKB transactions.

**Port next (~400 lines each, pure logic):**
- `molecule` — CKB wire format. Run their compiler, get C headers. Enables on-device transaction construction.
- `golomb-coded-set` — BIP158 compact block filters. The missing piece for a trustless light client.
- `sparse-merkle-tree` — Already has `no_std` + C bindings. Verify Fiber channel state on-device.

**Later:**
- `merkle-mountain-range` — trustless header chain proofs
- `fiber-sphinx` — onion routing privacy (SBC territory)

**Moonshot:**
- `ckb-vm` — run CKB scripts locally. Wild on ESP32, plausible on ESP32-P4.
