# Fiber Network for Gaming — Research

*Research: 2026-03-02*

## The question

Can Fiber Network carry arbitrary data as well as payments — specifically real-time gaming state?

**Short answer:** Not natively, but there are three legitimate paths, and one of them is genuinely novel.

---

## What Fiber actually is at the protocol level

Fiber is a Lightning Network-style payment channel protocol on CKB. Its P2P messages carry:
- Channel state updates (balance deltas)
- TLC (Time Lock Contract) hashes and preimages
- Schnorr partial signatures
- Routing onion packets (Sphinx)

There is no arbitrary data field in the core protocol. But the underlying transport (Tentacle P2P + Noise encryption) is designed to be extensible.

---

## Option 1 — Onion message piggyback (most protocol-native)

Lightning Network added "onion messages" in BOLT 12 — arbitrary data routed through the same Sphinx onion structure, at zero cost, with no payment attached. Fiber is designed for Lightning Network compatibility and `fiber-sphinx` is already implemented.

**How it works for gaming:**
- Two players open a Fiber channel (game session begins)
- Game state updates are routed as onion messages between nodes
- No payment per message — just authenticated, encrypted data
- Latency: P2P hop latency, roughly 20–100ms per hop (fine for turn-based)

**Status:** BOLT 12 onion messages are on Fiber's Lightning compatibility roadmap. Not implemented yet. This is the "right" path architecturally — worth watching.

---

## Option 2 — Micropayment per game event (most economically interesting)

Each game state update *is* a micropayment. 0.0001 CKB per packet, settled off-chain instantly via channel balance update.

**The elegant properties this creates:**
- Cheating has a cost — sending invalid state costs you the micropayment
- Winning earns money in real time — balance shifts with each move
- The channel IS the economic game state — no separate ledger needed
- Rage-quitting means forfeiting locked funds (enforced by the timelock)
- Settlement is automatic — close channel, on-chain result is final

**Real-world precedent:** Satoshi's Place on Lightning, various Lightning-native games. Fiber enables this at far lower fees than Lightning (CKB economics are more favourable for tiny amounts).

**Suitable for:** Turn-based games, card games, chess, strategy, any game where moves are discrete events.

---

## Option 3 — Custom protocol over Tentacle P2P (fastest to prototype)

Fiber nodes connect via Tentacle, Nervos' custom P2P stack. Tentacle explicitly supports mounting multiple protocols on the same connection — each protocol gets its own ID and message handler, sharing the underlying encrypted TCP connection.

**How it works:**
- Extend the Fiber node with a custom `GameProtocol` mounted on the existing Tentacle session
- Arbitrary bytes, fully encrypted (Noise XX), authenticated by the same keys used for the payment channel
- No payment required per message
- Game settlement triggered by Fiber payment at the end

**Suitable for:** Any latency class where P2P TCP is acceptable — turn-based through to casual real-time. Not FPS.

**Implementation path:** Requires modifying/extending the Fiber node software. Rust, using the Tentacle library. Harder than Options 1/2 but most flexible.

---

## The genuinely novel idea: channel-as-game-session

> *The payment channel IS the game session.*

This maps cleanly onto channel semantics:

| Game event | Channel event |
|---|---|
| Game lobby / matchmaking | Channel negotiation |
| Game starts | `OpenChannel` — both players lock funds |
| Player makes a move | Off-chain balance update (or micropayment) |
| Player wins a round | Balance shifts in their favour |
| Game ends (normal) | Cooperative `CloseChannel` — final score settled |
| Player rage-quits | Unilateral close — timelock punishes the quitter |
| Cheating attempt | Invalid state rejected, PTLC ensures honest settlement |

**The rage-quit problem is solved by design.** In normal multiplayer games, a losing player can just disconnect. With a Fiber channel, disconnecting mid-game means the timelock expires and the other player claims the full locked amount. The protocol enforces honest completion.

**Nobody has built this on Fiber yet.** Lightning Network has some game experiments but Fiber's lower fees and PTLC privacy make it a better fit.

---

## Latency reality check

| Game type | Latency needed | Fiber suitable? |
|---|---|---|
| Turn-based (chess, cards) | Seconds | ✅ Yes — native |
| Strategy (real-time lite) | 200–500ms | ✅ Yes — with side channel |
| Casual real-time | 50–150ms | 🟡 Maybe — direct P2P path |
| Competitive (MOBA, RTS) | 20–50ms | ⚠️ Marginal |
| FPS / twitch | <20ms | ❌ Wrong tool |

**For fast real-time:** Use WebRTC/UDP for game state, settle winnings via Fiber at game end. Best of both worlds — low latency gameplay, trustless settlement.

---

## What would need to be built

### Minimum viable: turn-based game on Fiber
1. Game client that talks to a Fiber node via RPC
2. Move encoding as payment memo or micropayment amount
3. Game state machine that maps channel balance to game score
4. UI that shows both game state and channel balance simultaneously

**Stack:** Node.js or Python client + existing Fiber RPC. No protocol changes needed.

### Full channel-as-game-session
1. Game-specific channel open parameters (timelock = game timeout)
2. State machine for valid moves encoded in TLC conditions
3. Dispute resolution via on-chain CKB script if player claims invalid state
4. Referee contract (optional) — third-party Fiber node acts as neutral arbiter

**Stack:** CKB script (RISC-V C or Rust) for dispute resolution + Fiber node extension.

### Arbitrary data transport (future)
Wait for BOLT 12 onion messages in Fiber, then build game state transport on top. Zero protocol modification needed on the game side.

---

## Related

- [BitChat CKB integration](./bitchat-ckb-integration.md) — another approach to P2P data + payments on embedded hardware
- [Fiber Network repo](https://github.com/nervosnetwork/fiber)
- [fiber-sphinx](https://github.com/nervosnetwork/fiber-sphinx) — Sphinx onion implementation (prerequisite for onion messages)
- [BOLT 12 spec](https://bolt12.org) — Lightning onion messages reference
