# 🏎️ Convoy

A lean, racing-styled live map for driving together with friends. Everyone in a convoy sees each other as an **emoji avatar** moving on a dark neon map, with live **speed**. Works in any phone browser (iOS Safari & Android Chrome) — no app install.

Two files:

- `index.html` — the whole app (map + UI + multiplayer). One file, no build step.
- `database.rules.json` — secure Firebase rules (locks data down so people can only write their own position, and only into a convoy).

You can try it **right now in demo mode** (simulated convoy mates) by opening `index.html` over HTTPS. To make it real multiplayer, do the 4-minute setup below.

---

## Quick start (demo)

Location sharing requires HTTPS, so you can't just double-click the file. Easiest free option:

1. Drag this folder onto **https://app.netlify.com/drop** (or push to GitHub Pages / Vercel).
2. Open the URL on your phone → allow location → you'll see two demo cars near you.

---

## Real multiplayer setup (Firebase — free tier)

**1. Create the project**
- Go to https://console.firebase.google.com → *Add project* (disable Analytics, it's not needed).

**2. Turn on the two services**
- *Build → Authentication → Get started → Sign-in method → enable **Anonymous**.*
- *Build → Realtime Database → Create database → start in **locked mode**.*

**3. Paste the security rules**
- In Realtime Database → **Rules** tab, paste the contents of `database.rules.json` → **Publish**.

**4. Add your config to the app**
- *Project settings (gear) → Your apps → Web app (`</>`)* → copy the `firebaseConfig` values.
- Open `index.html`, find the `firebaseConfig` block near the top of the `<script>`, and paste your `apiKey`, `authDomain`, `databaseURL`, `projectId`, `appId`. The `databaseURL` is the important one — once it's filled, demo mode turns off automatically.

**5. Host it over HTTPS** (Netlify Drop, GitHub Pages, Vercel, Firebase Hosting — any works). Done.

---

## How friends join

One person taps **Create** → gets a 5-letter code (e.g. `K7P2M`). Everyone else types that code and taps **Join**. Same code = same convoy. Pick an emoji + name first.

> Tip: add it to your home screen (Safari *Share → Add to Home Screen*, Chrome *⋮ → Add to Home screen*) so it opens fullscreen like a real app.

---

## Safety & privacy (what makes the backend "safe")

- **Anonymous auth** — no email, no password, no personal account.
- **You can only write your own dot.** The rules enforce `auth.uid === $uid`, so nobody can spoof or move another driver.
- **Strict validation** — lat/lng/speed must be in valid ranges; name ≤ 12 chars, emoji ≤ 8 chars; no extra fields can be injected.
- **Scoped to the convoy code** — your location lives only under `rooms/<code>/` and is only readable by signed-in users who have the code.
- **Auto-cleanup** — `onDisconnect` removes your position the moment you close the tab or lose signal; **Leave** deletes it immediately.
- **Open while driving only** — nothing is tracked in the background; location streams only while the page is open.

### Recommended extra hardening
- Room codes are short for convenience. For sensitive use, lengthen `randCode()` in `index.html`.
- Firebase free (Spark) plan is enough for friends. Set a billing budget alert if you upgrade.
- Old rooms aren't auto-deleted from the DB (only player nodes are). To auto-expire, add a scheduled Cloud Function later, or just delete `rooms/` occasionally in the console.

---

## Tech (kept intentionally lean)
- **Leaflet + OpenStreetMap** for the map (free, no API key), dark-filtered for the racing look.
- **Firebase Realtime Database** for live positions (low-latency, simple, secure rules).
- Plain HTML/CSS/JS — no framework, no bundler, ~1 file.
