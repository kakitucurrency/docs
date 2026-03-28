---
title: Block Confirmation
description: How to reliably confirm blocks in a Kakitu integration
---

# Block Confirmation

Confirming that a block has been accepted by the network before crediting a user account is **the most important safety requirement** in any Kakitu integration.

---

## What is Confirmation?

A block is **confirmed** when representatives holding more than **67% of online voting weight** have voted in favor of it. At that point, the block is considered irreversible under normal network conditions.

Unconfirmed blocks (those broadcast but not yet confirmed) can theoretically be rolled back if the network receives a competing block — this is resolved by the ORV consensus protocol, but your integration should never act on unconfirmed state.

---

## Two Methods for Tracking Confirmation

### Method 1: WebSocket Subscription (Recommended)

WebSockets provide real-time push notifications as blocks are confirmed. This is the most reliable and efficient method for active transaction processing.

Enable WebSockets in `config-node.toml`:

```toml
[websocket]
enabled = true
address = "::ffff:127.0.0.1"
port = 7078
```

Subscribe to confirmation events using any WebSocket client:

```javascript
const ws = new WebSocket('ws://localhost:7078');

ws.onopen = () => {
  ws.send(JSON.stringify({
    action: 'subscribe',
    topic: 'confirmation',
    options: {
      confirmation_type: 'active_quorum',
      all_local_accounts: true,
      // Filter to specific accounts (optional):
      accounts: [
        'kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3'
      ]
    }
  }));
};

ws.onmessage = (event) => {
  const msg = JSON.parse(event.data);
  if (msg.topic === 'confirmation') {
    const block = msg.message;
    console.log(`Confirmed: ${block.hash}`);
    console.log(`Amount: ${block.amount} raw`);
    console.log(`Account: ${block.account}`);
  }
};
```

Example confirmation message:

```json
{
  "topic": "confirmation",
  "time": "1648000000000",
  "message": {
    "account": "kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3",
    "amount": "1000000000000000000000000000000",
    "hash": "A170D51B94E00371ACE76E35AC81DC9405D5D04D4CEBC399AEACE07AE05DD293",
    "confirmation_type": "active_quorum",
    "block": {
      "type": "state",
      "account": "kshs_3t6k35...",
      "previous": "PREVIOUS_HASH",
      "representative": "kshs_REPRESENTATIVE",
      "balance": "0",
      "link": "DESTINATION_OR_SOURCE",
      "signature": "SIGNATURE",
      "work": "WORK_VALUE"
    }
  }
}
```

### Method 2: Polling (Fallback)

For simpler integrations or situations where WebSockets are not available, poll the `block_info` RPC:

```bash
curl -s -d '{
  "action": "block_info",
  "hash": "A170D51B94E00371ACE76E35AC81DC9405D5D04D4CEBC399AEACE07AE05DD293",
  "json_block": "true"
}' http://localhost:7076
```

Check the `confirmed` field:

```json
{
  "block_account": "kshs_3t6k35...",
  "amount": "1000000000000000000000000000000",
  "balance": "0",
  "height": "5",
  "local_timestamp": "1648000000",
  "successor": "",
  "confirmed": "true",
  "contents": { ... }
}
```

Poll until `confirmed` is `"true"`. Under normal network conditions this takes less than 1 second.

!!! warning "Do not use unconfirmed block data for balance calculations"
    RPC calls like `account_balance` and `account_info` may include unconfirmed blocks unless you explicitly request confirmed-only data. Always check the `confirmed` field on individual blocks before crediting accounts.

---

## Confirmation Height

Use the `confirmation_height` field in `account_info` to efficiently detect new confirmations:

```bash
curl -s -d '{
  "action": "account_info",
  "account": "kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3",
  "confirmation_height": "true"
}' http://localhost:7076
```

```json
{
  "frontier": "LATEST_BLOCK_HASH",
  "confirmation_height": "5",
  "confirmation_height_frontier": "CONFIRMED_BLOCK_HASH",
  "block_count": "5"
}
```

When `confirmation_height` equals `block_count`, all blocks on this account are confirmed.

---

## Receivable (Pending) Blocks

To check for incoming funds that have been sent but not yet received by this account:

```bash
curl -s -d '{
  "action": "receivable",
  "account": "kshs_3t6k35...",
  "count": "64",
  "threshold": "1000000000000000000000000",
  "source": "true"
}' http://localhost:7076
```

```json
{
  "blocks": {
    "A170D51B94E00371ACE76E35AC81DC9405D5D04D4CEBC399AEACE07AE05DD293": {
      "amount": "1000000000000000000000000000000",
      "source": "kshs_SENDER_ADDRESS"
    }
  }
}
```

!!! note
    Only **confirmed** send blocks appear in the receivable list by default. This is important — it means you can safely create receive blocks for anything in this list.

---

## Election State

For in-progress transactions, check the election state:

```bash
curl -s -d '{
  "action": "confirmation_active"
}' http://localhost:7076
```

```bash
curl -s -d '{
  "action": "confirmation_info",
  "root": "BLOCK_ROOT"
}' http://localhost:7076
```

---

## Confirmed Balance

To get the confirmed balance (excluding unconfirmed pending amounts):

```bash
curl -s -d '{
  "action": "account_balance",
  "account": "kshs_3t6k35...",
  "include_only_confirmed": "true"
}' http://localhost:7076
```

---

## Best Practices Summary

1. Always use `"include_only_confirmed": "true"` in balance checks
2. Subscribe to WebSocket confirmations for real-time processing
3. Store confirmed block hashes in your database — never re-process the same hash
4. For receivable blocks: only create receive blocks for confirmed send blocks
5. Poll `block_info` as a fallback if WebSocket is disconnected
6. Test your confirmation flow on the test network before going live
