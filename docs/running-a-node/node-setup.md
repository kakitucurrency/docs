---
title: Node Setup
description: Step-by-step guide to setting up a Kakitu node
---

# Node Setup

This guide walks through setting up a Kakitu node on any of the available networks. The recommended method is Docker, which simplifies upgrades and ensures a consistent runtime environment.

---

## Networks

| Network | Purpose | Port | RPC Port | Has value? |
|---|---|---|---|---|
| Main | Production network for real KSHS | 7075 | 7076 | Yes |
| Test | Integration testing (disposable) | 17075 | 17076 | No |
| Beta | Pre-release testing for node upgrades | 54000 | 55000 | No |

For new integrations, start on the **test network** to validate your implementation before going live on main.

---

## Method 1: Docker (Recommended)

Docker is the recommended approach for most operators. It isolates the node, simplifies upgrades, and is easy to script.

### Step 1 — Install Docker

```bash
# Ubuntu/Debian
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
```

Log out and back in for group membership to take effect.

### Step 2 — Pull the Kakitu Node Image

=== "Main Network"
    ```bash
    docker pull kakitucurrency/kakitu-node:latest
    ```

=== "Test Network"
    ```bash
    docker pull kakitucurrency/kakitu-node:latest-test
    ```

=== "Beta Network"
    ```bash
    docker pull kakitucurrency/kakitu-node:latest-beta
    ```

### Step 3 — Start the Node

Set environment variables for convenience:

```bash
export CONTAINER_NAME="kakitu-node"
export KAKITU_TAG="latest"          # or latest-test, latest-beta
export HOST_LEDGER_DIR="$HOME/kakitu-ledger"

mkdir -p "$HOST_LEDGER_DIR"
```

=== "Main Network"
    ```bash
    docker run --restart=unless-stopped \
      --name "$CONTAINER_NAME" \
      -p 7075:7075 \
      -v "$HOST_LEDGER_DIR":/root \
      "kakitucurrency/kakitu-node:$KAKITU_TAG" \
      kakitu_node daemon --network=live
    ```

=== "Test Network"
    ```bash
    docker run --restart=unless-stopped \
      --name "${CONTAINER_NAME}-test" \
      -p 17075:17075 \
      -v "$HOST_LEDGER_DIR":/root \
      "kakitucurrency/kakitu-node:latest-test" \
      kakitu_node daemon --network=test
    ```

!!! note "RPC Port"
    Port `7076` (RPC) is intentionally not published to the host interface. Access RPC via `docker exec` or by binding to localhost only. See [Node Security](security.md).

### Step 4 — Verify Startup

```bash
docker logs -f kakitu-node
```

Confirm the output shows the correct network and version:

```
[kakitu_node] Starting Kakitu node v1.0.0 (live network)
[kakitu_node] Ledger: /root/data.ldb
[kakitu_node] Listening on 0.0.0.0:7075
[kakitu_node] RPC listening on 127.0.0.1:7076
```

### Step 5 — Check Block Count via RPC

```bash
docker exec kakitu-node kakitu_node rpc --network=live block_count
```

Or via HTTP:

```bash
curl -s -d '{"action":"block_count"}' http://localhost:7076
```

Expected response:

```json
{
  "count": "1234567",
  "unchecked": "0",
  "cemented": "1234560"
}
```

### Step 6 — Monitor Sync Progress

```bash
curl -s -d '{"action":"telemetry"}' http://localhost:7076
```

Compare your node's `block_count` against the `peer_count` metric. Your node is considered synced when within 1% of the network's reported block count.

---

## Method 2: Native Binary

For operators who prefer a native installation without Docker:

### Download

Download the latest binary from [github.com/kakitucurrency/kakitu-node/releases](https://github.com/kakitucurrency/kakitu-node/releases).

```bash
# Example (replace VERSION with the latest release tag)
wget https://github.com/kakitucurrency/kakitu-node/releases/download/VERSION/kakitu_node_linux_x86_64
chmod +x kakitu_node_linux_x86_64
sudo mv kakitu_node_linux_x86_64 /usr/local/bin/kakitu_node
```

### Run as a Systemd Service

Create a service file at `/etc/systemd/system/kakitu-node.service`:

```ini
[Unit]
Description=Kakitu Node
After=network.target

[Service]
Type=simple
User=kakitu
ExecStart=/usr/local/bin/kakitu_node daemon --network=live
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable kakitu-node
sudo systemctl start kakitu-node
sudo systemctl status kakitu-node
```

---

## Method 3: Build from Source

For contributors and advanced operators. See [Build Options](../integration-guides/build-options.md).

---

## Next Steps

After your node is running and synced:

- [Configure the node](configuration.md) to customize behavior
- [Set up a wallet](wallet-setup.md) for managing KSHS
- [Enable representative voting](representative-voting.md) to participate in consensus
- [Set up monitoring](monitoring.md) for alerting and telemetry
