# R36S CKB Hardware Wallet — Design Doc
_Started 2026-03-02 overnight_

## Concept
A $30 open-source CKB hardware wallet using the R36S handheld gaming console.
Custom Linux firmware replaces the gaming UI with a CKB light client wallet.

**Tagline:** "A $30 open source CKB hardware wallet with a real screen and buttons."

## Why R36S
- RK3326, 1GB RAM, 3.5" IPS display (640×480), D-pad + ABXY buttons, battery
- Armbian confirmed working — full mainline Linux
- Ships with Linux already — no Android unlock needed
- Phill has 6 units ready to distribute to community
- ~$30 AliExpress price — accessible globally

## Hardware Capabilities
- Display: 3.5" IPS 640×480 — plenty for wallet UI
- Input: D-pad, A/B/X/Y, Start, Select, L/R — full navigation
- WiFi: yes (for light client sync)
- Battery: built-in — portable, offline signing
- Storage: microSD — firmware + wallet data
- No camera — QR scanning out, QR display in (receive only)
- USB-C: charging + data (ADB access)

## Architecture

### Base OS
- Armbian minimal (no desktop) on microSD
- Custom boot: splash screen → wallet app directly
- Locked down: no shell access by default, SSH optional via hidden button combo

### Light Client
- Port of `ckb-light-esp` logic to Node.js on Linux
- Or: compile our C library for ARM Linux (already host-testable)
- Connects to: public CKB light node OR Phill's node (192.168.68.87)
- MINIMAL profile: balance check, tip header sync, send/receive

### UI Framework (options, pick one)
1. **SDL2 + custom C UI** — fastest, most control, fits the hardware aesthetic
2. **ncurses terminal UI** — simple, lightweight, keyboard-navigable
3. **Node.js + framebuffer** — reuse existing JS light client code
4. **Python + pygame** — rapid prototyping

Recommendation: **SDL2** — gives pixel-perfect control, hardware-accelerated on Mali-G31 via Panfrost, fits the "hardware wallet" aesthetic best.

### Key Management
- BIP39 seed generation (already in CKB-ESP32, port to Linux)
- Seed stored encrypted on microSD (AES-256)
- PIN entry via D-pad (like hardware wallet PIN pad)
- secp256k1 signing on-device (already implemented)
- NO private key ever leaves the device

### Screens / UX Flow
```
[Boot] → [PIN Entry] → [Home Dashboard]
                              ↓
              ┌───────────────┼───────────────┐
           [Balance]      [Send]          [Receive]
              ↓               ↓               ↓
        [TX History]   [Enter amount]   [QR Code display]
                       [Scan/paste addr]
                       [Confirm + sign]
                       [Broadcast]
```

Button mapping:
- D-pad: navigate
- A: confirm/select
- B: back/cancel
- Start: home
- Select + Start: settings (hidden)
- L+R+Start+Select hold 3s: recovery mode

### Network
- WiFi credentials stored on SD (setup via USB/ADB on first boot)
- Light client syncs tip header every 30s when WiFi available
- Offline mode: can sign transactions, broadcast when connected
- Optional: connect to Phill's node for community units

## Distribution Plan

### For community units (pre-flashed):
1. Flash Armbian + wallet firmware to microSD
2. First boot: generate or restore seed on-device
3. Ship to community member
4. They set their own PIN on first boot

### For DIY:
1. GitHub repo with firmware image + build instructions
2. Flash guide (balenaEtcher, dd)
3. Source code fully open

### Nervos Talk follow-up post:
"The R36S CKB Wallet — $30 open source hardware wallet"
- How we built it
- Flash guide
- GitHub link
- "Grab one from Phill if you're in the community"

## Development Phases

### Phase 1 — Proof of concept (1-2 weeks)
- [ ] Armbian on R36S confirmed booting
- [ ] SDL2 rendering on framebuffer
- [ ] Basic UI: balance display, address QR
- [ ] Light client connecting to node, fetching balance

### Phase 2 — Wallet core (2-3 weeks)
- [ ] BIP39 seed generation + encrypted storage
- [ ] PIN entry UI
- [ ] Send transaction (manual address entry via D-pad)
- [ ] Sign + broadcast

### Phase 3 — Polish + distribute (1 week)
- [ ] Boot splash, custom UI theme
- [ ] Setup wizard (WiFi, seed)
- [ ] Flash image build pipeline
- [ ] Documentation + Nervos Talk post

## Repo Plan
- `toastmanAu/r36s-ckb-wallet` — firmware + UI
- Depends on `CKB-ESP32` (crypto primitives, Linux-compatible)
- Depends on `ckb-light-esp` logic (ported to Linux/C or Node.js)

## Armbian Setup Notes (researched overnight)
- Repo: `R36S-Stuff/R36S-Armbian` on GitHub — community-maintained RK3326 builds
- Multiboot option: `R36S-Stuff/R36S-Multiboot` — Armbian + ArkOS + ROCKNIX on one card
- Flash: balenaEtcher → `.img` to microSD (top TF-OS slot)
- **DTB / Screen panel variants — CRITICAL:**
  - R36S has 4+ known panel variants — wrong DTB = black screen
  - Newer 2024/2025 units need `mipi-panel.dtbo` overlay in `overlays/` on boot partition
  - Some Armbian builds include multi-panel picker (hold R1 + D-pad on boot)
  - Our firmware should ship with ALL known panel DTBs and auto-detect or prompt
  - Keep stock SD card as backup to extract original DTB if needed
- Screen res: **640×480** — design UI at this resolution
- Mali-G31 Panfrost: fully open source, OpenGL ES 3.1/3.2, actively maintained
- SDL2 on RK3326: works well — can use fbdev or Wayland/Weston backend
  - Wayland/Weston + SDL2 = zero-copy GPU rendering, best performance
  - fbdev fallback = simpler, no compositor needed, fine for wallet UI
- dArkOS (Debian-based) also an option if we want apt package manager directly

## Prior Art
- **SeedSigner** — closest comparison: Pi Zero + display + camera, open source Bitcoin wallet running Linux
  - Our advantages: better hardware, battery+buttons built-in, $30 complete, CKB-specific
  - No camera needed: receive via QR display, send via D-pad address entry or future QR scan
- **PiTrezor** — Trezor firmware on Raspberry Pi Linux — similar concept
- **Coldcard** — uses SDL2 in dev/simulator builds — validates our UI approach
- None of these target CKB. We're first.

## Notes
- Screen resolution 640×480 — design UI at this resolution
- Mali-G31 Panfrost driver works in mainline — hardware accel available
- RK3326 Armbian images exist: use as base, strip to minimal
- WiFi setup: first-boot wizard via USB-connected laptop (ADB or serial)
- Consider air-gap mode: WiFi off by default, sign offline, export via QR or USB

## Community Angle
"We built the CKB light client stack on ESP32. Now we're putting it in a $30 handheld.
Open source. Flash it yourself. Or ask Phill."

This is the kind of thing that gets people excited about a blockchain ecosystem —
real hardware, real utility, real price point.
