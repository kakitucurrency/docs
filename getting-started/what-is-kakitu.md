# What is Kakitu?

**Kakitu** is Kenya's digital shilling — a feeless, instant cryptocurrency designed to make digital payments accessible to every Kenyan.

---

## The Vision

Kenya has a strong culture of mobile money. M-Pesa revolutionised payments, but it still charges transaction fees, requires a telecom account, and operates as a closed, centralised system. Kakitu is the next step: an open, permissionless digital cash layer that anyone can use — with no fees, no intermediaries, and no gatekeeping.

Kakitu's currency symbol is **KSHS** and all addresses use the `kshs_` prefix.

---

## How It Works

Kakitu is built on the **Nano protocol** — a battle-tested, energy-efficient block-lattice architecture where every account has its own blockchain. This design means:

- **No miners** — transactions are confirmed by a network of representatives using delegated proof-of-stake voting (Open Representative Voting, ORV)
- **No fees** — there is no block reward, so there is no incentive to charge fees
- **Near-instant confirmation** — under 1 second in normal network conditions
- **Environmentally friendly** — no proof-of-work mining; negligible energy use per transaction

Each account on the Kakitu network has a unique `kshs_` address derived from a 256-bit seed or private key.

---

## Key Properties

| Property | Value |
|---|---|
| Currency symbol | KSHS |
| Address prefix | `kshs_` |
| Protocol | Nano block-lattice |
| Fees | 0 |
| Confirmation time | < 1 second |
| Supply | Fixed (no inflation) |

---

## Why Kenya?

Kenya is one of the world's most innovative mobile money markets. But existing solutions:

- Charge per-transaction fees that add up for low-income users
- Depend on a single telecom provider's infrastructure
- Are not globally interoperable without expensive conversion

Kakitu is designed to fill that gap — a Kenya-native digital currency that works on any smartphone with internet access, integrates with local services, and has zero transaction costs.

---

## Relationship to Nano

Kakitu is a fork of the open-source Nano node software. It uses the same block-lattice data model and ORV consensus but has its own:

- Genesis block and initial supply distribution
- Network magic bytes (`KL` for live, `KD` for dev)
- P2P and RPC ports (44075 / 44076 / 44078)
- Address prefix (`kshs_` instead of `nano_`)
- Brand, explorer, and community infrastructure

For deep technical background on the underlying protocol, see the [Nano documentation](https://docs.nano.org).

---

## Useful Links

- Website: [kakitu.org](https://kakitu.org)
- Explorer: [explorer.kakitu.org](https://explorer.kakitu.org)
- GitHub: [github.com/kakitucurrency](https://github.com/kakitucurrency)
