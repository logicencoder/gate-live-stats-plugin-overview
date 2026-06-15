# Gate.io Live Stats

**Live Gate.io spot statistics** on Logic Encoder — the interactive product lives at **[logicencoder.com/gate-app/](https://logicencoder.com/gate-app/)**, not on static article pages. Open the app, pick any USDT pair from the chip grid (or land with **`?coin=SYMBOL`** such as [DOGEUSDT](https://logicencoder.com/gate-app/?coin=DOGEUSDT)), and get the same realtime trading console as MEXC Live Stats: trade tape with freeze and export, favorites, collapsible panels, TradingView chart, 24h analytics, bot scoring, and hourly volume chart with hover tooltips. Shortcode **`[gate_dashboard]`** embeds the shell; **`symbol="ETHUSDT"`** sets the default pair.

## Tech stack

| Layer | Technologies |
|-------|--------------|
| WordPress plugin | PHP, shortcodes, wp-admin dashboards, WordPress AJAX, WordPress REST cache warmup |
| Public UI | HTML, CSS, vanilla JavaScript, Chart.js, TradingView embed, MessagePack WebSocket client |
| Live backend | Python 3, FastAPI, Gate.io spot WebSocket ingest, PostgreSQL |
| SEO SSR | Node.js Express — crawler HTML + Schema.org JSON-LD via WordPress routing |
| Search | XML sitemap, IndexNow queue (15-minute cron), coin sync from backend API |
| Data | PostgreSQL `gateio_trades` on backend; WordPress options for coin lists and SEO state |
| Hosting | WordPress on shared hosting; Python/Node on self-hosted Linux servers |

## Shared hosting, heavy work on the server

Gate.io Live Stats uses the **same split** as [MEXC Live Stats](https://github.com/logicencoder/mexc-live-stats-plugin-overview): WordPress on **shared hosting** renders the public shell and SEO hooks; **roughly 800 USDT spot pairs** ingest, aggregate, and broadcast from a **self-hosted Linux server**. The browser connects over WebSocket; PHP never becomes the trade database. Hostinger resource graphs stay flat under load because compute lives off the shared plan — see the MEXC overview for the Hostinger proof screenshots and architecture story.

## Live trading app (`/gate-app/`)

The dashboard at [logicencoder.com/gate-app/](https://logicencoder.com/gate-app/) is a **single-page trading console** (`gateDashboard` in JavaScript). Feature parity with MEXC Live Stats — same panel layout, same interaction patterns, Gate.io branding and precision metadata.

### Realtime connection

- **WebSocket** — MessagePack `gate_trade` ticks and `gate_stats` aggregates; active-symbol trades update hero price, arrow, sparkline, tape, and current hourly bar immediately.
- **REST fallback** — `GET /api/stats/memory/{symbol}` on symbol switch; hero price seeds from `current_price` (last trade in window) before the next print.
- **Subscription** — narrow subscribe to active pair; ignore stats/trades for other symbols.

### Symbol header and navigation

- **Breadcrumb** — Home › Gate.io › current pair.
- **Favorite star**, **full name — TICKER**, Gate.io badge, **Share** (copies `?coin=` URL with confirmation feedback).
- **Hero price** with **direction arrow** from latest trade side.
- **15-minute sparkline** with **hover tooltip** on the mini chart.
- **Last updated** timestamp.

### Favorites

Header star, per-chip stars, **favorites row** above the grid, **localStorage** persistence (`gate_favorites`), wp-admin **max favorites** cap.

### Active Coins picker

Symbol count, **search** with clear button, **Ticker / Name / Both** display modes (localStorage), **collapse** for the chip grid, alphabetical **chip grid** with active highlight.

### Collapsible panels

**Active Coins**, **TradingView chart**, and **24h Trading Activity** each support **− / +** collapse with localStorage persistence (`gate_panel_collapse`).

### TradingView chart

Embedded widget (`GATEIO:SYMBOL`); symbol label in header; collapse; iframe reload on pair switch.

### 24h analytics panels

**Current Market Overview** — open, price 24h ago, change vs open and realtime change with **−10% / 0% / +10%** bars, high/low, VWAP, volatility scale.

**Volume Analysis** — USDT and base volume, buy/sell bars, ratio scale, net flow, whale activity scale.

**Buy/Sell Analysis** — trade counts, count ratio, trading direction bar, buy volume % gauge.

### Trade analytics, bot activity, and live tape

**Trade Analytics** — average size, interval, largest trade, small/medium/large tier distribution.

**Bot Activity** — bot score, repeat size/interval, round lot, burst score with labeled scales.

**Real-Time Trades** — scrolling ledger with BUY/SELL color coding.

| Control | Behavior |
|---------|----------|
| **Realtime feed** | Append at bottom; auto-follow tail when scrolled to end. |
| **Freeze ❄️** | Buffer incoming rows; show pending count; flush on unfreeze. |
| **Export ↓** | **TXT**, **CSV**, **JSON** from DB history + WebSocket merge. |

### 24h Trading Activity chart

Hourly buy/sell bars; **fixed tooltip strip** above chart (hour, buy/sell volumes in coin and USDT); hover updates tooltip; live current-hour growth; collapsible panel.

### Symbol switch workflow

Chip click or `?coin=` clears stale UI, reloads TradingView, REST stats for immediate hero + panels, WebSocket resubscribe, tape backfill, URL `history.replaceState`.

## WordPress admin

**Gate.io Trading Dashboard** settings — WebSocket URL, SSR base URL, snapshot toggles.

**Coin Manager** — add/remove symbols, dead-symbol list, reload from backend, stream status table.

**Google validation dashboard** — SSR structure checks and JSON-LD inventory (same pattern as MEXC).

**Sitemap & IndexNow** — queue status, auto-push on publish, **Sync now** from backend bootstrap. Hard-excluded from sitemap: `BRISEUSDT`, `MOGUSDT`.

**AJAX** — API key proxy, server stats, monitoring, visitor logs, coin full name, health check.

**REST** — `POST /wp-json/gate/v1/cache-warmup` for backend cache sync after symbol reload.

## Search discovery (crawlers only)

Humans use **`/gate-app/`**. SEO URLs under **`/gate/{SYMBOL}/`** serve crawler HTML through WordPress `template_redirect` + Node SSR — live stats embedded for search, not the interactive app shell.

Private code: [gate-live-stats-plugin](https://github.com/logicencoder/gate-live-stats-plugin) · live data [gate-live-stats-backend](https://github.com/logicencoder/gate-live-stats-backend)

Backend overview: [gate-live-stats-backend-overview](https://github.com/logicencoder/gate-live-stats-backend-overview)

MEXC sibling (same UX patterns): [mexc-live-stats-plugin-overview](https://github.com/logicencoder/mexc-live-stats-plugin-overview)

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
