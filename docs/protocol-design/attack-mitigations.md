---
title: Attack Mitigations
description: How the Kakitu protocol defends against common attacks
---

# Attack Mitigations

The Kakitu protocol is designed to be resilient against a variety of attacks. This page describes the most relevant attack vectors and how the protocol mitigates them.

---

## Spam Attack

### Threat

An attacker creates thousands of valid transactions to flood the network, overwhelming nodes and slowing confirmations for legitimate users.

### Mitigations

1. **Proof-of-Work** — Every block requires computational work. At scale, generating work for millions of spam transactions has significant hardware cost.
2. **Election throttling** — The number of simultaneously active elections is bounded. Low-priority elections (from accounts with no history) are queued.
3. **Account priority weighting** — Elections from accounts with a recent confirmed transaction history receive processing priority over new accounts.
4. **Bandwidth limits** — Nodes enforce bandwidth caps, limiting the amount of data that can be pushed to peers.

---

## 51% / Majority Voting Attack

### Threat

An attacker acquires more than 51% of online voting weight, allowing them to confirm fraudulent transactions or prevent legitimate ones from confirming.

### Mitigations

1. **Economic cost** — Acquiring 51% of online voting weight requires buying or controlling more than half of the actively delegated KSHS supply. This is extremely expensive.
2. **No reward for attacking** — Unlike PoW chains where 51% attacks can be used for double-spend profit, a successful ORV attack only harms the network — there is no direct financial gain from controlling votes.
3. **Community response** — The Kakitu community can observe and respond to centralization of voting weight. Accounts can immediately redelegate away from a malicious representative.
4. **Sybil resistance** — Voting weight is proportional to delegated balance, not to the number of nodes. Creating many nodes does not increase voting power.

---

## Double-Spend Attack

### Threat

An attacker broadcasts two conflicting send blocks from the same account (a fork), attempting to spend the same funds twice.

### Mitigations

1. **Fork detection** — When the network sees two blocks at the same chain position, an election starts for both. Representatives vote for the one they saw first.
2. **Quorum requirement** — Only one of the conflicting blocks can accumulate 67% of online voting weight.
3. **Cementing** — Once confirmed, the winning block is cemented locally. Future blocks must build on the confirmed state.
4. **Confirmation before crediting** — Integrations that wait for confirmed status before crediting users are immune to this attack. See [Block Confirmation](../integration-guides/block-confirmation.md).

---

## Eclipse Attack

### Threat

An attacker controls all of a target node's peer connections, isolating it from the honest network and feeding it fabricated data.

### Mitigations

1. **Peer diversity** — The node maintains connections to many peers discovered through keepalive gossip.
2. **Preconfigured peers** — The node is pre-configured with well-known peers (`peering.kakitu.org`) that are difficult to Sybil-attack.
3. **Genesis block verification** — The node verifies the genesis block hash on connection. Peers on a different network (or an attacker's fork) are immediately rejected.

---

## Replay Attack

### Threat

An attacker resubmits a previously valid vote to fraudulently influence a new election.

### Mitigations

1. **Vote sequence numbers** — Each vote includes a monotonically increasing `sequence` number. Nodes reject votes with sequence numbers lower than or equal to the last seen sequence for that representative.
2. **Final votes** — A vote with `sequence = 0xFFFFFFFFFFFFFFFF` (max uint64) is a final vote. It cannot be replayed against a different block.

---

## Penny-Flooding (Dust Attack)

### Threat

An attacker sends many tiny (dust) transactions to a large number of accounts, creating receivable entries that bloat the ledger.

### Mitigations

1. **Receive minimum** — Nodes can be configured with `receive_minimum` to automatically ignore receivable blocks below a threshold.
2. **Work requirement** — Each spam send requires valid PoW, imposing a hardware cost.
3. **No ledger bloat from unsolicited receives** — The receive block must be explicitly created by the receiver. Dust sends only affect ledger state if the receiver actively creates a receive block.

---

## Node Denial-of-Service

### Threat

An attacker sends malformed or excessive RPC requests to exhaust node resources.

### Mitigations

1. **RPC binding** — RPC should only be accessible from localhost or trusted IPs.
2. **Request size limit** — Maximum RPC request size is enforced (default: 32 MB).
3. **Rate limiting** — Use a reverse proxy (nginx) in front of the RPC endpoint for production deployments.
4. **Firewall** — Port 7076 should never be exposed to the public internet.

---

## Summary

| Attack | Primary defense | Secondary defense |
|---|---|---|
| Spam | Proof-of-Work | Election throttling |
| 51% voting | Economic cost | Community redelgation |
| Double spend | Fork resolution via ORV | Confirmation requirement |
| Eclipse | Peer diversity | Genesis hash verification |
| Vote replay | Sequence numbers | Final vote mechanism |
| Dust flooding | receive_minimum | PoW per block |
| RPC DoS | Localhost binding | Reverse proxy + rate limiting |
