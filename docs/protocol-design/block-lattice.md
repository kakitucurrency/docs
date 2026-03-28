---
title: Block Lattice
description: The Block Lattice data structure used by Kakitu
---

# Block Lattice

The Block Lattice is the fundamental data structure of the Kakitu protocol. It replaces the traditional single-chain blockchain with a collection of independent account chains.

---

## Structure

Each account in Kakitu has its own **account chain** — an ordered sequence of blocks. Each block references the hash of the previous block in the same chain, forming a linked list.

```
Account A:                    Account B:

┌─────────┐                   ┌─────────┐
│  open   │                   │  open   │
│ balance │                   │ balance │
│   10    │                   │    5    │
└────┬────┘                   └────┬────┘
     │                             │
┌────▼────┐                   ┌────▼────┐
│  send   │──────────────────►│ receive │
│   →B    │    "5 KSHS"       │  ←A     │
│ balance │                   │ balance │
│    5    │                   │   10    │
└─────────┘                   └─────────┘
```

The transfer of value between accounts requires **two blocks**:

1. A **send block** on Account A's chain, reducing A's balance and specifying B as the destination
2. A **receive block** on Account B's chain, increasing B's balance and referencing the send block hash

---

## Why Two Blocks?

The two-block model enables **asynchronous transfers**:

- The send block is broadcast immediately when the sender is ready
- The receive block is created when the receiver comes online and processes receivables
- Funds remain in a **receivable** state indefinitely between the send and receive

This is different from traditional blockchains where both parties must be represented in a single shared transaction.

Benefits:
- Receiver does not need to be online for the sender to send
- Each account chain can be updated independently and in parallel
- No global ordering of transactions is required

---

## Block Relationships

Each state block on a Kakitu account chain contains:

- `previous` — hash of the prior block (creates the chain link)
- `balance` — the account balance after this block executes
- `link` — the connection to another account chain (send target or receive source)

The `link` field's meaning depends on the block's implied action:

| Action determined by | `link` contains |
|---|---|
| Balance decreases (send) | Destination account public key |
| Balance increases (receive/open) | Hash of the source send block |
| Balance unchanged (change rep) | `0000...0000` |

---

## The Ledger

All account chains together form the **ledger** — Kakitu's global state. The ledger is stored as a key-value database (LMDB) on every full node.

The ledger contains:
- All confirmed blocks, indexed by hash
- The current frontier (latest block) of each account
- Confirmation height for each account (highest confirmed block)
- Receivable entries (confirmed sends that haven't been received yet)
- Vote records and representative weights

---

## Double-Spend Prevention

Because each account chain is sequential, double-spending is structurally difficult:

- An account can only have one valid frontier block at a time
- If two conflicting blocks are broadcast for the same account (a fork), representatives vote on which one to confirm
- The losing fork is rejected by the network
- Once a block is cemented (confirmed and recorded locally as irreversible), it cannot be undone

---

## Comparison to Traditional Blockchains

| Property | Traditional Blockchain | Kakitu Block Lattice |
|---|---|---|
| Global order of transactions | Required | Not required |
| Transaction throughput | Limited by block size | Scales with accounts |
| Transaction fees | Required (to prioritize in block) | Not required |
| Finality | Probabilistic (reorgs possible) | Deterministic (ORV quorum) |
| Orphan blocks | Common | Not applicable |
