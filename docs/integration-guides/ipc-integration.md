---
title: IPC Integration
description: Integrating with the Kakitu node over IPC (TCP and local socket)
---

# IPC Integration

In addition to HTTP RPC and WebSockets, the Kakitu node exposes an IPC interface for lower-overhead local integration.

---

## IPC vs RPC

| Feature | HTTP RPC | IPC |
|---|---|---|
| Protocol | HTTP/1.1 | Raw TCP / Unix socket |
| Overhead | Medium (HTTP headers) | Low |
| Use case | General integration, remote access | High-frequency local clients |
| Authentication | IP-based (bind to localhost) | File permissions (local socket) |

For most integrations, HTTP RPC is simpler and sufficient. IPC is useful when minimizing latency for high-frequency block processing on the same machine.

---

## Enabling IPC

In `config-node.toml`:

### TCP IPC

```toml
[ipc.tcp]
enabled = true
port = 7077
```

### Local Socket (Unix domain socket)

```toml
[ipc.local]
enabled = true
path = "/tmp/kakitu"
```

---

## IPC Message Format

IPC uses a custom framing protocol. Messages are length-prefixed raw bytes. The body format mirrors the JSON used in HTTP RPC, but the framing and encoding differ.

The recommended approach for most developers is to use a library that wraps IPC:

- [gokakitu](https://github.com/kakitucurrency/gokakitu) — Go library with native IPC support
- [kakitucurrency-js](https://github.com/kakitucurrency/kakitucurrency-js) — JavaScript library

### Example (Go)

```go
import "github.com/kakitucurrency/gokakitu"

client, err := gokakitu.NewIPCClient("/tmp/kakitu")
if err != nil {
    log.Fatal(err)
}
defer client.Close()

result, err := client.Call("account_balance", map[string]string{
    "account": "kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3",
})
```

---

## Security

- **Unix socket**: Access is controlled by file system permissions. Restrict to the user running the node.
- **TCP IPC**: Bind to `127.0.0.1` only. Do not expose to the network.

```toml
[ipc.tcp]
enabled = true
port = 7077
# Bind to localhost implicitly when IPC is used on the same machine
```

For Docker deployments, IPC over TCP (port 7077) is more practical than Unix sockets. Unix sockets require sharing the socket file between containers.
