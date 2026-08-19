# Trekking Miles — Munnar Attractions Finder

A single-file, location-aware web app. Guests set their base (or use GPS) and see
nearby Munnar-area attractions sorted by proximity, with one-tap Google Maps directions.

## Deploy to Vercel (no build step)

**Recommended path (GitHub Desktop → Vercel, auto-redeploys on every change):**
1. In GitHub Desktop: File → New Repository (or add this folder as an existing repository). Add `index.html` (and this README).
2. Commit, then "Publish repository" to GitHub.
3. Go to vercel.com/new, sign in with GitHub, and import that repository. Framework preset: **Other**, no build command, output directory: root.
4. Deploy. Vercel gives a temporary URL like `your-project.vercel.app` — check everything works there first.
5. From now on: edit `index.html`, commit + push in GitHub Desktop, and Vercel redeploys automatically. No manual re-uploading.

**Quick path (no GitHub, for a one-off test):** drag the folder onto vercel.com/new (Framework preset: **Other**). Works, but every future edit means re-dragging the whole folder — fine for a first look, not for ongoing use.

Either way, Vercel serves it over HTTPS, which the GPS feature requires.

## Going live on your real domain

**Recommended: a subdomain of trekkingmiles.com** — e.g. `munnar.trekkingmiles.com` — not a separate domain. It inherits your existing domain's trust and SEO authority, guests recognize it as genuinely Trekking Miles (important since it drives WhatsApp bookings), and it costs nothing extra to register or renew. A brand-new domain would start at zero SEO authority and be one more asset to maintain.

Steps:
1. In the Vercel project → Settings → Domains → Add `munnar.trekkingmiles.com`.
2. Vercel shows a DNS record to add (typically a CNAME pointing to `cname.vercel-dns.com`).
3. Add that record wherever trekkingmiles.com's DNS is managed (your domain registrar or DNS provider — GoDaddy, Namecheap, Cloudflare, Google Domains, or your web host's DNS panel). This does **not** touch or affect the existing trekkingmiles.com site — it's a new, separate subdomain.
4. Wait for DNS to propagate (usually minutes, sometimes up to 24–48h). Vercel auto-issues a free HTTPS certificate once it resolves.

A subpath instead (`trekkingmiles.com/munnar`) can be marginally better for SEO, but only works cleanly if your main site's current platform/host supports routing part of the domain to a different app — depends what trekkingmiles.com runs on today. Worth checking before ruling it out.

## Editing content — two ways

**A. Edit in-file (default).** All data lives in the `DATA` object near the top of the <script> block:
- `attractions[]`: name, corridor, cats, blurb, lat, lng, added, verified, bookable, photo (optional image URL)
- `bases[]`: the "You are here" dropdown options
- `cats{}`: category → colour
- `guideInterests[]`: options shown in the local guide request form
- `guideRequestWebhookUrl`: optional — set to a Google Apps Script URL to also log guide requests to a Sheet (see `claude/guide-request-backend-plan.md`)
- `comingSoon[]`: the Resorts / Homestays / Local Experiences / Munnar Packages teaser cards
- `whatsapp`: E.164 number without '+' (used for all wa.me links)
- `sheetCsvUrl`: leave empty to use in-file data (below)

**B. Live from a Google Sheet (no redeploy to change content).**
1. Open `attractions.csv` in Google Sheets. Columns: `name, corridor, categories, blurb, lat, lng, bookable, added, verified, photo` (`categories` separated by `|`; `photo` is an optional image URL — see note on image hosting below).
2. File → Share → **Publish to web** → choose the sheet → **CSV** → copy the URL.
3. Paste that URL into `sheetCsvUrl` in `index.html` and redeploy once.
Thereafter, edits in the Sheet appear on next page load. If the fetch fails, the app falls back to the in-file data.

**Adding photos:** the `photo` column takes a direct image URL — the Sheet doesn't host the file itself. Use an image host (Cloudinary or similar recommended; Google Drive share links work but need reformatting and can load slowly under traffic). Don't embed photos as base64 like the logo — fine for one small logo, not for 49 photos in a single HTML file. A missing or broken photo URL fails gracefully — the card just falls back to its normal no-image layout.

## Lead capture & bookings
- Spots with `bookable:true` show a green **Book / Enquire** button (prefilled WhatsApp).
- A floating **Plan my trip** button (WhatsApp) is always visible.
- Guests can **＋ Day** spots and **Send to WhatsApp** — that message is a qualified lead.
- **Need a Munnar Local Guide?** section (live today) collects date/group size/interest/language and sends a structured WhatsApp request — fulfilled manually until a guide directory exists. See `claude/guide-request-backend-plan.md` for the backend logging plan and the future guide-directory schema.
- All chats route to the `whatsapp` number. Add UTM tags to `trekkingmiles.com` links if you want web attribution.

## ⚠️ Before going public
- Coordinates are **approximate seeds** (`verified:false`). Verify each lat/lng
  (right-click a spot in Google Maps → copy coordinates) so proximity sorting is correct.
- Distances shown are **straight-line**; the Directions button gives the real route.
- OpenStreetMap tiles are fine at low traffic; switch to a keyed provider (Mapbox/MapTiler) if usage grows.
