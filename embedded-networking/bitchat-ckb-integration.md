# BitChat Integration Research
_Last updated: 2026-03-01_

## What is BitChat

Jack Dorsey / Block, Inc. — announced July 2025. Open source (Unlicense).
Decentralised BLE mesh chat. No servers, no accounts, no phone numbers.
iOS + Android native apps. Reddit/community growing fast.

Repo: https://github.com/permissionlesstech/bitchat

## Protocol Stack

```
Application:  BitchatMessage (nickname, content, UUID, mentions)
Session:      BitchatPacket  (type, TTL, sender/recipient ID, flags)
Encryption:   Noise_XX_25519_ChaChaPoly_SHA256
Transport:    BLE (GATT), Wi-Fi Direct (planned), LoRa (community)
```

## Wire Format (BinaryProtocol.swift)

### BitchatPacket (V1 header = 14 bytes)

| Field | Size | Notes |
|---|---|---|
| version | 1 | 0x01 = V1, 0x02 = V2 (4-byte payloadLen + routing) |
| type | 1 | see MessageType below |
| ttl | 1 | decremented at each hop |
| timestamp | 8 | milliseconds since Unix epoch, big-endian |
| flags | 1 | HAS_RECIPIENT=0x01, HAS_SIG=0x02, COMPRESSED=0x04, HAS_ROUTE=0x08, IS_RSR=0x10 |
| payloadLen | 2 (V1) / 4 (V2) | |
| senderID | 8 | always present |
| recipientID | 8 (opt) | broadcast = all 0xFF |
| payload | variable | |
| signature | 64 (opt) | Ed25519 |

Padding: PKCS#7-style to nearest block: 256/512/1024/2048 bytes.

### MessageType enum

| Value | Name | Description |
|---|---|---|
| 0x01 | announce | "I'm here" + nickname + Noise fingerprint |
| 0x02 | message | Public broadcast chat |
| 0x03 | leave | "I'm leaving" |
| 0x10 | noiseHandshake | Noise XX init or response |
| 0x11 | noiseEncrypted | Private msg/receipts (encrypted) |
| 0x20 | fragment | Large message chunk |
| 0x21 | requestSync | GCS filter sync |
| 0x22 | fileTransfer | Binary payloads |

### BitchatMessage payload

| Field | Size | Notes |
|---|---|---|
| flags | 1 | isRelay=0x01, isPrivate=0x02, hasOrigSender=0x04, hasRecipNick=0x08 |
| timestamp | 8 | ms |
| id | 1+len | UUID string, 1-byte length prefix |
| sender | 1+len | nickname |
| content | 2+len | UTF-8, 2-byte length prefix |
| origSender | 1+len (opt) | |
| recipNick | 1+len (opt) | |

## Identity & Crypto

- **Noise static key**: Curve25519 keypair (long-term identity)
- **Signing key**: Ed25519 (for announces + authentication)
- **Fingerprint**: SHA-256(noise_static_pub_key) — 32 bytes
- **Noise protocol**: `Noise_XX_25519_ChaChaPoly_SHA256`
  - XX = mutual auth, no pre-shared keys
  - 3-message handshake: `→e`, `←e,ee,s,es`, `→s,se`
  - Transport keys derived from final handshake state
- **Private messages**: Noise transport encryption (inner BitchatMessage)
- **Relay nodes**: forward encrypted Noise envelope opaquely (can't read private msgs)

## BLE GATT UUIDs (from AppConstants.kt)

- SERVICE: `F47B5E2D-4A9E-4C5A-9B3F-8E1D2C3A4B5C`
- CHARACTERISTIC: `A1B2C3D4-E5F6-4A5B-8C9D-0E1F2A3B4C5D`
- DESCRIPTOR: `00002902-0000-1000-8000-00805f9b34fb` (standard CCCD)
- Same on iOS + Android (confirmed in source comments)

## Routing (whitepaper §7)

**Gossip flood with Bloom dedup:**
1. Receive packet
2. Check Bloom filter (seen before?) → drop if yes
3. Add to Bloom filter
4. Dispatch to app if for-us or broadcast
5. Decrement TTL. If TTL > 0 → relay to all peers except source

**TTL defaults:** 7 (message), 3 (announce/leave), 1 (direct known peer)

**Private messages:** relay nodes forward the Noise-encrypted envelope without
decrypting. Only the intended recipient (who holds the Noise session key) can read it.

## Fragmentation

- BLE MTU: ~512 bytes (ATT layer max on modern devices, often 247 bytes)
- BitChat fragments at ~469 bytes
- Fragment packet types: fragment(0x20) — simplified to single type (not start/continue/end)
- LoRa MTU: 255 bytes — needs full reassemble/re-fragment at bridge

## Our Implementation (src/bitchat/)

### Done (commit 144f9b7)

**bitchat_packet.h / .cpp** — Wire codec
- `bc_packet_encode()` / `bc_packet_decode()` — full V1 packet codec
- `bc_message_encode()` / `bc_message_decode()` — application message
- `bc_announce_encode()` / `bc_announce_decode()` — presence packets
- `bc_pad()` / `bc_unpad()` — PKCS#7 block padding
- Zero-copy decode (payload ptr into caller buffer)
- `padding=false` for LoRa (no MTU headroom)

**bitchat_mesh.h / .cpp** — Relay engine
- `BitchatMesh` — full mesh state machine
- `BitchatBloom` — 512-bit Bloom filter, 3 hash functions, 5-min rotation
- Peer table: 16 peers, LRU eviction, 2-min timeout
- `bc_mesh_receive()` — decode → dedup → echo-suppress → dispatch → relay (TTL--)
- `bc_mesh_send_message()` / `send_announce()` / `send_leave()`
- `bc_mesh_tick()` — peer aging + bloom rotation
- Packet builders: `bc_build_message_packet()` / `announce` / `leave`
- Callbacks: `on_message`, `on_peer`, `on_relay`, `on_noise`

108 host tests, all passing.

### Next: BLE Transport Layer (bitchat_ble.h)

On ESP32 with NimBLE (Arduino NimBLE-Arduino library):
- Advertise the BitChat GATT service UUID
- Central + peripheral simultaneously (ESP32 supports this)
- Connect to nearby peers advertising the service
- Write characteristic = send packet to peer
- Notify characteristic = receive packet from peer
- `on_relay` callback → write to all connected peers except source

Estimated: 2-3 days for basic BLE functional relay.

### Next: LoRa Bridge (bitchat_lora.h)

The gap nobody has shipped yet. On T-Deck or T-Beam:
- BLE side: act as BitChat mesh peer (same as above)
- LoRa side: relay packets to distant LoRa nodes
- Bridge: reassemble BLE fragments → re-fragment for LoRa (255-byte MTU)
- LoRa nodes can relay to other LoRa nodes (TTL-based, same flood algorithm)
- Then hand off to BLE on the far end

This is novel — nobody has BLE↔LoRa BitChat bridge yet (community is asking for it).

### Future: Noise Session Layer (bitchat_noise.h)

For private encrypted messages TO the ESP32:
- Implement Noise XX handshake (Curve25519 + ChaCha20-Poly1305)
- `trezor_crypto` already has curve25519 and chacha20
- The XX pattern is 3 messages, stateful
- Would allow ESP32 to send/receive private messages, not just relay

### Future: CKB Payment Layer

- BitChat identity fingerprint = SHA-256(Noise_static_pub_key)
- Could derive a CKB lock script from this fingerprint
- Allows: "pay this BitChat peer" without exchanging addresses
- The T-Deck device would then be: BitChat mesh node + CKB light client + payment terminal

## Key Technical Notes

- **Relay is packet-transparent**: relay nodes don't need a Noise session
- **Echo suppression**: nodes track their own sender_id, drop own packets
- **Bloom filter false positives**: ~0.1% at 512 bits, acceptable (redundant mesh delivery)
- **ESP32 BLE limitations**: can be central + peripheral, but simultaneous connections are limited (~3-7 depending on mode and chip)
- **No server, no registration**: pure P2P — ideal for ESP32 fixed nodes (coffee shops, etc.)

## Integration Priority Order

1. ✅ Wire codec + mesh relay engine (done)
2. BLE transport — NimBLE on ESP32, advertise + scan + connect
3. LoRa bridge — reassemble/re-fragment for 255-byte LoRa MTU
4. Noise session — for private messages TO the ESP32
5. CKB payment layer — identity → lock script mapping

## Boards

- **T-Deck** (in our targets): BLE + LoRa SX1262 + keyboard + display = ideal dev board
- **T-Beam**: BLE + LoRa + GPS — mobile relay node
- **CYD (ESP32-2432S028R)**: BLE only (no LoRa) — can be BLE mesh node with display
