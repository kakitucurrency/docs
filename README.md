# Kakitu Documentation

Welcome to the official documentation for **Kakitu** — Kenya's digital shilling (KSHS).

Kakitu is a feeless, instant cryptocurrency built on the Nano protocol, designed as a modern digital cash layer for Kenya and beyond. Every transaction is free and confirms in under a second.

---

## Quick Links

| Resource | URL |
|---|---|
| Explorer | [explorer.kakitu.org](https://explorer.kakitu.org) |
| API | [api.kakitu.org](https://api.kakitu.org) |
| Website | [kakitu.org](https://kakitu.org) |
| GitHub Org | [github.com/kakitucurrency](https://github.com/kakitucurrency) |

---

## Documentation Sections

### Getting Started
- [What is Kakitu?](./getting-started/what-is-kakitu.md) — Introduction, vision, and how it works
- [Quick Start](./getting-started/quick-start.md) — Get a `kshs_` address and make your first transaction
- [Install a Node](./getting-started/install-node.md) — Run the Kakitu node software

### Network
- [Network Parameters](./network/network-parameters.md) — Ports, genesis account, magic bytes, endpoints
- [Genesis](./network/genesis.md) — The genesis block and initial supply

### Tools & Libraries
- [kakitu-keytools](./tools/keytools.md) — Key generation and signing utilities
- [gokakitu](./tools/gokakitu.md) — Go library for Kakitu integration
- [kakitucurrency-js](./tools/kakitucurrency-js.md) — JavaScript/TypeScript library
- [atto-wallet](./tools/atto-wallet.md) — CLI wallet for sending and receiving KSHS
- [MCP Server](./tools/mcp-server.md) — AI assistant integration via Model Context Protocol

### Integration
- [RPC Reference](./integration/rpc-reference.md) — Full RPC API with `kshs_` examples
- [Send & Receive](./integration/send-receive.md) — How to programmatically send and receive KSHS

### Community
- [Run a Node](./community/run-a-node.md) — Deploy a Kakitu representative node on Railway or VPS

---

## GitHub Repositories

| Repo | Description |
|---|---|
| [kakitucurrency/kakitu-node](https://github.com/kakitucurrency/kakitu-node) | Core node software (fork of Nano) |
| [kakitucurrency/keytools](https://github.com/kakitucurrency/keytools) | Key generation utilities |
| [kakitucurrency/gokakitu](https://github.com/kakitucurrency/gokakitu) | Go library |
| [kakitucurrency/kakitucurrency-js](https://github.com/kakitucurrency/kakitucurrency-js) | JavaScript library |
| [kakitucurrency/atto](https://github.com/kakitucurrency/atto) | CLI wallet |
| [kakitucurrency/kakitu_mcp_server](https://github.com/kakitucurrency/kakitu_mcp_server) | MCP server for AI assistants |

---

## Currency Details

| Property | Value |
|---|---|
| Symbol | KSHS |
| Address prefix | `kshs_` |
| Smallest unit | raw (10^-30 KSHS) |
| Transaction fees | 0 (feeless) |
| Confirmation time | < 1 second |

---

*Kakitu — Kenya's digital shilling. Fast, free, forever.*
