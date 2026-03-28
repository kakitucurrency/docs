# RPC Reference

The Kakitu node exposes a JSON-RPC API over HTTP. The public endpoint is:

```
https://api.kakitu.org
```

For local development with your own node:

```
http://localhost:44076
```

All requests are HTTP POST with a JSON body containing an `action` field. All `kshs_` addresses are used as-is.

---

## block_count

Returns the number of blocks in the ledger.

**Request:**

```bash
curl -d '{"action":"block_count"}' https://api.kakitu.org
```

**Response:**

```json
{
  "count": "1042839",
  "unchecked": "0",
  "cemented": "1042839"
}
```

---

## account_info

Returns balance, frontier block, representative, and other metadata for a `kshs_` account.

**Request:**

```bash
curl -d '{
  "action": "account_info",
  "account": "kshs_3ab7yz...xk9m",
  "representative": "true",
  "weight": "true",
  "pending": "true"
}' https://api.kakitu.org
```

**Response:**

```json
{
  "frontier": "A3F1C2D4...9D2C",
  "open_block": "7B4EAD...1A08",
  "representative_block": "C92D44...4F77",
  "balance": "12500000000000000000000000000000",
  "modified_timestamp": "1709123456",
  "block_count": "42",
  "representative": "kshs_1mqj8myiphp7uzzoopegxogwdqosrd4n9ybckp5kh3f8o1yuo974dywt7k7h",
  "weight": "0",
  "pending": "0"
}
```

Balances are in **raw** (1 KSHS = 10^30 raw).

---

## pending

Returns a list of pending (unreceived) block hashes for an account.

**Request:**

```bash
curl -d '{
  "action": "pending",
  "account": "kshs_3ab7yz...xk9m",
  "count": "10"
}' https://api.kakitu.org
```

**Response:**

```json
{
  "blocks": {
    "D1E2F3...A4B5": "1000000000000000000000000000000",
    "F9G8H7...C6D5": "500000000000000000000000000000"
  }
}
```

Keys are block hashes; values are amounts in raw.

---

## process (send block)

Publish a pre-built, signed block to the network. Used for sending KSHS when managing keys locally.

**Request:**

```bash
curl -d '{
  "action": "process",
  "json_block": "true",
  "subtype": "send",
  "block": {
    "type": "state",
    "account": "kshs_3ab7yz...xk9m",
    "previous": "A3F1C2D4...9D2C",
    "representative": "kshs_1mqj8myiphp7uzzoopegxogwdqosrd4n9ybckp5kh3f8o1yuo974dywt7k7h",
    "balance": "11500000000000000000000000000000",
    "link": "kshs_1dest...",
    "link_as_account": "kshs_1dest...",
    "signature": "AABBCC...",
    "work": "0000000000000000"
  }
}' https://api.kakitu.org
```

**Response:**

```json
{
  "hash": "D4E5F6...7G8H"
}
```

---

## process (receive block)

Publish a receive block to claim pending KSHS.

**Request:**

```bash
curl -d '{
  "action": "process",
  "json_block": "true",
  "subtype": "receive",
  "block": {
    "type": "state",
    "account": "kshs_3ab7yz...xk9m",
    "previous": "A3F1C2D4...9D2C",
    "representative": "kshs_1mqj8myiphp7uzzoopegxogwdqosrd4n9ybckp5kh3f8o1yuo974dywt7k7h",
    "balance": "13500000000000000000000000000000",
    "link": "<pending-block-hash>",
    "signature": "AABBCC...",
    "work": "0000000000000000"
  }
}' https://api.kakitu.org
```

---

## block_info

Returns full details of a specific block by its hash.

**Request:**

```bash
curl -d '{
  "action": "block_info",
  "json_block": "true",
  "hash": "D1E2F3...A4B5"
}' https://api.kakitu.org
```

**Response:**

```json
{
  "block_account": "kshs_3ab7yz...xk9m",
  "amount": "1000000000000000000000000000000",
  "balance": "12500000000000000000000000000000",
  "height": "38",
  "local_timestamp": "1709123456",
  "confirmed": "true",
  "contents": { ... }
}
```

---

## account_history

Returns a list of confirmed blocks for an account.

**Request:**

```bash
curl -d '{
  "action": "account_history",
  "account": "kshs_3ab7yz...xk9m",
  "count": "20"
}' https://api.kakitu.org
```

---

## Unit Conversion

| Amount | Raw value |
|---|---|
| 1 KSHS | `1000000000000000000000000000000` (10^30) |
| 0.1 KSHS | `100000000000000000000000000000` |
| 0.001 KSHS | `1000000000000000000000000000` |

---

## Related

- [Send & Receive](./send-receive.md)
- [kakitucurrency-js](../tools/kakitucurrency-js.md) — JS library that wraps these calls
- [gokakitu](../tools/gokakitu.md) — Go library that wraps these calls
- [Network Parameters](../network/network-parameters.md)
