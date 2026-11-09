# Release infrastructure — how releases are cut today

Rewritten 2026-11-09. The 2026-10-12 version described blockers that have since been cleared
(Play API key, iOS signing, a Mac). This is what works now, end to end. Both stores are fed
from the app repo's `eth` branch (`kakitucurrency/Kakitu_wallet_flutter`).

---

## 1. Android — Google Play (automated)

**Where it runs.** On the Linux build PC, from the main app checkout (the only checkout with
the real upload key, `android/key.properties`; worktrees under `~/build/wt/` carry a throwaway
key for test builds and must never be used for a store build).

```
cd Kakitu_wallet_flutter/android
export JAVA_HOME=$HOME/.local/jdk TMPDIR=$HOME/build/tmp \
       SUPPLY_JSON_KEY=$HOME/.playconsole/kakitu-play-api.json \
       KAKITU_ENV=production \
       LOAN_POT_ADDRESS=0xb38dbA7abD392fF1488e325170DE4b9F4EC05a1F
fastlane play_release track:production release_status:completed
```

- The lane builds through Flutter (`flutter build appbundle --flavor play`), not gradle, and
  refuses to start without a well-formed `LOAN_POT_ADDRESS` (without it the app refuses loan,
  Boost and repay signing).
- `release_status:completed` is what makes it public. The lane's default is `draft`.
- Policy: production, 100% rollout, never staged.
- **Verify against Google, not the log.** A short Ruby script (googleauth, scope
  `androidpublisher`) opens an edit and reads `tracks/production`; the expected answer is
  `status=completed versions=["<code>"] fraction=full`. Service account:
  `kakitu-play-ci@kakitu-play.iam.gserviceaccount.com`, package `com.kakitu.app`.
- Do not pipe the lane through `tail`: fastlane prints its changelog last and the build log is
  lost; exit code 0 alone does not prove an upload.
- Build on `$HOME/build` with `TMPDIR` there: the per-user `/tmp` quota fills and Flutter then
  "hangs" (errno 122).

## 2. iOS — App Store (GitHub Actions, no Mac)

**Where it runs.** `.github/workflows/ios-testflight.yml` on a GitHub macOS runner. The file
must exist on `main` for `workflow_dispatch` to be offered; the build uses `eth`:

```
gh workflow run ios-testflight.yml --ref eth -f flutter_env=production
gh run watch <run-id> --exit-status
```

- The workflow builds Flutter with `KAKITU_ENV` and the pinned `LOAN_POT_ADDRESS`, then
  fastlane signs (distribution cert `6WCC8QB6WA`, expires 2027-10-30, manual signing), archives
  and uploads. With no `build_number` input the lane takes the pubspec build number.
- Repository secrets it needs: `ASC_KEY_ID`, `ASC_ISSUER_ID`, `ASC_KEY_P8_BASE64`,
  `FASTLANE_TEAM_ID` (`JK46FCYNG4`, KAKITU SECURITIES LIMITED), `IOS_DIST_CERT_P12_BASE64`,
  `IOS_DIST_CERT_PASSWORD`. All set since 2026-10-30.
- `Info.plist` declares `ITSAppUsesNonExemptEncryption=false`, so builds no longer hold at
  "Missing Compliance".
- **Uploading is not submitting.** After the build shows `VALID` in App Store Connect, submit
  through the App Store Connect API (key `HV84YXW8JC`, `.p8` under
  `~/.appstoreconnect/private_keys/`, 600): cancel any open review submission and wait for
  `DEVELOPER_REJECTED`, PATCH the version string and attach the build, then create a review
  submission, add the version as an item, and PATCH `submitted: true`. Use
  `releaseType: AFTER_APPROVAL` so approval publishes it with no further step. Apple's cancel
  can take several minutes.
- Resolution Center (rejection reasons) is not reachable with the API key; it needs a web
  login on the Account Holder's Apple ID.
- App Store guideline 3.2.2(ix): personal-loan apps may not charge more than 36% APR including
  fees and may not require full repayment within 60 days. Production loan pricing is set in the
  admin console; check it against this before an iOS submission that shows Loans.

## 3. Secrets to back up

| what | where | why it matters |
|---|---|---|
| App Store Connect API key `.p8` | `~/.appstoreconnect/private_keys/` | Apple never re-issues it |
| iOS distribution key + p12 | `~/.appstoreconnect/dist/` (600) | signs every iOS build |
| Play service-account JSON | `~/.playconsole/kakitu-play-api.json` | every Play upload |
| Android upload keystore | referenced by `android/key.properties` in the main checkout | Play rejects a different key |
| Loan pot / data keys, phone-code key | `~/kakitu-secrets/` (700) | set on Railway; losing them stops loans / phone proof |

## 4. Machine

The build PC is Linux with 14 GiB RAM. Run heavy jobs one at a time (Flutter release build,
the full Flutter suite, the server's controller and repository suites, the Android emulator):
three at once has already had the desktop killed by the out-of-memory daemon. The server's
`repository` and `controller` test packages need `-timeout 50m`; Go's default 10 minutes cuts
`repository` off. Keep the charger in: the PC has died on battery mid-build, and a power loss
corrupted the local test Postgres once.

## 5. Debts carried from 2026-10-12 (not re-verified since unless marked)

- **Rotate the Fonbnk production credentials** (pasted in cleartext in an earlier session).
- **Revoke the Alchemy and Stripe keys** in the Flutter git history; rewriting history is not
  needed, revoking is.
- **Firebase is split**: Android and the server on `kakitu-wallet`, iOS on
  `kakitu-wallet-fc743`. The server now sends FCM pushes (Android); iOS pushes still need an
  APNs key and the consolidation.
- **`main` / `master` lag `eth`.** Everything real is on `eth`; `main` exists mainly so the iOS
  workflow can be dispatched.
- **Committed `ios/8H48RKDJ83.p12`** belongs to an old team; revoke it.
- **`RESERVE_CHECK_ENABLED` is false** on Railway (last read 2026-10-29), so nothing enforces
  KSHS supply ≤ backing at mint time; supply is still read off the chain for reporting.
- **`kakitu-pulse`** was merged but not running as of 2026-10-12.
