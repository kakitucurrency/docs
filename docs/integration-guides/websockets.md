---
title: WebSockets
description: Real-time event streaming from the Kakitu node via WebSockets
---

# WebSockets

The Kakitu node exposes a WebSocket interface for real-time event streaming. This is the most efficient way to monitor transactions, votes, and network state without polling.

---

## Enabling WebSockets

Add to `config-node.toml`:

```toml
[websocket]
enabled = true
address = "::ffff:127.0.0.1"
port = 7078
```

Restart the node. The WebSocket server is available at:

```
ws://localhost:7078
```

---

## Connection

All communication uses JSON messages. Subscribe to topics with an `action: subscribe` message.

```javascript
const ws = new WebSocket('ws://localhost:7078');

ws.onopen = () => {
  console.log('Connected to Kakitu node WebSocket');
};

ws.onmessage = (event) => {
  const message = JSON.parse(event.data);
  console.log(message);
};

ws.onerror = (error) => {
  console.error('WebSocket error:', error);
};

ws.onclose = () => {
  console.log('Connection closed — reconnecting...');
  // Implement reconnect with exponential backoff
};
```

---

## Topics

### confirmation

Emitted when a block is confirmed by the network.

**Subscribe:**

```json
{
  "action": "subscribe",
  "topic": "confirmation",
  "options": {
    "confirmation_type": "active_quorum",
    "all_local_accounts": true,
    "accounts": [
      "kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3"
    ]
  }
}
```

| Option | Values | Description |
|---|---|---|
| `confirmation_type` | `active_quorum`, `active_confirmation_height`, `inactive` | Filter by confirmation mechanism |
| `all_local_accounts` | `true`/`false` | Subscribe to all accounts in local wallets |
| `accounts` | array of `kshs_` addresses | Subscribe to specific accounts |

**Message:**

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
      "previous": "...",
      "representative": "kshs_...",
      "balance": "0",
      "link": "...",
      "signature": "...",
      "work": "..."
    }
  }
}
```

---

### vote

Emitted when a representative vote is observed.

**Subscribe:**

```json
{
  "action": "subscribe",
  "topic": "vote",
  "options": {
    "representatives": [
      "kshs_REPRESENTATIVE_ADDRESS"
    ]
  }
}
```

**Message:**

```json
{
  "topic": "vote",
  "time": "1648000000000",
  "message": {
    "account": "kshs_REPRESENTATIVE",
    "signature": "VOTE_SIGNATURE",
    "sequence": "42",
    "timestamp": "1648000000000",
    "duration": "0",
    "blocks": [
      "A170D51B94E00371ACE76E35AC81DC9405D5D04D4CEBC399AEACE07AE05DD293"
    ],
    "type": "replay"
  }
}
```

---

### new_unconfirmed_block

Emitted when the node receives a new block before it is confirmed. Use only for visibility; **never credit funds** based on unconfirmed blocks.

**Subscribe:**

```json
{
  "action": "subscribe",
  "topic": "new_unconfirmed_block"
}
```

---

### stopped_election

Emitted when a block election ends without confirmation (e.g., due to a fork resolution).

```json
{
  "action": "subscribe",
  "topic": "stopped_election"
}
```

---

### active_difficulty

Emitted when the network's minimum work difficulty changes.

```json
{
  "action": "subscribe",
  "topic": "active_difficulty"
}
```

**Message:**

```json
{
  "topic": "active_difficulty",
  "time": "1648000000000",
  "message": {
    "network_minimum": "fffffff800000000",
    "network_receive_minimum": "fffffe0000000000",
    "network_current": "fffffff800000000",
    "multiplier": "1.0"
  }
}
```

---

### telemetry

Emitted periodically with node telemetry data.

```json
{
  "action": "subscribe",
  "topic": "telemetry"
}
```

---

## Unsubscribe

```json
{
  "action": "unsubscribe",
  "topic": "confirmation"
}
```

---

## Updating a Subscription

Modify an existing subscription without reconnecting:

```json
{
  "action": "update",
  "topic": "confirmation",
  "options": {
    "accounts_add": ["kshs_NEW_ACCOUNT"],
    "accounts_del": ["kshs_OLD_ACCOUNT"]
  }
}
```

---

## Reconnection Strategy

WebSocket connections can drop. Always implement reconnection with exponential backoff:

```javascript
let reconnectDelay = 1000; // 1 second

function connect() {
  const ws = new WebSocket('ws://localhost:7078');

  ws.onopen = () => {
    reconnectDelay = 1000; // Reset on successful connection
    subscribeToTopics(ws);
  };

  ws.onclose = () => {
    setTimeout(() => {
      reconnectDelay = Math.min(reconnectDelay * 2, 30000); // Cap at 30s
      connect();
    }, reconnectDelay);
  };
}

connect();
```

When reconnecting, check `receivable` and `account_info` via RPC to catch any confirmations that occurred during the disconnected period.
