# Network Parameters

This page lists all network-level constants for the Kakitu live network.

---

## Identity

| Parameter | Value |
|---|---|
| Currency name | Kakitu |
| Currency symbol | KSHS |
| Address prefix | `kshs_` |
| Protocol base | Nano block-lattice |

---

## Magic Bytes

Magic bytes are used to identify Kakitu network traffic at the wire level.

| Network | Magic bytes |
|---|---|
| Live | `KL` |
| Dev | `KD` |

---

## Ports

| Service | Port | Protocol |
|---|---|---|
| P2P (node peering) | 44075 | TCP + UDP |
| RPC | 44076 | TCP (HTTP) |
| WebSocket | 44078 | TCP (WS) |

These ports must be open inbound on any node that wants to be reachable by peers.

---

## Endpoints

| Service | URL |
|---|---|
| Explorer | [https://explorer.kakitu.org](https://explorer.kakitu.org) |
| Public RPC / API | [https://api.kakitu.org](https://api.kakitu.org) |
| Peering node | `peering.kakitu.org:56111` |

---

## Genesis Account

The genesis account holds the initial supply of KSHS and is the root of the block-lattice.

```
kshs_1mqj8myiphp7uzzoopegxogwdqosrd4n9ybckp5kh3f8o1yuo974dywt7k7h
```

View on explorer: [explorer.kakitu.org/account/kshs_1mqj8myiphp7...](https://explorer.kakitu.org/account/kshs_1mqj8myiphp7uzzoopegxogwdqosrd4n9ybckp5kh3f8o1yuo974dywt7k7h)

---

## Supply

| Parameter | Value |
|---|---|
| Total supply | Fixed at genesis |
| Smallest unit | 1 raw = 10^-30 KSHS |
| 1 KSHS | 1,000,000,000,000,000,000,000,000,000,000 raw |
| Inflation | None |

---

## Node Configuration Snippets

### config-node.toml

```toml
[node]
preconfigured_peers = ["peering.kakitu.org:56111"]
```

### config-rpc.toml

```toml
[rpc]
enable = true
port = 44076
enable_control = true
```

---

## Related

- [Genesis block](./genesis.md)
- [Install a Node](../getting-started/install-node.md)
- [RPC Reference](../integration/rpc-reference.md)
