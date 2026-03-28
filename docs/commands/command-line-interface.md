---
title: Command Line Interface
description: Kakitu node command-line interface reference
---

# Command Line Interface

The `kakitu_node` binary accepts command-line arguments for configuration, diagnostics, and maintenance tasks.

---

## Basic Usage

```bash
kakitu_node [OPTIONS] COMMAND
```

For Docker:
```bash
docker exec kakitu-node kakitu_node [OPTIONS] COMMAND
```

---

## Daemon

Start the node daemon:

```bash
kakitu_node daemon [OPTIONS]
```

| Option | Description |
|---|---|
| `--network=live` | Main network (default) |
| `--network=test` | Test network |
| `--network=beta` | Beta network |
| `--data_path=PATH` | Override ledger data directory |
| `--config KEY=VALUE` | Override a specific config value |
| `--daemon` | Run as a background daemon (native only) |

Examples:

```bash
# Start on main network
kakitu_node daemon --network=live

# Override config inline
kakitu_node daemon --network=live \
  --config node.enable_voting=true \
  --config rpc.enable_control=true

# Custom data directory
kakitu_node daemon --network=live --data_path=/data/kakitu
```

---

## Configuration Generation

Generate default config files:

```bash
# Node config
kakitu_node --generate_config node

# RPC config
kakitu_node --generate_config rpc
```

Redirect to files:

```bash
kakitu_node --generate_config node > /data/kakitu/config-node.toml
kakitu_node --generate_config rpc > /data/kakitu/config-rpc.toml
```

---

## Key Generation

Generate a key pair from the command line:

```bash
kakitu_node --generate_keypair
```

**Output:**
```
Private: 9F0E444C69F353324A31FC6D4B28C1CF1E3FA6C08A2A3FC37A9F9A83F4EB6340
Public:  C008B814A7D269A1FA3C6528B19201A24D797912DB9996FF02A1FF356E45552B
Account: kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3
```

---

## Diagnostics

Run system diagnostics (useful for checking OpenCL GPU availability):

```bash
kakitu_node --diagnostics
```

Output includes:
- Platform info
- Available OpenCL devices
- Ledger status
- Network connectivity tests

---

## Version

```bash
kakitu_node --version
```

---

## Wallet Management (CLI)

### Create a wallet

```bash
kakitu_node wallet_create
```

### Import a seed

```bash
kakitu_node wallet_add_adhoc --wallet=WALLET_ID --key=PRIVATE_KEY_HEX
```

### List wallet accounts

```bash
kakitu_node wallet_list --wallet=WALLET_ID
```

---

## RPC as CLI

Send an RPC command directly without a running HTTP server:

```bash
kakitu_node rpc --network=live '{"action":"block_count"}'
```

This starts a temporary RPC session against the running node's IPC socket.

---

## Ledger Operations

### Print account info

```bash
kakitu_node account_info --account=kshs_3t6k35...
```

### Print block info

```bash
kakitu_node block_info --hash=BLOCK_HASH
```

---

## Full Option Reference

```
Usage: kakitu_node [OPTIONS] [COMMAND]

Commands:
  daemon              Start the node
  snapshot            Create a ledger snapshot

Options:
  --network=NETWORK   Network to use: live, test, beta
  --data_path=PATH    Path to ledger data directory
  --config KEY=VALUE  Override config value (repeatable)
  --version           Print version and exit
  --diagnostics       Run system diagnostics
  --generate_config   Generate default config: node or rpc
  --generate_keypair  Generate a new key pair
  --help              Show this help message
```
