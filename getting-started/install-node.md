# Install a Kakitu Node

This guide covers how to install and run the Kakitu node software (`kakitu_node`), a fork of the Nano node.

---

## Requirements

| Resource | Minimum | Recommended |
|---|---|---|
| CPU | 2 cores | 4+ cores |
| RAM | 4 GB | 8+ GB |
| Disk | 20 GB SSD | 100+ GB SSD |
| OS | Ubuntu 20.04+ | Ubuntu 22.04 LTS |
| Network | 10 Mbps | 100 Mbps |

---

## Option A: Docker (Recommended)

The quickest way to get a node running is via Docker.

```bash
docker pull kakitucurrency/kakitu-node:latest

docker run -d \
  --name kakitu-node \
  -p 44075:44075/udp \
  -p 44075:44075/tcp \
  -p 44076:44076 \
  -v kakitu-data:/root/KakituData \
  kakitucurrency/kakitu-node:latest \
  kakitu_node daemon --network=live
```

Check the node is running:

```bash
docker logs -f kakitu-node
```

---

## Option B: Build from Source

### Install Dependencies

```bash
sudo apt-get update
sudo apt-get install -y \
  cmake ninja-build git \
  libboost-all-dev \
  libssl-dev \
  libcurl4-openssl-dev
```

### Clone and Build

```bash
git clone https://github.com/kakitucurrency/kakitu-node
cd kakitu-node
git submodule update --init --recursive

mkdir build && cd build
cmake .. -G Ninja \
  -DCMAKE_BUILD_TYPE=Release \
  -DACTIVE_NETWORK=kakitu_live_network

ninja kakitu_node
```

### Run the Node

```bash
./kakitu_node daemon --network=live
```

Data is stored in `~/KakituData/` by default.

---

## Enable RPC

To expose the RPC interface (required for integration), edit `~/KakituData/config-rpc.toml`:

```toml
[rpc]
enable = true
address = "::ffff:127.0.0.1"
port = 44076
enable_control = true
```

Then restart the node. The RPC will be available at `http://localhost:44076`.

---

## Peering

The node will automatically discover peers. To add a static peer manually, edit `~/KakituData/config-node.toml`:

```toml
[node]
preconfigured_peers = ["peering.kakitu.org:56111"]
```

---

## Verify the Node is Syncing

```bash
curl -d '{"action":"block_count"}' http://localhost:44076
```

The `count` value should increase over time as the node syncs blocks.

---

## Next Steps

- [Run a Representative Node](../community/run-a-node.md) — Help secure the network
- [RPC Reference](../integration/rpc-reference.md) — Use the local RPC for development
