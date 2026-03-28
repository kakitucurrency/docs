---
title: Networking
description: Kakitu network architecture — live network and bootstrap
---

# Networking

Kakitu uses two distinct sub-networks: the **live network** for real-time block and vote propagation, and the **bootstrap network** for bulk ledger synchronization.

---

## Live Network

The live network is the primary communication channel for:

- Broadcasting new blocks to the network
- Propagating representative votes
- Exchanging keepalive messages with peers

### Transport

TCP on port **7075** (main network).

| Network | Live Port |
|---|---|
| Main | 7075 |
| Test | 17075 |
| Beta | 54000 |

### Message Types

| Message | Purpose |
|---|---|
| `keepalive` | Peer discovery and connectivity check |
| `publish` | Broadcast a new block |
| `confirm_req` | Request confirmation votes for a block |
| `confirm_ack` | Send a vote for a block |
| `bulk_pull` | Request multiple blocks (bootstrap) |
| `telemetry_req` / `telemetry_ack` | Node health metrics exchange |

### Block Propagation

1. Account creates a signed block with valid PoW
2. Block is submitted to any node via the `process` RPC
3. Node validates the block (signature, PoW, chain linkage)
4. Valid block is broadcast to connected peers via `publish` messages
5. Peers rebroadcast to their peers (gossip protocol)
6. Representatives vote via `confirm_ack` messages

---

## Bootstrap Network

The bootstrap network is used by new or recovering nodes to download the full ledger history. It is optimized for bulk data transfer rather than real-time propagation.

### Transport

TCP on the same port as the live network. Bootstrap and live traffic are differentiated by message type.

### Bootstrapping Process

1. New node connects to preconfigured seed peers (`peering.kakitu.org`)
2. Requests the genesis block hash to identify the network
3. Downloads account frontiers (the latest block hash for each account)
4. Pulls missing blocks for accounts whose local state is behind
5. Verifies each block's signature and PoW
6. Once within 1% of network block count, node is considered synced

### Bootstrap Modes

| Mode | Description |
|---|---|
| Legacy bootstrap | Pull frontiers for all accounts, then fill in missing blocks |
| Lazy bootstrap | Request missing blocks by hash as needed |
| Wallet bootstrap | Prioritize blocks for accounts in the local wallet |

---

## Peer Discovery

### Preconfigured Peers

The node ships with a list of well-known peers in `config-node.toml`:

```toml
[node]
preconfigured_peers = [
  "peering.kakitu.org"
]
```

### Keepalive Messages

Nodes exchange `keepalive` messages with known peers. Each keepalive includes a list of other known peers, allowing organic peer discovery across the network.

### UPnP

By default, the node attempts to use UPnP to open port 7075 on your router. This can be disabled:

```toml
[node]
upnp_enabled = false
```

---

## Network Identity

The Kakitu network is identified by the **genesis block hash** — the hash of the very first block. Nodes that do not share the same genesis block hash are on different networks and will not communicate.

This means running `--network=test` and `--network=live` nodes are completely isolated, even if they run on the same machine.

---

## Bandwidth

Node bandwidth usage depends on:

- Number of connected peers
- Network transaction volume
- Whether the node is a Principal Representative (PR votes are rebroadcast)

Approximate bandwidth for a PR node on a busy network: **50–200 Mbps**.

Limit bandwidth in `config-node.toml`:

```toml
[node]
bandwidth_limit = 10485760      # 10 MB/s
bandwidth_limit_burst_ratio = 3.0
bootstrap_bandwidth_limit = 5242880  # 5 MB/s
```

---

## Telemetry

Nodes exchange telemetry metrics with peers. Retrieve your peers' telemetry:

```bash
curl -s -d '{"action":"telemetry","raw":"true"}' http://localhost:7076
```

This returns block counts, uptime, version, and bandwidth data for all connected peers, which helps you assess network health and your sync status.
