# r36s-ckb-wallet — Repo Structure Draft

```
r36s-ckb-wallet/
├── README.md                    # Flash guide + what it is
├── DESIGN.md                    # Links to full design doc
├── LICENSE                      # MIT
│
├── firmware/                    # Build system for the SD card image
│   ├── build.sh                 # Main build script (Armbian base + overlay)
│   ├── armbian-config/          # Armbian customisation hooks
│   ├── overlays/                # Panel DTBs for all R36S variants
│   │   ├── panel1.dtbo
│   │   ├── panel2.dtbo
│   │   ├── panel3.dtbo
│   │   └── panel4.dtbo
│   └── firstboot/               # First-boot setup wizard scripts
│       ├── panel-detect.sh      # Auto-detect / prompt screen panel
│       └── wifi-setup.sh        # WiFi credentials setup
│
├── wallet/                      # The wallet application
│   ├── CMakeLists.txt           # Build: SDL2 + CKB crypto libs
│   ├── src/
│   │   ├── main.c               # Entry point, SDL2 init, screen loop
│   │   ├── ui/
│   │   │   ├── screen_home.c    # Home: balance + address
│   │   │   ├── screen_send.c    # Send: amount + address entry
│   │   │   ├── screen_receive.c # Receive: QR code display
│   │   │   ├── screen_pin.c     # PIN entry (d-pad)
│   │   │   ├── screen_seed.c    # Seed generate / restore
│   │   │   ├── screen_history.c # Transaction history
│   │   │   └── widgets.c        # Buttons, lists, text, QR renderer
│   │   ├── wallet/
│   │   │   ├── keystore.c       # Encrypted seed storage (AES-256)
│   │   │   ├── bip39.c          # Mnemonic generation (from CKB-ESP32)
│   │   │   ├── address.c        # CKB address encode/decode (from CKB-ESP32)
│   │   │   └── signer.c         # secp256k1 tx signing (from CKB-ESP32)
│   │   └── network/
│   │       ├── light_client.c   # CKB light client (from ckb-light-esp, ported)
│   │       ├── rpc.c            # CKB RPC calls over WiFi
│   │       └── sync.c           # Tip header sync, balance fetch
│   └── assets/
│       ├── font.ttf             # UI font (small, readable at 640×480)
│       ├── splash.bmp           # Boot splash
│       └── icons/               # CKB logo, status icons
│
├── tools/
│   ├── qr-gen/                  # QR code generation (libqrencode)
│   └── panel-id/                # Tool to identify R36S panel variant
│
├── docs/
│   ├── FLASH-GUIDE.md           # Step by step flashing
│   ├── BUILD-FROM-SOURCE.md     # For devs
│   ├── SECURITY.md              # Key storage, threat model
│   └── PANEL-VARIANTS.md        # R36S screen panel identification
│
└── releases/                    # Pre-built images (GitHub Releases)
    └── README.md                # Points to GitHub releases page
```

## Key Dependencies
- Armbian RK3326 base image
- SDL2 (display + input)
- libqrencode (QR code generation)
- CKB-ESP32 crypto primitives (bip39, secp256k1, address) — reuse C code directly
- ckb-light-esp light client logic — port C code to Linux ARM
- libsodium (key derivation, AES-256 encryption for keystore)
- cJSON (RPC JSON parsing)

## Button Mapping
```
D-pad UP/DOWN    → navigate menu items
D-pad LEFT/RIGHT → change value / cursor
A (right)        → confirm / select
B (bottom)       → back / cancel
X (top)          → secondary action (e.g. paste address)
Y (left)         → unused / future
START            → home screen
SELECT           → settings (long press)
L1               → scroll up (history)
R1               → scroll down (history)
L1+R1+START+SELECT (hold 3s) → recovery mode
```

## Seed/PIN Security Model
- 12 or 24 word BIP39 seed, generated on-device (entropy from /dev/random)
- Seed encrypted with AES-256-GCM, key derived from PIN via PBKDF2
- Encrypted blob stored on SD card (/boot/wallet.enc or dedicated partition)
- PIN entered via d-pad: up/down to change digit, A to confirm each
- Wrong PIN: exponential backoff (1s, 2s, 4s... up to 30s after 10 attempts)
- Air-gap option: disable WiFi chip entirely via rfkill, sign offline

## Build Pipeline
1. Download Armbian minimal RK3326 image
2. Chroot into image
3. apt install: libsdl2-dev libqrencode-dev libsodium-dev nodejs
4. Build wallet/ with CMake
5. Install wallet binary + assets to /opt/ckb-wallet/
6. Set wallet as systemd service, auto-start on boot
7. Disable unnecessary services (bluetooth, avahi, etc.)
8. Lock down: no SSH by default, no root login
9. Compress image → distribute via GitHub Releases
