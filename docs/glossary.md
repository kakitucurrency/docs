---
title: Glossary
description: Definitions of key terms used in the Kakitu protocol
---

# Glossary

---

## Account

A `kshs_`-prefixed address that you control with a private key. Each account has its own account chain within the Block Lattice. An account can hold a balance of KSHS and send, receive, or delegate to a representative.

---

## Account Chain

The sequence of state blocks belonging to a single account. Every block in an account chain references the hash of the previous block, forming a linked list. The most recent block is called the **frontier**.

---

## Active Election

A consensus election that is currently in progress. Active elections collect representative votes and update their quorum status until they are confirmed or abandoned.

---

## Block

A single transaction in the Kakitu protocol. All blocks use the **state block** format and represent one of: send, receive, open, or change-representative actions.

---

## Block Lattice

The overall data structure of the Kakitu ledger. A directed acyclic graph where each account maintains its own account chain. The "lattice" is formed by the cross-links between chains created by send/receive block pairs.

---

## Bootstrap Network

A TCP-based sub-network used by nodes to download the full ledger history from peers. The bootstrap network is optimized for bulk data transfer rather than real-time propagation.

---

## Cemented Block

A confirmed block that has been recorded as locally irreversible by the node. Once cemented, a block's confirmation height is updated and it cannot be undone.

---

## Cementing

The process of recording a confirmed block as locally irreversible. Cementing updates the confirmation height for the block's account chain.

---

## Change Block

A state block that changes an account's representative without moving any funds. The `balance` remains the same, and the `link` field is all zeros.

---

## Confirmation

A block is confirmed when representatives holding more than 67% of online voting weight have voted for it. Confirmed blocks should be treated as settled.

---

## Confirmation Height

The block height up to which all blocks on an account chain are confirmed. Used to efficiently detect new confirmations without re-checking old blocks.

---

## Delegated Voting Weight

The KSHS balance that an account has assigned to a representative. The representative's total voting weight is the sum of all delegated balances. Delegating does not transfer custody of funds.

---

## Epoch Block

A special block signed by the network genesis account that upgrades the protocol rules for all accounts. Epoch blocks are used for coordinated network upgrades.

---

## Final Vote

A representative vote with a sequence number of `0xFFFFFFFFFFFFFFFF` (maximum uint64). A final vote signals that the representative will not change their vote, allowing nodes to confirm faster.

---

## Fork

Two competing blocks at the same position in an account chain. A fork is resolved by Open Representative Voting — the block that first accumulates quorum wins; the other is discarded.

---

## Frontier

The hash of the most recent block in an account chain. All new blocks must reference the current frontier as their `previous` field.

---

## Genesis Account

The first account in the Kakitu network, whose open block defines the initial supply of KSHS. All KSHS in circulation originates from the genesis account.

---

## Genesis Block

The first block on the genesis account chain. It defines the network's total supply and serves as the root of the entire Block Lattice.

---

## KSHS

The native currency of the Kakitu network — Kenya's digital shilling. All addresses start with `kshs_`. The smallest unit is **raw** (1 KSHS = 10^30 raw).

---

## kshs_ (Address Prefix)

The prefix that identifies Kakitu accounts. All valid Kakitu addresses begin with `kshs_`. The prefix was chosen to distinguish Kakitu from the Nano protocol's `nano_` prefix.

---

## Lazy Bootstrap

A bootstrap mode where the node requests specific missing blocks by hash as they are referenced, rather than downloading all frontiers first.

---

## Ledger

The complete record of all account chains in the Kakitu network. Stored locally by each node as an LMDB database file (`data.ldb`).

---

## Live Network

The TCP-based network used for real-time block and vote propagation during normal network operation (as opposed to the bootstrap network used for ledger sync).

---

## Open Block

The first block on an account chain. An open block has `previous = 0000...0000` and establishes the account's initial balance and representative.

---

## Online Voting Weight

The sum of delegated voting weight held by representatives that have recently sent votes. Quorum is calculated as a percentage of online voting weight, not total supply.

---

## ORV (Open Representative Voting)

Kakitu's consensus mechanism. Account holders delegate their balance weight to representatives, who vote on the validity of transactions. A block is confirmed once 67% of online voting weight has voted for it.

---

## Principal Representative (PR)

A representative account holding at least 0.1% of online voting weight. PR votes are rebroadcast by all nodes that receive them, giving PRs network-wide reach and influence over confirmation speed.

---

## Proof-of-Work (PoW)

A computational value that must be attached to every block before it can be broadcast. PoW raises the cost of spam without affecting fees or consensus. Generated using the Blake2b hash function.

---

## Quorum

The confirmation threshold. A block is confirmed when representatives holding more than 67% of online voting weight have voted for it.

```
quorum_delta = 0.67 × online_voting_weight
```

---

## Raw

The smallest unit of KSHS. Not divisible further.

```
1 KSHS = 1,000,000,000,000,000,000,000,000,000,000 raw
       = 10^30 raw
```

All RPC operations use raw values.

---

## Receive Block

A state block that increases an account's balance by claiming a previously sent (but unclaimed) send block. The `link` field contains the hash of the source send block.

---

## Receivable

A confirmed send block that has been sent to an account but not yet claimed with a receive block. Receivable funds are safe to receive — only confirmed send blocks appear in the receivable list.

---

## Representative

An account that votes on behalf of other accounts in ORV consensus. Any account can be a representative; voting weight is proportional to delegated balance.

---

## Seed

A 32-byte (64 hex character) random value from which multiple private keys are derived deterministically. The seed is the master backup for all accounts derived from it.

---

## Send Block

A state block that decreases an account's balance and specifies a destination. The `link` field contains the destination account's public key.

---

## State Block

The universal block format in Kakitu. All actions (send, receive, open, change representative) use the state block format. Identified by `"type": "state"`.

---

## Unchecked Block

A block received by the node that cannot yet be fully validated because a preceding block in its chain has not been received yet. Unchecked blocks are stored temporarily and processed once their dependencies arrive.

---

## Voting Weight

See **Delegated Voting Weight**.

---

## Wallet ID

A locally generated identifier for a wallet stored in the node's database. The wallet ID is not recoverable — it cannot be used to restore funds on another machine. Always back up the **seed** instead.

---

## Work

See **Proof-of-Work (PoW)**.
