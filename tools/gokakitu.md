# gokakitu

**Repository:** [github.com/kakitucurrency/gokakitu](https://github.com/kakitucurrency/gokakitu)

`gokakitu` is the official Go library for interacting with the Kakitu network. It provides key generation, block construction, signing, and RPC client functionality — everything you need to integrate KSHS payments into a Go application.

---

## Installation

```bash
go get github.com/kakitucurrency/gokakitu
```

---

## Key Generation

```go
package main

import (
    "fmt"
    "github.com/kakitucurrency/gokakitu/wallet"
)

func main() {
    seed, _ := wallet.NewSeed()
    privateKey, publicKey := wallet.KeypairFromSeed(seed, 0)
    address := wallet.AddressFromPublicKey(publicKey)

    fmt.Println("Address:", address) // kshs_3ab7...
}
```

---

## RPC Client

```go
package main

import (
    "fmt"
    "github.com/kakitucurrency/gokakitu/rpc"
)

func main() {
    client := rpc.NewClient("https://api.kakitu.org")

    info, err := client.AccountInfo("kshs_3ab7...")
    if err != nil {
        panic(err)
    }
    fmt.Println("Balance:", info.Balance)
    fmt.Println("Frontier:", info.Frontier)
}
```

---

## Send KSHS

```go
hash, err := client.Send(rpc.SendParams{
    Wallet:      "<wallet-id>",
    Source:      "kshs_3ab7...",
    Destination: "kshs_1mqj8myiphp7uzzoopegxogwdqosrd4n9ybckp5kh3f8o1yuo974dywt7k7h",
    Amount:      "1000000000000000000000000000000", // 1 KSHS in raw
})
```

---

## Receive Pending Blocks

```go
pending, _ := client.Pending("kshs_3ab7...", 10)
for hash := range pending.Blocks {
    client.Receive(rpc.ReceiveParams{
        Wallet:  "<wallet-id>",
        Account: "kshs_3ab7...",
        Block:   hash,
    })
}
```

---

## Block Construction (Raw)

For applications that manage keys locally (without a node wallet), `gokakitu` supports constructing and signing blocks from scratch:

```go
block := wallet.NewSendBlock(wallet.SendBlockParams{
    Previous:    frontier,
    Destination: "kshs_1mqj8...",
    Balance:     newBalance,
    PrivateKey:  privateKey,
    RepAddress:  "kshs_3ab7...",
})

// Publish to network
result, _ := client.Process(block, "send")
fmt.Println("Block hash:", result.Hash)
```

---

## Related

- [kakitucurrency-js](./kakitucurrency-js.md) — JavaScript equivalent
- [RPC Reference](../integration/rpc-reference.md)
- [Send & Receive](../integration/send-receive.md)
