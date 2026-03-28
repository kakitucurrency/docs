---
title: Key Management
description: Private key management strategies for Kakitu integrations
---

# Key Management

Robust key management is the most critical part of any Kakitu integration. This guide covers three approaches, ranging from quick internal management to production-grade external key vaults.

---

## Seeds and Key Derivation

### Hex Seeds

A Kakitu seed is a 64-character uppercase hex string (32 random bytes):

```
0000000000000000000000000000000000000000000000000000000000000001
```

Private keys are derived deterministically from a seed:

```
private_key[index] = blake2b(seed || uint32_be(index))
```

Where `uint32_be(index)` is the 4-byte big-endian encoding of the account index (0, 1, 2, ...).

### Mnemonic Seeds (BIP39)

Kakitu supports 24-word (and 12-word) BIP39 mnemonics using coin-type **165** in the BIP44 derivation path:

```
m/44'/165'/[index]'
```

Example test vector (24-word):

| Field | Value |
|---|---|
| Mnemonic | `edge ability shrug bamboo relax loyal more hundred limit appear trade divorce coil thing push grocery shiver chapter devote century minor canoe maple` |
| Seed (hex) | `0DC285FDE768F7FF29B66CE7252D56ED92FE003B605907F7A4F683C3DC8586D34` |
| Account 0 private key | `3BE4FC2EF3F3B7374E6FC4FB6E7BB153F8A2998B3B3DAB50853EABE128024143` |
| Account 0 address | `kshs_1pu7jyqontmattfn3sl5uea2j38et1329wpfmx3qhny7m9xq695hswkqnepd` |

---

## Approach 1: External Key Management (Recommended for Production)

For high-volume integrations (exchanges, payment processors, service wallets):

1. Generate seeds and private keys using a cryptographically secure random number generator — **outside** the node
2. Store private keys in an encrypted database or HSM
3. Use the node **only** for:
   - Querying receivable blocks (`receivable` RPC)
   - Broadcasting signed blocks (`process` RPC)
   - Querying account state (`account_info` RPC)
4. Sign all blocks in your application, not in the node

### Generating a Key Pair (Node RPC)

```bash
curl -s -d '{"action":"key_create"}' http://localhost:7076
```

```json
{
  "private": "781186FB9EF17DB6E3D1056550D9FAE5D5BBADA6A6BC370E4CBB938B1DC71DA3",
  "public": "3068BB1CA04525BB0E416C485FE6A67FD52540227D267CC8B6E8DA958A7FA039",
  "account": "kshs_1e5aqegc1jb7qe514kov498kangreutujrc8wtp2tscs4xr8o39bgtx4izhik"
}
```

### Deriving from a Seed (Node RPC)

```bash
curl -s -d '{
  "action": "deterministic_key",
  "seed": "YOUR_SEED_HEX",
  "index": "0"
}' http://localhost:7076
```

```json
{
  "private": "9F0E444C69F353324A31FC6D4B28C1CF1E3FA6C08A2A3FC37A9F9A83F4EB6340",
  "public": "C008B814A7D269A1FA3C6528B19201A24D797912DB9996FF02A1FF356E45552B",
  "account": "kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3"
}
```

### Signing Blocks Externally

Using [kakitu-keytools](https://github.com/kakitucurrency/kakitu-keytools) (Go):

```go
import "github.com/kakitucurrency/kakitu-keytools"

block := keytools.StateBlock{
    Account:        "kshs_3t6k35...",
    Previous:       "previous_hash",
    Representative: "kshs_representative...",
    Balance:        "990000000000000000000000000000",
    Link:           "destination_public_key",
}

signature := keytools.Sign(privateKey, block.Hash())
block.Signature = signature
```

---

## Approach 2: Internal Key Management (Suitable for Small Deployments)

For services managing fewer than ~1000 accounts, the node's built-in wallet is convenient:

### Create and manage wallets

See [Wallet Setup](../running-a-node/wallet-setup.md) for full wallet RPC documentation.

### Critical rules for internal management

1. **Always back up the seed** — not the wallet ID
2. **Set a wallet password** — encrypt the wallet at rest
3. **Never enable `enable_control` on a public RPC port**
4. **Track transactions by block hash** — not by order of creation

---

## Confirmation is Mandatory

!!! danger "Never credit funds without confirmation"
    Unconfirmed blocks can be rolled back. Always wait for a block to reach **confirmed** status before considering a payment received.

See [Block Confirmation](block-confirmation.md) for how to reliably detect confirmed transactions.

---

## Idempotency

Duplicate `send` RPC calls can create duplicate transactions. Prevent this by passing a unique `id` field:

```bash
curl -s -d '{
  "action": "send",
  "wallet": "YOUR_WALLET_ID",
  "source": "kshs_SOURCE",
  "destination": "kshs_DESTINATION",
  "amount": "1000000000000000000000000000000",
  "id": "payment_invoice_12345"
}' http://localhost:7076
```

Subsequent calls with the same `id` return the same block hash without sending again.

For external signing, track the block hash and check for its existence before broadcasting:

```bash
curl -s -d '{
  "action": "block_info",
  "hash": "BLOCK_HASH"
}' http://localhost:7076
```

If the block exists, it has already been broadcast — do not submit again.

---

## Security Checklist

- [ ] Seed generated with a CSPRNG (cryptographically secure random number generator)
- [ ] Seed stored offline or in an encrypted vault (not in `config-node.toml` or environment variables in plaintext)
- [ ] Wallet encrypted with a strong password
- [ ] `enable_control` only accessible from localhost
- [ ] Block confirmation verified before crediting user accounts
- [ ] Transaction idempotency implemented (unique IDs or block hash checks)
- [ ] Private keys never transmitted over unencrypted connections
