# Handheld Gaming Console Linux Research
_Researched 2026-03-02 overnight_

## Summary Table

| Device | SoC | RAM | Armbian/Mainline | Agent Node Potential |
|--------|-----|-----|-----------------|---------------------|
| Anbernic RG-ARC D | RK3566 | 2GB | ✅ Excellent | ⭐⭐⭐⭐ HIGH |
| R36S | RK3326 | 1GB | ✅ Yes | ⭐⭐⭐ MEDIUM |
| XU10 (Ampown) | RK3326S | 1GB | ✅ Yes | ⭐⭐⭐ MEDIUM |
| TrimUI Pro | Allwinner A133P | 1GB | ⚠️ Limited | ⭐⭐ LOW-MED |
| RG35XX H | Allwinner H700 | 1GB | ⚠️ Limited | ⭐⭐ LOW-MED |
| Retroid Pocket 4 Pro | Dimensity 1100 | 8GB | ⚠️ Partial | ⭐⭐⭐ MEDIUM |
| Powkiddy X39 Pro | ATM7051 | 128MB | ❌ None | ❌ No |

---

## Device Details

### 🟢 Anbernic RG-ARC D — BEST CANDIDATE
- **SoC**: RockChip RK3566 (same as Orange Pi 3B!) — quad-core Cortex-A55 @ 1.8GHz, Mali-G52 2EE
- **RAM**: 2GB LPDDR4
- **OS**: Ships with dual-boot Android 11 + Linux
- **Custom OS**: ArkOS, JELOS, ROCKNIX, Batocera — all well supported
- **Armbian**: RK3566 is fully supported in Armbian — same kernel/DTBs as OPi3B
- **Agent potential**: HIGH — same SoC as our OPi3B, 2GB RAM, display built-in, battery. Could run OpenClaw + light CKB client. Battery-powered portable agent node.
- **Notes**: This is basically an OPi3B with a screen, battery, and controllers. Drop Armbian on it and it's another fleet member.

---

### 🟡 R36S — GOOD CANDIDATE
- **SoC**: RockChip RK3326 — quad-core Cortex-A35 @ 1.5GHz, Mali-G31MP2
- **RAM**: 1GB DDR3L
- **OS**: Linux-based stock, ships with ArkOS
- **Custom OS**: ArkOS, ROCKNIX, JELOS, Armbian (confirmed working), Arch Linux ARM
- **Armbian**: Yes — community has working Armbian port with XFCE desktop
- **Agent potential**: MEDIUM — 1GB RAM is tight for OpenClaw but doable for lightweight tasks. RK3326 is older/weaker than RK3566 but well supported.
- **Notes**: Cheap clone market device but well supported. Good for a dedicated CKB display node. Screen panel DTB variations — check which variant you have.

---

### 🟡 XU10 (Ampown/Powkiddy) — GOOD CANDIDATE  
- **SoC**: RockChip RK3326S — quad-core Cortex-A35 @ 1.6GHz, Mali-G31
- **RAM**: 1GB DDR4
- **OS**: Linux-based
- **Custom OS**: ROCKNIX, AmberELEC, REG Linux (official support)
- **Armbian**: RK3326 family is supported — likely works with R36S DTBs
- **Agent potential**: MEDIUM — same tier as R36S. REG Linux has official builds. Active community.
- **Notes**: Powkiddy branded. Rocknix officially supports it. REG Linux has dedicated builds.

---

### 🟠 TrimUI Smart Pro — LIMITED
- **SoC**: Allwinner A133P — quad-core Cortex-A53 @ 1.8GHz, PowerVR GE8300
- **RAM**: 1GB LPDDR4x + 8GB eMMC
- **OS**: Linux stock
- **Custom OS**: MinUI, NextUI, CrossMix-OS, Knulli (Batocera), GammaOS
- **Armbian**: ⚠️ No — Allwinner A133P is NOT in Armbian. PowerVR GPU = closed source, pain to support.
- **Agent potential**: LOW-MEDIUM — runs Linux natively, good community, but Allwinner A133P has no Armbian support and PowerVR GPU is a dead end for open drivers. Could still run a minimal OpenClaw if you use stock Linux base.
- **Notes**: Nice hardware but SoC is a dead end for mainline Linux. Stick to gaming CFW.

---

### 🟠 Anbernic RG35XX H — LIMITED
- **SoC**: Allwinner H700 — quad-core Cortex-A53 @ 1.5GHz, Mali-G31 MP2
- **RAM**: 1GB LPDDR4
- **OS**: Linux 64-bit
- **Custom OS**: GarlicOS, MinUI, muOS, Batocera Lite
- **Armbian**: ⚠️ H700 has partial Armbian support (experimental) — not mainstream
- **Agent potential**: LOW-MEDIUM — Allwinner H700 has some mainline kernel support but weaker than RK3566. Mali-G31 has Panfrost open driver though.
- **Notes**: The Allwinner H700 is better than A133P for mainline but still not as solid as RK3566. Good for gaming CFW, marginal for agent use.

---

### 🟡 Retroid Pocket 4 Pro — POWERFUL BUT LOCKED
- **SoC**: MediaTek Dimensity 1100 — 4× Cortex-A78 @ 2.6GHz + 4× A55 @ 2.0GHz, Mali-G77 MC9
- **RAM**: 8GB LPDDR4x
- **OS**: Android 13
- **Custom OS**: Android-based only currently — no real Linux CFW
- **Armbian**: ❌ No — Dimensity 1100 is improving in mainline kernel (6.7+) but no practical Armbian port
- **Agent potential**: MEDIUM — most powerful device on the list by far, but locked to Android. Could run Termux + OpenClaw on Android. Not a true Linux node.
- **Notes**: Best emulation device here by a wide margin. For agent use, Termux is the realistic path. Watch for future Linux ports — kernel support improving.

---

### 🔴 Powkiddy X39 Pro — SKIP
- **SoC**: ATM7051 (Actions Semi) — quad-core Cortex-A9 @ 900MHz, SGX540
- **RAM**: 128MB LPDDR2 (!)
- **OS**: Linux-based
- **Custom OS**: Very limited CFW, niche community
- **Armbian**: ❌ No — Actions Semi SoC, zero Armbian support, SGX540 GPU = closed/dead
- **Agent potential**: ❌ None — 128MB RAM, weak SoC, no community support. Gaming only, and barely.
- **Notes**: The black sheep of the list. Keep for gaming or sell.

---

## Recommendations

### For agent node / CKB display:
1. **RG-ARC D** — flash Armbian, same as OPi3B setup. Portable battery-powered fleet member. Best pick.
2. **R36S** — flash Armbian. Cheap, works, 1GB is tight but viable for lightweight tasks.
3. **XU10** — similar to R36S, ROCKNIX supported, could work.

### For gaming only (keep as-is):
- **TrimUI Pro** — great gaming device, stick to CrossMix-OS or Knulli
- **RG35XX H** — good gaming device, muOS or GarlicOS
- **Retroid Pocket 4 Pro** — best emulation machine, keep on Android, use Termux if needed

### Write-off:
- **X39 Pro** — 128MB RAM, dead-end SoC

---

## CKB Fleet Integration — RG-ARC D Plan

If you want to bring the RG-ARC D into the fleet:
1. Flash Armbian (RK3566 image — same as OPi3B)
2. `npm install -g openclaw`  
3. Configure with HF free models (low power device)
4. Add to heartbeat monitoring
5. Could run `minimal_watch_cyd`-style CKB display using the built-in screen
6. Battery-powered = portable CKB node 🔋

The RK3566 is **identical silicon** to the OPi3B — we already know exactly how to set it up.
