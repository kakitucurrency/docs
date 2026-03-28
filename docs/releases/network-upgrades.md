---
title: Network Upgrades
description: History of Kakitu network protocol upgrades
---

# Network Upgrades

Network upgrades change the protocol rules for all nodes. Most upgrades are backward-compatible (soft) or require coordinated node upgrades before activation.

---

## Upgrade Process

Network upgrades in Kakitu are coordinated through **epoch blocks** — special blocks signed by the network's genesis account that trigger protocol changes when they reach the required confirmation height.

The upgrade process:

1. A new node version is released with support for the new epoch
2. Node operators are given a notice period to upgrade (minimum 2 weeks)
3. Once sufficient adoption is confirmed, the epoch block is broadcast
4. All accounts are upgraded to the new epoch as they process their next block

---

## Epoch History

### Epoch 1 — Genesis

*Activated: March 2026 (genesis)*

Initial network state. All accounts start at Epoch 1.

- Minimum PoW difficulty: single threshold for all block types
- Block format: state blocks required

### Epoch 2 — Differentiated Work

*Planned*

- Send/change blocks: `fffffff800000000` (higher difficulty)
- Receive/open blocks: `fffffe0000000000` (lower difficulty)
- Purpose: reduce the cost of receiving while maintaining spam resistance on sends

---

## Upcoming Upgrades

See [Roadmap](roadmap.md) for planned protocol changes and their timeline.

---

## Node Operator Responsibilities

When a network upgrade is announced:

1. **Read the release notes** for the new node version
2. **Upgrade your node** before the activation date
3. **Monitor the upgrade** via telemetry and confirmation stats
4. **Report issues** to the [GitHub repository](https://github.com/kakitucurrency/kakitu-node/issues)

Nodes running outdated software after an epoch upgrade may fail to process blocks correctly and will eventually be disconnected by peers.
