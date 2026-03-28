---
title: Build Options
description: Building Kakitu node and tools from source
---

# Build Options

This page covers building Kakitu node software from source, for contributors and operators who need custom builds.

---

## kakitu-node (Core Node)

The node is built with CMake and requires a C++17-compatible compiler.

### Prerequisites

```bash
# Ubuntu 22.04
sudo apt install -y \
  build-essential cmake \
  libboost-all-dev \
  libssl-dev \
  pkg-config \
  git

# macOS
brew install cmake boost openssl pkg-config
```

### Clone and Build

```bash
git clone --recursive https://github.com/kakitucurrency/kakitu-node.git
cd kakitu-node

mkdir build && cd build

cmake .. \
  -DCMAKE_BUILD_TYPE=Release \
  -DACTIVE_NETWORK=kakitu_live_network

make -j$(nproc) kakitu_node
```

### Build Targets

| Target | Description |
|---|---|
| `kakitu_node` | The main node binary |
| `kakitu_rpc` | Standalone RPC server |
| `core_test` | Unit test suite |
| `load_test` | Network load testing tool |

### Network Flags

```bash
# Live network (default)
-DACTIVE_NETWORK=kakitu_live_network

# Test network
-DACTIVE_NETWORK=kakitu_test_network

# Beta network
-DACTIVE_NETWORK=kakitu_beta_network
```

---

## kakitu-work-server

The work server is a separate Rust project:

```bash
git clone https://github.com/kakitucurrency/kakitu-work-server.git
cd kakitu-work-server

# CPU only
cargo build --release

# With GPU (OpenCL)
cargo build --release --features opencl
```

---

## gokakitu (Go Library)

```bash
go get github.com/kakitucurrency/gokakitu
```

Or for development:

```bash
git clone https://github.com/kakitucurrency/gokakitu.git
cd gokakitu
go build ./...
go test ./...
```

---

## kakitucurrency-js (JavaScript/TypeScript)

```bash
git clone https://github.com/kakitucurrency/kakitucurrency-js.git
cd kakitucurrency-js
npm install
npm run build
npm test
```

---

## Docker Build

Build the node Docker image locally:

```bash
git clone --recursive https://github.com/kakitucurrency/kakitu-node.git
cd kakitu-node

docker build -t kakitucurrency/kakitu-node:local .
```

---

## Running Tests

```bash
cd kakitu-node/build

# Unit tests
./core_test

# RPC tests (requires running node)
./rpc_test
```

---

## Contributing

See [Contribution Guidelines](../community/contributing.md) for code standards, pull request procedures, and the development workflow.
