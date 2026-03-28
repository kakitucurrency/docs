---
title: Running a Node — Overview
description: Why and how to run a Kakitu node
---

# Running a Node — Overview

Running a Kakitu node helps decentralize the network and provides a direct access point for systems built on top of Kakitu. Unlike proof-of-work blockchains, Kakitu has no mining rewards — incentives for running infrastructure come from participation in the network and supporting the ecosystem.

---

## Why Run a Node?

- **Support decentralization** — The more independent nodes the network has, the more resilient and censorship-resistant it becomes
- **Direct ledger access** — Your applications can query the ledger without depending on a third-party API
- **Become a representative** — Hold delegated voting weight and participate directly in consensus
- **Lower latency** — Confirm transactions instantly without API rate limits

---

## Node Types

### Non-Voting Node

A non-voting node connects to the network, maintains a full copy of the ledger, and can send and receive transactions via RPC — but does not participate in voting.

This is the **recommended starting point** for integrations, exchanges, and services. A non-voting node has no minimum balance requirement and no obligation to be online continuously.

### Representative Node

A representative node holds delegated voting weight below the **Principal Representative** threshold (less than 0.1% of online voting weight). It votes on transactions but its votes are not rebroadcast by peers.

### Principal Representative Node

A Principal Representative (PR) holds at least **0.1% of online voting weight** delegated to its account. PR votes are rebroadcast by peers across the network, meaning they directly influence confirmation speed and quorum achievement.

Running a PR node requires:

- Consistently high uptime
- Sufficient hardware resources (see below)
- A `kshs_` address with significant delegated balance

---

## Hardware Recommendations

| Component | Minimum | Recommended (PR) |
|---|---|---|
| CPU | 4 cores, 2 GHz | 8 cores, 3+ GHz |
| RAM | 8 GB | 32 GB |
| Storage | 200 GB SSD | 500 GB NVMe SSD |
| Bandwidth | 100 Mbps | 500 Mbps+ |
| OS | Ubuntu 22.04 LTS | Ubuntu 22.04 LTS |

!!! warning "Disk I/O is Critical"
    Kakitu's ledger is I/O intensive. Mechanical hard drives (HDDs) will cause severe performance degradation. Use SSD or NVMe storage.

---

## Ongoing Responsibilities

Running a node is an ongoing commitment. Operators should:

- Apply OS-level security patches regularly
- Upgrade to the latest Kakitu node releases
- Monitor disk usage as the ledger grows
- Protect private keys and RPC endpoints
- Ensure the node remains reachable on the correct ports

For detailed guidance, see [Node Security](security.md).

---

## Next Steps

1. [Secure your server](security.md) before installing
2. [Set up your node](node-setup.md) with Docker or a native binary
3. [Configure the node](configuration.md) for your use case
4. [Set up a wallet](wallet-setup.md) if managing funds directly
5. [Enable representative voting](representative-voting.md) to participate in consensus
6. [Monitor your node](monitoring.md) with telemetry and alerts
