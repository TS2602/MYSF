# MYSF Fitness App

A personal strength training and nutrition tracking PWA, built as a single self-contained `index.html` (no build step, no framework, no backend).

## Stack
- Vanilla HTML/CSS/JS — no bundler, no dependencies to install
- Data persistence: browser `localStorage` (per-device, no account/login)
- Food data: local curated database + USDA FoodData Central API + Open Food Facts API
- Barcode scanning: ZXing (loaded lazily from CDN, only when the scanner is opened)

## Local development
There's no build step. Just open `index.html` directly in a browser, or serve the folder with any static file server:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Deployment
This repo is connected to Netlify for auto-deploy on every push to `main`. No manual build command is needed — Netlify just serves `index.html` as a static site.

## Important architectural notes
- **Storage is localStorage, not any cloud/account-based sync.** Each browser/device has its own independent data. There is no server-side database.
- **The USDA API key is intentionally embedded in client-side code.** It's a personal, free, non-sensitive API key (rate-limits public nutrition data, nothing else) — not a security concern at this scale. If this app ever gets real public traffic, revisit this via a serverless proxy (see "Future: scaling the food database" below).
- Two external, keyless food data sources are used in parallel: USDA (better for whole/raw foods) and Open Food Facts (better for branded/packaged products, including barcode lookups). Attribution to both is shown in the UI per Open Food Facts' license terms.

## Future: scaling the food database
If this app ever gets real public adoption beyond a small circle, the embedded API key becomes a shared bottleneck across all users. The fix is a Netlify Function that holds the key server-side and caches repeated queries — not yet built, since it's not needed at current scale.

## Backup & data safety
Since all data lives in localStorage only, use the in-app "Export Full Backup" (Program or Plan tab) regularly. There is no server-side backup.
