# kakitu-keytools

**Repository:** [github.com/kakitucurrency/keytools](https://github.com/kakitucurrency/keytools)

kakitu-keytools is a command-line utility for generating and managing Kakitu cryptographic keys. It produces seeds, private keys, public keys, and `kshs_` addresses compatible with the Kakitu network.

---

## Installation

### From Source

```bash
git clone https://github.com/kakitucurrency/keytools
cd keytools
# Follow the build instructions in the repo README
```

---

## Commands

### Generate a New Key Pair

```bash
./keytools generate
```

Output:

```
Seed:        A3F1C2...9D2C
Private key: 7B4EAD...1A08
Public key:  C92D44...4F77
Address:     kshs_3ab7yz...xk9m
```

- The **seed** is 64 hex characters (256 bits). Back it up securely — it controls your funds.
- The **address** is derived from the public key using the `kshs_` prefix and base32 encoding.

### Derive Address from Seed

```bash
./keytools derive --seed <YOUR_SEED> --index 0
```

Each seed can derive up to 2^32 unique addresses (index 0 through 4294967295).

### Sign Data

```bash
./keytools sign --private-key <PRIVATE_KEY> --data <HEX_DATA>
```

Returns a 128-character hex signature.

### Verify a Signature

```bash
./keytools verify --public-key <PUBLIC_KEY> --data <HEX_DATA> --signature <SIG>
```

---

## Address Format

Kakitu addresses follow the same format as Nano but with the `kshs_` prefix:

```
kshs_[52 character base32 encoded public key][8 character checksum]
```

Example:

```
kshs_1mqj8myiphp7uzzoopegxogwdqosrd4n9ybckp5kh3f8o1yuo974dywt7k7h
```

---

## Security Notes

- Never share your seed or private key with anyone.
- Store seeds offline (paper, hardware wallet, encrypted file).
- Each index derivation is deterministic — the same seed + index always produces the same key pair.

---

## Related

- [Quick Start](../getting-started/quick-start.md)
- [gokakitu](./gokakitu.md) — Go library with key generation built in
- [kakitucurrency-js](./kakitucurrency-js.md) — JavaScript key generation
