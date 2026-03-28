---
title: Docker Management
description: Managing a Kakitu node running in Docker
---

# Docker Management

This page covers common Docker operations for Kakitu node operators.

---

## Useful Commands

### Check container status

```bash
docker ps -a | grep kakitu
```

### View logs (live)

```bash
docker logs -f kakitu-node
```

### View logs (last 100 lines)

```bash
docker logs --tail 100 kakitu-node
```

### Execute a command inside the container

```bash
docker exec kakitu-node kakitu_node --version
```

### Run an RPC command

```bash
docker exec kakitu-node curl -s -d '{"action":"version"}' http://127.0.0.1:7076
```

### Stop the node

```bash
docker stop kakitu-node
```

### Start the node

```bash
docker start kakitu-node
```

### Restart the node

```bash
docker restart kakitu-node
```

---

## Upgrading the Node

Upgrading is a two-step process: pull the new image and recreate the container. The ledger data is stored on the host (`$HOST_LEDGER_DIR`) and is preserved across upgrades.

```bash
# Pull the new image
docker pull kakitucurrency/kakitu-node:latest

# Stop and remove the old container
docker stop kakitu-node
docker rm kakitu-node

# Start a new container with the same ledger volume
docker run --restart=unless-stopped \
  --name kakitu-node \
  -p 7075:7075 \
  -v "$HOME/kakitu-ledger":/root \
  kakitucurrency/kakitu-node:latest \
  kakitu_node daemon --network=live
```

!!! tip "Zero-downtime upgrades"
    The node will resume from the existing ledger on restart. Downtime during upgrade is typically under 30 seconds.

---

## Configuration Overrides

To pass custom configuration flags to the node:

```bash
docker run --restart=unless-stopped \
  --name kakitu-node \
  -p 7075:7075 \
  -v "$HOME/kakitu-ledger":/root \
  kakitucurrency/kakitu-node:latest \
  kakitu_node daemon --network=live \
  --config node.enable_voting=true \
  --config rpc.enable_control=true
```

For persistent configuration changes, edit the `config-node.toml` file in your ledger directory directly. See [Configuration](configuration.md).

---

## Accessing Config Files

The configuration files are stored in the ledger directory on the host:

```bash
ls "$HOME/kakitu-ledger/"
# config-node.toml
# config-rpc.toml
# data.ldb
# log/
```

Edit them with any text editor. Changes take effect on the next container restart.

---

## Docker Compose

For operators who prefer Docker Compose:

```yaml
# docker-compose.yml
version: "3.8"

services:
  kakitu-node:
    image: kakitucurrency/kakitu-node:latest
    container_name: kakitu-node
    restart: unless-stopped
    ports:
      - "7075:7075"
    volumes:
      - ./kakitu-ledger:/root
    command: kakitu_node daemon --network=live
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "5"
```

Start with:

```bash
docker compose up -d
docker compose logs -f
```

---

## Resource Limits

Prevent the node from consuming excessive host resources:

```bash
docker run --restart=unless-stopped \
  --name kakitu-node \
  --memory="8g" \
  --cpus="4" \
  -p 7075:7075 \
  -v "$HOME/kakitu-ledger":/root \
  kakitucurrency/kakitu-node:latest \
  kakitu_node daemon --network=live
```

Adjust values based on your hardware. See [hardware recommendations](overview.md#hardware-recommendations).
