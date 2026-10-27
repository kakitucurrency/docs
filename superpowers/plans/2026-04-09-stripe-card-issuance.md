# Stripe Virtual Card Issuance Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Allow Kakitu users to issue a free virtual Visa card (via Stripe) or cash out to M-Pesa through a unified "Sell" flow in the Flutter wallet.

**Architecture:** The Go wallet server gains a Postgres-backed EU address pool, a Stripe Issuing wrapper, and two new HTTP endpoints (`POST /issuing/card`, `GET /issuing/card/details`). The Flutter app replaces the direct Sell→M-Pesa path with a method-selection sheet that routes to either the new Card screen or the existing SellSheet.

**Tech Stack:** Go 1.25 + GORM + stripe-go/v76 + Chi router (backend); Flutter/Dart + http package (frontend).

---

## File Map

### Wallet Server (`kakitu_wallet_server/`)
| Action | File | Responsibility |
|--------|------|---------------|
| Create | `stripe/cardholder.go` | Stripe API calls: create cardholder, issue card, fetch sensitive card details |
| Create | `models/dbmodels/stripe_address.go` | GORM model for address pool |
| Create | `models/dbmodels/stripe_card.go` | GORM model for issued cards |
| Create | `repository/stripe_address_repo.go` | `AssignAddress()` — atomic pool claim |
| Create | `repository/stripe_card_repo.go` | `CreateCard()`, `GetCardByAccount()` |
| Create | `controller/issuing_c.go` | HTTP handlers for card issuance and details |
| Create | `cmd/seed-addresses/main.go` | One-time CLI to bulk-import EU addresses from CSV |
| Modify | `database/postgres.go` | Add `StripeAddress`, `StripeCard` to `Migrate()` |
| Modify | `main.go` | Instantiate `IssuingController`, register `/issuing` routes |

### Flutter Wallet (`kakitu_wallet_flutter-master/`)
| Action | File | Responsibility |
|--------|------|---------------|
| Create | `lib/ui/card/card_service.dart` | HTTP calls to `/issuing/card` and `/issuing/card/details` |
| Create | `lib/ui/sell/sell_method_sheet.dart` | Bottom sheet: choose Card or M-Pesa |
| Create | `lib/ui/card/card_screen.dart` | Full card screen (empty + active states + Google Pay button) |
| Create | `lib/ui/card/card_details_sheet.dart` | Show full number, CVC, billing ZIP with 60s auto-hide |
| Modify | `lib/ui/home_page.dart` | Wire Sell pill to open `SellMethodSheet` instead of `SellSheet` |

---

## Task 1: Add stripe-go dependency

**Files:**
- Modify: `kakitu_wallet_server/go.mod`

- [ ] **Step 1: Add stripe-go**

```bash
cd /Users/kiptengwer/Documents/kakitu/kakitu_wallet_server
go get github.com/stripe/stripe-go/v76
```

Expected: `go.mod` and `go.sum` updated with `github.com/stripe/stripe-go/v76 v76.x.x`

- [ ] **Step 2: Verify build still compiles**

```bash
/opt/homebrew/bin/go build ./...
```

Expected: no errors

- [ ] **Step 3: Commit**

```bash
cd /Users/kiptengwer/Documents/kakitu/kakitu_wallet_server
git add go.mod go.sum
git commit -m "chore: add stripe-go/v76 dependency"
```

---

## Task 2: DB models — StripeAddress and StripeCard

**Files:**
- Create: `kakitu_wallet_server/models/dbmodels/stripe_address.go`
- Create: `kakitu_wallet_server/models/dbmodels/stripe_card.go`

- [ ] **Step 1: Write stripe_address.go**

```go
// kakitu_wallet_server/models/dbmodels/stripe_address.go
package dbmodels

import "time"

type StripeAddress struct {
	ID         string     `json:"id" gorm:"primaryKey;type:uuid;default:gen_random_uuid()"`
	Line1      string     `json:"line1" gorm:"not null"`
	City       string     `json:"city" gorm:"not null"`
	Country    string     `json:"country" gorm:"not null;size:2"`
	PostalCode string     `json:"postal_code" gorm:"not null"`
	AssignedTo *string    `json:"assigned_to" gorm:"uniqueIndex;default:null"`
	AssignedAt *time.Time `json:"assigned_at" gorm:"default:null"`
	CreatedAt  time.Time  `json:"created_at"`
}
```

- [ ] **Step 2: Write stripe_card.go**

```go
// kakitu_wallet_server/models/dbmodels/stripe_card.go
package dbmodels

import "time"

type StripeCard struct {
	ID                 string    `json:"id" gorm:"primaryKey;type:uuid;default:gen_random_uuid()"`
	KakituAddress      string    `json:"kakitu_address" gorm:"uniqueIndex;not null"`
	StripeCardholderID string    `json:"stripe_cardholder_id" gorm:"not null"`
	StripeCardID       string    `json:"stripe_card_id" gorm:"not null"`
	AddressID          string    `json:"address_id" gorm:"not null"`
	CreatedAt          time.Time `json:"created_at"`
}
```

- [ ] **Step 3: Verify compilation**

```bash
cd /Users/kiptengwer/Documents/kakitu/kakitu_wallet_server
/opt/homebrew/bin/go build ./...
```

Expected: no errors

- [ ] **Step 4: Commit**

```bash
git add models/dbmodels/stripe_address.go models/dbmodels/stripe_card.go
git commit -m "feat: add StripeAddress and StripeCard DB models"
```

---

## Task 3: Update database migrations

**Files:**
- Modify: `kakitu_wallet_server/database/postgres.go`

- [ ] **Step 1: Add new models to Migrate()**

In `database/postgres.go`, replace the existing `Migrate` function:

```go
func Migrate(db *gorm.DB) error {
	return db.AutoMigrate(
		&dbmodels.FcmToken{},
		&dbmodels.MpesaTransaction{},
		&dbmodels.StripeAddress{},
		&dbmodels.StripeCard{},
	)
}
```

- [ ] **Step 2: Verify compilation**

```bash
cd /Users/kiptengwer/Documents/kakitu/kakitu_wallet_server
/opt/homebrew/bin/go build ./...
```

Expected: no errors

- [ ] **Step 3: Commit**

```bash
git add database/postgres.go
git commit -m "feat: migrate StripeAddress and StripeCard tables"
```

---

## Task 4: StripeAddress repository

**Files:**
- Create: `kakitu_wallet_server/repository/stripe_address_repo.go`

- [ ] **Step 1: Write the repository**

```go
// kakitu_wallet_server/repository/stripe_address_repo.go
package repository

import (
	"errors"
	"time"

	"github.com/kakitucurrency/kakitu-wallet-server/models/dbmodels"
	"gorm.io/gorm"
	"gorm.io/gorm/clause"
)

var ErrNoAddressesAvailable = errors.New("no addresses available in pool")

type StripeAddressRepo struct {
	DB *gorm.DB
}

// AssignAddress atomically claims one unassigned address for the given kakituAddress.
// Uses SELECT FOR UPDATE SKIP LOCKED so concurrent requests never race on the same row.
// Returns ErrNoAddressesAvailable if the pool is empty.
func (r *StripeAddressRepo) AssignAddress(kakituAddress string) (*dbmodels.StripeAddress, error) {
	var addr dbmodels.StripeAddress
	err := r.DB.Transaction(func(tx *gorm.DB) error {
		res := tx.Clauses(clause.Locking{Strength: "UPDATE", Options: "SKIP LOCKED"}).
			Where("assigned_to IS NULL").
			First(&addr)
		if res.Error != nil {
			if errors.Is(res.Error, gorm.ErrRecordNotFound) {
				return ErrNoAddressesAvailable
			}
			return res.Error
		}
		now := time.Now().UTC()
		return tx.Model(&addr).Updates(map[string]interface{}{
			"assigned_to": kakituAddress,
			"assigned_at": now,
		}).Error
	})
	if err != nil {
		return nil, err
	}
	return &addr, nil
}

// AvailableCount returns how many unassigned addresses remain in the pool.
func (r *StripeAddressRepo) AvailableCount() (int64, error) {
	var count int64
	err := r.DB.Model(&dbmodels.StripeAddress{}).Where("assigned_to IS NULL").Count(&count).Error
	return count, err
}
```

- [ ] **Step 2: Verify compilation**

```bash
cd /Users/kiptengwer/Documents/kakitu/kakitu_wallet_server
/opt/homebrew/bin/go build ./...
```

Expected: no errors

- [ ] **Step 3: Commit**

```bash
git add repository/stripe_address_repo.go
git commit -m "feat: add StripeAddressRepo with atomic AssignAddress"
```

---

## Task 5: StripeCard repository

**Files:**
- Create: `kakitu_wallet_server/repository/stripe_card_repo.go`

- [ ] **Step 1: Write the repository**

```go
// kakitu_wallet_server/repository/stripe_card_repo.go
package repository

import (
	"errors"

	"github.com/kakitucurrency/kakitu-wallet-server/models/dbmodels"
	"gorm.io/gorm"
)

type StripeCardRepo struct {
	DB *gorm.DB
}

// GetCardByAccount returns the card for a kakituAddress, or nil if none exists.
func (r *StripeCardRepo) GetCardByAccount(kakituAddress string) (*dbmodels.StripeCard, error) {
	var card dbmodels.StripeCard
	err := r.DB.Where("kakitu_address = ?", kakituAddress).First(&card).Error
	if err != nil {
		if errors.Is(err, gorm.ErrRecordNotFound) {
			return nil, nil
		}
		return nil, err
	}
	return &card, nil
}

// CreateCard persists a newly issued Stripe card.
func (r *StripeCardRepo) CreateCard(kakituAddress, cardholderID, cardID, addressID string) (*dbmodels.StripeCard, error) {
	card := &dbmodels.StripeCard{
		KakituAddress:      kakituAddress,
		StripeCardholderID: cardholderID,
		StripeCardID:       cardID,
		AddressID:          addressID,
	}
	if err := r.DB.Create(card).Error; err != nil {
		return nil, err
	}
	return card, nil
}
```

- [ ] **Step 2: Verify compilation**

```bash
cd /Users/kiptengwer/Documents/kakitu/kakitu_wallet_server
/opt/homebrew/bin/go build ./...
```

Expected: no errors

- [ ] **Step 3: Commit**

```bash
git add repository/stripe_card_repo.go
git commit -m "feat: add StripeCardRepo"
```

---

## Task 6: Stripe API wrapper

**Files:**
- Create: `kakitu_wallet_server/stripe/cardholder.go`

- [ ] **Step 1: Write the Stripe wrapper**

```go
// kakitu_wallet_server/stripe/cardholder.go
package stripeissuing

import (
	"os"

	"github.com/stripe/stripe-go/v76"
	"github.com/stripe/stripe-go/v76/issuing/card"
	"github.com/stripe/stripe-go/v76/issuing/cardholder"
)

func init() {
	stripe.Key = os.Getenv("STRIPE_SECRET_KEY")
}

type CardholderParams struct {
	Name       string
	Phone      string
	Line1      string
	City       string
	Country    string
	PostalCode string
}

type IssuedCard struct {
	CardholderID string
	CardID       string
	Last4        string
	ExpMonth     uint64
	ExpYear      uint64
}

type CardDetails struct {
	Number     string
	CVC        string
	ExpMonth   uint64
	ExpYear    uint64
	PostalCode string
}

// CreateCardholder creates a Stripe Issuing cardholder and immediately issues a virtual card.
func CreateCardholder(p CardholderParams) (*IssuedCard, error) {
	ch, err := cardholder.New(&stripe.IssuingCardholderParams{
		Name:  stripe.String(p.Name),
		Type:  stripe.String("individual"),
		Email: stripe.String(""),
		Phone: stripe.String(p.Phone),
		Billing: &stripe.IssuingCardholderBillingParams{
			Address: &stripe.AddressParams{
				Line1:      stripe.String(p.Line1),
				City:       stripe.String(p.City),
				Country:    stripe.String(p.Country),
				PostalCode: stripe.String(p.PostalCode),
			},
		},
	})
	if err != nil {
		return nil, err
	}

	c, err := card.New(&stripe.IssuingCardParams{
		Cardholder: stripe.String(ch.ID),
		Currency:   stripe.String("eur"),
		Type:       stripe.String("virtual"),
	})
	if err != nil {
		return nil, err
	}

	// Activate the card
	active := stripe.IssuingCardStatusActive
	_, err = card.Update(c.ID, &stripe.IssuingCardParams{
		Status: &active,
	})
	if err != nil {
		return nil, err
	}

	return &IssuedCard{
		CardholderID: ch.ID,
		CardID:       c.ID,
		Last4:        c.Last4,
		ExpMonth:     c.ExpMonth,
		ExpYear:      c.ExpYear,
	}, nil
}

// GetCardDetails fetches the sensitive card data (number, CVC) from Stripe.
// Never cache the result — call fresh each time the user requests to see details.
func GetCardDetails(cardID string, postalCode string) (*CardDetails, error) {
	c, err := card.Get(cardID, &stripe.IssuingCardParams{
		Params: stripe.Params{
			Expand: []*string{
				stripe.String("number"),
				stripe.String("cvc"),
			},
		},
	})
	if err != nil {
		return nil, err
	}
	return &CardDetails{
		Number:     c.Number,
		CVC:        c.CVC,
		ExpMonth:   c.ExpMonth,
		ExpYear:    c.ExpYear,
		PostalCode: postalCode,
	}, nil
}
```

- [ ] **Step 2: Verify compilation**

```bash
cd /Users/kiptengwer/Documents/kakitu/kakitu_wallet_server
/opt/homebrew/bin/go build ./...
```

Expected: no errors

- [ ] **Step 3: Commit**

```bash
git add stripe/cardholder.go
git commit -m "feat: add Stripe Issuing wrapper (create cardholder, issue card, get details)"
```

---

## Task 7: Issuing HTTP controller

**Files:**
- Create: `kakitu_wallet_server/controller/issuing_c.go`

- [ ] **Step 1: Write the controller**

```go
// kakitu_wallet_server/controller/issuing_c.go
package controller

import (
	"encoding/json"
	"errors"
	"net/http"

	"github.com/go-chi/render"
	"github.com/kakitucurrency/kakitu-wallet-server/repository"
	stripeissuing "github.com/kakitucurrency/kakitu-wallet-server/stripe"
	"github.com/kakitucurrency/kakitu-wallet-server/utils"
	"k8s.io/klog/v2"
)

type IssuingController struct {
	AddressRepo *repository.StripeAddressRepo
	CardRepo    *repository.StripeCardRepo
}

type issueCardRequest struct {
	Account string `json:"account"`
	Name    string `json:"name"`
	Phone   string `json:"phone"`
}

type cardDetailsRequest struct {
	Account string `json:"account"`
}

// HandleIssueCard handles POST /issuing/card
// Issues a new virtual Visa card for the given Kakitu account.
// If the account already has a card, returns the existing card ID.
func (ic *IssuingController) HandleIssueCard(w http.ResponseWriter, r *http.Request) {
	var req issueCardRequest
	if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
		ErrBadrequest(w, r, "invalid request body")
		return
	}
	if !utils.ValidateAddress(req.Account) {
		ErrBadrequest(w, r, "invalid account address")
		return
	}
	if req.Name == "" {
		ErrBadrequest(w, r, "name is required")
		return
	}
	if req.Phone == "" {
		ErrBadrequest(w, r, "phone is required")
		return
	}

	// Check for existing card
	existing, err := ic.CardRepo.GetCardByAccount(req.Account)
	if err != nil {
		klog.Errorf("GetCardByAccount error: %v", err)
		ErrInternalServerError(w, r, "database error")
		return
	}
	if existing != nil {
		render.Status(r, http.StatusOK)
		render.JSON(w, r, map[string]interface{}{
			"card_id": existing.StripeCardID,
			"last4":   "",
			"status":  "active",
		})
		return
	}

	// Claim an address from the pool
	addr, err := ic.AddressRepo.AssignAddress(req.Account)
	if err != nil {
		if errors.Is(err, repository.ErrNoAddressesAvailable) {
			render.Status(r, http.StatusServiceUnavailable)
			render.JSON(w, r, map[string]string{"error": "no_addresses_available"})
			return
		}
		klog.Errorf("AssignAddress error: %v", err)
		ErrInternalServerError(w, r, "address pool error")
		return
	}

	// Issue the card via Stripe
	issued, err := stripeissuing.CreateCardholder(stripeissuing.CardholderParams{
		Name:       req.Name,
		Phone:      req.Phone,
		Line1:      addr.Line1,
		City:       addr.City,
		Country:    addr.Country,
		PostalCode: addr.PostalCode,
	})
	if err != nil {
		klog.Errorf("Stripe CreateCardholder error: %v", err)
		ErrInternalServerError(w, r, "stripe error")
		return
	}

	// Persist the card
	if _, err := ic.CardRepo.CreateCard(req.Account, issued.CardholderID, issued.CardID, addr.ID); err != nil {
		klog.Errorf("CreateCard DB error: %v", err)
		ErrInternalServerError(w, r, "database error")
		return
	}

	render.Status(r, http.StatusOK)
	render.JSON(w, r, map[string]interface{}{
		"card_id":   issued.CardID,
		"last4":     issued.Last4,
		"exp_month": issued.ExpMonth,
		"exp_year":  issued.ExpYear,
		"status":    "active",
	})
}

// HandleCardDetails handles GET /issuing/card/details
// Returns the sensitive card data (number, CVC, expiry, billing ZIP).
// Never stored server-side — fetched fresh from Stripe on every call.
func (ic *IssuingController) HandleCardDetails(w http.ResponseWriter, r *http.Request) {
	account := r.URL.Query().Get("account")
	if !utils.ValidateAddress(account) {
		ErrBadrequest(w, r, "invalid account address")
		return
	}

	card, err := ic.CardRepo.GetCardByAccount(account)
	if err != nil {
		klog.Errorf("GetCardByAccount error: %v", err)
		ErrInternalServerError(w, r, "database error")
		return
	}
	if card == nil {
		render.Status(r, http.StatusNotFound)
		render.JSON(w, r, map[string]string{"error": "no card found for this account"})
		return
	}

	// Fetch address postal code for AVS display
	var addr struct{ PostalCode string }
	ic.AddressRepo.DB.Raw("SELECT postal_code FROM stripe_addresses WHERE id = ?", card.AddressID).Scan(&addr)

	details, err := stripeissuing.GetCardDetails(card.StripeCardID, addr.PostalCode)
	if err != nil {
		klog.Errorf("GetCardDetails Stripe error: %v", err)
		ErrInternalServerError(w, r, "stripe error")
		return
	}

	render.Status(r, http.StatusOK)
	render.JSON(w, r, map[string]interface{}{
		"number":      details.Number,
		"cvc":         details.CVC,
		"exp_month":   details.ExpMonth,
		"exp_year":    details.ExpYear,
		"postal_code": details.PostalCode,
	})
}
```

- [ ] **Step 2: Verify compilation**

```bash
cd /Users/kiptengwer/Documents/kakitu/kakitu_wallet_server
/opt/homebrew/bin/go build ./...
```

Expected: no errors

- [ ] **Step 3: Commit**

```bash
git add controller/issuing_c.go
git commit -m "feat: add IssuingController for card issuance and details endpoints"
```

---

## Task 8: Register routes in main.go

**Files:**
- Modify: `kakitu_wallet_server/main.go`

- [ ] **Step 1: Add imports**

Add to the import block in `main.go` (around line 17):

```go
stripeissuing "github.com/kakitucurrency/kakitu-wallet-server/stripe"
```

Wait — the `stripe` package init already sets `stripe.Key` from env. No import needed in main. Skip this sub-step.

- [ ] **Step 2: Instantiate repos and controller**

In `main.go`, after the existing `mpesaTxnRepo` instantiation (around line 164), add:

```go
stripeAddressRepo := &repository.StripeAddressRepo{DB: db}
stripeCardRepo := &repository.StripeCardRepo{DB: db}
ic := controller.IssuingController{
    AddressRepo: stripeAddressRepo,
    CardRepo:    stripeCardRepo,
}
```

- [ ] **Step 3: Register the routes**

In `main.go`, after the M-Pesa route block (around line 246), add:

```go
// Stripe Issuing routes
app.Route("/issuing", func(r chi.Router) {
    r.Post("/card", ic.HandleIssueCard)
    r.Get("/card/details", ic.HandleCardDetails)
})
```

- [ ] **Step 4: Verify compilation**

```bash
cd /Users/kiptengwer/Documents/kakitu/kakitu_wallet_server
/opt/homebrew/bin/go build ./...
```

Expected: binary `kakitu-server` builds cleanly

- [ ] **Step 5: Smoke test with test key**

```bash
export STRIPE_SECRET_KEY=sk_test_REDACTED
./kakitu-server &
sleep 1
curl -s -X POST http://localhost:3000/issuing/card \
  -H "Content-Type: application/json" \
  -d '{"account":"kshs_3t6xviu57d4x4szqfcq3mev9otc3mq3ys8j4s4ycwhoyax4y5kgsfez4","name":"Test User","phone":"+254712345678"}'
kill %1
```

Expected: `{"error":"no_addresses_available"}` (pool is empty — correct, seeding comes next) or a Stripe error about the test key. If you see `{"error":"invalid account address"}` recheck the address format.

- [ ] **Step 6: Commit**

```bash
git add main.go
git commit -m "feat: register /issuing routes and wire IssuingController"
```

---

## Task 9: Address pool seeding CLI

**Files:**
- Create: `kakitu_wallet_server/cmd/seed-addresses/main.go`

- [ ] **Step 1: Download a sample EU address CSV**

OpenAddresses.io provides free datasets. For testing, create a small sample CSV at `/tmp/eu_addresses_sample.csv`:

```bash
cat > /tmp/eu_addresses_sample.csv << 'EOF'
line1,city,country,postal_code
Hauptstraße 12,Berlin,DE,10115
Kurfürstendamm 45,Berlin,DE,10707
Unter den Linden 7,Berlin,DE,10117
Friedrichstraße 100,Berlin,DE,10117
Alexanderplatz 1,Berlin,DE,10178
Prenzlauer Allee 36,Berlin,DE,10405
Torstraße 220,Berlin,DE,10115
Rosenthaler Str 40,Berlin,DE,10178
Oranienburger Str 67,Berlin,DE,10117
Kastanienallee 15,Berlin,DE,10435
Hermannstraße 84,Berlin,DE,12051
Sonnenallee 101,Berlin,DE,12045
Kottbusser Damm 24,Berlin,DE,10967
Bergmannstraße 7,Berlin,DE,10961
Graefestraße 19,Berlin,DE,10967
Weserstraße 46,Berlin,DE,12047
Mainzer Straße 9,Berlin,DE,10247
Rigaer Straße 71,Berlin,DE,10247
Warschauer Str 34,Berlin,DE,10243
Boxhagener Str 82,Berlin,DE,10245
EOF
```

- [ ] **Step 2: Write the seeding CLI**

```go
// kakitu_wallet_server/cmd/seed-addresses/main.go
package main

import (
	"encoding/csv"
	"flag"
	"fmt"
	"os"
	"time"

	"github.com/kakitucurrency/kakitu-wallet-server/database"
	"github.com/kakitucurrency/kakitu-wallet-server/models/dbmodels"
	"gorm.io/gorm"
	"gorm.io/gorm/clause"
)

func main() {
	csvPath := flag.String("csv", "", "Path to CSV file with columns: line1,city,country,postal_code")
	batchSize := flag.Int("batch", 500, "Insert batch size")
	flag.Parse()

	if *csvPath == "" {
		fmt.Fprintln(os.Stderr, "Usage: seed-addresses -csv /path/to/addresses.csv")
		os.Exit(1)
	}

	db, err := database.NewConnection(&database.Config{
		URL:     os.Getenv("DATABASE_URL"),
		Host:    os.Getenv("DB_HOST"),
		Port:    os.Getenv("DB_PORT"),
		User:    os.Getenv("DB_USER"),
		Password: os.Getenv("DB_PASS"),
		DBName:  os.Getenv("DB_NAME"),
		SSLMode: os.Getenv("DB_SSLMODE"),
	})
	if err != nil {
		fmt.Fprintf(os.Stderr, "DB connection failed: %v\n", err)
		os.Exit(1)
	}

	f, err := os.Open(*csvPath)
	if err != nil {
		fmt.Fprintf(os.Stderr, "Cannot open CSV: %v\n", err)
		os.Exit(1)
	}
	defer f.Close()

	r := csv.NewReader(f)
	rows, err := r.ReadAll()
	if err != nil {
		fmt.Fprintf(os.Stderr, "CSV parse error: %v\n", err)
		os.Exit(1)
	}

	if len(rows) < 2 {
		fmt.Fprintln(os.Stderr, "CSV must have header + at least 1 data row")
		os.Exit(1)
	}

	var batch []dbmodels.StripeAddress
	inserted := 0
	skipped := 0

	for _, row := range rows[1:] { // skip header
		if len(row) < 4 {
			skipped++
			continue
		}
		batch = append(batch, dbmodels.StripeAddress{
			Line1:      row[0],
			City:       row[1],
			Country:    row[2],
			PostalCode: row[3],
			CreatedAt:  time.Now().UTC(),
		})

		if len(batch) >= *batchSize {
			if err := insertBatch(db, batch); err != nil {
				fmt.Fprintf(os.Stderr, "Insert error: %v\n", err)
				os.Exit(1)
			}
			inserted += len(batch)
			batch = batch[:0]
			fmt.Printf("Inserted %d rows...\n", inserted)
		}
	}

	if len(batch) > 0 {
		if err := insertBatch(db, batch); err != nil {
			fmt.Fprintf(os.Stderr, "Insert error: %v\n", err)
			os.Exit(1)
		}
		inserted += len(batch)
	}

	fmt.Printf("Done. Inserted: %d, Skipped (malformed): %d\n", inserted, skipped)
}

func insertBatch(db *gorm.DB, batch []dbmodels.StripeAddress) error {
	return db.Clauses(clause.OnConflict{DoNothing: true}).Create(&batch).Error
}
```

- [ ] **Step 3: Build and test the CLI**

```bash
cd /Users/kiptengwer/Documents/kakitu/kakitu_wallet_server
/opt/homebrew/bin/go build -o /tmp/seed-addresses ./cmd/seed-addresses/
/tmp/seed-addresses -csv /tmp/eu_addresses_sample.csv
```

Expected output (with DB running):
```
Inserted 20 rows...
Done. Inserted: 20, Skipped (malformed): 0
```

- [ ] **Step 4: Verify rows exist**

```bash
psql $DATABASE_URL -c "SELECT count(*) FROM stripe_addresses WHERE assigned_to IS NULL;"
```

Expected: `20` (or however many rows were in the CSV)

- [ ] **Step 5: Commit**

```bash
git add cmd/seed-addresses/main.go
git commit -m "feat: add address pool seeding CLI (seed-addresses)"
```

---

## Task 10: Flutter — CardService

**Files:**
- Create: `kakitu_wallet_flutter-master/lib/ui/card/card_service.dart`

- [ ] **Step 1: Write CardService**

```dart
// lib/ui/card/card_service.dart
import 'dart:convert';
import 'package:http/http.dart' as http;

const String _BASE_URL = 'https://walletapi.kakitu.org';

class IssuedCard {
  final String cardId;
  final String last4;
  final int expMonth;
  final int expYear;
  final String status;

  IssuedCard({
    required this.cardId,
    required this.last4,
    required this.expMonth,
    required this.expYear,
    required this.status,
  });

  factory IssuedCard.fromJson(Map<String, dynamic> json) => IssuedCard(
        cardId: json['card_id'] as String,
        last4: json['last4'] as String? ?? '',
        expMonth: (json['exp_month'] as num?)?.toInt() ?? 0,
        expYear: (json['exp_year'] as num?)?.toInt() ?? 0,
        status: json['status'] as String? ?? 'active',
      );
}

class CardDetails {
  final String number;
  final String cvc;
  final int expMonth;
  final int expYear;
  final String postalCode;

  CardDetails({
    required this.number,
    required this.cvc,
    required this.expMonth,
    required this.expYear,
    required this.postalCode,
  });

  factory CardDetails.fromJson(Map<String, dynamic> json) => CardDetails(
        number: json['number'] as String,
        cvc: json['cvc'] as String,
        expMonth: (json['exp_month'] as num).toInt(),
        expYear: (json['exp_year'] as num).toInt(),
        postalCode: json['postal_code'] as String,
      );
}

class CardService {
  /// Issues a virtual Visa card for the given Kakitu account.
  /// Returns null if the account already has a card (server returns existing card_id).
  static Future<IssuedCard> issueCard({
    required String account,
    required String name,
    required String phone,
  }) async {
    final resp = await http.post(
      Uri.parse('$_BASE_URL/issuing/card'),
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode({'account': account, 'name': name, 'phone': phone}),
    );
    final body = jsonDecode(resp.body) as Map<String, dynamic>;
    if (resp.statusCode == 200) return IssuedCard.fromJson(body);
    final error = body['error'] as String? ?? 'unknown error';
    throw Exception(error);
  }

  /// Fetches sensitive card details. Never cache — call fresh each time.
  static Future<CardDetails> getCardDetails(String account) async {
    final resp = await http.get(
      Uri.parse('$_BASE_URL/issuing/card/details?account=${Uri.encodeComponent(account)}'),
    );
    final body = jsonDecode(resp.body) as Map<String, dynamic>;
    if (resp.statusCode == 200) return CardDetails.fromJson(body);
    final error = body['error'] as String? ?? 'unknown error';
    throw Exception(error);
  }
}
```

- [ ] **Step 2: Verify Flutter analyzes cleanly**

```bash
cd /Users/kiptengwer/Documents/kakitu/kakitu_wallet_flutter-master
flutter analyze lib/ui/card/card_service.dart
```

Expected: `No issues found!`

- [ ] **Step 3: Commit**

```bash
git add lib/ui/card/card_service.dart
git commit -m "feat: add CardService for Stripe card issuance API calls"
```

---

## Task 11: Flutter — SellMethodSheet

**Files:**
- Create: `kakitu_wallet_flutter-master/lib/ui/sell/sell_method_sheet.dart`

- [ ] **Step 1: Write SellMethodSheet**

```dart
// lib/ui/sell/sell_method_sheet.dart
import 'package:flutter/material.dart';
import 'package:kakitu_wallet_flutter/styles.dart';
import 'package:kakitu_wallet_flutter/ui/sell/sell_sheet.dart';
import 'package:kakitu_wallet_flutter/ui/card/card_screen.dart';
import 'package:kakitu_wallet_flutter/ui/widgets/sheet_util.dart';

class SellMethodSheet extends StatelessWidget {
  const SellMethodSheet({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return SafeArea(
      minimum: EdgeInsets.only(bottom: MediaQuery.of(context).viewInsets.bottom),
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          // Handle
          Container(
            margin: const EdgeInsets.only(top: 12, bottom: 8),
            width: 40,
            height: 4,
            decoration: BoxDecoration(
              color: Colors.white24,
              borderRadius: BorderRadius.circular(2),
            ),
          ),
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 24),
            child: Text(
              'How do you want to sell?',
              style: AppStyles.textStyleSize20W700Primary(context),
              textAlign: TextAlign.center,
            ),
          ),
          const SizedBox(height: 4),
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 24),
            child: Text(
              'Choose how to receive your KES',
              style: AppStyles.textStyleSize14W100Primary(context),
              textAlign: TextAlign.center,
            ),
          ),
          const SizedBox(height: 20),
          // Card option
          _MethodTile(
            icon: Icons.credit_card_rounded,
            title: 'Spend with Visa Card',
            subtitle: 'Use your virtual card online, in-app & with Google Pay',
            color: const Color(0xFFFFD700),
            onTap: () {
              Navigator.of(context).pop();
              Sheets.showAppHeightNineSheet(
                context: context,
                widget: const CardScreen(),
              );
            },
          ),
          const SizedBox(height: 10),
          // M-Pesa option
          _MethodTile(
            icon: Icons.phone_android_rounded,
            title: 'Cash Out to M-Pesa',
            subtitle: 'Receive KES directly to your M-Pesa number',
            color: const Color(0xFF4CAF50),
            onTap: () {
              Navigator.of(context).pop();
              Sheets.showAppHeightNineSheet(
                context: context,
                widget: const SellSheet(),
              );
            },
          ),
          const SizedBox(height: 12),
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 24),
            child: Text(
              '1 KSHS = 1 KES  ·  No conversion fee  ·  Instant',
              style: AppStyles.textStyleSize12W400Primary(context),
              textAlign: TextAlign.center,
            ),
          ),
          const SizedBox(height: 24),
        ],
      ),
    );
  }
}

class _MethodTile extends StatelessWidget {
  final IconData icon;
  final String title;
  final String subtitle;
  final Color color;
  final VoidCallback onTap;

  const _MethodTile({
    required this.icon,
    required this.title,
    required this.subtitle,
    required this.color,
    required this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 16),
      child: InkWell(
        onTap: onTap,
        borderRadius: BorderRadius.circular(14),
        child: Container(
          padding: const EdgeInsets.all(14),
          decoration: BoxDecoration(
            color: color.withOpacity(0.08),
            border: Border.all(color: color.withOpacity(0.3)),
            borderRadius: BorderRadius.circular(14),
          ),
          child: Row(
            children: [
              Container(
                padding: const EdgeInsets.all(10),
                decoration: BoxDecoration(
                  color: color.withOpacity(0.15),
                  borderRadius: BorderRadius.circular(10),
                ),
                child: Icon(icon, color: color, size: 24),
              ),
              const SizedBox(width: 14),
              Expanded(
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(title,
                        style: AppStyles.textStyleSize16W700Primary(context)
                            .copyWith(color: color)),
                    const SizedBox(height: 2),
                    Text(subtitle,
                        style: AppStyles.textStyleSize12W400Primary(context)),
                  ],
                ),
              ),
              Icon(Icons.chevron_right_rounded, color: color),
            ],
          ),
        ),
      ),
    );
  }
}
```

- [ ] **Step 2: Check which AppStyles methods exist**

```bash
grep -n "textStyleSize" /Users/kiptengwer/Documents/kakitu/kakitu_wallet_flutter-master/lib/styles.dart | head -20
```

If any style method used above doesn't exist, substitute the closest match found in that output.

- [ ] **Step 3: Verify Flutter analyzes cleanly**

```bash
cd /Users/kiptengwer/Documents/kakitu/kakitu_wallet_flutter-master
flutter analyze lib/ui/sell/sell_method_sheet.dart
```

Expected: `No issues found!`

- [ ] **Step 4: Commit**

```bash
git add lib/ui/sell/sell_method_sheet.dart
git commit -m "feat: add SellMethodSheet (choose Card or M-Pesa)"
```

---

## Task 12: Flutter — CardDetailsSheet

**Files:**
- Create: `kakitu_wallet_flutter-master/lib/ui/card/card_details_sheet.dart`

- [ ] **Step 1: Write CardDetailsSheet**

```dart
// lib/ui/card/card_details_sheet.dart
import 'dart:async';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:kakitu_wallet_flutter/styles.dart';
import 'package:kakitu_wallet_flutter/ui/card/card_service.dart';
import 'package:kakitu_wallet_flutter/appstate_container.dart';

class CardDetailsSheet extends StatefulWidget {
  const CardDetailsSheet({Key? key}) : super(key: key);

  @override
  State<CardDetailsSheet> createState() => _CardDetailsSheetState();
}

class _CardDetailsSheetState extends State<CardDetailsSheet> {
  CardDetails? _details;
  bool _loading = true;
  String? _error;
  int _secondsLeft = 60;
  Timer? _timer;

  @override
  void initState() {
    super.initState();
    _load();
  }

  Future<void> _load() async {
    final account = StateContainer.of(context).wallet.address;
    try {
      final details = await CardService.getCardDetails(account);
      if (!mounted) return;
      setState(() {
        _details = details;
        _loading = false;
      });
      _startTimer();
    } catch (e) {
      if (!mounted) return;
      setState(() {
        _error = e.toString();
        _loading = false;
      });
    }
  }

  void _startTimer() {
    _timer = Timer.periodic(const Duration(seconds: 1), (t) {
      if (!mounted) { t.cancel(); return; }
      setState(() => _secondsLeft--);
      if (_secondsLeft <= 0) {
        t.cancel();
        Navigator.of(context).pop();
      }
    });
  }

  @override
  void dispose() {
    _timer?.cancel();
    super.dispose();
  }

  void _copy(String value, String label) {
    Clipboard.setData(ClipboardData(text: value));
    ScaffoldMessenger.of(context)
        .showSnackBar(SnackBar(content: Text('$label copied')));
  }

  @override
  Widget build(BuildContext context) {
    return SafeArea(
      minimum: EdgeInsets.only(bottom: MediaQuery.of(context).viewInsets.bottom),
      child: Padding(
        padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 16),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            // Handle
            Container(
              width: 40, height: 4,
              margin: const EdgeInsets.only(bottom: 16),
              decoration: BoxDecoration(
                color: Colors.white24,
                borderRadius: BorderRadius.circular(2),
              ),
            ),
            Row(
              children: [
                Text('Card Details',
                    style: AppStyles.textStyleSize20W700Primary(context)),
                const Spacer(),
                if (_details != null)
                  Container(
                    padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 3),
                    decoration: BoxDecoration(
                      color: Colors.orange.withOpacity(0.2),
                      borderRadius: BorderRadius.circular(6),
                    ),
                    child: Text('⏱ ${_secondsLeft}s',
                        style: const TextStyle(color: Colors.orange, fontSize: 12)),
                  ),
              ],
            ),
            const SizedBox(height: 16),
            if (_loading)
              const Padding(
                padding: EdgeInsets.all(24),
                child: CircularProgressIndicator(),
              )
            else if (_error != null)
              Text(_error!, style: const TextStyle(color: Colors.red))
            else if (_details != null) ...[
              _DetailRow(
                label: 'Card Number',
                value: _details!.number,
                onCopy: () => _copy(_details!.number, 'Card number'),
              ),
              _DetailRow(
                label: 'Expiry',
                value:
                    '${_details!.expMonth.toString().padLeft(2, '0')} / ${_details!.expYear}',
              ),
              _DetailRow(
                label: 'CVC',
                value: _details!.cvc,
                onCopy: () => _copy(_details!.cvc, 'CVC'),
              ),
              _DetailRow(
                label: 'Billing ZIP',
                value: _details!.postalCode,
                onCopy: () => _copy(_details!.postalCode, 'Billing ZIP'),
              ),
              const SizedBox(height: 8),
              Text(
                'These details are hidden after ${_secondsLeft}s for your security',
                style: AppStyles.textStyleSize12W400Primary(context),
                textAlign: TextAlign.center,
              ),
            ],
            const SizedBox(height: 16),
          ],
        ),
      ),
    );
  }
}

class _DetailRow extends StatelessWidget {
  final String label;
  final String value;
  final VoidCallback? onCopy;

  const _DetailRow({required this.label, required this.value, this.onCopy});

  @override
  Widget build(BuildContext context) {
    return Container(
      margin: const EdgeInsets.only(bottom: 8),
      padding: const EdgeInsets.symmetric(horizontal: 14, vertical: 10),
      decoration: BoxDecoration(
        color: Colors.white.withOpacity(0.05),
        borderRadius: BorderRadius.circular(10),
      ),
      child: Row(
        children: [
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(label,
                    style: const TextStyle(color: Colors.grey, fontSize: 11)),
                const SizedBox(height: 2),
                Text(value,
                    style: const TextStyle(
                        color: Colors.white,
                        fontSize: 14,
                        fontWeight: FontWeight.w600,
                        letterSpacing: 1.5)),
              ],
            ),
          ),
          if (onCopy != null)
            IconButton(
              icon: const Icon(Icons.copy_rounded, size: 18, color: Color(0xFFFFD700)),
              onPressed: onCopy,
            ),
        ],
      ),
    );
  }
}
```

- [ ] **Step 2: Verify Flutter analyzes cleanly**

```bash
cd /Users/kiptengwer/Documents/kakitu/kakitu_wallet_flutter-master
flutter analyze lib/ui/card/card_details_sheet.dart
```

Expected: `No issues found!`

- [ ] **Step 3: Commit**

```bash
git add lib/ui/card/card_details_sheet.dart
git commit -m "feat: add CardDetailsSheet (60s auto-hide, copy buttons)"
```

---

## Task 13: Flutter — CardScreen

**Files:**
- Create: `kakitu_wallet_flutter-master/lib/ui/card/card_screen.dart`

- [ ] **Step 1: Write CardScreen**

```dart
// lib/ui/card/card_screen.dart
import 'package:flutter/material.dart';
import 'package:kakitu_wallet_flutter/appstate_container.dart';
import 'package:kakitu_wallet_flutter/styles.dart';
import 'package:kakitu_wallet_flutter/ui/card/card_service.dart';
import 'package:kakitu_wallet_flutter/ui/card/card_details_sheet.dart';
import 'package:kakitu_wallet_flutter/ui/widgets/sheet_util.dart';
import 'package:kakitu_wallet_flutter/model/vault.dart';
import 'package:kakitu_wallet_flutter/service_locator.dart';
import 'package:shared_preferences/shared_preferences.dart';

const _kCardIdKey = 'stripe_card_id';
const _kCardLast4Key = 'stripe_card_last4';
const _kCardExpMonthKey = 'stripe_card_exp_month';
const _kCardExpYearKey = 'stripe_card_exp_year';

class CardScreen extends StatefulWidget {
  const CardScreen({Key? key}) : super(key: key);

  @override
  State<CardScreen> createState() => _CardScreenState();
}

class _CardScreenState extends State<CardScreen> {
  IssuedCard? _card;
  bool _loading = true;
  bool _issuing = false;
  String? _error;

  // Onboarding form fields
  final _nameController = TextEditingController();
  final _phoneController = TextEditingController();

  @override
  void initState() {
    super.initState();
    _loadCachedCard();
  }

  @override
  void dispose() {
    _nameController.dispose();
    _phoneController.dispose();
    super.dispose();
  }

  Future<void> _loadCachedCard() async {
    final prefs = await SharedPreferences.getInstance();
    final cardId = prefs.getString(_kCardIdKey);
    if (cardId != null) {
      setState(() {
        _card = IssuedCard(
          cardId: cardId,
          last4: prefs.getString(_kCardLast4Key) ?? '',
          expMonth: prefs.getInt(_kCardExpMonthKey) ?? 0,
          expYear: prefs.getInt(_kCardExpYearKey) ?? 0,
          status: 'active',
        );
        _loading = false;
      });
    } else {
      setState(() => _loading = false);
    }
  }

  Future<void> _issueCard() async {
    if (_nameController.text.trim().isEmpty || _phoneController.text.trim().isEmpty) {
      setState(() => _error = 'Please enter your name and phone number');
      return;
    }
    setState(() { _issuing = true; _error = null; });
    final account = StateContainer.of(context).wallet.address;
    try {
      final issued = await CardService.issueCard(
        account: account,
        name: _nameController.text.trim(),
        phone: _phoneController.text.trim(),
      );
      final prefs = await SharedPreferences.getInstance();
      await prefs.setString(_kCardIdKey, issued.cardId);
      await prefs.setString(_kCardLast4Key, issued.last4);
      await prefs.setInt(_kCardExpMonthKey, issued.expMonth);
      await prefs.setInt(_kCardExpYearKey, issued.expYear);
      if (!mounted) return;
      setState(() { _card = issued; _issuing = false; });
    } catch (e) {
      if (!mounted) return;
      setState(() { _error = e.toString(); _issuing = false; });
    }
  }

  void _showDetails() {
    Sheets.showAppHeightNineSheet(
      context: context,
      widget: const CardDetailsSheet(),
    );
  }

  @override
  Widget build(BuildContext context) {
    return SafeArea(
      minimum: EdgeInsets.only(bottom: MediaQuery.of(context).viewInsets.bottom),
      child: _loading
          ? const Center(child: CircularProgressIndicator())
          : _card == null
              ? _buildEmptyState()
              : _buildActiveCard(),
    );
  }

  Widget _buildEmptyState() {
    return SingleChildScrollView(
      padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 20),
      child: Column(
        children: [
          // Handle
          Container(width: 40, height: 4, margin: const EdgeInsets.only(bottom: 20),
            decoration: BoxDecoration(color: Colors.white24, borderRadius: BorderRadius.circular(2))),
          const Icon(Icons.credit_card_outlined, size: 60, color: Color(0xFFFFD700)),
          const SizedBox(height: 12),
          Text('Get Your Kakitu Visa Card',
              style: AppStyles.textStyleSize20W700Primary(context),
              textAlign: TextAlign.center),
          const SizedBox(height: 8),
          Text('A free virtual Visa card to spend your KSHS online & in-store',
              style: AppStyles.textStyleSize14W100Primary(context),
              textAlign: TextAlign.center),
          const SizedBox(height: 20),
          // Features
          _FeatureRow(icon: Icons.check_circle_outline, text: 'Free virtual Visa card'),
          _FeatureRow(icon: Icons.check_circle_outline, text: 'Works online worldwide'),
          _FeatureRow(icon: Icons.check_circle_outline, text: 'Add to Google Pay'),
          _FeatureRow(icon: Icons.check_circle_outline, text: 'Instant — no wait'),
          const SizedBox(height: 20),
          // Form
          TextField(
            controller: _nameController,
            decoration: InputDecoration(
              labelText: 'Full Name',
              labelStyle: const TextStyle(color: Colors.grey),
              filled: true,
              fillColor: Colors.white.withOpacity(0.05),
              border: OutlineInputBorder(borderRadius: BorderRadius.circular(10),
                borderSide: BorderSide.none),
            ),
            style: const TextStyle(color: Colors.white),
          ),
          const SizedBox(height: 10),
          TextField(
            controller: _phoneController,
            keyboardType: TextInputType.phone,
            decoration: InputDecoration(
              labelText: 'Phone Number (e.g. +254712345678)',
              labelStyle: const TextStyle(color: Colors.grey),
              filled: true,
              fillColor: Colors.white.withOpacity(0.05),
              border: OutlineInputBorder(borderRadius: BorderRadius.circular(10),
                borderSide: BorderSide.none),
            ),
            style: const TextStyle(color: Colors.white),
          ),
          if (_error != null) ...[
            const SizedBox(height: 8),
            Text(_error!, style: const TextStyle(color: Colors.red, fontSize: 12)),
          ],
          const SizedBox(height: 16),
          SizedBox(
            width: double.infinity,
            child: ElevatedButton(
              onPressed: _issuing ? null : _issueCard,
              style: ElevatedButton.styleFrom(
                backgroundColor: const Color(0xFFFFD700),
                foregroundColor: Colors.black,
                padding: const EdgeInsets.symmetric(vertical: 14),
                shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
              ),
              child: _issuing
                  ? const SizedBox(height: 20, width: 20,
                      child: CircularProgressIndicator(color: Colors.black, strokeWidth: 2))
                  : const Text('Get My Free Card', style: TextStyle(fontWeight: FontWeight.w800, fontSize: 16)),
            ),
          ),
          const SizedBox(height: 8),
          Text('Free to issue. Spending funded from your KSHS balance.',
              style: AppStyles.textStyleSize12W400Primary(context),
              textAlign: TextAlign.center),
        ],
      ),
    );
  }

  Widget _buildActiveCard() {
    final expMonthStr = _card!.expMonth.toString().padLeft(2, '0');
    final name = _nameController.text.isNotEmpty
        ? _nameController.text.toUpperCase()
        : 'KAKITU USER';

    return SingleChildScrollView(
      padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 20),
      child: Column(
        children: [
          // Handle
          Container(width: 40, height: 4, margin: const EdgeInsets.only(bottom: 16),
            decoration: BoxDecoration(color: Colors.white24, borderRadius: BorderRadius.circular(2))),
          Row(
            children: [
              Text('My Kakitu Card',
                  style: AppStyles.textStyleSize20W700Primary(context)),
              const Spacer(),
              Container(
                padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 3),
                decoration: BoxDecoration(
                  color: Colors.green.withOpacity(0.2),
                  borderRadius: BorderRadius.circular(6),
                ),
                child: const Text('● ACTIVE',
                    style: TextStyle(color: Colors.green, fontSize: 11, fontWeight: FontWeight.w700)),
              ),
            ],
          ),
          const SizedBox(height: 16),
          // Card visual
          Container(
            width: double.infinity,
            padding: const EdgeInsets.all(20),
            decoration: BoxDecoration(
              gradient: const LinearGradient(
                colors: [Color(0xFFFFD700), Color(0xFFFFA500), Color(0xFFFF6B00)],
                begin: Alignment.topLeft,
                end: Alignment.bottomRight,
              ),
              borderRadius: BorderRadius.circular(16),
              boxShadow: [BoxShadow(color: Colors.orange.withOpacity(0.35), blurRadius: 20, offset: const Offset(0, 8))],
            ),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                const Text('KAKITU VIRTUAL CARD',
                    style: TextStyle(color: Colors.white70, fontSize: 10, letterSpacing: 2)),
                const SizedBox(height: 12),
                Text('•••• •••• •••• ${_card!.last4}',
                    style: const TextStyle(color: Colors.white, fontSize: 18,
                        fontWeight: FontWeight.w700, letterSpacing: 3)),
                const SizedBox(height: 12),
                Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    Column(crossAxisAlignment: CrossAxisAlignment.start, children: [
                      Text(name, style: const TextStyle(color: Colors.white,
                          fontSize: 13, fontWeight: FontWeight.w700)),
                      Text('EXP $expMonthStr/${_card!.expYear}',
                          style: const TextStyle(color: Colors.white70, fontSize: 11)),
                    ]),
                    const Text('VISA', style: TextStyle(color: Colors.white,
                        fontSize: 20, fontWeight: FontWeight.w900, fontStyle: FontStyle.italic)),
                  ],
                ),
              ],
            ),
          ),
          const SizedBox(height: 16),
          // Actions grid
          GridView.count(
            crossAxisCount: 4,
            shrinkWrap: true,
            physics: const NeverScrollableScrollPhysics(),
            mainAxisSpacing: 8,
            crossAxisSpacing: 8,
            children: [
              _ActionTile(icon: Icons.visibility_outlined, label: 'Details', onTap: _showDetails),
              _ActionTile(icon: Icons.ac_unit_outlined, label: 'Freeze', onTap: () {}),
              _ActionTile(icon: Icons.receipt_long_outlined, label: 'History', onTap: () {}),
              _ActionTile(icon: Icons.settings_outlined, label: 'Settings', onTap: () {}),
            ],
          ),
          const SizedBox(height: 12),
          // Google Pay button
          SizedBox(
            width: double.infinity,
            child: OutlinedButton.icon(
              onPressed: () {},
              icon: const Text('G',
                  style: TextStyle(fontWeight: FontWeight.w900, fontSize: 18,
                      foreground: Paint()..shader = const LinearGradient(
                        colors: [Color(0xFF4285F4), Color(0xFF34A853), Color(0xFFFBBC05), Color(0xFFEA4335)],
                      ).createShader(const Rect.fromLTWH(0, 0, 40, 40)))),
              label: const Text('Add to Google Pay',
                  style: TextStyle(color: Colors.white, fontWeight: FontWeight.w600)),
              style: OutlinedButton.styleFrom(
                side: const BorderSide(color: Colors.white24),
                padding: const EdgeInsets.symmetric(vertical: 12),
                shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(10)),
              ),
            ),
          ),
        ],
      ),
    );
  }
}

class _FeatureRow extends StatelessWidget {
  final IconData icon;
  final String text;
  const _FeatureRow({required this.icon, required this.text});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 3),
      child: Row(children: [
        Icon(icon, color: const Color(0xFFFFD700), size: 18),
        const SizedBox(width: 8),
        Text(text, style: AppStyles.textStyleSize14W100Primary(context)),
      ]),
    );
  }
}

class _ActionTile extends StatelessWidget {
  final IconData icon;
  final String label;
  final VoidCallback onTap;
  const _ActionTile({required this.icon, required this.label, required this.onTap});

  @override
  Widget build(BuildContext context) {
    return InkWell(
      onTap: onTap,
      borderRadius: BorderRadius.circular(10),
      child: Container(
        decoration: BoxDecoration(
          color: Colors.white.withOpacity(0.07),
          borderRadius: BorderRadius.circular(10),
        ),
        child: Column(mainAxisAlignment: MainAxisAlignment.center, children: [
          Icon(icon, color: Colors.white70, size: 22),
          const SizedBox(height: 4),
          Text(label, style: const TextStyle(color: Colors.white54, fontSize: 9)),
        ]),
      ),
    );
  }
}
```

- [ ] **Step 2: Verify Flutter analyzes cleanly**

```bash
cd /Users/kiptengwer/Documents/kakitu/kakitu_wallet_flutter-master
flutter analyze lib/ui/card/card_screen.dart
```

Expected: `No issues found!` — fix any style method mismatches found in step 2 of Task 11.

- [ ] **Step 3: Commit**

```bash
git add lib/ui/card/card_screen.dart
git commit -m "feat: add CardScreen (empty state + active Visa card with actions)"
```

---

## Task 14: Wire home page Sell button to SellMethodSheet

**Files:**
- Modify: `kakitu_wallet_flutter-master/lib/ui/home_page.dart`

- [ ] **Step 1: Add import for SellMethodSheet**

In `home_page.dart`, find the existing imports block. The file already imports `sell_sheet.dart` at line 30. Add the new import directly after it:

```dart
import 'package:kakitu_wallet_flutter/ui/sell/sell_method_sheet.dart';
```

- [ ] **Step 2: Change the Sell button's onTap**

In `home_page.dart` around line 1204, find:

```dart
_buildActionPill(
  context: context,
  icon: Icons.currency_exchange_rounded,
  label: 'Sell',
  color: const Color(0xFFF59E0B),
  onTap: () {
    Sheets.showAppHeightNineSheet(
        context: context, widget: SellSheet());
  },
),
```

Replace with:

```dart
_buildActionPill(
  context: context,
  icon: Icons.currency_exchange_rounded,
  label: 'Sell',
  color: const Color(0xFFF59E0B),
  onTap: () {
    Sheets.showAppHeightNineSheet(
        context: context, widget: const SellMethodSheet());
  },
),
```

- [ ] **Step 3: Verify Flutter analyzes cleanly**

```bash
cd /Users/kiptengwer/Documents/kakitu/kakitu_wallet_flutter-master
flutter analyze lib/ui/home_page.dart
```

Expected: `No issues found!`

- [ ] **Step 4: Full project analyze**

```bash
cd /Users/kiptengwer/Documents/kakitu/kakitu_wallet_flutter-master
flutter analyze
```

Expected: `No issues found!`

- [ ] **Step 5: Commit**

```bash
git add lib/ui/home_page.dart lib/ui/sell/sell_method_sheet.dart
git commit -m "feat: wire Sell button to SellMethodSheet with Card and M-Pesa paths"
```

---

## Self-Review Checklist

- [x] Address pool model + `AssignAddress()` with `SKIP LOCKED` — Task 2, 4
- [x] StripeCard model + repo — Task 2, 5
- [x] Migrations updated — Task 3
- [x] Stripe wrapper: create cardholder, issue card, activate, get details — Task 6
- [x] `POST /issuing/card` endpoint with all validation — Task 7
- [x] `GET /issuing/card/details` endpoint — Task 7
- [x] Routes registered in main.go — Task 8
- [x] Seeding CLI — Task 9
- [x] CardService (Flutter HTTP client) — Task 10
- [x] SellMethodSheet (choose Card or M-Pesa) — Task 11
- [x] CardDetailsSheet (60s auto-hide, copy buttons) — Task 12
- [x] CardScreen (empty + active states) — Task 13
- [x] Home page Sell button wired to SellMethodSheet — Task 14
- [x] `STRIPE_SECRET_KEY` env var documented in spec
