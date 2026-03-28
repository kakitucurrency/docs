---
title: Wallet Setup
description: Setting up and managing wallets on a Kakitu node
---

# Wallet Setup

The Kakitu node includes a built-in software wallet suitable for development and small-scale deployments. For production systems managing significant KSHS balances, see [Key Management](../integration-guides/key-management.md) for external key management approaches.

---

## Creating a Wallet

A **wallet** in the Kakitu node is a local container for one or more seeds and their derived accounts. Each wallet is identified by a `wallet_id` — a random 64-character hex string used only on this node.

!!! warning "wallet_id is not a backup"
    The `wallet_id` is a local database reference only. It cannot be used to restore funds on another machine. Always back up your **seed** instead.

### Create a new wallet

```bash
curl -s -d '{"action":"wallet_create"}' http://localhost:7076
```

```json
{
  "wallet": "6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B6B"
}
```

Save the returned `wallet` value. This is your `wallet_id`.

---

## Adding Accounts

### Create a new account (derived from wallet seed)

```bash
curl -s -d '{
  "action": "account_create",
  "wallet": "YOUR_WALLET_ID"
}' http://localhost:7076
```

```json
{
  "account": "kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3"
}
```

### Create multiple accounts at once

```bash
curl -s -d '{
  "action": "accounts_create",
  "wallet": "YOUR_WALLET_ID",
  "count": "10"
}' http://localhost:7076
```

---

## Backing Up the Seed

```bash
curl -s -d '{
  "action": "wallet_export",
  "wallet": "YOUR_WALLET_ID"
}' http://localhost:7076
```

The returned JSON contains the seed. Store it **offline and encrypted**. This seed is the only thing that can restore your accounts.

---

## Importing a Seed

```bash
curl -s -d '{
  "action": "wallet_change_seed",
  "wallet": "YOUR_WALLET_ID",
  "seed": "YOUR_64_CHAR_HEX_SEED"
}' http://localhost:7076
```

After importing, use `accounts_create` to regenerate the accounts derived from this seed.

---

## Locking and Unlocking Wallets

Wallets can be encrypted with a password:

### Set a password

```bash
curl -s -d '{
  "action": "password_change",
  "wallet": "YOUR_WALLET_ID",
  "password": "your-secure-password"
}' http://localhost:7076
```

### Unlock a wallet

```bash
curl -s -d '{
  "action": "password_enter",
  "wallet": "YOUR_WALLET_ID",
  "password": "your-secure-password"
}' http://localhost:7076
```

### Check lock status

```bash
curl -s -d '{
  "action": "wallet_locked",
  "wallet": "YOUR_WALLET_ID"
}' http://localhost:7076
```

---

## Checking Balances

### Single account balance

```bash
curl -s -d '{
  "action": "account_balance",
  "account": "kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3"
}' http://localhost:7076
```

```json
{
  "balance": "10000000000000000000000000000000",
  "pending": "0",
  "receivable": "0"
}
```

Balance is returned in **raw**. Divide by `10^30` to get KSHS.

### All wallet balances

```bash
curl -s -d '{
  "action": "wallet_balances",
  "wallet": "YOUR_WALLET_ID"
}' http://localhost:7076
```

---

## Sending KSHS

```bash
curl -s -d '{
  "action": "send",
  "wallet": "YOUR_WALLET_ID",
  "source": "kshs_SOURCE_ACCOUNT",
  "destination": "kshs_DESTINATION_ACCOUNT",
  "amount": "1000000000000000000000000000000"
}' http://localhost:7076
```

The `amount` is in **raw**. `1000000000000000000000000000000` raw = 1 KSHS.

!!! tip "Use a unique ID to prevent duplicate sends"
    Pass an `"id"` field (any unique string) to ensure idempotency — repeated calls with the same `id` return the same block hash without creating a duplicate transaction.

---

## Receiving Pending Funds

The node can automatically receive pending (receivable) blocks:

### Receive for a specific account

```bash
curl -s -d '{
  "action": "receive",
  "wallet": "YOUR_WALLET_ID",
  "account": "kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3",
  "block": "PENDING_BLOCK_HASH"
}' http://localhost:7076
```

### Auto-receive for all wallet accounts

Set `receive_minimum` in `config-node.toml` to automatically receive any receivable block above a minimum threshold:

```toml
[node]
receive_minimum = "1000000000000000000000000"  # 0.001 KSHS
```

The node will automatically receive blocks for all wallet accounts.
