# Release infrastructure — what is missing and how to finish it

Written 2026-10-12. Three of these block automation; the rest are debts that will
bite at a bad moment.

---

## 1. Automatic Play Store uploads  ← the one you asked about

**Where it stands.** `android/fastlane` already has the `play_release` lane. It builds
the bundle through the Flutter tool (not gradle directly, which reads a stale version)
and uploads to the `internal` track as a **draft**, so a human still decides when it
reaches users. It needs exactly one thing: a service-account JSON key at the path
`SUPPLY_JSON_KEY` points to. That key has never been created.

**Why it has never been created.** Creating one is refused with:

> An Organization Policy that blocks service account key creation has been enforced
> on your organization.

That is `constraints/iam.disableServiceAccountKeyCreation`. You disabled it and the
error persisted, which almost always means one of three things:

1. The constraint is set at the **organization** node and you changed it on the
   **project**. A project does not inherit a change made beside it — it has to
   explicitly override the parent.
2. Your account does not hold **Organization Policy Administrator**
   (`roles/orgpolicy.policyAdmin`). Project Owner is not enough, and the console
   will often show the toggle without being able to save it.
3. You changed it and did not wait. Propagation is usually under a minute but is
   not instant.

### Path A — fix the policy (do this if you own the organization)

`gcloud` is now installed. It is not authenticated; that step is interactive and has
to be you:

```
gcloud auth login
gcloud organizations list                       # note the ORG_ID
gcloud projects list                            # note the PROJECT_ID you want
```

See what is actually in force, at both levels:

```
gcloud resource-manager org-policies describe \
  constraints/iam.disableServiceAccountKeyCreation --organization=ORG_ID

gcloud resource-manager org-policies describe \
  constraints/iam.disableServiceAccountKeyCreation --project=PROJECT_ID --effective
```

The `--effective` read on the project is the one that matters: it is what the API
will enforce. If it still says enforced, either lift it at the org:

```
gcloud resource-manager org-policies delete \
  constraints/iam.disableServiceAccountKeyCreation --organization=ORG_ID
```

or override it for just this project, which is the safer choice because it leaves
the guard-rail standing everywhere else:

```
cat > /tmp/allow-keys.yaml <<'EOF'
constraint: constraints/iam.disableServiceAccountKeyCreation
booleanPolicy:
  enforced: false
EOF
gcloud resource-manager org-policies set-policy /tmp/allow-keys.yaml --project=PROJECT_ID
```

Re-run the `--effective` read until it reports not enforced.

### Path B — sidestep it entirely (do this if you do not own the org, or Path A stalls)

The service account does **not** have to live in the organization's project. Play
Console grants access to a service-account *email address*; it does not care which
Google Cloud project issued it. So create it in a project owned by a personal Google
account, where no org policy applies:

1. Sign in to <https://console.cloud.google.com> as a **personal** Google account
   (not the org one). Create a new project, e.g. `kakitu-play-release`.
2. Enable the Google Play Android Developer API for it:
   APIs & Services → Library → "Google Play Android Developer API" → Enable.
3. IAM & Admin → Service Accounts → Create service account.
   Name it `play-publisher`. No project roles are needed — its power comes from
   Play Console, not from Google Cloud.
4. Open it → Keys → Add key → Create new key → **JSON**. It downloads.
5. In **Play Console** → Users and permissions → Invite new user → paste the
   service account's email (`play-publisher@kakitu-play-release.iam.gserviceaccount.com`).
   Give it, for the Kakitu app only: **Release to testing tracks** and
   **Release apps to production** if you want it to be able to. Nothing else.

### Then, either path

```
mkdir -p ~/.playconsole
mv ~/Downloads/<that-file>.json ~/.playconsole/kakitu-play-api.json
chmod 600 ~/.playconsole/kakitu-play-api.json
```

Do not put it in the repo and do not paste its contents anywhere — it is a
credential with release rights over your listing. `android/fastlane/Appfile`
already expects it by environment:

```
export SUPPLY_JSON_KEY=~/.playconsole/kakitu-play-api.json
cd android && bundle exec fastlane play_release
```

Verify before trusting it:

```
cd android && bundle exec fastlane run validate_play_store_json_key \
  json_key:$HOME/.playconsole/kakitu-play-api.json
```

A first upload through the API sometimes fails with "the caller does not have
permission" for up to 24 hours after the invite — that is Play propagating, not a
wrong key.

---

## 2. `ios/fastlane/.env` does not survive

It is gitignored, correctly — but that means it exists only on this machine and
vanishes on any fresh clone, and it has already gone missing once. It holds no
secret, only identifiers; the secret is the `.p8`.

Keep a copy outside the repo, e.g. `~/.appstoreconnect/kakitu-ios.env`, and either
symlink it in or `source` it before releasing. Its contents:

```
ASC_KEY_ID=W3DD5X485T
ASC_ISSUER_ID=aead8807-efc9-416d-84a6-4393aa91913b
ASC_KEY_PATH=$HOME/.appstoreconnect/private_keys/AuthKey_W3DD5X485T.p8
FASTLANE_TEAM_ID=JK46FCYNG4
```

The `.p8` at `~/.appstoreconnect/private_keys/` is the thing to back up properly.
Apple will not re-issue it; losing it means revoking and re-creating the key.

---

## 3. Disk

A release needs roughly 8–10 GiB of headroom and the machine has been running at
under 1 GiB twice, killing both an Xcode archive ("database or disk is full") and a
Gradle R8 pass ("No space left on device"). The second failure also left a stale
Gradle lock that broke the *next* build for a different reason.

Caches cleared so far, all of which refill: DerivedData, Go build cache, Homebrew,
pip, Gradle `jars-9`.

The standing 15 GiB is three iOS simulators:

| simulator | size | state |
|---|---|---|
| iPhone 17 | 6.6G | shutdown |
| iPhone 17 Pro | 4.7G | shutdown |
| iPhone 17 Pro Max | 3.7G | booted |

Erasing one wipes the Kakitu wallets on it, seed phrases included — which is why
none were touched. Decide which hold wallets worth keeping, then:

```
xcrun simctl erase <UDID>
```

Before any release, check: `df -h /`.

---

## 4. Debts that are not blocking today

- **Rotate the Fonbnk production credentials.** They were pasted in cleartext in an
  earlier session, so treat them as exposed. Do it before real user money moves.
- **Revoke the Alchemy key** that is in the Flutter git history, and the Stripe key.
  History rewriting is not needed — revoking is.
- **Firebase is split.** Android and the server are on project `kakitu-wallet`; iOS
  is on `kakitu-wallet-fc743`. Push notifications will not work across both until
  they are consolidated. Nothing sends pushes yet, so this is cheap to fix now and
  expensive later.
- **`main` / `master` are ~137 commits behind `eth`.** Everything real is on `eth`.
  Either merge or retire those branches, so nobody clones the default branch and
  gets a year-old app.
- **`kakitu-pulse` is merged but not running.** It needs KSHS in a wallet and
  somewhere to run.
- **Reserves are about 15 KES short** — 133 KSHS issued against roughly 118 KES of
  backing. `RESERVE_CHECK_ENABLED=false`, so nothing is enforcing the peg.
