# SBC Deployment — Research

Running CKB infrastructure on single-board computers: full nodes, Fiber payment channel nodes, solo mining setups, and build pipelines.

## Topics

### Armbian Build System
Armbian supports 356 boards via a single build framework. The `userpatches/` system lets you inject custom software at build time without forking Armbian.

See [armbian-ckb-userpatches](https://github.com/toastmanAu/armbian-ckb-userpatches) for the implementation.

### Hardware tested

| Board | SoC | RAM | Role | Notes |
|---|---|---|---|---|
| Orange Pi 3B | RK3566 | 4GB | CKB node | Working, image built |
| Orange Pi 5+ | RK3588S | 16GB | Fiber node, build host | Running Fiber + Ollama |
| Orange Pi 3B | RK3566 | 4GB | CKB devchain | OPi3B-Armbian |
| N100 Mini PC | Intel N100 | 16GB | Agent node, build host | x86_64, Docker |
| Raspberry Pi 5 | BCM2712 | 8GB | Orchestration | Runs OpenClaw |

### Fiber Network on SBCs
Running Fiber Network Node (FNN) on SBCs. Key lessons:
- Always `pkill -9 fnn` before restart — DB lock issue
- N100 needs 100+ CKB funded before it auto-accepts channels
- SSH tunnel pattern for exposing localhost-only RPC: `autossh -L 8237:127.0.0.1:8227 ckbnode`
- `FIBER_KEY_PASSWORD` must be backed up — loss = loss of funds in channels

### Solo Mining Setup
CKB stratum proxy running on Pi 5, miners pointing at local stratum port, rewards to own address. No pool cut. Works with NerdMiner CKB and any Eaglesong stratum miner.
