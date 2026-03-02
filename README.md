# CKB Embedded Research

Public research notes from [Wyltek Industries](https://wyltekindustries.com) on deploying CKB blockchain infrastructure on embedded hardware — ESP32 microcontrollers, single-board computers, and low-cost consumer devices.

This isn't polished documentation. It's working notes: what we found, what we tried, what the next steps are. The goal is to save other builders the research time so they can skip straight to building.

## What's here

### [`nervos-protocol-ports/`](./nervos-protocol-ports/)
Analysis of Nervos Network's Rust/C libraries and what it takes to port them to C++ for embedded use. Molecule serialization, BIP158 compact filters, sparse merkle trees, multi-chain authentication — the building blocks for trustless embedded CKB clients.

### [`hardware-wallets/`](./hardware-wallets/)
Research on turning cheap consumer hardware into CKB signing devices. The $30 R36S handheld gaming console as a hardware wallet. SPHINCS+ post-quantum signatures on ESP32-P4. What's feasible, what isn't, and why.

### [`embedded-networking/`](./embedded-networking/)
BitChat BLE mesh integration with CKB light client. LoRa bridging. Getting blockchain data to devices that aren't always online. Offline payment channel settlement.

### [`sbc-deployment/`](./sbc-deployment/)
Running CKB full nodes, Fiber payment channel nodes, and solo mining setups on Orange Pi, Raspberry Pi, and other SBCs. Build systems, Armbian userpatches, image pipelines.

### [`ideas/`](./ideas/)
Half-baked ideas that might be good. Recorded so they don't get lost.

## Philosophy

> *Research that stays private helps nobody.*

Everything here was going to be figured out eventually by someone in the community. We're just publishing our notes so you don't have to repeat the work.

Extrapolate freely. Build on it. If you ship something based on this, a mention is appreciated but not required.

## Related projects

| Repo | What it is |
|---|---|
| [ckb-light-esp](https://github.com/toastmanAu/ckb-light-esp) | First C implementation of CKB light client on ESP32 |
| [NerdMiner_CKB](https://github.com/toastmanAu/NerdMiner_CKB) | ESP32 Eaglesong miner |
| [TelegramSerial](https://github.com/toastmanAu/TelegramSerial) | Remote ESP32 serial debug via Telegram |
| [TelegramOTA](https://github.com/toastmanAu/TelegramOTA) | Remote OTA flash via Telegram |
| [wyltek-embedded-builder](https://github.com/toastmanAu/wyltek-embedded-builder) | PlatformIO board + component library |
| [armbian-ckb-userpatches](https://github.com/toastmanAu/armbian-ckb-userpatches) | Turn any SBC into a CKB node via Armbian |

## Contributing

See something wrong? Have better numbers? Open an issue or PR. This is a living document.

---
*Wyltek Industries — [wyltekindustries.com](https://wyltekindustries.com)*
