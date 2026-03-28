---
title: RPC Protocol
description: Complete RPC reference for the Kakitu node with kshs_ examples
---

# RPC Protocol

The Kakitu node exposes an HTTP RPC interface that accepts JSON-encoded POST requests. All commands follow the format:

```bash
curl -s -d '{"action":"COMMAND_NAME", ...}' http://localhost:7076
```

---

## Configuration

The RPC server runs on port **7076** (main network) and is bound to `127.0.0.1` by default. Sensitive commands require `enable_control = true` in `config-rpc.toml`.

Commands that require `enable_control` are marked with **[control]** below.

---

## Account Commands

### account_balance

Returns the balance for a `kshs_` account.

```bash
curl -s -d '{
  "action": "account_balance",
  "account": "kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3"
}' http://localhost:7076
```

Optional: `"include_only_confirmed": "true"` — exclude unconfirmed receivable amounts.

**Response:**
```json
{
  "balance": "10000000000000000000000000000000",
  "pending": "0",
  "receivable": "5000000000000000000000000000000"
}
```

---

### account_block_count

Returns the number of blocks in the given account's chain.

```bash
curl -s -d '{
  "action": "account_block_count",
  "account": "kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3"
}' http://localhost:7076
```

**Response:** `{"block_count": "42"}`

---

### account_info

Returns detailed information about an account.

```bash
curl -s -d '{
  "action": "account_info",
  "account": "kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3",
  "representative": "true",
  "weight": "true",
  "receivable": "true",
  "confirmation_height": "true"
}' http://localhost:7076
```

**Response:**
```json
{
  "frontier": "FF84533A571D953A596EA401FD41743AC85D04F406E76FDA3641F52F1B03461",
  "open_block": "991CF190094C00F0B68E2E5F75F6BEE95A2E0BD93CEAA4A6734DB9F19B728948",
  "representative_block": "991CF190094C00F0B68E2E5F75F6BEE95A2E0BD93CEAA4A6734DB9F19B728948",
  "balance": "10000000000000000000000000000000",
  "modified_timestamp": "1586118900",
  "block_count": "42",
  "account_version": "1",
  "confirmation_height": "41",
  "confirmation_height_frontier": "PREV_HASH",
  "representative": "kshs_1kakituxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "weight": "0",
  "receivable": "5000000000000000000000000000000"
}
```

---

### account_history

Returns transaction history for an account.

```bash
curl -s -d '{
  "action": "account_history",
  "account": "kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3",
  "count": "10",
  "raw": "false"
}' http://localhost:7076
```

**Response:**
```json
{
  "account": "kshs_3t6k35...",
  "history": [
    {
      "type": "send",
      "account": "kshs_DESTINATION",
      "amount": "1000000000000000000000000000000",
      "local_timestamp": "1648000000",
      "height": "5",
      "hash": "BLOCK_HASH",
      "confirmed": "true"
    }
  ]
}
```

---

### account_key

Returns the public key for a `kshs_` address.

```bash
curl -s -d '{
  "action": "account_key",
  "account": "kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3"
}' http://localhost:7076
```

**Response:** `{"key": "E89208DD038FBB269987689621D52292AE9C35941A7484756ECCED92A65093BA"}`

---

### account_get

Returns the `kshs_` address for a public key.

```bash
curl -s -d '{
  "action": "account_get",
  "key": "E89208DD038FBB269987689621D52292AE9C35941A7484756ECCED92A65093BA"
}' http://localhost:7076
```

**Response:** `{"account": "kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3"}`

---

### account_representative

Returns the representative for an account.

```bash
curl -s -d '{
  "action": "account_representative",
  "account": "kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3"
}' http://localhost:7076
```

**Response:** `{"representative": "kshs_1kakituxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"}`

---

### account_weight

Returns the delegated voting weight for a representative account.

```bash
curl -s -d '{
  "action": "account_weight",
  "account": "kshs_1kakituxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}' http://localhost:7076
```

**Response:** `{"weight": "10000000000000000000000000000000000000"}`

---

### accounts_balances

Returns balances for multiple accounts.

```bash
curl -s -d '{
  "action": "accounts_balances",
  "accounts": [
    "kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3",
    "kshs_ANOTHER_ACCOUNT"
  ]
}' http://localhost:7076
```

---

### accounts_frontiers

Returns the frontier (latest block hash) for multiple accounts.

```bash
curl -s -d '{
  "action": "accounts_frontiers",
  "accounts": ["kshs_3t6k35...", "kshs_ANOTHER..."]
}' http://localhost:7076
```

---

### accounts_receivable

Returns receivable (pending) blocks for multiple accounts.

```bash
curl -s -d '{
  "action": "accounts_receivable",
  "accounts": ["kshs_3t6k35...", "kshs_ANOTHER..."],
  "count": "64",
  "threshold": "1000000000000000000000000",
  "source": "true"
}' http://localhost:7076
```

---

### receivable

Returns receivable blocks for a single account.

```bash
curl -s -d '{
  "action": "receivable",
  "account": "kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3",
  "count": "64",
  "threshold": "1000000000000000000000000",
  "source": "true"
}' http://localhost:7076
```

**Response:**
```json
{
  "blocks": {
    "A170D51B94E00371ACE76E35AC81DC9405D5D04D4CEBC399AEACE07AE05DD293": {
      "amount": "1000000000000000000000000000000",
      "source": "kshs_SENDER"
    }
  }
}
```

---

## Block Commands

### block_count

Returns the total number of blocks in the ledger.

```bash
curl -s -d '{"action":"block_count"}' http://localhost:7076
```

**Response:**
```json
{
  "count": "12345678",
  "unchecked": "8",
  "cemented": "12345670"
}
```

---

### block_info

Returns detailed information about a block.

```bash
curl -s -d '{
  "action": "block_info",
  "hash": "A170D51B94E00371ACE76E35AC81DC9405D5D04D4CEBC399AEACE07AE05DD293",
  "json_block": "true"
}' http://localhost:7076
```

**Response:**
```json
{
  "block_account": "kshs_3t6k35...",
  "amount": "1000000000000000000000000000000",
  "balance": "0",
  "height": "5",
  "local_timestamp": "1648000000",
  "successor": "",
  "confirmed": "true",
  "contents": {
    "type": "state",
    "account": "kshs_3t6k35...",
    "previous": "...",
    "representative": "kshs_...",
    "balance": "0",
    "link": "...",
    "signature": "...",
    "work": "..."
  },
  "subtype": "send"
}
```

---

### blocks_info

Returns information for multiple blocks.

```bash
curl -s -d '{
  "action": "blocks_info",
  "hashes": ["HASH1", "HASH2"],
  "json_block": "true",
  "source": "true",
  "balance": "true",
  "include_not_found": "false"
}' http://localhost:7076
```

---

### block_hash

Computes the hash of a block without broadcasting it.

```bash
curl -s -d '{
  "action": "block_hash",
  "json_block": "true",
  "block": {
    "type": "state",
    "account": "kshs_3t6k35...",
    "previous": "PREV_HASH",
    "representative": "kshs_REP",
    "balance": "9000000000000000000000000000000",
    "link": "DESTINATION_PUBKEY",
    "signature": "SIGNATURE",
    "work": "WORK"
  }
}' http://localhost:7076
```

**Response:** `{"hash": "COMPUTED_HASH"}`

---

### process **[control]**

Broadcasts a signed block to the network.

```bash
curl -s -d '{
  "action": "process",
  "json_block": "true",
  "subtype": "send",
  "block": {
    "type": "state",
    "account": "kshs_SOURCE",
    "previous": "PREV_HASH",
    "representative": "kshs_REP",
    "balance": "9000000000000000000000000000000",
    "link": "DESTINATION_PUBKEY",
    "signature": "SIGNATURE",
    "work": "WORK"
  }
}' http://localhost:7076
```

**Response:** `{"hash": "BLOCK_HASH"}`

---

## Work Commands

### work_generate **[control]**

Generates Proof-of-Work for a block.

```bash
curl -s -d '{
  "action": "work_generate",
  "hash": "PREVIOUS_HASH_OR_PUBLIC_KEY",
  "difficulty": "fffffff800000000"
}' http://localhost:7076
```

**Response:**
```json
{
  "work": "2bf29ef00786a6bc",
  "difficulty": "ffffffff8ba4a60c",
  "multiplier": "8.00",
  "hash": "PREVIOUS_HASH"
}
```

---

### work_validate

Validates a Proof-of-Work value.

```bash
curl -s -d '{
  "action": "work_validate",
  "work": "2bf29ef00786a6bc",
  "hash": "PREVIOUS_HASH",
  "difficulty": "fffffff800000000"
}' http://localhost:7076
```

**Response:**
```json
{
  "valid_all": "1",
  "valid_receive": "1",
  "difficulty": "ffffffff8ba4a60c",
  "multiplier": "8.00"
}
```

---

### work_cancel **[control]**

Cancels an active work generation request.

```bash
curl -s -d '{
  "action": "work_cancel",
  "hash": "PREVIOUS_HASH"
}' http://localhost:7076
```

---

### work_peers

Returns configured work peers.

```bash
curl -s -d '{"action":"work_peers"}' http://localhost:7076
```

---

## Key Commands

### key_create

Generates a new random private key, public key, and `kshs_` address.

```bash
curl -s -d '{"action":"key_create"}' http://localhost:7076
```

**Response:**
```json
{
  "private": "781186FB9EF17DB6E3D1056550D9FAE5D5BBADA6A6BC370E4CBB938B1DC71DA3",
  "public": "3068BB1CA04525BB0E416C485FE6A67FD52540227D267CC8B6E8DA958A7FA039",
  "account": "kshs_1e5aqegc1jb7qe514kov498kangreutujrc8wtp2tscs4xr8o39bgtx4izhik"
}
```

---

### key_expand

Returns the public key and `kshs_` address for a private key.

```bash
curl -s -d '{
  "action": "key_expand",
  "key": "781186FB9EF17DB6E3D1056550D9FAE5D5BBADA6A6BC370E4CBB938B1DC71DA3"
}' http://localhost:7076
```

---

### deterministic_key

Derives a private key, public key, and address from a seed at a given index.

```bash
curl -s -d '{
  "action": "deterministic_key",
  "seed": "YOUR_SEED_HEX",
  "index": "0"
}' http://localhost:7076
```

---

## Unit Conversion Commands

### kshs_to_raw

Converts KSHS to raw.

```bash
curl -s -d '{
  "action": "kshs_to_raw",
  "amount": "1"
}' http://localhost:7076
```

**Response:** `{"amount": "1000000000000000000000000000000"}`

---

### raw_to_kshs

Converts raw to KSHS.

```bash
curl -s -d '{
  "action": "raw_to_kshs",
  "amount": "1000000000000000000000000000000"
}' http://localhost:7076
```

**Response:** `{"amount": "1"}`

---

## Network / Node Commands

### version

Returns node version information.

```bash
curl -s -d '{"action":"version"}' http://localhost:7076
```

**Response:**
```json
{
  "rpc_version": "1",
  "store_version": "21",
  "protocol_version": "18",
  "node_vendor": "Kakitu v1.0.0",
  "store_vendor": "LMDB",
  "network": "live",
  "network_identifier": "GENESIS_HASH",
  "build_info": "..."
}
```

---

### telemetry

Returns network telemetry from connected peers.

```bash
curl -s -d '{"action":"telemetry"}' http://localhost:7076
```

---

### peers

Returns connected peers.

```bash
curl -s -d '{
  "action": "peers",
  "peer_details": "true"
}' http://localhost:7076
```

---

### representatives

Returns representative accounts and their voting weight.

```bash
curl -s -d '{
  "action": "representatives",
  "count": "100",
  "sorting": "true"
}' http://localhost:7076
```

---

### representatives_online

Returns representatives that have recently voted.

```bash
curl -s -d '{
  "action": "representatives_online",
  "weight": "true"
}' http://localhost:7076
```

---

### confirmation_quorum

Returns current quorum and online voting weight stats.

```bash
curl -s -d '{
  "action": "confirmation_quorum",
  "peer_details": "true"
}' http://localhost:7076
```

---

### stats

Returns internal node statistics.

```bash
curl -s -d '{
  "action": "stats",
  "type": "counters"
}' http://localhost:7076
```

Types: `counters`, `samples`, `all`.

---

### stop **[control]**

Gracefully stops the node.

```bash
curl -s -d '{"action":"stop"}' http://localhost:7076
```

---

## Wallet Commands **[control]**

All wallet commands require `enable_control = true`.

### wallet_create

Creates a new wallet and returns its ID.

```bash
curl -s -d '{"action":"wallet_create"}' http://localhost:7076
```

---

### account_create

Creates a new account in a wallet.

```bash
curl -s -d '{
  "action": "account_create",
  "wallet": "WALLET_ID"
}' http://localhost:7076
```

---

### send

Sends KSHS from a wallet account.

```bash
curl -s -d '{
  "action": "send",
  "wallet": "WALLET_ID",
  "source": "kshs_SOURCE",
  "destination": "kshs_DESTINATION",
  "amount": "1000000000000000000000000000000",
  "id": "UNIQUE_IDEMPOTENCY_KEY"
}' http://localhost:7076
```

**Response:** `{"block": "BLOCK_HASH"}`

---

### receive

Receives a pending block for a wallet account.

```bash
curl -s -d '{
  "action": "receive",
  "wallet": "WALLET_ID",
  "account": "kshs_YOUR_ACCOUNT",
  "block": "PENDING_BLOCK_HASH"
}' http://localhost:7076
```

---

### wallet_balances

Returns balances for all accounts in a wallet.

```bash
curl -s -d '{
  "action": "wallet_balances",
  "wallet": "WALLET_ID",
  "threshold": "1000000000000000000000000"
}' http://localhost:7076
```

---

### wallet_export

Exports the wallet seed (for backup).

```bash
curl -s -d '{
  "action": "wallet_export",
  "wallet": "WALLET_ID"
}' http://localhost:7076
```

---

### account_representative_set

Changes the representative for a wallet account.

```bash
curl -s -d '{
  "action": "account_representative_set",
  "wallet": "WALLET_ID",
  "account": "kshs_YOUR_ACCOUNT",
  "representative": "kshs_NEW_REP"
}' http://localhost:7076
```

---

## Confirmation Commands

### confirmation_history

Returns recently confirmed blocks.

```bash
curl -s -d '{
  "action": "confirmation_history",
  "hash": "OPTIONAL_SPECIFIC_HASH"
}' http://localhost:7076
```

---

### confirmation_active

Returns blocks with active elections.

```bash
curl -s -d '{"action":"confirmation_active"}' http://localhost:7076
```

---

### confirmation_info

Returns detailed information about a specific election.

```bash
curl -s -d '{
  "action": "confirmation_info",
  "root": "BLOCK_ROOT_HASH",
  "representatives": "true",
  "json_block": "true"
}' http://localhost:7076
```

---

## Ledger Commands

### ledger

Returns ledger entries (account state blocks).

```bash
curl -s -d '{
  "action": "ledger",
  "account": "kshs_3t6k35...",
  "count": "10",
  "representative": "true",
  "weight": "true",
  "receivable": "true",
  "modified_since": "0",
  "sorting": "false"
}' http://localhost:7076
```

---

### frontier_count

Returns the number of accounts in the ledger.

```bash
curl -s -d '{"action":"frontier_count"}' http://localhost:7076
```

---

### chain

Returns a list of block hashes going backwards from a frontier.

```bash
curl -s -d '{
  "action": "chain",
  "block": "FRONTIER_HASH",
  "count": "10"
}' http://localhost:7076
```

---

### successors

Returns blocks that come after a given block in an account chain.

```bash
curl -s -d '{
  "action": "successors",
  "block": "BLOCK_HASH",
  "count": "10"
}' http://localhost:7076
```

---

## Bootstrap Commands

### bootstrap

Initiates bootstrap from a specific peer.

```bash
curl -s -d '{
  "action": "bootstrap",
  "address": "peering.kakitu.org",
  "port": "7075"
}' http://localhost:7076
```

---

### bootstrap_any

Initiates bootstrap from any available peer.

```bash
curl -s -d '{"action":"bootstrap_any"}' http://localhost:7076
```

---

### bootstrap_status

Returns current bootstrap status.

```bash
curl -s -d '{"action":"bootstrap_status"}' http://localhost:7076
```
