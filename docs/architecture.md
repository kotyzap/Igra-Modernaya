# Architecture & hosting

```
             im.4xs.dev (Cloudflare custom domain)
                          │
                ┌─────────▼──────────┐
                │  Cloudflare Worker  │  im-4xs-dev
                │  run_worker_first:  │  /api/*, /imadmin*, /_admin*
                └──┬───────┬──────┬───┘
     static assets │       │      │ Access JWT check
   (SPA fallback)  │       │      │
   index.html ◄────┘       │      └──► /_admin/  (dashboard page)
   manifest, icons         │
                ┌──────────┴──────────┐
                │                     │
        KV "SCORES"              D1 "im-4xs-stats"
        records → Top 5          events (view/start/end/rec)
```

## Front end

- **One HTML file** (`index.html`, ~70 KB) — all games, CSS and JS inline, no frameworks.
- Each game is a small object: `reset(mode)`, `press(button)`, `update(now, demo)`, `draw(ctx, state, now)`, plus a `face` layout.
- Rendering: one `<canvas>` with a 400 × 260 logical LCD, scaled with `devicePixelRatio`.
- Routing: real paths on im.4xs.dev (`/btc`, `/kitchen`, `/keys`, `/moon`), `#hash` elsewhere; retired paths go to the list.
- PWA: `manifest.webmanifest` (`display: fullscreen`, `orientation: landscape`) and Apple Home Screen meta tags.
- i18n: Czech for `cs`/`sk` devices, English otherwise.
- `_headers`: `Cache-Control: no-cache`, so updates appear on the next reload.

## API

| Method | Path | Auth | Purpose |
|---|---|---|---|
| GET | `/api/records` | public | all Top 5 lists |
| POST | `/api/records` | public | submit `{g, m, s, n}`; returns rank |
| POST | `/api/event` | public | anonymous play event (`view`, `start`, `end`, `rec`) |
| GET | `/api/admin/stats` | Access | dashboard data |
| POST | `/api/admin/records/delete` | Access | remove a Top 5 entry |

Validation: game id `[a-z]{2,12}`, mode `A`/`B`, score `1–9999`, initials `[A-Z]{3}`.

## Data

- **KV `records`** — `{ game: { A: [{s, n, t}…5], B: […] } }`.
- **D1 `events`** — `ts, type, game, mode, score, halving, country, device, os, browser, pwa, sid, ini` (indexed on `ts`, `sid`).

## Deploy

```bash
npx wrangler deploy
```
