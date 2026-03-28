---
title: Integration Basics
description: Core concepts for integrating with Kakitu
---

# Integration Basics

This guide covers the foundational concepts every developer needs before building on Kakitu. Read this page first before moving on to key management, block confirmation, or specific integration patterns.

---

## Block Lattice Design

Kakitu uses a **Block Lattice** — a directed acyclic graph where every account has its own independent blockchain.

Unlike traditional blockchains where all transactions compete to be included in a shared block, in Kakitu:

- Each account chain is controlled exclusively by its private key holder
- Transactions on one account do not block or delay transactions on another
- There is no global block size or throughput ceiling

Every transaction consists of two operations:

1. **Send** — the sender creates a block reducing their balance and specifying a destination
2. **Receive** — the receiver creates a block increasing their balance, referencing the send block

The receiver does not need to be online when the send occurs. Funds remain **receivable** indefinitely until the receiver claims them.

```
Account A Chain:          Account B Chain:
[open]                    [open]
[receive: genesis]        [receive: genesis]
[send → B: 5 KSHS]  ───► [receive ← A: 5 KSHS]
[send → C: 2 KSHS]
```

---

## Representatives and Consensus

Kakitu uses **Open Representative Voting (ORV)** for consensus:

1. Account holders assign a **representative** — a node that votes on their behalf
2. Representatives accumulate **voting weight** proportional to delegated KSHS
3. When a block is broadcast, representatives vote on it
4. Once votes from representatives holding more than **67% of online voting weight** are collected, the block is confirmed

Key properties:
- Voting is free — no staking, no lockup
- Any account can change its representative at any time (creates a change block)
- Representatives vote on behalf of delegators; delegators retain full custody of their funds

---

## Account Identifiers

Understanding the difference between these identifiers is critical:

### Wallet ID

```
6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B
```

- 64-character hex string
- Local to this node's database only
- Not recoverable if the database is deleted
- Do not use as a backup mechanism

### Seed

```
0000000000000000000000000000000000000000000000000000000000000001
```

- 64-character hex string (32 bytes)
- Deterministically derives multiple private keys via Blake2b hashing
- **This is your backup.** Store it offline and encrypted.
- Also available as a 24-word BIP39 mnemonic

### Private Key

```
9F0E444C69F353324A31FC6D4B28C1CF1E3FA6C08A2A3FC37A9F9A83F4EB6340
```

- 64-character hex string (32 bytes)
- Derived from a seed at a specific index
- Signs all blocks for the associated account
- Never share or transmit

### Public Key

```
C008B814A7D269A1FA3C6528B19201A24D797912DB9996FF02A1FF356E45552B
```

- 64-character hex string (32 bytes)
- Derived from the private key using Ed25519
- Used in open block and change block operations

### Account Address

```
kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3
```

- The public key encoded in Base32 with a `kshs_` prefix and a Blake2b-40 checksum
- This is the identifier you share with others to receive KSHS
- Always begins with `kshs_`

---

## Units

All Kakitu RPC operations use **raw** — the smallest indivisible unit.

```
1 KSHS = 1,000,000,000,000,000,000,000,000,000,000 raw
       = 10^30 raw
```

**Never use floating-point arithmetic** for KSHS amounts. Use arbitrary-precision integer libraries (e.g., Python's `int`, Java's `BigInteger`, Go's `big.Int`).

| Amount | Raw value |
|---|---|
| 1 KSHS | `1000000000000000000000000000000` |
| 0.001 KSHS | `1000000000000000000000000000` |
| 0.000001 KSHS | `1000000000000000000000000` |

Unit conversion RPC:

```bash
# KSHS to raw
curl -s -d '{
  "action": "kshs_to_raw",
  "amount": "1"
}' http://localhost:7076
# → {"amount": "1000000000000000000000000000000"}

# Raw to KSHS
curl -s -d '{
  "action": "raw_to_kshs",
  "amount": "1000000000000000000000000000000"
}' http://localhost:7076
# → {"amount": "1"}
```

---

## Block Structure

All Kakitu transactions use the **state block** format. Every block contains:

| Field | Description |
|---|---|
| `type` | Always `"state"` |
| `account` | The `kshs_` address of the account chain |
| `previous` | Hash of the previous block in this account's chain (or `0000...0000` for open block) |
| `representative` | The representative for this account |
| `balance` | Account balance **after** this block, in raw |
| `link` | Context-dependent (see below) |
| `signature` | Ed25519 signature of the block hash |
| `work` | Proof-of-Work value |

### The `link` Field

| Block Action | `link` value |
|---|---|
| Send | Destination account's **public key** (32 bytes hex) |
| Receive / Open | Hash of the **source send block** |
| Change representative | `0000...0000` (64 zeros) |

---

## URIs and QR Codes

Kakitu uses standardized URI schemes for payments and key imports:

| Scheme | Purpose | Example |
|---|---|---|
| `kshs:` | Payment request | `kshs:kshs_3t6k35...?amount=1000000000000000000000000000000` |
| `kshsrep:` | Change representative | `kshsrep:kshs_3t6k35...` |
| `kshskey:` | Import private key | `kshskey:PRIVATE_KEY_HEX` |
| `kshsseed:` | Import seed | `kshsseed:SEED_HEX` |
| `kshsblock:` | Import raw block | `kshsblock:{"type":"state",...}` |

Payment URI with amount:

```
kshs:kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3?amount=1000000000000000000000000000000&label=KakituPay
```

---

## Next Steps

- [Key Management](key-management.md) — generating and storing keys securely
- [Block Confirmation](block-confirmation.md) — how to reliably detect confirmed transactions
- [Work Generation](work-generation.md) — generating PoW for blocks
- [WebSockets](websockets.md) — real-time event streaming
- [RPC Reference](../commands/rpc-protocol.md) — complete API documentation
