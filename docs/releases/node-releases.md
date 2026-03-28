---
title: Node Releases
description: Kakitu node release history and changelogs
---

# Node Releases

This page tracks all Kakitu node releases. Each release includes a changelog, upgrade notes, and links to binaries.

---

## Release Channels

| Channel | Description |
|---|---|
| **Stable** | Recommended for all production deployments |
| **Beta** | Pre-release testing; no monetary value on beta network |
| **Development** | Nightly builds; may be unstable |

---

## v1.0.0 — Genesis Release

*Released: March 2026*

The first public release of the Kakitu node — a fork of Nano V28 adapted for the Kenyan digital shilling.

### Changes

- Address prefix changed from `nano_` / `xrb_` to `kshs_`
- Currency renamed from NANO to KSHS
- Unit conversion RPC commands renamed: `kshs_to_raw`, `raw_to_kshs`
- URI scheme updated: `kshs:`, `kshsrep:`, `kshskey:`, `kshsseed:`, `kshsblock:`
- Genesis account uses `kshs_` prefix
- Network magic bytes updated (separate from Nano mainnet)
- P2P ports: 7075 (live), 17075 (test), 54000 (beta)
- Default representative: Kakitu Foundation representative

### Binaries

| Platform | Download |
|---|---|
| Linux x86_64 | [kakitu_node_v1.0.0_linux_x86_64](https://github.com/kakitucurrency/kakitu-node/releases/tag/v1.0.0) |
| macOS arm64 | [kakitu_node_v1.0.0_macos_arm64](https://github.com/kakitucurrency/kakitu-node/releases/tag/v1.0.0) |
| Docker | `docker pull kakitucurrency/kakitu-node:v1.0.0` |

### Docker

```bash
docker pull kakitucurrency/kakitu-node:v1.0.0
docker pull kakitucurrency/kakitu-node:latest
```

### Upgrade Notes

This is the initial release. No upgrade path from a prior version.

---

## Upgrade Instructions (General)

For any release:

```bash
# Docker upgrade
docker stop kakitu-node
docker rm kakitu-node
docker pull kakitucurrency/kakitu-node:latest
docker run --restart=unless-stopped \
  --name kakitu-node \
  -p 7075:7075 \
  -v "$HOME/kakitu-ledger":/root \
  kakitucurrency/kakitu-node:latest \
  kakitu_node daemon --network=live
```

The ledger data is preserved in the mounted volume — no re-sync required on upgrade.
