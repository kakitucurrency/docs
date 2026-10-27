# Stripe Virtual Card Issuance — Design Spec
**Date:** 2026-04-09  
**Status:** Approved

---

## Overview

Kakitu users can convert their KSHS holdings into spendable value via two paths: a Stripe-issued virtual Visa card, or a direct M-Pesa cash-out. Both paths are accessed through a single **Sell** button on the home screen.

---

## Background & Constraints

- Stripe Issuing requires cardholder billing address to be in a Stripe-supported country. Kenya is not supported; EU countries are.
- Kakitu operates a European Stripe account, making EU card issuance available.
- Dedicated card issuance platforms (Marqeta, Thredd) are out of scope — too costly and require additional licensing at this stage.
- The cardholder billing address is used for AVS (Address Verification). In Europe, AVS checks mainly the postal code. A real EU postal code matched to a real city is sufficient to avoid card declines.
- Card details (full number, CVC) must never be stored server-side — only retrieved from Stripe and forwarded to the client once.

---

## Address Pool

### Purpose
Each Kakitu user who requests a card is assigned a unique real EU address. This address becomes their Stripe cardholder billing address.

### Data Model — `stripe_addresses` table

| Column | Type | Notes |
|--------|------|-------|
| `id` | uuid PK | auto-generated |
| `line1` | string | real street + number |
| `city` | string | real city |
| `country` | string | ISO 2-letter (DE, NL, FR, ES, IT) |
| `postal_code` | string | valid postal code for the city |
| `assigned_to` | string (nullable) | kakitu_address of the assigned user |
| `assigned_at` | timestamp (nullable) | when assigned |
| `created_at` | timestamp | |

### Address Source
Real EU addresses sourced from [openaddresses.io](https://openaddresses.io) open datasets. Initial seed: 100,000 addresses across DE, NL, FR, ES, IT — sufficient for early growth. A seeding CLI (`cmd/seed-addresses/main.go`) handles bulk import from CSV.

### Assignment
When a user requests a card, `AssignAddress(kakituAddress)` atomically claims one row using `SELECT FOR UPDATE SKIP LOCKED` to handle concurrent requests safely. Assignment is permanent — one address per user for life.

---

## Card Issuance Flow (Server)

### New table — `stripe_cards`

| Column | Type | Notes |
|--------|------|-------|
| `id` | uuid PK | |
| `kakitu_address` | string | user's kshs_ address |
| `stripe_cardholder_id` | string | Stripe cardholder ID |
| `stripe_card_id` | string | Stripe card ID |
| `address_id` | uuid FK | → stripe_addresses |
| `created_at` | timestamp | |

### API Endpoint — `POST /api/issuing/card`

**Request:** `{ "account": "kshs_...", "name": "John Kamau", "phone": "+254712345678" }`  
Name and phone are collected by the Flutter app during the "Get My Free Card" onboarding step and passed in the request. They are not stored server-side beyond what Stripe holds.

**Flow:**
1. Check if user already has a card → return existing `stripe_card_id` if so
2. `AssignAddress(account)` — claim an unused address; return HTTP 503 with `{ "error": "no_addresses_available" }` if pool is exhausted (triggers manual re-seed alert)
3. `CreateCardholder(name, phone, address)` → Stripe API
4. `IssueVirtualCard(cardholderID)` → Stripe API
5. Persist to `stripe_cards`
6. Return `stripe_card_id` to client

**Response:** `{ "card_id": "ic_...", "status": "active" }`

Card details (number, CVC, expiry) are **never returned here**. The Flutter app fetches them separately via a second call that proxies Stripe's sensitive card data endpoint — forwarded to the client and not stored.

### Rate limiting
One card per `kakitu_address`. Enforced at the DB level (unique index on `kakitu_address`).

---

## New Files (wallet server)

| File | Purpose |
|------|---------|
| `models/dbmodels/stripe_address.go` | GORM model for address pool |
| `models/dbmodels/stripe_card.go` | GORM model for issued cards |
| `repository/stripe_address_repo.go` | `AssignAddress()` with SELECT FOR UPDATE SKIP LOCKED |
| `repository/stripe_card_repo.go` | `CreateCard()`, `GetCardByAccount()` |
| `stripe/cardholder.go` | Stripe API: create cardholder, issue card, fetch card details |
| `controller/issuing_c.go` | HTTP handler for `POST /api/issuing/card` and `GET /api/issuing/card/details` |
| `cmd/seed-addresses/main.go` | One-time CLI to bulk-import EU addresses from CSV |

### Migration
`database.Migrate()` updated to auto-migrate `StripeAddress` and `StripeCard`.

### Environment variables added
```
STRIPE_SECRET_KEY=sk_live_...
```

---

## Flutter Wallet — UI Flow

### Home Screen
The existing "Card" quick-action button is replaced with a **Sell** button (green, 💰 icon).

### Sell Bottom Sheet (new screen)
Tapping Sell opens a bottom sheet with two options:
- **Spend with Visa Card** → navigates to Card Screen
- **Cash Out to M-Pesa** → navigates to existing M-Pesa B2C flow

### Card Screen (new screen)
Accessed via Card tab in bottom nav or via Sell → Card path.

**Empty state** (user has no card):
- Illustrated empty card outline
- "Get My Free Card" CTA button
- Feature list: free, instant, works online, Google/Apple Pay eligible

**Active state** (user has a card):
- Gold Kakitu Visa card (dark background)
- Card number masked (last 4 visible)
- Status badge (ACTIVE / FROZEN)
- Action grid: Show Details · Freeze · History · Settings
- "Add to Google Pay" button

**Show Details sub-screen**:
- Full card number, expiry, CVC, billing ZIP — all with copy buttons
- Auto-hides after 60 seconds
- Details fetched fresh from server each time (never cached locally)

---

## M-Pesa Sell Flow
No new backend work — reuses existing `mpesa/b2c.go`. The Sell → M-Pesa path routes to the existing cash-out screen with amount input, 1 KSHS = 1 KES rate display, destination M-Pesa number, and confirm button.

---

## Out of Scope
- Physical card issuance
- Apple Pay provisioning (Google Pay only for MVP)
- Card spend notifications (future)
- Spending limits UI (Stripe default €500/day applies)
- Address pool top-up automation (manual re-seed when pool runs low)
