---
title: Representative Voting
description: How to configure your Kakitu node as a representative
---

# Representative Voting

Representatives are the backbone of Kakitu's consensus mechanism. They hold delegated voting weight from other accounts and vote on the validity of transactions.

---

## Overview

In Kakitu's Open Representative Voting (ORV) system:

1. Account holders delegate their balance to a representative address
2. When a transaction is broadcast, representatives vote on it
3. Once votes exceeding 67% of online voting weight are collected, the transaction is confirmed
4. Voting costs nothing — no staking, no lockup, no fees

Any node with delegated voting weight can participate. Nodes holding **at least 0.1% of online voting weight** become **Principal Representatives** — their votes are rebroadcast across the network, giving them direct influence over confirmation speed.

---

## Prerequisites

Before enabling voting:

1. Your node must be **fully synced** with the network
2. Your representative account (`kshs_` address) must have KSHS delegated to it by other accounts
3. Your node must have reliable uptime — votes missed during downtime affect confirmation times

---

## Enabling Voting

Edit `config-node.toml`:

```toml
[node]
enable_voting = true

# Your representative's kshs_ address
# This should match the account whose private key is in your wallet
representative = "kshs_YOUR_REPRESENTATIVE_ADDRESS"
```

Or use command-line override:

```bash
kakitu_node daemon --network=live --config node.enable_voting=true
```

---

## Checking Voting Status

```bash
curl -s -d '{"action":"confirmation_quorum"}' http://localhost:7076
```

```json
{
  "quorum_delta": "67000000000000000000000000000000000000",
  "online_weight_minimum": "60000000000000000000000000000000000000",
  "online_weight_quorum_percent": "67",
  "online_stake_total": "95000000000000000000000000000000000000",
  "peers_stake_total": "90000000000000000000000000000000000000",
  "trended_stake_total": "92000000000000000000000000000000000000"
}
```

---

## Checking Your Voting Weight

```bash
curl -s -d '{
  "action": "representatives",
  "count": "100",
  "sorting": "true"
}' http://localhost:7076
```

Find your representative's address in the returned list to see your delegated weight.

---

## Delegating to Your Representative

To encourage others to delegate to your representative, share your `kshs_` representative address publicly. Account holders can change their representative with:

```bash
curl -s -d '{
  "action": "account_representative_set",
  "wallet": "THEIR_WALLET_ID",
  "account": "kshs_THEIR_ACCOUNT",
  "representative": "kshs_YOUR_REPRESENTATIVE_ADDRESS"
}' http://localhost:7076
```

This creates a **change block** on their account chain — it's a free, instant operation.

---

## Principal Representative Threshold

To become a Principal Representative (PR) and have your votes rebroadcast by peers, you need at least **0.1% of online voting weight** delegated to your representative address.

Monitor your progress:

```bash
curl -s -d '{
  "action": "account_info",
  "account": "kshs_YOUR_REPRESENTATIVE_ADDRESS",
  "representative": "true"
}' http://localhost:7076
```

---

## Uptime Requirements

Representative nodes should maintain as close to **100% uptime** as possible. When a representative goes offline:

- Their delegated weight is removed from the online voting weight calculation
- This reduces the quorum delta, potentially slowing confirmations
- Persistent offline PRs may be filtered by the network's online weight tracking

Use a process manager (systemd, Docker `--restart=always`) and set up uptime monitoring with alerts. See [Monitoring](monitoring.md).

---

## Voting Weight Delegation Best Practices

- Encourage users of your service to delegate their KSHS to your representative
- Publish your representative address on your website and in community channels
- Maintain transparency about your node's uptime and infrastructure
- Consider running a redundant node in a different region for failover
