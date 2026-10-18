# Card Sourcing Dashboard

A single static page to browse and search sourced Japanese Pokémon card listings.
No build step, no dependencies: one `index.html` (inline CSS + vanilla JS) and one
data file, `data/listings.json`.

## Demo data

The site currently ships **DEMO data only** (`"demo": true` in the JSON). Live
sourcing is not connected yet, so the page shows a banner saying exactly that.
Do not treat any listing, price, or status on the page as a real finding.

## What the data contains

Only this safe projection is stored and displayed:

- `title`, `price_jpy` (integer yen), `condition`, `status`, `images`, `detected_at`

Deliberately **not** included anywhere: store/shop name, source URL, item IDs,
seller names, tokens, internal IDs, or any market/fair-value/discount figures.

## Updating the listings

1. Edit `data/listings.json` by hand (or generate it from your sourcing tool).
2. Keep the shape: `generated_at`, `demo`, and a `listings` array of objects with
   the fields above. `images` is an ordered array; the first image is the cover.
3. Keep `"demo": true` until live data is real. When real data lands, set it to
   `false` and refresh `generated_at` — the banner switches to a live-data line.
4. Commit to `main`; GitHub Pages redeploys automatically.

Images may be data URIs (as in the demo) or ordinary files you add to the repo.

## How the page works

- Search filters by title (case-insensitive, partial match, trimmed).
- Sort by price (low → high / high → low); checkbox shows/hides `INACTIVE` items.
- A results line shows "N of M shown" and an empty state appears when nothing matches.
- If `data/listings.json` cannot be loaded, an inline error is shown instead of a
  blank page.

## GitHub Pages

Settings → Pages → Build and deployment → Deploy from a branch: branch `main`,
folder `/ (root)`. The empty `.nojekyll` file stops Jekyll from processing the
site, so `data/listings.json` is served as-is.

## Local preview

Serve the folder over HTTP (opening `index.html` via `file://` blocks `fetch`):

```
python3 -m http.server 8000
```

Then open http://localhost:8000/
