# Hardware Wallets — Research

Turning cheap, accessible consumer hardware into CKB signing devices.

## Devices researched

### R36S (~$30 AliExpress handheld)
See [`r36s-hardware-wallet-design.md`](./r36s-hardware-wallet-design.md)

The R36S is an RK3326 handheld gaming console sold for ~$30. It runs mainline Linux, has a 3.5" IPS display, physical buttons, and a USB-C port. It's essentially a tiny secured computer in a pocketable form factor — and it's already rooted by design (comes with custom firmware).

Key finding: the hardware is genuinely suitable. The bottleneck is software — CKB CLI needs packaging for arm32/arm64, and a purpose-built signing UI needs building.

### ESP32-P4 + SPHINCS+
Post-quantum signing research. The P4's hardware SHA accelerators make SPHINCS+ potentially viable — Trezor's pure-software implementation takes ~2 minutes on similar hardware, but hardware SHA could bring that to seconds.

### Handheld Linux compatibility
See [`handheld-linux-compatibility.md`](./handheld-linux-compatibility.md)

Survey of $20-50 handheld Linux devices (R36S, RG35XX, Anbernic series) and their suitability as embedded Linux compute nodes or signing devices.
