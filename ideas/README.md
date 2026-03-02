# Ideas

Half-baked things that might be worth building. Recorded so they don't evaporate.

---

## NerdMiner Mesh Network
A mesh of NerdMiner CKB devices coordinating nonce ranges via BitChat LoRa bridge. No central server. Each device mines a non-overlapping nonce space. Solution broadcast over mesh, submitted by whichever node has WiFi.

**Feasibility:** High — extranonce partitioning already implemented in stratum proxy. LoRa bridge is the missing piece.

---

## R36S Hardware Wallet
$30 RK3326 handheld + mainline Linux + CKB CLI = pocketable hardware wallet. Physical buttons for confirmation. Air-gap via QR code camera. See hardware-wallets/ for full design.

**Feasibility:** Medium — hardware is right, software needs building.

---

## Cross-Chain POS Terminal
ESP32 touchscreen POS that accepts payment to a CKB address from any blockchain wallet (Bitcoin, Ethereum, Cardano, etc.) using ckb-auth's multi-chain signature verification. Customer scans QR, pays with their existing wallet, merchant receives CKB.

**Feasibility:** High once WyAuth lands — hardware already built (Elecrow ESP32 HMI).

---

## Distributed Solo Mining via LoRa Mesh
Like the NerdMiner mesh but extended: multiple ESP32 miners on different LoRa nodes, coordinated by a gateway that has internet. Enables mining in locations without per-device internet access.

---

## CKB-VM on ESP32-P4
Run CKB's RISC-V virtual machine on RISC-V hardware (ESP32-P4 is RISC-V). Execute CKB lock scripts locally — true trustless transaction verification without any node dependency. P4 has hardware SHA accelerators and 32MB PSRAM.

**Feasibility:** Low-medium — technically possible, significant work. Would be a genuine first.

---

## SPHINCS+ Hardware Wallet on ESP32-P4
Post-quantum CKB signing device. SPHINCS+ is slow in software (~2min on Trezor-class hardware) but ESP32-P4's hardware SHA accelerators could bring it to seconds. First quantum-resistant embedded CKB signer.

**Feasibility:** Medium — depends on benchmarking P4 hardware SHA with SPHINCS+.
