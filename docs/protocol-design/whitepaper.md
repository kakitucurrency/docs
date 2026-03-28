---
title: Kakitu Whitepaper
description: The Kakitu protocol whitepaper — feeless digital cash for Kenya
---

# Kakitu: A Feeless Digital Currency for Kenya

**Version 1.0 — March 2026**

---

## Abstract

Kakitu is a digital currency designed for Kenya and the East African region. It provides feeless, near-instant transfers of value with cryptographic security and no central point of control. Built on the Block Lattice architecture and Open Representative Voting (ORV) consensus, Kakitu achieves transaction throughput and confirmation speeds that existing financial infrastructure cannot match, with a fraction of the energy consumption of conventional cryptocurrencies.

---

## 1. Introduction

The dominant mechanism for digital value transfer in Kenya is M-Pesa, operated by Safaricom. While M-Pesa achieved impressive adoption, it has fundamental limitations:

- **Transaction fees**: Every transfer carries a fee, disproportionately affecting small, frequent transactions
- **Centralization**: A single corporate entity controls issuance, access, and operation
- **Exclusion**: KYC requirements exclude undocumented individuals
- **No programmability**: No pathway to DeFi, smart contracts, or composable financial services

Kakitu addresses these limitations using the Nano protocol — one of the most efficient distributed ledger designs in existence — adapted for the Kenyan context with the `kshs_` address prefix and KSHS currency unit.

---

## 2. The Block Lattice

### 2.1 Account Chains

Traditional blockchains serialize all transactions into a single global chain. This creates bottlenecks: transactions compete for block space, fees emerge as a prioritization mechanism, and throughput is bounded by block size and interval.

The Block Lattice replaces this with a directed acyclic graph where each account has its own chain. Account A's transactions never block or delay Account B's transactions.

### 2.2 Asynchronous Transfers

A transfer in Kakitu is a pair of blocks:

1. **Send block** (on sender's chain): reduces sender's balance, specifies destination
2. **Receive block** (on receiver's chain): increases receiver's balance, references the send block hash

The two blocks are created independently. The send block is broadcast when the sender is ready. The receive block is created when the receiver comes online.

This asynchrony is a feature: it enables offline receiving, batched processing, and eliminates the ordering constraints of shared-chain models.

---

## 3. Open Representative Voting

### 3.1 Delegated Voting

Consensus in Kakitu does not require mining or staking lockups. Account holders delegate their balance weight to a representative of their choosing. Delegation:
- Costs nothing (only a change block with PoW)
- Does not move funds
- Can be changed at any time

### 3.2 Quorum

A block is confirmed when cumulative votes from representatives holding more than 67% of online voting weight are collected. This threshold was chosen to:
- Be high enough to make attacks expensive
- Be low enough that the network can confirm transactions even when some representatives are offline

### 3.3 Finality

Once a block reaches quorum, it is considered final. There are no reorganizations, no forks, and no probabilistic confirmation delays. A confirmed Kakitu transaction cannot be reversed.

---

## 4. Spam Prevention

Because Kakitu transactions are feeless, an alternative spam prevention mechanism is required. Kakitu uses **Proof-of-Work**:

- Each block requires a valid PoW value meeting a minimum difficulty threshold
- Work is generated against the previous block hash (or account public key for open blocks)
- Send/change blocks require higher difficulty than receive/open blocks
- Work can be pre-generated and cached for low-latency sending

The computational cost of generating valid PoW for millions of spam transactions is significant, making spam attacks expensive while keeping the cost for legitimate users negligible.

---

## 5. Security Model

Kakitu's security properties:

| Property | Mechanism |
|---|---|
| Authenticity | Ed25519 signatures on every block |
| Integrity | Blake2b hashing of block contents |
| Anti-spam | Proof-of-Work per block |
| Anti-double-spend | ORV fork resolution |
| Anti-sybil | Voting weight is balance-weighted, not node-count-weighted |
| Censorship resistance | Any node can broadcast any valid block |

---

## 6. Network Parameters

| Parameter | Value |
|---|---|
| Currency | KSHS (Kenya Shilling digital) |
| Address prefix | `kshs_` |
| Maximum supply | Fixed at genesis |
| Smallest unit | raw (1 KSHS = 10^30 raw) |
| Block format | State block |
| Signature algorithm | Ed25519 |
| Hash function | Blake2b-256 |
| PoW algorithm | Blake2b-64 |
| Consensus | Open Representative Voting |
| P2P port (main) | 7075 TCP |
| RPC port (main) | 7076 TCP |

---

## 7. Roadmap

See [Roadmap](../releases/roadmap.md) for planned protocol upgrades and ecosystem development.

---

## 8. Acknowledgements

Kakitu is a fork of the Nano protocol, originally designed by Colin LeMahieu. The Block Lattice architecture and ORV consensus are described in the original Nano whitepaper (2015). Kakitu's contribution is the adaptation of this protocol for Kenya, with the `kshs_` address scheme, KSHS currency branding, and an ecosystem of tools tailored to Kenyan developers and users.

---

*Kakitu — Kenya's digital shilling. Fast, free, forever.*
