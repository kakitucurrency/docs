# Send & Receive KSHS

This guide covers how to programmatically send and receive KSHS in your application. It uses the `kakitucurrency-js` library for examples, but the same logic applies in Go (`gokakitu`) or via raw RPC.

---

## Overview

Kakitu uses a **block-lattice** model. Every account has its own chain of blocks. Sending KSHS creates a **send block** on the sender's chain. The recipient must then publish a **receive block** to credit their account.

Both operations require:
1. Fetching the account's current **frontier** (latest block hash)
2. Computing the new **balance** (after the send or receive)
3. Signing the block with the account's **private key**
4. Broadcasting via the `process` RPC call

---

## Sending KSHS

### Step 1 — Fetch Account Info

```javascript
const rpc = new KakituRPC('https://api.kakitu.org');
const info = await rpc.accountInfo(senderAddress);
// info.frontier — hash of the latest block
// info.balance  — current balance in raw
// info.representative — current rep address
```

### Step 2 — Compute New Balance

```javascript
import { toRaw } from 'kakitucurrency-js';

const amountRaw = toRaw('5.0');  // 5 KSHS in raw
const newBalance = BigInt(info.balance) - BigInt(amountRaw);

if (newBalance < 0n) throw new Error('Insufficient balance');
```

### Step 3 — Build and Sign the Send Block

```javascript
import { createSendBlock, signBlock } from 'kakitucurrency-js';

const block = createSendBlock({
    account:        senderAddress,
    previous:       info.frontier,
    representative: info.representative,
    balance:        newBalance.toString(),
    link:           destinationAddress,   // for send blocks, link = destination address
});

const signedBlock = signBlock(block, privateKey);
```

### Step 4 — Broadcast

```javascript
const result = await rpc.process(signedBlock, 'send');
console.log('Send block hash:', result.hash);
```

---

## Receiving KSHS

Incoming KSHS sits as a **pending block** until you explicitly receive it.

### Step 1 — Check for Pending Blocks

```javascript
const pending = await rpc.pending(receiverAddress, 10);
// Returns: { blocks: { "<hash>": "<amount_raw>", ... } }
```

### Step 2 — For Each Pending Block, Build a Receive Block

```javascript
import { createReceiveBlock, signBlock } from 'kakitucurrency-js';

for (const [pendingHash, pendingAmount] of Object.entries(pending.blocks)) {
    const info = await rpc.accountInfo(receiverAddress);

    const newBalance = BigInt(info.balance) + BigInt(pendingAmount);

    const block = createReceiveBlock({
        account:        receiverAddress,
        previous:       info.frontier,
        representative: info.representative,
        balance:        newBalance.toString(),
        link:           pendingHash,   // for receive blocks, link = pending block hash
    });

    const signedBlock = signBlock(block, privateKey);
    const result = await rpc.process(signedBlock, 'receive');
    console.log('Receive block hash:', result.hash);
}
```

---

## Opening a New Account

If an account has never been used (no frontier block), the first receive is an **open** block. Most libraries handle this automatically by checking whether `info.frontier` is the zero hash (`000...000`).

```javascript
const isNewAccount = info.frontier === '0'.repeat(64);
```

For new accounts, use `createOpenBlock` instead of `createReceiveBlock`.

---

## Work (Proof of Work)

Kakitu does not use proof of work for spam prevention on the live network by default. If your node requires work, you can generate it server-side:

```bash
curl -d '{"action":"work_generate","hash":"<frontier-hash>"}' http://localhost:44076
```

Or use the `work` field in the block object. Check your node's configuration for `work_watcher_period` and `work_peers` settings.

---

## WebSocket Subscription

To receive real-time notifications when new blocks arrive for your account, connect to the WebSocket endpoint (port 44078):

```javascript
const ws = new WebSocket('wss://api.kakitu.org');

ws.send(JSON.stringify({
    action: 'subscribe',
    topic: 'confirmation',
    options: {
        accounts: ['kshs_3ab7yz...xk9m']
    }
}));

ws.onmessage = (event) => {
    const data = JSON.parse(event.data);
    console.log('Confirmed block:', data.message.hash);
};
```

---

## Related

- [RPC Reference](./rpc-reference.md)
- [kakitucurrency-js](../tools/kakitucurrency-js.md)
- [gokakitu](../tools/gokakitu.md)
- [Network Parameters](../network/network-parameters.md)
