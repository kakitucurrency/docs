---
title: Node Configuration
description: Configuring the Kakitu node with config-node.toml and config-rpc.toml
---

# Node Configuration

The Kakitu node uses TOML configuration files. These files are created automatically with default values on first launch and can be customized for your deployment.

---

## Configuration Files

| File | Purpose |
|---|---|
| `config-node.toml` | Node behavior, networking, voting, work generation |
| `config-rpc.toml` | RPC server settings and access control |

Default location: the **data directory** (same folder as `data.ldb`).

For Docker setups, this is the mounted volume directory (e.g., `~/kakitu-ledger/`).

---

## Reference Format

Settings in this documentation use dot notation:

```
node.enable_voting = true
```

This maps to the TOML file as:

```toml
[node]
enable_voting = true
```

---

## config-node.toml

### Core Node Settings

```toml
[node]

# Enable this node to vote in consensus (requires delegated balance)
enable_voting = false

# Number of CPU threads used for PoW generation (0 = disabled)
work_threads = 0

# Maximum PoW difficulty multiplier (default: 64)
max_work_generate_multiplier = 64

# External work generation peers (GPU work servers)
# Format: ["::ffff:127.0.0.1:7076"]
work_peers = []

# Maximum number of peers to connect to
max_peers_per_ip = 10

# Path to the ledger database
data_path = ""  # defaults to platform data directory

# Bootstrap bandwidth limits
bootstrap_bandwidth_limit = 5242880  # 5 MB/s

# Logging verbosity: off, critical, error, warning, info, debug, trace
log_level_console = "warning"
log_level_file = "info"
```

### Voting & Representative Settings

```toml
[node]
enable_voting = true

# Your representative's kshs_ address (for delegation)
representative = "kshs_1kakituxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

### Network Settings

```toml
[node]
# Peers to connect to on startup (useful for private networks or faster initial sync)
preconfigured_peers = [
  "peering.kakitu.org"
]
```

### WebSocket Settings

```toml
[websocket]
enabled = true
address = "::ffff:127.0.0.1"
port = 7078
```

### IPC Settings

```toml
[ipc.tcp]
enabled = false
port = 7077

[ipc.local]
enabled = false
path = "/tmp/kakitu"
```

### OpenCL (GPU Work Generation)

```toml
[opencl]
enable = false
device = 0
platform = 0
threads = 1048576
```

---

## config-rpc.toml

```toml
[rpc]

# Bind address for RPC server
address = "::ffff:127.0.0.1"

# RPC port
port = 7076

# Enable privileged RPC commands (DANGEROUS if exposed publicly)
enable_control = false

# Maximum RPC request size in bytes (default 32 MB)
max_request_size = 33554432

# Number of RPC threads
threads = 4
```

---

## Command-Line Overrides

Settings can be overridden on the command line without editing the TOML file. This is useful for Docker and scripted deployments:

```bash
kakitu_node daemon \
  --network=live \
  --config node.enable_voting=true \
  --config rpc.enable_control=true \
  --config node.work_threads=0
```

!!! note "Version note"
    Command-line `--config` overrides are supported from v1.0.0+. Earlier versions require TOML file edits.

---

## Network-Specific Ports

| Network | P2P Port | RPC Port | WebSocket Port |
|---|---|---|---|
| Main | 7075 | 7076 | 7078 |
| Test | 17075 | 17076 | 17078 |
| Beta | 54000 | 55000 | 57000 |

---

## Generating a Default Config

To generate the default config files without starting the daemon:

```bash
kakitu_node --generate_config node > config-node.toml
kakitu_node --generate_config rpc > config-rpc.toml
```

Or with Docker:

```bash
docker exec kakitu-node kakitu_node --generate_config node
```

---

## Viewing Active Config

```bash
curl -s -d '{"action":"config"}' http://localhost:7076
```

!!! warning
    This requires `enable_control = true` and should only be called from localhost.
