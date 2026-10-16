# MyOutfits — project context for Claude Code

Personal wardrobe & outfit app. Single file `index.html` (HTML + CSS + JS inline), hosted on GitHub Pages.
UI language: English. Owner talks in Turkish; reply in Turkish, keep UI copy in English.
No framework, no server, no accounts, no paid services. Never upload user photos to external services.
Keep the cream/sage/green look (Young Serif + Onest, hanger logo, Pip mascot, Pinterest-like minimal layout).

## Code map (inside index.html)
- `build(s)` + renderer IIFE: custom loft mesh builder, Three.js 0.160 (jsDelivr, on demand), 2D canvas fallback.
  Models cached by outfit JSON (LRU 10, GPU resources disposed). Redraws batched via rAF (fixes old recursion bug).
- App script sections: basics · catalog · data store · photo store · backup · suggestion engine (`Engine`) ·
  weather & today · photo import · UI · events · start.

## Data
- `localStorage['myoutfits-v4']` = `{schema:4, savedAt, pieces, state, looks, wear, prefs}`
  - wear: one entry per day `[{date,ids}]`; prefs: `{occasion, location, lastWeather, lastBackupAt}`
  - Old keys `myoutfits-v3`, `myoutfits-v2`, `myoutfits-worn` are migrated once and left untouched.
- IndexedDB `myoutfits-photos` / store `images` `{id, blob, createdAt}`. Photo IDs are immutable;
  looks share IDs with pieces; a photo is deleted only when nothing references it.
- Piece fields: id, name, type, color, cut, sleeves, photoId?, formality(1-3), warmth, material(standard|suede),
  rainReady, occasions[everyday|work|special], inLaundry, favorite. Missing fields → `itemDefaults(type)`.

## Backup (.zip, version 1)
`myoutfits-backup.json` + `photos/<id>.<ext>` + `README.txt`. Restore: validate → CRC/signature check →
show counts & confirm → write all photos in one IDB transaction (new IDs) → swap localStorage + verify →
delete old photos. Any failure before the swap leaves current data untouched.

## Suggestion engine rules
Hard rules only: no laundry pieces; needs top + bottom + shoes. Everything else is a score
(warmth vs feels-like curve, rain/snow/wind with `rainReady`, occasion, formality, favorites, recency penalties,
color harmony modes). Bounded search ≤ 10,368 core combos, top 48 kept, accessories picked afterwards.
"Another one" must change a main piece. Forecast uses the city's time zone (Open-Meteo `timezone=auto`).
Weather failure → same-day cached forecast (labeled) or manual weather.

## Testing
Playwright + headless Chromium was used (network mocked: Open-Meteo, three.js, fake background remover).
186 checks passed on v16 (engine, flows, backup/restore, layout at 360/390/768/1440).
Not verified: real iPhone/Android, Safari/Firefox, real IMG.LY model download, real GPU speed.

## Open items / next ideas
1. Photo print on mannequin looks muddy for plain or wrinkled photos → add per-piece
   "Show on mannequin: print / color only", default color-only for non-cut-out or shadowy photos.
2. Scarf and bag strap clip through the body ("physics later").
3. Cloud backup/sync later (Supabase was considered), keep structure ready for profiles + friend-only DMs.
4. Name/domain undecided (myoutfits.co and myout.fit exist; .ca needs Canadian presence — owner is an
   international student).
5. Background remover is AGPL (@imgly) — check license before any commercial launch.
