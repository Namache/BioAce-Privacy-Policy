# Google Play — Data Safety form answers

Completed answer sheet for the Play Console **App content → Data safety** section, derived
line-by-line from [privacy_policy.md](privacy_policy.md) and from what the code actually does.
**Re-check this file whenever the policy or an SDK changes** — Play requires the form, the policy,
and app behaviour to agree, and over-declaring is as much a mismatch as under-declaring.

Verified against the MVP build on 2026-07-30: no ad SDK, no analytics SDK, no billing, no social
pings. The only third-party SDKs that touch user data are Google Sign-In (Credential Manager),
Supabase, and Firebase Crashlytics; Resend is server-side, called by the `send-feedback-email`
edge function.

---

## Definitions used

- **Collected** — transmitted off the device.
- **Shared** — transferred to a *third party*. Transfers to a **service provider** that only
  processes data on our behalf (Supabase, Crashlytics, Resend) do **not** count as sharing under
  Play's definition, so every answer below is **Shared: No**.
- Settings and caches that never leave the device (theme, font size, sound, reminder preference,
  Discover cache, seen-request IDs) are **not collected** and are therefore not declared.

---

## Section 1 — Data collection and security

| Question | Answer |
|---|---|
| Does your app collect or share any of the required user data types? | **Yes** |
| Is all of the user data collected by your app encrypted in transit? | **Yes** — `network_security_config.xml` sets `cleartextTrafficPermitted="false"` app-wide |
| Do you provide a way for users to request that their data is deleted? | **Yes** — in-app (Settings → Privacy → Delete account, `delete_account` RPC) and by email |

---

## Section 2 — Data types

### Personal info

| Data type | Collected | Shared | Purposes | Optional? |
|---|---|---|---|---|
| **Name** | Yes | No | App functionality, Account management | Required |
| **Email address** | Yes | No | App functionality, Account management | Required |
| **User IDs** | Yes | No | App functionality, Account management | Required |

- *Name* — from Google Sign-In, plus the username you choose (shown to friends and on the leaderboard).
- *Email address* — from Google Sign-In; also the email you type into the feedback form.
- *User IDs* — Google account ID and the Supabase user ID.
- Do **not** declare: Address, Phone number, Race and ethnicity, Political or religious beliefs,
  Sexual orientation, Other info. None are collected.

### App activity

| Data type | Collected | Shared | Purposes | Optional? |
|---|---|---|---|---|
| **App interactions** | Yes | No | App functionality | Required |
| **Other user-generated content** | Yes | No | App functionality, Developer communications | Optional |

- *App interactions* — lesson progress, quiz/practice results, XP, streaks, ranks, achievements,
  weekly quests, chest claims, unlocked avatars, friend requests/connections. This is core
  functionality data, **not** analytics; there is no analytics SDK in the app.
- *Other user-generated content* — the free-text message in the feedback form (an open-ended
  response). Optional because the feedback form is never required to use the app.
- Do **not** declare: In-app search history, Installed apps, Other actions.

### App info and performance

| Data type | Collected | Shared | Purposes | Optional? |
|---|---|---|---|---|
| **Crash logs** | Yes | No | App functionality | Required |
| **Diagnostics** | Yes | No | App functionality | Required |

- Both via Firebase Crashlytics: stack traces, device model and manufacturer, OS version, app version.
- Do **not** declare: Other app performance data.

### Device or other IDs

| Data type | Collected | Shared | Purposes | Optional? |
|---|---|---|---|---|
| **Device or other IDs** | Yes | No | App functionality | Required |

- The **Crashlytics installation UUID**, used only to group one device's crash reports together.
- This is **not** an advertising ID. The app has no ad SDK and never reads the advertising identifier.

---

## Section 3 — Data types to leave UNCHECKED

Every category below must be left **not collected**. Each was checked against the current build.

| Category | Why not |
|---|---|
| **Location** (approximate, precise) | No location permission, no location API use |
| **Financial info** | No payments, no billing library, no paid tier |
| **Health and fitness** | Not collected |
| **Messages** (emails, SMS, other in-app messages) | No messaging feature — social pings were removed in the MVP; feedback is declared as user-generated content instead |
| **Photos and videos** | Avatars are in-app artwork; no photo picker, no camera |
| **Audio files** (voice, sound recordings, music) | No microphone permission; text-to-speech is output-only and on-device |
| **Files and docs** | Not collected |
| **Calendar** | Not collected |
| **Contacts** | Friends are found by in-app username search; no contacts permission |
| **Web browsing history** | The lab WebView loads only bundled assets; no browsing history is read or sent |
| **Purchase history** | No purchases |
| **Advertising ID** (under Device or other IDs) | No ad SDK; identifier never accessed |

---

## Section 4 — Other Play Console declarations

| Question | Answer |
|---|---|
| Target age group | **13+** — app shows a 13+ age gate before first use (`WelcomeScreen.kt`, `AgeGateDialog`) |
| Does your app target children? | **No** — therefore **not** subject to the Play Families policy |
| Has your app undergone an independent security review? | **No** |
| Privacy policy URL | `https://namache.github.io/BioAce-Privacy-Policy/` — live, and verified on 2026-07-30 to serve the current policy |
| Account deletion URL | Drafted, not yet hosted — `delete_account.md` in this repo; needs publishing to the Pages repo (see below) |

---

## Before you submit

### Done

- **Policy published and matching.** `https://namache.github.io/BioAce-Privacy-Policy/` returns 200
  and its text is content-identical to `privacy_policy.md` (checked heading-by-heading on
  2026-07-30; the only diffs are GitHub Pages cosmetics — the repo-name title it prepends and a
  smart-quoted apostrophe). `PRIVACY_POLICY_URL` in `SettingsScreen.kt` points at it, so the in-app
  link, the Play listing, and the build now agree.

### Still outstanding

1. **Publish the account-deletion page.** [`delete_account.md`](delete_account.md) in this repo is
   drafted and matches what `SupabaseAuthRepository.deleteAccount()` actually does (deletes the
   profile — cascading to all user data — and the auth user; Crashlytics logs are excluded because
   they're keyed by a per-device install ID, not the account). Push it to the same Pages repo as the
   privacy policy (e.g. as `delete-account.md` / `delete-account/index.html`) so it gets a public
   URL, then enter that URL in Play Console. Until it's published,
   `https://namache.github.io/BioAce-Privacy-Policy/#10-your-privacy-rights` is a fallback that at
   least describes both the in-app and email deletion paths.
2. **Re-verify if Crashlytics is ever removed or an SDK added.** The Crashlytics rows above are the
   *only* reason "Device or other IDs" and the crash/diagnostics rows are declared at all — and the
   *only* reason `delete_account.md`'s "what isn't deleted" section exists.

### Whenever the policy changes

Re-run the check that the hosted page still matches `privacy_policy.md`. The hosted copy lives in a
**separate repository**, so nothing in this repo's CI will catch it drifting out of sync.
