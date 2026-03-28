---
title: Work Generation
description: Configuring Proof-of-Work generation for the Kakitu node
---

# Work Generation

Every Kakitu block requires a valid Proof-of-Work (PoW) value before it can be broadcast. Work is a spam-prevention mechanism — it doesn't affect consensus or add fees, but it requires computational effort to produce.

---

## Overview

Work generation uses the **Blake2b** hash function. A valid work value satisfies:

```
blake2b(nonce || prev_block_hash) >= difficulty_threshold
```

For the first block in an account (open block):

```
blake2b(nonce || public_key) >= difficulty_threshold
```

### Difficulty Thresholds (Epoch v2)

| Block Type | Threshold |
|---|---|
| Send / Change | `fffffff800000000` |
| Receive / Open / Epoch | `fffffe0000000000` |

Send/change blocks require higher difficulty because they can be used for spam; receive blocks are less costly to generate work for.

---

## Hardware Recommendations

| Hardware | Work/second (approx.) |
|---|---|
| Nvidia RTX 3080 | ~60 work/s |
| Nvidia RTX 2080 Ti | ~47 work/s |
| AMD RX 6700 XT | ~40 work/s |
| AMD Ryzen 5 3600 (CPU) | ~0.6 work/s |

GPUs are strongly preferred. CPU work generation is acceptable for low-volume operations but will cause RPC delays under load.

---

## Recommended Configurations

### High-Volume Node (Exchange / Service)

- Dedicated machine with GPU running the Kakitu Work Server
- Configure as a `work_peer` in the node's config
- Disable local CPU work: `node.work_threads = 0`

### Low-Volume Node (Personal / Small Integration)

- Run the Kakitu Work Server on the same machine as the node
- Use GPU acceleration (`--gpu 0:0`)
- Set `node.work_threads = 0`

---

## Kakitu Work Server

The Kakitu Work Server is the preferred work generation tool. It runs independently of the node and can service multiple nodes simultaneously.

### Install

```bash
# Download from releases
wget https://github.com/kakitucurrency/kakitu-work-server/releases/latest/download/kakitu_work_server_linux_x86_64
chmod +x kakitu_work_server_linux_x86_64
sudo mv kakitu_work_server_linux_x86_64 /usr/local/bin/kakitu_work_server
```

### Run with GPU

```bash
kakitu_work_server --listen 127.0.0.1:7078 --gpu 0:0
```

### Run with CPU only

```bash
kakitu_work_server --listen 127.0.0.1:7078 --cpu-threads 4
```

### Configure as a work peer

In `config-node.toml`:

```toml
[node]
work_threads = 0
work_peers = ["::ffff:127.0.0.1:7078"]
```

---

## Enabling OpenCL (GPU) in the Node Directly

If you prefer to use OpenCL directly in the node rather than the work server:

```toml
[opencl]
enable = true
platform = 0
device = 0
threads = 1048576

[node]
work_threads = 0
```

List available OpenCL devices:

```bash
kakitu_node --diagnostics
```

---

## Work Generation via RPC

Generate work manually:

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

!!! note
    `work_generate` requires `enable_control = true` in `config-rpc.toml`.

---

## Work Validation via RPC

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

---

## Pre-caching Work

For optimal latency, generate work for the next block immediately after each confirmed transaction — before the next send is needed.

After a block is confirmed:

1. Store the block hash
2. Immediately submit a `work_generate` request using that hash
3. Cache the result
4. When the next block is needed, use the cached work

For wallets where the next block type is unknown, generate at the higher `send/change` difficulty threshold to ensure the work is valid for any block type.

---

## Work Cancellation

Cancel an in-progress work generation:

```bash
curl -s -d '{
  "action": "work_cancel",
  "hash": "PREVIOUS_BLOCK_HASH"
}' http://localhost:7076
```
