---
title: Kakitu documentation has moved
description: The Kakitu developer documentation now lives at developer.kakitu.org
---

<meta http-equiv="refresh" content="0; url=https://developer.kakitu.org/docs/intro">
<link rel="canonical" href="https://developer.kakitu.org/docs/intro">

# This documentation has moved

The Kakitu developer documentation is now at
**[developer.kakitu.org](https://developer.kakitu.org/docs/intro)**.

If you are not redirected automatically, follow that link.

## Why

Everything previously published here described an **earlier version of Kakitu built on Nano** —
a block-lattice network, feeless transactions, RPC endpoints, work generation, IPC. None of it
is accurate any more.

**KSHS is now an ERC-20 token on Base**, backed one-to-one by USDC held in reserve. One KSHS is
one Kenyan shilling. Wallets are self-custody: the key is generated on the device and never
leaves it. Cash in and cash out run through licensed partner rails.

Reserve backing is public and live at
[`walletapi.kakitu.org/reserves`](https://walletapi.kakitu.org/reserves).

Rather than leave the old pages up, we have taken them down. Working from them would have cost
you real time and, if you got far enough, real money.

The new site covers authentication, merchant registration, collections, webhooks, errors and
rate limits, with SDKs for TypeScript, Python and PHP.
