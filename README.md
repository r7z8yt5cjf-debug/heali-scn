# SCN Desk — PWA

Offline reference for the NYS SCN Operations Manual. Fuzzy search, tap-to-copy citation chips, version picker, version ledger. Installs to the home screen and works with no connection after first load.

## Folder layout

```
scn-desk/
├── index.html            app (all CSS/JS inline)
├── manifest.webmanifest  PWA manifest
├── sw.js                 service worker (offline cache)
├── logo.png              header logo
├── icons/                app icons (from HEALI logo)
└── data/
    ├── versions.json     version registry (the ledger)
    └── v8.json           full manual v8
```

## Deploy (one-time, ~5 minutes)

A PWA **must be served over HTTPS** — service workers do not run from `file://` or from SharePoint/Teams file previews. Easiest free option is GitHub Pages:

1. Create a free account at github.com → New repository → name it `scn-desk`, set **Public** (or Private on a paid plan with Pages enabled).
2. Upload everything in this folder (drag-and-drop works: "uploading an existing file"). Keep the folder structure — `data/` and `icons/` as folders.
3. Repo → Settings → Pages → Source: **Deploy from a branch** → Branch: `main`, folder `/ (root)` → Save.
4. After ~1 minute your app is live at `https://<your-username>.github.io/scn-desk/`.

Share that link with the team. On iPhone: open in Safari → Share → **Add to Home Screen**. On Android/desktop Chrome: the install prompt appears in the address bar. After the first load, it works fully offline.

**Test locally** (optional): in the folder, run `python3 -m http.server 8080` and open `http://localhost:8080`. Localhost is the one non-HTTPS place service workers run.

## Adding v7 (or any version)

1. Drop `v7.json` into `data/` (same schema: `version/label/effective/source/coverage/coverage_note/sections[]`).
2. Add an entry to `data/versions.json`:
```json
{ "version": "v7", "label": "Version 7", "effective": "2025-10-01",
  "file": "v7.json", "coverage": "partial", "note": "…" }
```
3. Optionally add `"v7.json"` path to the `SHELL` list in `sw.js` so it precaches for offline.

## "Changed in v8" flags

The app renders a gold **Changed in v8** badge on any section that has either:
```json
"changed": true          // or
"flags": ["v8"]
```
Tag the sections the §0 change log touches (e.g., `5f-service-re-authorizations`, the 5.J services with revised criteria) and the badge appears in browse, search results, and the reader.

## Shipping updates

When you change any file, bump the cache name in `sw.js` (`scn-desk-v1` → `scn-desk-v2`) so installed clients pick up the new version. Data files (`data/*.json`) refresh automatically in the background even without a bump.

## Citation chips

Tapping any §-chip copies a formatted citation, e.g.:
> SCN Operations Manual Version 8 (eff. 5/1/2026), §5.J (2.3) — HRSN Services — 2.3 Asthma Remediation, pp. 170–178

Search accepts plain language ("asthma remediation eligibility"), tolerates one-letter typos on longer words, and understands cite lookups like `5.J 2.3` or `5d`.
