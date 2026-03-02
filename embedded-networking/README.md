# Embedded Networking — Research

Getting blockchain data to and from devices that aren't always connected, and building mesh networks for offline-capable payment infrastructure.

## Topics

### BitChat + CKB Integration
See [`bitchat-ckb-integration.md`](./bitchat-ckb-integration.md)

BitChat (Jack Dorsey, July 2025) is an open-source BLE mesh chat protocol using Noise_XX encryption and Ed25519 signing. All the crypto primitives are already in our embedded stack (trezor-crypto).

Key opportunity: nobody has shipped a LoRa bridge for BitChat yet. A T-Deck (BLE + LoRa SX1262 + keyboard + display) could bridge the BLE mesh to LoRa, extending range to kilometres.

CKB integration angle: BitChat identity fingerprints map naturally to CKB lock scripts — enabling payments to mesh chat identities.

### LoRa Transport for CKB Light Client
The CKB light client in `ckb-light-esp` has a transport-agnostic design. LoRa transport profile is stubbed — completing it would enable CKB light client operation in areas without WiFi or cellular.

### Offline Payment Channels
Fiber's PTLC-based payment channels can be settled offline if both parties have the channel state. Research into offline settlement proof construction on embedded devices.
