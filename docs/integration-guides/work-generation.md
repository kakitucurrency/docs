---
title: Work Generation (Integration)
description: Generating Proof-of-Work for Kakitu blocks in your integration
---

# Work Generation for Integrations

Every Kakitu block requires a valid Proof-of-Work value before it can be broadcast. This guide covers work generation from the perspective of an application developer.

---

## When to Generate Work

Work is tied to the **previous block hash** of an account chain. For the first block (open block), work is generated against the **account's public key**.

| Block type | Work input |
|---|---|
| Open block (first ever) | Account public key |
| All subsequent blocks | Hash of the most recent block |

Work must meet a minimum difficulty threshold:

| Block type | Minimum difficulty |
|---|---|
| Send / Change | `fffffff800000000` |
| Receive / Open / Epoch | `fffffe0000000000` |

When in doubt, always generate at the higher Send/Change difficulty — it is valid for any block type.

---

## Requesting Work from the Node

```bash
curl -s -d '{
  "action": "work_generate",
  "hash": "PREVIOUS_BLOCK_HASH_OR_PUBLIC_KEY",
  "difficulty": "fffffff800000000"
}' http://localhost:7076
```

```json
{
  "work": "2bf29ef00786a6bc",
  "difficulty": "ffffffff8ba4a60c",
  "multiplier": "8.00",
  "hash": "PREVIOUS_BLOCK_HASH_OR_PUBLIC_KEY"
}
```

Use the returned `work` value in your block's `work` field.

---

## Pre-caching Work

For low-latency transaction sending, generate work for the **next** block immediately after a transaction is confirmed — before the user needs to send again.

```python
# After a block is confirmed:
def on_confirmation(block_hash):
    # Immediately start generating work for the next block
    work = generate_work(block_hash, difficulty="fffffff800000000")
    cache.set(f"work:{block_hash}", work, ttl=3600)

# When building the next block:
def build_block(previous_hash):
    work = cache.get(f"work:{previous_hash}")
    if not work:
        # Cache miss — generate synchronously (adds latency)
        work = generate_work(previous_hash)
    return Block(previous=previous_hash, work=work, ...)
```

---

## Validating Work

Before broadcasting a block, validate that the work value is correct:

```bash
curl -s -d '{
  "action": "work_validate",
  "work": "2bf29ef00786a6bc",
  "hash": "PREVIOUS_BLOCK_HASH_OR_PUBLIC_KEY",
  "difficulty": "fffffff800000000"
}' http://localhost:7076
```

```json
{
  "valid_all": "1",
  "valid_receive": "1",
  "difficulty": "ffffffff8ba4a60c",
  "multiplier": "8.00"
}
```

`valid_all` = `"1"` means the work meets the send/change threshold (valid for any block type).

---

## External Work Generation

For high-throughput systems, use a dedicated work server:

### Kakitu Work Server API

The work server accepts the same format as the node RPC:

```bash
curl -s -d '{
  "action": "work_generate",
  "hash": "PREVIOUS_BLOCK_HASH",
  "difficulty": "fffffff800000000"
}' http://work-server:7078
```

See [Work Generation (Node)](../running-a-node/work-generation.md) for setup instructions.

---

## Libraries

### Go — gokakitu

```go
import "github.com/kakitucurrency/gokakitu"

work, err := gokakitu.GenerateWork(previousHash, gokakitu.DifficultyBase)
```

### JavaScript — kakitucurrency-js

```javascript
import { generateWork } from 'kakitucurrency-js';

const work = await generateWork(previousHash, { difficulty: 'fffffff800000000' });
```

---

## Difficulty Multiplier

The `multiplier` in the work_generate response indicates how far above the minimum threshold the generated work is. A multiplier of `1.0` means exactly at minimum; higher values provide additional margin.

For most use cases, the default difficulty is sufficient. Increasing difficulty does not improve transaction speed or priority — work only needs to meet the minimum threshold.
