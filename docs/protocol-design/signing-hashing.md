---
title: Signing and Hashing
description: Ed25519 signatures and Blake2b hashing in the Kakitu protocol
---

# Signing and Hashing

Kakitu uses two cryptographic primitives for security: **Ed25519** for digital signatures and **Blake2b** for hashing.

---

## Ed25519 Signatures

All Kakitu blocks are signed with **Ed25519** — a high-performance elliptic-curve digital signature algorithm using Curve25519.

### Key Generation

Private keys in Kakitu are 32-byte (256-bit) random values. Public keys are derived deterministically:

```
public_key = ed25519_derive_public_key(private_key)
```

The corresponding `kshs_` account address is then derived from the public key:

```
address = "kshs_" + base32_encode(public_key || blake2b_40(public_key))
```

Where `blake2b_40` is the first 5 bytes (40 bits) of a Blake2b hash, used as a checksum.

### Block Signing

Every block must be signed by the private key of the account that owns the block:

```
block_hash = blake2b_256(preamble || account_pubkey || previous || rep_pubkey || balance || link)
signature = ed25519_sign(private_key, block_hash)
```

The preamble is a constant 32-byte value that domain-separates Kakitu block hashes from other uses of the same inputs.

### Vote Signing

Representative votes are also signed with Ed25519:

```
vote_hash = blake2b_256("vote " || blocks_hashes || sequence)
vote_signature = ed25519_sign(representative_private_key, vote_hash)
```

---

## Blake2b Hashing

Kakitu uses **Blake2b** for all hashing operations. Blake2b is fast, secure, and does not require padding or initialization vectors.

### Uses in Kakitu

| Operation | Hash function | Output size |
|---|---|---|
| Block hash | Blake2b | 256 bits (32 bytes) |
| Address checksum | Blake2b | 40 bits (5 bytes) |
| Seed → private key derivation | Blake2b | 256 bits |
| PoW verification | Blake2b | 64 bits (first 8 bytes) |

### Seed to Private Key Derivation

```
private_key[index] = blake2b_256(seed || uint32_be(index))
```

Where `uint32_be(index)` is the 4-byte big-endian representation of the derivation index (0, 1, 2, ...).

---

## Address Format

A Kakitu address is structured as follows:

```
kshs_[base32(public_key)][base32(checksum)]
```

- Prefix: `kshs_`
- Alphabet: `13456789abcdefghijkmnopqrstuwxyz` (custom 32-character set, no `0`, `2`, `l`, `v`)
- Public key: 52 base32 characters (encoding 260 bits; 256 bits public key + 4 padding)
- Checksum: 8 base32 characters (encoding 5 bytes / 40 bits of Blake2b hash)
- Total length: `kshs_` + 60 characters = 65 characters

---

## Libraries

### Go — gokakitu

```go
import "github.com/kakitucurrency/gokakitu"

// Generate key pair
privateKey, publicKey := gokakitu.GenerateKeyPair()

// Get address from public key
address := gokakitu.PublicKeyToAddress(publicKey)

// Sign a block hash
signature := gokakitu.Sign(privateKey, blockHash)

// Verify a signature
valid := gokakitu.Verify(publicKey, blockHash, signature)
```

### JavaScript — kakitucurrency-js

```javascript
import { generateKeyPair, publicKeyToAddress, signBlock, verifyBlock } from 'kakitucurrency-js';

const { privateKey, publicKey } = generateKeyPair();
const address = publicKeyToAddress(publicKey);
const signature = signBlock(privateKey, blockHash);
const valid = verifyBlock(publicKey, blockHash, signature);
```

---

## Test Vectors

### Address Derivation

| Private Key | Public Key | Address |
|---|---|---|
| `000...001` | `19D3D919475DEED4696B5D13018151D1AF88B2BD3BCFF048B45031C1F36D1858` | `kshs_1pu7jyqontmattfn3sl5uea2j38et1329wpfmx3qhny7m9xq695hswkqnepd` |

### Block Hash

A correctly constructed state block with:
- Account: `kshs_3t6k35gi95xu6tergt6p69ck76ogmitsa8mnijtpxm9fkcm736xtoncuohr3`
- Previous: `A170D51B...`
- Balance: `1000000000000000000000000000000`
- Link: `E89208DD...`

Should produce a deterministic block hash that can be verified with any compliant Ed25519 library.
