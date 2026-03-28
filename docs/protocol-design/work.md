---
title: Proof-of-Work
description: The Proof-of-Work algorithm used for spam prevention in Kakitu
---

# Proof-of-Work

Kakitu uses Proof-of-Work (PoW) as a spam prevention mechanism. Unlike Bitcoin's mining, Kakitu's PoW does not determine consensus or create new coins — it simply raises the cost of submitting large numbers of transactions.

---

## Purpose

Without any cost per transaction, a malicious actor could flood the network with millions of transactions at no expense. PoW imposes a small computational cost per block, making spam attacks prohibitively expensive at scale while remaining negligible for normal use.

PoW does not affect:
- Which blocks are confirmed
- Transaction order
- Fees (there are none)

---

## Algorithm

Kakitu uses the **Blake2b** hash function:

### Block height > 1 (all non-open blocks)

```
Find nonce such that:
blake2b(nonce || previous_block_hash) >= difficulty_threshold
```

### Block height == 1 (open block — first on account)

```
Find nonce such that:
blake2b(nonce || account_public_key) >= difficulty_threshold
```

Where:
- `nonce` is an 8-byte (64-bit) random value
- `||` denotes concatenation
- The comparison is unsigned integer comparison of the first 8 bytes of the hash

---

## Difficulty Thresholds (Epoch v2)

| Block Type | Threshold | Notes |
|---|---|---|
| Send / Change | `fffffff800000000` | Higher — these can broadcast value |
| Receive / Open / Epoch | `fffffe0000000000` | Lower — these can only claim |

Send and change blocks require ~8× more work than receive blocks, because send blocks are the primary mechanism for potential spam.

---

## Work Field Format

The `work` field in a block is an 8-byte (64-bit) hex-encoded nonce:

```json
{
  "work": "2bf29ef00786a6bc"
}
```

---

## Epoch Upgrades

Work difficulty can be upgraded via epoch blocks — special blocks signed by the network genesis account that upgrade all accounts to a new minimum difficulty. This allows the network to adjust spam resistance over time without a hard fork.

| Epoch | Introduced | Change |
|---|---|---|
| v1 | Genesis | Initial difficulty |
| v2 | Upgrade | Differentiated send/receive thresholds |

---

## Pre-caching

Because generating work takes time (fractions of a second on a GPU, seconds on a CPU), work should be **pre-generated** for the next block as soon as the current block is confirmed.

After block `H` is confirmed:
- Immediately generate `work(H)` in the background
- Store it in a cache keyed by `H`
- When the next block is needed, retrieve from cache for instant broadcast

---

## Energy Efficiency

Kakitu's PoW is dramatically more energy-efficient than Proof-of-Work mining:

- Work is generated once per block, not continuously
- No mining race — only the block creator generates work
- GPU work generation is optional and not required for normal operation
- A typical send transaction requires < 0.001 kWh of energy to generate work

---

## Adjusting Difficulty

Work can be generated above the minimum threshold. A higher-difficulty block is valid but not prioritized differently — there is no advantage to generating extra-hard work for transaction processing.

To generate at higher difficulty (e.g., for testing):

```bash
curl -s -d '{
  "action": "work_generate",
  "hash": "PREVIOUS_HASH",
  "difficulty": "ffffffff00000000",
  "multiplier": "8.0"
}' http://localhost:7076
```

The `multiplier` field can be used instead of specifying an absolute difficulty.
