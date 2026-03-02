# Fiber Network for Gaming — Research

*Research: 2026-03-02*

## Prior Art: Kabletop (Cryptape)

Before going further — Cryptape (the team behind CKB) already built a complete turn-based gaming framework on CKB payment channels called **Kabletop**. This is the most important reference for anyone building games on CKB.

**Repos:**
- [`cryptape/kabletop-contracts`](https://github.com/cryptape/kabletop-contracts) — CKB smart contracts (NFT, Wallet, Payment, Kabletop channel contract)
- [`cryptape/kabletop-ckb-sdk`](https://github.com/cryptape/kabletop-ckb-sdk) — Rust SDK: CKB tx construction, built-in P2P module, wallet manager
- [`cryptape/kabletop-godot`](https://github.com/cryptape/kabletop-godot) — Godot engine bindings via gdnative (`.dll`/`.so`/`.dylib`)
- [`cryptape/kabletop-demo`](https://github.com/cryptape/kabletop-demo) — Playable two-player card game (Slay the Spire/Hearthstone inspired)
- [`cryptape/kabletop-godot-relayserver`](https://github.com/cryptape/kabletop-godot-relayserver) — NAT-traversal relay server for P2P matchmaking

**What Kabletop proved:**
- Game logic runs in **Lua** — compiled into binaries, deployed on-chain via CKB-VM
- The Kabletop contract validates game state transitions on-chain when disputed
- Channels are state channels — all moves are off-chain, only open/close/dispute hits the chain
- NFTs (cards) are CKB cells — owning more cards means more CKB staked as storage rent (elegant tokenomics)
- P2P is direct node-to-node, relay server handles NAT traversal
- Godot engine supported on Windows/Linux/macOS

**Current status:** Development stalled — last commits circa 2021-2022, deployed on testnet. Not production-ready, but the contract architecture and SDK are a solid foundation to build from.

---

## The Question

Can Fiber Network carry arbitrary data as well as payments — specifically real-time gaming state?

**Short answer:** Not natively in Fiber's current protocol, but Kabletop's architecture shows the right approach: state channels with on-chain dispute resolution. Fiber extends this with better cryptography (PTLC vs HTLC), multi-asset support, and a more mature network.

---

## What Kabletop Showed Works

### State channel architecture
```
Open channel (lock funds) → off-chain game moves → close channel (settle)
```
Every game move is a signed state update exchanged P2P. No on-chain tx per move. The contract only runs if there's a dispute — one player claims the other cheated and submits the last valid state on-chain.

### Lua for game logic — brilliant design decision
Game rules are written in Lua, compiled to bytecode, deployed as CKB cells. The Kabletop contract runs the Lua VM to validate any disputed state. This means:
- Game rules are on-chain verifiable
- No trust in either player's client
- Updating game rules = deploying new Lua bytecode cell
- Any game expressible in Lua works with the same contract infrastructure

### NFT-as-stake
Cards are NFTs stored in CKB cells. Owning more cards = more CKB locked as storage. This creates natural scarcity without artificial supply caps — the network's storage economics enforce it.

### Relay server for NAT traversal
Direct P2P doesn't work behind NAT (most home networks). Kabletop's relay server is a simple matchmaking/relay layer — clients register, connect via relay, then ideally upgrade to direct. Same pattern as WebRTC signaling.

---

## How Fiber Improves on Kabletop

| Feature | Kabletop (2021) | Fiber (2024) |
|---|---|---|
| Crypto | HTLC | PTLC (Schnorr, more private) |
| Assets | CKB + custom NFT | CKB + any xUDT/RGB++ asset |
| Network | Custom P2P | Tentacle (production-grade) |
| Lightning compat | No | Yes (cross-chain payments) |
| Onion routing | No | Yes (Sphinx) |
| Watchtower | No | Yes |
| Status | Stalled | Active development |

A Fiber-native gaming framework would inherit all of this — private moves via PTLC, any asset as stake, Lightning Network interoperability for cross-chain tournaments.

---

## Three Approaches for Gaming on Fiber

### Option 1 — State channels (Kabletop pattern, updated for Fiber)

Port Kabletop's architecture to Fiber:
- Replace HTLC channel logic with Fiber's PTLC channels
- Keep Lua game logic on-chain for dispute resolution
- Use Fiber's Tentacle P2P for move exchange
- Use Fiber's existing watchtower for channel monitoring

**This is the right path.** Kabletop already proved the architecture works.

### Option 2 — Micropayment per game event

Each move is a Fiber payment — 0.0001 CKB per move, balance shifts with game state. No separate dispute contract needed — the channel balance IS the score.

**Elegant but limited** — works for simple zero-sum games (chess, checkers), less clean for complex card games where state is more than a number.

### Option 3 — Custom protocol over Tentacle P2P

Mount a custom game protocol on the existing Tentacle connection between Fiber nodes. Arbitrary bytes, fully encrypted, same keys as the payment channel. Settlement via Fiber payment at game end.

**Fastest to prototype** — no on-chain contract needed for simple games.

---

## What a Modern CKB Gaming SDK Would Look Like

Building on Kabletop's lessons, a 2024-era SDK would:

```
ckb-game-sdk/
├── contracts/           # CKB scripts (Rust/C)
│   ├── game-channel/    # state channel open/close/dispute
│   └── game-vm/         # Lua VM for on-chain game validation
├── sdk/                 # Rust SDK
│   ├── channel.rs       # Fiber channel management
│   ├── p2p.rs           # Tentacle P2P + relay
│   ├── game.rs          # game state signing + verification
│   └── lua.rs           # Lua game logic execution
├── bindings/
│   ├── godot/           # GDNative/GDExtension (Godot 4)
│   ├── unity/           # C# bindings
│   └── js/              # WebAssembly for browser games
└── examples/
    ├── card-game/       # Kabletop-style card game
    └── chess/           # Simple turn-based
```

**Key differences from Kabletop:**
- Godot 4 (GDExtension instead of GDNative — Godot 4 API)
- Fiber channels instead of raw CKB channels
- xUDT asset stakes (stablecoins, not just CKB)
- WebAssembly bindings for browser-playable games
- Relay server upgraded to handle Fiber's Tentacle protocol

---

## Latency Reality Check

| Game type | Latency needed | Suitable approach |
|---|---|---|
| Turn-based (chess, cards) | Seconds | ✅ State channels — full Kabletop pattern |
| Strategy (lite real-time) | 200–500ms | ✅ State channels + Tentacle P2P |
| Casual real-time | 50–150ms | 🟡 Tentacle side channel, settle at end |
| Competitive (MOBA, RTS) | 20–50ms | ⚠️ WebRTC for game, Fiber for settlement |
| FPS / twitch | <20ms | ❌ Wrong tool — use WebRTC, settle via Fiber |

---

## The Rage-Quit Problem — Solved by Design

In normal multiplayer games, losing players disconnect. In a state channel game:

| Scenario | What happens |
|---|---|
| Normal game end | Cooperative close — instant, no dispute |
| Player rage-quits | Other player submits last signed state on-chain |
| Cheating attempt | Submitting invalid state triggers dispute — Lua VM runs on-chain and rejects it |
| Player goes offline | Timelock expires — other player claims the channel |

The protocol enforces honest play. No moderators, no appeals, no refunds — the contract is the referee.

---

## What Needs Building

### To revive Kabletop for modern CKB:
1. Port contracts from old CKB script API to current (capsule v0.10+)
2. Update SDK to use Fiber RPC instead of raw CKB channels
3. Rebuild Godot bindings for Godot 4 (GDExtension)
4. Ship a simple demo game (chess or card game)

**Estimated effort:** 4-8 weeks for a team that knows Rust + CKB

### To build something new from scratch using Fiber today:
1. Write a Fiber-aware game channel contract (CKB script)
2. Build a simple Node.js or Rust SDK wrapping Fiber RPC
3. Write a turn-based game client (web or native)
4. Minimal relay server for matchmaking

**Estimated effort:** 2-4 weeks for MVP, browser-based card game

---

## Related

- [Kabletop contracts](https://github.com/cryptape/kabletop-contracts)
- [Kabletop demo](https://github.com/cryptape/kabletop-demo)
- [Fiber Network repo](https://github.com/nervosnetwork/fiber)
- [fiber-sphinx](https://github.com/nervosnetwork/fiber-sphinx) — onion routing (privacy for game moves)
- [BOLT 12 spec](https://bolt12.org) — onion messages reference
- [BitChat CKB integration](./bitchat-ckb-integration.md) — P2P data + payments on embedded
