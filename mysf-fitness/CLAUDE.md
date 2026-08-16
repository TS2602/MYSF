# MYSF Fitness App — Project Context

Personal strength training + nutrition tracking PWA. Single `index.html` file, no build step, no framework, no backend. Built for the owner's personal use and a small circle (training partner), self-hosted on Netlify, deployed from this repo.

## Who this is for
Owner is cutting from 280+lb toward a 240lb goal, training a 4-day Upper/Lower split (twice-weekly muscle group frequency). Nutrition targets are set accordingly (~2,400-2,500 cal, high protein) but are now user-editable per device, not hardcoded to one person's numbers — anyone using their own copy of the app sets their own targets in Plan.

## Architecture — read this before making storage changes
- **All persistence is `localStorage`, not any cloud/account system.** Each browser/device is fully independent — there is no sync, no login, no server-side database.
- **This was a deliberate, hard-won decision.** Earlier iterations used Claude.ai's `window.storage` API, which only works inside Claude's own artifact sandbox — it silently fails (or doesn't exist at all) once self-hosted outside Claude, which caused real data loss during development. Never reintroduce `window.storage`.
- **localStorage keys in use:** `mysf-workout-log`, `mysf-nutrition-log`, `mysf-session-log`, `mysf-bodyweight-log`, `mysf-program`, `mysf-nutrition-target`, `mysf-usda-api-key`, `mysf-onboarded`, `mysf-last-export`. All are per-device only.
- **"Full Backup" (Program and Plan tabs) is the only cross-device mechanism** — exports everything as one JSON file, importable via "Restore from Backup File." Any new persisted data type must be added to both `exportFullBackup()` and `performRestore()`, or it silently won't survive a device migration.

## Food data — two APIs, both keyless for the end user
- **USDA FoodData Central**: best for whole/raw foods. Uses a personal API key (not the public DEMO_KEY, which has a near-unusable shared rate limit) hardcoded as the default in `searchUSDA()`. This key is intentionally client-side visible — it's not a sensitive credential, just a public-data rate limiter. If this app ever gets real public traffic beyond a small circle, that shared ceiling becomes a bottleneck; the fix is a Netlify Function proxy with caching (not yet built).
- **Open Food Facts**: best for branded/packaged products (protein shakes, bars, anything with a barcode). No key, no meaningful rate limit, but their license requires attribution — shown in the meal builder and scanner UI. Don't remove that credit.
- **Serving-size handling is important and non-obvious**: both APIs report nutrition per 100g by default. USDA Branded Foods sometimes include `labelNutrients` + `servingSize` (the actual printed label data); Open Food Facts sometimes includes `serving_size` + `*_serving` nutrient fields. When present, use those directly (qty defaults to 1, unit = the real serving label). When absent, fall back to a per-gram basis (qty defaults to 100, unit = "g"). Getting this wrong previously caused a real 100x calorie miscalculation bug — don't regress it.
- Barcode scanning uses ZXing, lazy-loaded from CDN only when the scanner is opened (not on page load).

## Design system
- Dark theme only. Colors: `--bg`, `--surface`, `--accent` (steel blue, Train/interactive), `--brass` (gold, Fuel/achievement), `--macro-carb` (sage green), `--danger` (only for borders/backgrounds — use `--danger-text`, a lightened variant, for any actual red *text*, since the base danger red fails WCAG AA contrast at 3.4:1 against card backgrounds).
- Fonts: Oswald (headers), Inter (body), JetBrains Mono (numbers/data). Loaded via `<link rel="preconnect">` + stylesheet in `<head>`, not a render-blocking `@import`.
- Icons are hand-written inline SVGs (simple line-icon style, `stroke="currentColor"`), not an icon font or library. No icon should visually resemble an unrelated glyph — a past bug had a "dumbbell" mark that was actually shaped like a percent sign.
- Motion: cards have a staggered fade/lift entrance (`cardIn` animation), but only on genuine navigation to a new screen. Minor same-screen interactions (stepper taps, qty edits) suppress it via a `.no-anim` class toggled in the top-level `render()` — don't remove this or every small interaction will replay entrance animations on every card.
- Mobile-first: inputs must be `font-size: 16px` minimum or iOS Safari auto-zooms on focus. Touch targets aim for ~44px where feasible.

## Rendering pattern
Full `innerHTML` replacement per view on every state change (no virtual DOM, no diffing). This is intentional and fine at this app's data scale — don't "fix" it into a framework rewrite without a real reason.

## Testing
No automated tests. Every real bug in this app's history was caught by manual testing on an actual iPhone against the live Netlify deploy — syntax-checking and structural review (brace balance, duplicate function/CSS detection, orphaned DOM references) catches a different class of bug but not functional ones. Always ask for a real device test after non-trivial changes, especially anything touching storage, dates, or the camera/scanner.
