# atto-wallet

**Repository:** [github.com/kakitucurrency/atto](https://github.com/kakitucurrency/atto)

`atto` is the official command-line wallet for Kakitu. It lets you generate addresses, send and receive KSHS, and manage multiple accounts — all from the terminal.

---

## Installation

### Download Binary

Download the latest release from the [atto releases page](https://github.com/kakitucurrency/atto/releases).

```bash
# Linux/macOS example
chmod +x atto
sudo mv atto /usr/local/bin/atto
```

### Build from Source

```bash
git clone https://github.com/kakitucurrency/atto
cd atto
go build -o atto .
```

---

## Setup

On first run, `atto` will prompt you to create or import a seed:

```bash
atto init
```

Your seed is stored encrypted in `~/.atto/wallet.json`. You will be prompted for a password on each use.

---

## Generate an Address

```bash
atto address
```

Output:

```
Account 0: kshs_3ab7yz...xk9m
```

To see multiple accounts:

```bash
atto address --count 5
```

---

## Check Balance

```bash
atto balance
```

Output:

```
Account: kshs_3ab7yz...xk9m
Balance: 12.500000 KSHS
Pending:  0.000000 KSHS
```

---

## Receive Pending KSHS

```bash
atto receive
```

`atto` will scan for pending blocks and create receive blocks for each one. Transactions confirm in under 1 second.

---

## Send KSHS

```bash
atto send --to kshs_1mqj8myiphp7uzzoopegxogwdqosrd4n9ybckp5kh3f8o1yuo974dywt7k7h \
          --amount 5.0
```

Confirm the transaction when prompted. The block hash is printed on success:

```
Sent 5.0 KSHS
Block: A3F1C2D4...9D2C
```

---

## Transaction History

```bash
atto history
```

Shows the last 20 blocks for your account. Use `--count` to show more.

---

## Use a Custom Node

By default, `atto` connects to `https://api.kakitu.org`. To use your own node:

```bash
atto --rpc http://localhost:44076 balance
```

---

## Multiple Accounts

```bash
atto --account 1 balance       # use account index 1
atto --account 2 send --to ... # send from account index 2
```

---

## Related

- [Quick Start](../getting-started/quick-start.md)
- [kakitucurrency-js](./kakitucurrency-js.md) — For programmatic wallet operations
- [RPC Reference](../integration/rpc-reference.md)
