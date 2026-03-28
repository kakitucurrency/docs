---
title: Blocks
description: State block format and field definitions for Kakitu
---

# Blocks

All Kakitu transactions use the **state block** format. Every action — sending, receiving, changing a representative — is represented as a single state block on an account's chain.

---

## State Block Format

```json
{
  "type": "state",
  "account": "kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3",
  "previous": "A170D51B94E00371ACE76E35AC81DC9405D5D04D4CEBC399AEACE07AE05DD293",
  "representative": "kshs_1kakituxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "balance": "990000000000000000000000000000",
  "link": "E89208DD038FBB269987689621D52292AE9C35941A7484756ECCED92A65093BA",
  "link_as_account": "kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3",
  "signature": "SIGNATURE_HEX",
  "work": "WORK_HEX"
}
```

---

## Field Definitions

### `type`

Always `"state"`. All blocks in the modern Kakitu protocol use this type.

### `account`

The `kshs_` address of the account that owns this block. This is whose chain the block belongs to.

### `previous`

The hash of the immediately preceding block in this account's chain.

For the **first block** (open block), `previous` is all zeros:

```
0000000000000000000000000000000000000000000000000000000000000000
```

### `representative`

The `kshs_` address of the representative for this account. The representative's balance weight is counted from this account.

You can change your representative at any time by creating a new block with a different `representative` value and the same `balance` (no funds move).

### `balance`

The account's balance **after** this block executes, in **raw** (integer, not floating-point).

```
1 KSHS = 1000000000000000000000000000000 raw
```

### `link`

A 32-byte hex field whose meaning depends on the block action:

| Action | Determined by | `link` value |
|---|---|---|
| **Send** | `balance` decreases | Destination account's **public key** |
| **Receive / Open** | `balance` increases | Hash of the source **send block** |
| **Change representative** | `balance` unchanged | `0000...0000` (64 zeros) |

### `signature`

An Ed25519 signature of the block hash, signed with the account's private key. This proves the account owner authorized the block.

See [Signing & Hashing](signing-hashing.md) for the signing procedure.

### `work`

The Proof-of-Work value for this block. Must satisfy the minimum difficulty threshold for the block type.

See [Work](work.md) for the PoW algorithm.

---

## Block Hash

The block hash is a **Blake2b-256** hash of the block contents. It is computed before signing:

```
hash = blake2b_256(
  block_hash_preamble ||  // 32-byte constant: 0x6...
  account_public_key   ||  // 32 bytes
  previous_hash        ||  // 32 bytes
  representative_pubkey||  // 32 bytes
  balance              ||  // 16 bytes (128-bit big-endian)
  link                     // 32 bytes
)
```

The signature is computed over this hash.

---

## Block Types (Historical)

Prior to the state block format, Kakitu (and Nano) used separate block types for each action:

| Legacy Type | Replaced by |
|---|---|
| `send` | State block (balance decreases) |
| `receive` | State block (balance increases) |
| `open` | State block (first block in chain) |
| `change` | State block (representative only) |

All blocks on the current network use the state format. Legacy types may appear in historical data.

---

## Example: Send Block

Account A sends 1 KSHS to Account B:

```json
{
  "type": "state",
  "account": "kshs_ACCOUNT_A",
  "previous": "LAST_BLOCK_HASH_ON_A",
  "representative": "kshs_REPRESENTATIVE",
  "balance": "9000000000000000000000000000000",
  "link": "PUBLIC_KEY_OF_ACCOUNT_B",
  "signature": "SIGNATURE",
  "work": "WORK"
}
```

Account B's balance before: 0 KSHS, Account A's balance before: 10 KSHS. After this block: A has 9 KSHS, the 1 KSHS is receivable by B.

---

## Example: Receive Block

Account B receives the 1 KSHS sent by Account A:

```json
{
  "type": "state",
  "account": "kshs_ACCOUNT_B",
  "previous": "LAST_BLOCK_HASH_ON_B",
  "representative": "kshs_REPRESENTATIVE",
  "balance": "1000000000000000000000000000000",
  "link": "HASH_OF_THE_SEND_BLOCK",
  "signature": "SIGNATURE",
  "work": "WORK"
}
```

After confirmation: B has 1 KSHS.

---

## Broadcasting a Block

Once a block is built and signed, broadcast it via the `process` RPC:

```bash
curl -s -d '{
  "action": "process",
  "json_block": "true",
  "subtype": "send",
  "block": {
    "type": "state",
    "account": "kshs_ACCOUNT_A",
    "previous": "PREV_HASH",
    "representative": "kshs_REP",
    "balance": "9000000000000000000000000000000",
    "link": "DESTINATION_PUBLIC_KEY",
    "signature": "SIGNATURE",
    "work": "WORK"
  }
}' http://localhost:7076
```

The `subtype` field (`send`, `receive`, `open`, `change`, `epoch`) helps the node validate the block correctly.
