# FleetBuddy

**Fuel & toll slip reconciliation for WildTrust's WildOceans vehicle fleet.**

Pick a vehicle, upload its monthly Daily Movement Report and Vehicle Analysis Report, photograph each fuel or toll slip, let AI pull the amount/station/date/odometer, type in the driver and project by hand, search the real vote number, and sync straight to Google Sheets and Drive. No app store, no install, no server.

*Built in-house by WILDOCEANS IT.*

---

## How it's built

Same architecture as ReconEasy — a single static HTML file, no build step, no framework, no backend server, paired with the same Cloudflare Worker that already proxies ReconEasy's AI calls.

```
┌──────────────────────┐        ┌────────────────────────┐        ┌────────────────────┐
│   index.html          │──────▶│  reconeasy-worker        │──────▶│   Anthropic API     │
│  (GitHub Pages)        │        │  (Cloudflare, existing)  │        │  (reads the slip)   │
└──────────────────────┘        └────────────────────────┘        └────────────────────┘
          │
          ▼
┌─────────────────────────────────────┐
│  Signed-in user's own Google account │
│  → Drive (reports/slips) + Sheets    │
└─────────────────────────────────────┘
```

**The Cloudflare Worker is shared with ReconEasy — no new Worker was deployed.** Its CORS check only cares about the origin (`https://nxrth98.github.io`), not the path, so FleetBuddy living at `https://nxrth98.github.io/fleetbuddy` is already allowed. Nothing to configure there.

| Piece | Where it lives | Purpose |
|---|---|---|
| `index.html` | This repo, served via GitHub Pages | The entire app |
| Worker | `reconeasy-worker` repo (shared, unchanged) | Proxies the slip-reading AI call |
| Vote database | Same shared Google Sheet ReconEasy uses | Live source of vote/budget codes |
| Recon entries & receipts | Each signed-in user's own Google Drive, under a `FleetBuddy` folder, subfoldered per vehicle | Created automatically on first sync |
| `/images` folder (you create this) | This repo | Fleet gallery photos — see naming convention below |

---

## Deploying this repo

1. **Push this folder to GitHub** as a new repo named `fleetbuddy` under your account, then enable **GitHub Pages** (Settings → Pages → Deploy from branch → `main` / root). It'll be live at `https://nxrth98.github.io/fleetbuddy`.

2. **Add the redirect URI to your existing Google Cloud OAuth Client** (the same one ReconEasy uses — no new client needed):
   - Go to Google Cloud Console → APIs & Services → Credentials → your OAuth 2.0 Client ID
   - Under **Authorized JavaScript origins**, confirm `https://nxrth98.github.io` is already there (it will be, from ReconEasy)
   - Under **Authorized redirect URIs**, add `https://nxrth98.github.io/fleetbuddy`
   - Save

3. **Add yourself (and anyone else who'll use it) as a test user** on that same Google Cloud project if it's still in OAuth Testing mode — same list ReconEasy already uses.

4. **Nothing else to configure.** The Worker, the vote Sheet, and the Anthropic API key are all reused as-is from ReconEasy.

That's it — open the link, sign in, you're in.

---

## Fleet gallery photos

The gallery screen looks for images at `/images/{REG}-1.jpg` and `/images/{REG}-2.jpg` (reg number, spaces stripped, uppercase). Create an `images` folder in this repo and drop photos in with the matching filename — no code changes needed, missing files just show a placeholder icon. Full filename list is in an HTML comment directly above the gallery code in `index.html`.

---

## Data model, in short

- Entries, uploaded report files (in-memory for the session), and captured slip photos are stored **locally on the device** until you tap **Sync**
- **Sync** groups entries by month and writes each month to its own tab in that vehicle's Google Sheet, sorted by date — safe even if reports arrive out of order through the month
- **Upload** pushes any not-yet-uploaded slip photos to Drive
- Vote number is always a manual search-and-pick — never auto-filled — same for driver and project, which are always typed by hand, on purpose
- There is currently **no shared team view** — each person's synced data lives in their own Drive

---

## Configuration reference

| Setting | Value |
|---|---|
| Google OAuth Client ID | `419380799116-k9dnh5p4i66r1kulqdjirj6uh3ojenrv.apps.googleusercontent.com` (shared with ReconEasy) |
| OAuth redirect URI | `https://nxrth98.github.io/fleetbuddy` |
| Cloudflare Worker URL | `https://reconeasy-worker.nxrth98.workers.dev` (shared, unchanged) |
| Vote database Sheet ID | `1GrLCv_s_WTvjksuZI50NANHtXZnnwcdxVbYMWL-qurc` (shared with ReconEasy) |

The Anthropic API key is not in this repo — it lives only in the Cloudflare Worker's encrypted environment variables, same as ReconEasy.
