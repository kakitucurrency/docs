# Genesis Block

The genesis block is the first block in the Kakitu network and is hardcoded into the node software. It establishes the initial supply of KSHS and the root of the block-lattice.

---

## Genesis Account

```
kshs_1mqj8myiphp7uzzoopegxogwdqosrd4n9ybckp5kh3f8o1yuo974dywt7k7h
```

The entire initial supply of KSHS is opened in this account at network launch.

---

## Purpose of the Genesis Block

In the Nano block-lattice model, every account has its own chain of blocks. The genesis block is the `open` block for the genesis account and acts as the source of all KSHS in circulation. From here, funds are distributed to other accounts via standard send/receive block pairs.

There is no mining in Kakitu — the genesis block is pre-defined and its hash is embedded in the node binary.

---

## Supply Distribution

The initial supply is distributed from the genesis account to:

- Network bootstrap representative accounts
- Community treasury
- Development fund
- Public faucet (for onboarding new users)

Distribution details will be published at [kakitu.org](https://kakitu.org) when the live network launches.

---

## Viewing the Genesis Account

You can inspect the genesis account and all transactions that have flowed from it on the explorer:

```
https://explorer.kakitu.org/account/kshs_1mqj8myiphp7uzzoopegxogwdqosrd4n9ybckp5kh3f8o1yuo974dywt7k7h
```

---

## Related

- [Network Parameters](./network-parameters.md)
- [What is Kakitu?](../getting-started/what-is-kakitu.md)
