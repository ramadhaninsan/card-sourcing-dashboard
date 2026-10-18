# Card Sourcing Dashboard

A single static page to browse and search sourced Japanese Pokémon card listings.
No build step, no dependencies: one `index.html` (inline CSS + vanilla JS) and one
data file, `data/listings.json`.

## Data provenance

The site is a consumer only. `data/listings.json` is exported by the **private
sourcing pipeline** (currently its offline dry-run fixture) and copied here
verbatim. Nothing on this page is a live finding, and no data is fetched from
anywhere except that one file.

The banner is driven by the `data_label` field, so the page always states what
the data actually is (e.g. "Offline dry-run fixture (synthetic) — not live
sourcing"). If `data_label` is missing, a synthetic-data fallback sentence is
shown. The `generated_at` timestamp is shown under the banner.

## What the data contains

Top level:

- `generated_at` — ISO timestamp of the export.
- `demo` — whether the export is synthetic. Not used to word the banner.
- `data_label` — exact banner text describing the data.
- `market_note` — caveat about the market indicator, shown once below the list.
- `listings` — array of listing objects.
- `excluded` — array of rejected items, each with `title` and `reason`.
- `market` — array of cached market-estimate rows the run considered, in the
  export's order. Each row has `card`, `market_indicator_jpy`, `meets_floor`
  (boolean), `estimate_at` (ISO timestamp), and `ask_jpy` (the matched eligible
  listing's ask, or `null` when the estimate matched no listing).

Per listing:

- `title`, `price_jpy` (integer yen), `market_indicator_jpy`, `condition`
  (`"1/6"` … `"6/6"`), `status` (`NEW` / `SEEN` / `INACTIVE`), `images`
  (ordered array of data URIs; the first image is the cover), `detected_at`.

Deliberately **not** included anywhere: store/shop name, source URL, item IDs,
seller names, tokens, internal IDs.

## Market indicator caveat

The market indicator is a PokemonPriceTracker provider estimate converted with a
timestamped USD/JPY rate. It is an indicator for selecting cards, **not** a
valuation and **not** a deal verdict, and the page treats it that way:

- It is shown as its own line — `Market indicator ¥5,000` — next to the asking
  price, with the secondary note "provider estimate, not a valuation".
- The full `market_note` text is shown once below the results.
- If `market_indicator_jpy` is missing from a listing, the line is omitted
  entirely (no placeholder, no zero).
- The page never derives anything from the two numbers: no savings, percentage,
  gap, "under market" flag, or verdict of any kind.

## Market prices table

Below the caveat, the "Market prices (provider estimates)" section renders the
top-level `market` array as a real table, in the data file's order. The table is
independent of search, sort, and the INACTIVE toggle — it always shows every row
the run considered. Columns:

- **Card** — the row's `card` label.
- **Market indicator** — the provider estimate in JPY with thousands separators.
  A missing, null, or non-positive value is shown as `—`.
- **Ask** — the matched eligible listing's `ask_jpy` in JPY, or `—` when
  `ask_jpy` is absent or null.
- **Estimate date** — `estimate_at` in UTC, matching the listing date style
  (`YYYY-MM-DD HH:MM UTC`), or `—` when missing or invalid.
- **Rule (≥ ¥5,000)** — the run's selection-rule result, worded `qualifies`
  (`meets_floor: true`) or `below floor` (`meets_floor: false`), and `—` when the
  flag is missing. This is a rule outcome, never a valuation or a deal verdict.

The heading's explanation states that these are cached provider estimates the
run considered, shown in JPY, and that only estimates of at least ¥5,000 select
a card. The whole section is hidden when `market` is absent or empty, so no empty
table is rendered. Missing cells are dashed — never `¥0`, `NaN`, or `undefined`.

On narrow screens the table is wrapped in its own horizontally scrollable,
keyboard-focusable region. The document itself must not scroll sideways: at
390 px wide, `document.scrollWidth` should equal `window.innerWidth`.

## Excluded section

`excluded` entries are rendered as a small list titled "Excluded by this run",
each row showing only the item `title` and the recorded `reason` — nothing is
inferred or fabricated, and no ids or URLs exist in the data to show.

## Updating the listings

1. Re-run the private pipeline and take its exported JSON as-is.
2. Replace `data/listings.json` with that file. Do not hand-edit it and do not
   add fields: this repo only consumes the export.
3. Commit to `main`; GitHub Pages redeploys automatically.

Images are data URIs in the export (or ordinary files you add to the repo).

## How the page works

- Search filters by title (case-insensitive, partial match, trimmed).
- Sort by price (low → high / high → low); checkbox shows/hides `INACTIVE` items.
- A results line shows "N of M shown" and an empty state appears when nothing matches.
- The excluded list and the market prices table are not affected by search,
  sort, or the INACTIVE toggle.
- The market prices table scrolls horizontally inside its own wrapper on narrow
  screens; the page body never scrolls sideways.
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
