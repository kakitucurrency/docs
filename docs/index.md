---
title: Kakitu Documentation
description: Official documentation for Kakitu — Kenya's feeless digital shilling (KSHS)
---

# Kakitu Documentation

**Kakitu** is Kenya's digital shilling — a feeless, instant cryptocurrency built on a Block Lattice architecture. Every transaction is free and confirms in under a second. No miners. No fees. No waiting.

---

## Quick Links

| Resource | URL |
|---|---|
| Explorer | [explorer.kakitu.org](https://explorer.kakitu.org) |
| API | [api.kakitu.org](https://api.kakitu.org) |
| Website | [kakitu.org](https://kakitu.org) |
| GitHub | [github.com/kakitucurrency](https://github.com/kakitucurrency) |
| Discord | [discord.gg/kakitu](https://discord.gg/kakitu) |

---

## Currency Details

| Property | Value |
|---|---|
| Symbol | KSHS |
| Address prefix | `kshs_` |
| Smallest unit | raw (1 KSHS = 10^30 raw) |
| Transaction fees | 0 (feeless) |
| Confirmation time | < 1 second |
| Supply | Fixed — no inflation |

---

## Documentation Sections

### [What is Kakitu?](what-is-kakitu/index.md)
Understand what Kakitu is, why it was built, how it differs from M-Pesa and other cryptocurrencies, and what makes it uniquely suited to Kenya and East Africa.

### [Running a Node](running-a-node/overview.md)
Everything you need to run a Kakitu node — from security setup and Docker configuration to representative voting, work generation, and monitoring.

### [Integration Guides](integration-guides/the-basics.md)
Developer-focused guides covering key management, block confirmation, work generation, WebSockets, and building applications on top of Kakitu.

### [Protocol Design](protocol-design/overview.md)
Deep technical documentation on how Kakitu works at the protocol level: the Block Lattice, Open Representative Voting (ORV), blocks, networking, and attack mitigations.

### [Commands](commands/rpc-protocol.md)
Complete reference for the Kakitu RPC protocol and command-line interface, with full `kshs_` examples.

### [Releases](releases/node-releases.md)
Node release notes, network upgrade history, and the development roadmap.

### [Glossary](glossary.md)
Definitions of key terms used throughout the Kakitu protocol and documentation.

---

## GitHub Repositories

| Repository | Description |
|---|---|
| [kakitu-node](https://github.com/kakitucurrency/kakitu-node) | Core node software (Nano fork) |
| [kakitu-keytools](https://github.com/kakitucurrency/kakitu-keytools) | Key generation and signing utilities |
| [gokakitu](https://github.com/kakitucurrency/gokakitu) | Go library for Kakitu integration |
| [kakitucurrency-js](https://github.com/kakitucurrency/kakitucurrency-js) | JavaScript/TypeScript library |
| [atto](https://github.com/kakitucurrency/atto) | CLI wallet for KSHS |
| [kakitu_mcp_server](https://github.com/kakitucurrency/kakitu_mcp_server) | AI assistant integration via MCP |

---

*Kakitu — Kenya's digital shilling. Fast, free, forever.*
