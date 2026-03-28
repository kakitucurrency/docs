# kakitucurrency-js

**Repository:** [github.com/kakitucurrency/kakitucurrency-js](https://github.com/kakitucurrency/kakitucurrency-js)

`kakitucurrency-js` is the official JavaScript/TypeScript library for building Kakitu-powered applications. It supports key generation, block construction, signing, and RPC calls — compatible with both Node.js and browser environments.

---

## Installation

```bash
npm install kakitucurrency-js
# or
yarn add kakitucurrency-js
```

---

## Generate a kshs_ Address

```javascript
import { generateSeed, deriveKeypair, deriveAddress } from 'kakitucurrency-js';

const seed = generateSeed();
const { privateKey, publicKey } = deriveKeypair(seed, 0);
const address = deriveAddress(publicKey);

console.log('Address:', address); // kshs_3ab7...
```

---

## Check Account Balance

```javascript
import { KakituRPC } from 'kakitucurrency-js';

const rpc = new KakituRPC('https://api.kakitu.org');

const info = await rpc.accountInfo('kshs_3ab7...');
console.log('Balance (raw):', info.balance);
console.log('Frontier:', info.frontier);
```

---

## Send KSHS

```javascript
import { createSendBlock, signBlock, KakituRPC } from 'kakitucurrency-js';

const rpc = new KakituRPC('https://api.kakitu.org');
const info = await rpc.accountInfo(sourceAddress);

const block = createSendBlock({
    previous:        info.frontier,
    representative:  info.representative,
    balance:         newBalanceRaw,    // current balance minus amount to send (in raw)
    link:            destinationAddress,
});

const signedBlock = signBlock(block, privateKey);
const result = await rpc.process(signedBlock, 'send');
console.log('Block hash:', result.hash);
```

---

## Receive Pending KSHS

```javascript
import { createReceiveBlock, signBlock, KakituRPC } from 'kakitucurrency-js';

const rpc = new KakituRPC('https://api.kakitu.org');
const pending = await rpc.pending(address, 10);

for (const [hash, amount] of Object.entries(pending.blocks)) {
    const info = await rpc.accountInfo(address);
    const block = createReceiveBlock({
        previous:       info.frontier,
        representative: info.representative,
        balance:        BigInt(info.balance) + BigInt(amount),
        link:           hash,
    });
    const signed = signBlock(block, privateKey);
    await rpc.process(signed, 'receive');
    console.log('Received block:', hash);
}
```

---

## Unit Conversion

```javascript
import { toRaw, fromRaw } from 'kakitucurrency-js';

const raw = toRaw('1.5');        // '1500000000000000000000000000000'
const kshs = fromRaw(raw);       // '1.5'
```

1 KSHS = 10^30 raw.

---

## TypeScript Support

`kakitucurrency-js` ships with full TypeScript type definitions. Import types as needed:

```typescript
import type { AccountInfo, SendBlockParams } from 'kakitucurrency-js';
```

---

## Related

- [gokakitu](./gokakitu.md) — Go library equivalent
- [RPC Reference](../integration/rpc-reference.md)
- [Send & Receive](../integration/send-receive.md)
