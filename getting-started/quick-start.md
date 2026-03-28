# Quick Start

Get up and running with Kakitu in minutes. This guide walks you through getting a `kshs_` address, checking your balance, and making your first transaction.

---

## 1. Generate a kshs_ Address

You need a seed (or private key) to generate a Kakitu address. The easiest way is with **kakitu-keytools**:

```bash
# Install keytools (see tools/keytools.md for full setup)
git clone https://github.com/kakitucurrency/keytools
cd keytools
```

Generate a new seed and derive the first address:

```bash
./keytools generate
```

Example output:

```
Seed:        A3F1...9D2C
Private key: 7B4E...1A08
Public key:  C92D...4F77
Address:     kshs_3ab7...xk9m
```

Keep your **seed** secret and backed up. Your `kshs_` address is safe to share publicly.

---

## 2. View Your Address on the Explorer

Open [explorer.kakitu.org](https://explorer.kakitu.org) and search your `kshs_` address to see:

- Current balance (in KSHS)
- Pending (unreceived) blocks
- Full transaction history

---

## 3. Receive KSHS

Share your `kshs_` address with anyone who wants to send you KSHS. Incoming transfers appear as **pending blocks** on your account. You must explicitly receive them to credit your balance.

Using atto CLI wallet:

```bash
atto receive --account kshs_3ab7...xk9m
```

Or via RPC:

```bash
curl -d '{"action":"pending","account":"kshs_3ab7...xk9m"}' https://api.kakitu.org
```

See [RPC Reference](../integration/rpc-reference.md) for full details.

---

## 4. Send KSHS

Using atto CLI wallet:

```bash
atto send --from kshs_3ab7...xk9m \
          --to   kshs_1mqj8myiphp7uzzoopegxogwdqosrd4n9ybckp5kh3f8o1yuo974dywt7k7h \
          --amount 1.5
```

The transaction confirms in under 1 second. No fees are charged.

---

## 5. Check a Transaction

Every send and receive is a **block** with a unique hash. Look it up on the explorer:

```
https://explorer.kakitu.org/block/<block-hash>
```

Or query via RPC:

```bash
curl -d '{"action":"block_info","hash":"<block-hash>","json_block":"true"}' \
     https://api.kakitu.org
```

---

## Next Steps

- [Install a Node](./install-node.md) — Run your own Kakitu node
- [RPC Reference](../integration/rpc-reference.md) — Build integrations with the RPC API
- [Send & Receive (programmatic)](../integration/send-receive.md) — Integrate KSHS into your application
