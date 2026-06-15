# Gate.io Live Stats

![Gate.io Live Stats — live trading app at logicencoder.com/gate-app](assets/live-dashboard.png)

**Live Gate.io spot statistics** on Logic Encoder — the interactive product lives at **[logicencoder.com/gate-app/](https://logicencoder.com/gate-app/)**, not on static article pages. Open the app, pick any USDT pair from the chip grid (or land with **`?coin=SYMBOL`** such as [DOGEUSDT](https://logicencoder.com/gate-app/?coin=DOGEUSDT)), and get realtime trade tape, bot-activity scoring, TradingView chart, 24h analytics panels, hourly volume chart, favorites, export, and freeze — all in one WordPress-embedded shell. Shortcode **`[gate_dashboard]`** drops the same UI on any page; **`symbol="ETHUSDT"`** sets the default pair.

## Tech stack

| Layer | Technologies |
|-------|--------------|
| WordPress plugin | PHP, shortcodes, wp-admin dashboards, WordPress AJAX, WordPress REST cache warmup |
| Public UI | HTML, CSS, vanilla JavaScript, Chart.js, TradingView embed, MessagePack WebSocket client |
| Live backend | Python 3, FastAPI, Gate.io spot WebSocket ingest, PostgreSQL |
| SEO SSR | Node.js Express — crawler HTML + Schema.org JSON-LD via WordPress routing |
| Search | XML sitemap, IndexNow queue (15-minute cron), coin sync from backend API |
| Data | PostgreSQL trade history on backend; WordPress options for coin lists and SEO state |
| Hosting | WordPress on shared hosting; Python/Node on self-hosted Linux servers |

## Shared hosting, heavy work on the server

Gate.io Live Stats uses the **same split** as [MEXC Live Stats](https://github.com/logicencoder/mexc-live-stats-plugin-overview): WordPress on **shared hosting** renders the public shell, sitemaps, and IndexNow hooks; ingest, aggregation, PostgreSQL, MessagePack fan-out, chart generation, and SSR bundles run on a **self-hosted Linux server**. The browser connects over WebSocket; PHP never becomes the trade database.

**Roughly 700 USDT spot pairs** run on the live Gate install today (the parallel MEXC fleet carries **1,400+**). Visitors still get realtime tapes and rolling analytics in the browser; WordPress mostly **displays and indexes** what the backend already computed. The [MEXC Live Stats overview](https://github.com/logicencoder/mexc-live-stats-plugin-overview) includes Hostinger resource graphs that apply to the same architecture pattern on both products.

## Live trading app (`/gate-app/`)

The dashboard at [logicencoder.com/gate-app/](https://logicencoder.com/gate-app/) is a **single-page trading console**. Every control below is implemented in the plugin JavaScript (`gateDashboard`); data comes from the self-hosted Gate.io backend over WebSocket and REST.

### Realtime connection

- **WebSocket** — MessagePack frames carry `gate_trade` ticks and periodic `gate_stats` aggregates. Trades for the **active symbol** update the hero price, direction arrow, sparkline, tape rows, and the current hour on the 24h bar chart in the same event loop — no polling for headline price.
- **REST fallback** — `GET /api/stats/memory/{symbol}` loads full 24h stats when you switch pairs or when bootstrap has no cache yet. Hero **last price** seeds from `current_price`, then `24h_open`, VWAP, or high/low fallbacks; if still empty, `GET /api/trades?hours=24&limit=1` pulls the latest DB print. On symbol switch the hero clears to **—** immediately so the previous pair’s price never lingers.
- **Subscription** — switching chips sends a new `subscribe` action for the active pair; stats and trades for other symbols are ignored so panels never show cross-contaminated data.

### Symbol header and navigation

- **Breadcrumb** — Home › Applications › Gate.io › current pair name (updates on symbol switch).
- **Title row** — favorite star, **full name — TICKER**, Gate.io badge, **Share** button.
- **Share** — copies the current page URL with `?coin=SYMBOL` to clipboard; button flashes confirmation feedback.
- **Hero price** — large USDT last trade formatted with **exchange price precision** and thousand separators from **1,000** upward; **direction arrow** (↑ buy tint, ↓ sell tint) follows the latest print side.
- **15-minute sparkline** — Chart.js line beside the price, fed from per-symbol price history in memory; **hover tooltip** on the mini chart shows the sampled price at that point.
- **Last updated** — clock time of the last applied tick.

### Favorites

- **Header star** — toggle favorite for the active pair.
- **Per-chip star** on each coin button in the grid — same list.
- **Favorites row** — dedicated chip strip above the full grid when you have favorites; hidden when empty.
- **localStorage** — `gate_favorites` persists across sessions in the browser; wp-admin exposes **max favorites** (default cap stored in `gate_max_favorites`).

### Active Coins picker

- **Symbol count** in the panel title — fleet size from backend bootstrap (~700 USDT pairs on the live install).
- **Search** — filters chips by ticker or full name; **×** clears input.
- **Display mode** — **Ticker**, **Name**, or **Both** on chips; saved in `gate_coin_display_mode` localStorage.
- **Collapse (−/+)** — hides chip grid, search, and display toggle; state in `gate_panel_collapse`.
- **Chip grid** — alphabetical USDT pairs; active chip highlighted; one click rebinds all panels and updates the URL bar without reload.

The live app opens on app pages with WordPress chrome hidden: in-app breadcrumb, favorite star and pair title, hero price with direction arrow and mini sparkline, **Active Coins** search and display toggles, favorites row, and the scrollable chip grid for the full fleet.

### TradingView chart

Embedded **TradingView** widget (`GATEIO:SYMBOL`): timeframe toolbar, indicators, drawings, candlesticks, volume sub-chart. Panel header shows the active symbol. **Collapse** shrinks iframe height to zero and remembers preference in `gate_panel_collapse`. Iframe `src` reloads on every symbol switch.

### 24h analytics panels

Three side-by-side panels summarize **rolling 24h structure** for the active pair (stats broadcast + REST):

**Current Market Overview** — 24h open, price 24h ago, change vs open and real-time change (each with a **−10% / 0% / +10%** sentiment bar and context label), 24h high and low, **VWAP**, **volatility** on a stable → wild scale. Prices use per-symbol precision from bootstrap metadata.

**Volume Analysis** — total 24h volume in USDT and base asset; **buy vs sell volume** bars; **buy/sell volume ratio** on strong-sell → strong-buy scale; **net flow** in USDT and coins with direction arrow; **whale activity** on retail → whale dominance scale. USDT volumes round with thousand separators; coin amounts respect **amount precision**.

**Buy/Sell Analysis** — 24h trade count, buy/sell counts and percentages, **count ratio**, **trading direction** label and bar (all-sell → balanced → all-buy), **buy volume %** gauge. Trade counts use thousand separators.

### Trade analytics, bot activity, and live tape

**Trade Analytics** — average trade size in USDT and base asset (micro → retail → pro scale); **average interval** between prints (active → slow); **largest trade** size, side, and time; distribution bars for **small**, **medium**, and **large** USDT tiers.

**Bot Activity** — **bot score** on clean → bot scale; **repeat size**, **repeat interval**, **round lot**, and **burst score** each with organic → suspect → likely → confirmed style bars and text labels.

**Real-Time Trades** — live scrolling ledger: time, symbol, price, amount, USDT notional, color-coded **BUY/SELL** with side stripe. Price and amount columns format per-symbol from `symbolPrecision` cache.

| Control | Behavior |
|---------|----------|
| **Realtime feed** | New rows append at the bottom; if you are at the bottom, the tape **follows the tail**; scroll up to read without auto-jump. |
| **Freeze ❄️** | Pauses live inserts; trades buffer in memory; button shows pending count; unfreeze flushes in order. |
| **Export ↓** | **TXT**, **CSV**, or **JSON** — merges PostgreSQL history (rolling window) with in-memory WebSocket rows, deduplicated; download filename includes symbol and timestamp. |

Only trades matching the **active symbol** update the tape and hero; other symbols still contribute to sparkline history caches.

### 24h Trading Activity chart

Full-width **hourly bar chart** — green **buy** and red **sell** base-asset volume per clock hour for the active pair.

- **Fixed tooltip strip** above the chart — hour label, buy volume, sell volume (coin + USDT) with color swatches; defaults to the latest hour after load.
- **Hover / scrub** — moving across bars updates the fixed tooltip (Chart.js built-in tooltip disabled for stable mobile layout).
- **Live hour bar** — current hour grows as new trades arrive via `updateCurrentHourBar`.
- **Collapse** — same localStorage persistence as coins grid and TradingView.

### Symbol switch workflow

Clicking a chip (or loading `?coin=`): clears hero to **—**, clears tape and stats cache, resets sparkline datasets, reloads TradingView, fetches REST stats for immediate panel + hero price fill, seeds from recent DB trade if needed, re-subscribes WebSocket, and backfills the tape from DB + WS. URL updates via `history.replaceState` so Share stays accurate.

## Monitor Dashboard

wp-admin **Monitor Dashboard** is the operator nerve center when something looks stale on the public site or trade counts drop after a deploy. Three panels mirror what you would otherwise SSH in to check.

### WebSocket throughput and health

Top status bar shows **API** and **SSR** reachability with response times, plus a green **Connected** badge when the admin monitor socket is live.

The **WebSocket throughput** grid tracks:

- **Messages/sec** and **download KB/s** — is data actually flowing right now?
- **Peak rate** and **total downloaded** — cumulative volume since server start.
- **Reconnects** and **server uptime** — spot flapping connections or recent restarts.
- **Min / avg / current latency** — end-to-end delay from Gate.io ingest to your browser.
- **Compression ratio** — MessagePack savings vs raw payload size.

The **3-hour rolling chart** plots download speed and messages/sec per minute — correlate a traffic drop with a deploy, a Gate.io outage, or a subscription gap. If messages/sec flatlines while Gate.io is healthy, scroll to System logs for auth or subscribe failures.

### Subscribed pairs and server metrics

The **subscribed pairs** chip wall lists every USDT market on the Gate.io stream — a dense wall of symbol tags in alphanumeric order. Scroll it when you suspect a new listing never subscribed.

Four summary cards below:

| Card | What to watch |
|------|----------------|
| **Subscribed pairs** | Tracked vs actively printing counts. Footer: aggregate **trades/min** + **Gate.io connection healthy**. |
| **Broadcast queue** | Internal fan-out depth — should stay near zero; growth means clients or Postgres writes lag. |
| **PostgreSQL** | Total trades stored, messages sent to browsers, server start time. |
| **Connected clients** | Unique IPs and WebSocket sessions reading right now. |

When public users report “frozen tape” but trades/min is nonzero, check **connected clients** — you may have a front-end issue, not ingest.

### System logs

Dark **system log** console filtered to connection lifecycle — no per-trade spam. Typical boot sequence:

1. Config loaded from server.
2. Admin monitor initializes (MessagePack mode).
3. WebSocket connect attempt with URL and retry count.
4. Authentication success with masked API key prefix.
5. Bulk subscribe to **all** symbols (full fleet list; UI truncates display).
6. Server welcome and **successfully subscribed** confirmation.

Use when the public site shows zero trades/min but Gate.io itself is up — you will see whether auth failed, subscribe timed out, or the socket never reconnect after a key rotation.

## Coin Manager

wp-admin **Coin Manager** is fleet hygiene across every tracked USDT pair — add new listings, prune delisted symbols, reload from the server bootstrap, and validate SEO output without leaving WordPress.

### Overview and dead coins

Four stat cards summarize fleet health:

| Card | Meaning |
|------|---------|
| **Tracked coins** | Symbols on the server list; footer shows aggregate trades/min. |
| **Live stream** | Pairs receiving trades now; footer counts idle (no print in 60s). |
| **Never traded** | Added but no historical print yet — common right after listing. |
| **Dead / missing** | Delisted on Gate.io, removed from stream, or never subscribed. |

The **Dead & missing** panel lists red chips for symbols no longer on the exchange. One click **removes** them so sitemap and SSR do not 404 forever. **Refresh dead list** pulls the latest diff from the backend after a Gate.io maintenance window.

### Add, reload, and tracked list

**Reload symbols on server** syncs the WordPress coin list with the Python bootstrap — run after a backend deploy or Gate.io listing batch.

- **Single add** — type a symbol, press Add coin.
- **Bulk import** — paste one symbol per line, **Add all** for dozens of new listings.

The **All tracked coins** table is the sortable fleet roster: symbol, **trades/min**, **stream** status (live vs idle), **ever traded** flag, and **Remove** per row. **Refresh now** updates stream stats without reloading the page.

### Schema.org validation

Before you push a new coin to production SEO, pick a symbol and hit **Run validation**. Two layers:

**Python API — server stats**: connected clients, messages sent, latency (min / max / avg), compression ratio, subscription room counts.

**Node SSR — HTML structure checks**: Bot Activity section, 24h hourly table, KPI strip, ring gauge, Schema.org JSON-LD block, symbol string in document, HTML byte size sanity check.

The expandable **JSON-LD key inventory** lists every Schema.org field the SSR emitted. Operators confirm Google Rich Results will see a real `Dataset`, not a stub page.

### Activity log

Timestamped **activity log** records operator actions: coin add/remove, bulk import results, server reload (with POST status), auto-refresh coin-count changes, and success/warning/error color coding.

## Sitemap and IndexNow

Search discovery is first-class — a per-coin URL for every tracked symbol would be impossible to submit by hand.

### Sitemap status and IndexNow queue

**Gate.io Sitemap & Indexing** dashboard shows:

- **Public sitemap URL** with live HTTP check.
- **WordPress hook** registered and **static file** present for fallback.
- **Total URLs** in sitemap and cache timestamp.
- Buttons to **view XML**, open **Google Search Console**, and **save static sitemap file** after bulk coin changes.

**IndexNow queue** panel tracks pending URLs, last batch time/status, sent vs remaining counts, and **last push new coins** after a listing batch. A **15-minute cron** drains the queue in batches.

### Auto-push, coin table, and API sync

- **Auto push on publish** — when a WordPress post or page goes live, its permalink hits IndexNow automatically.
- **Coins in sitemap** — paginated table of symbol + path — catch typos before Google does.
- **Sync now** — pulls the authoritative coin list from the Python API bootstrap and rebuilds sitemap entries.
- **Hard exclusions** — `BRISEUSDT` and `MOGUSDT` never enter the sitemap (operator blocklist in plugin code).

Typical workflow after Gate.io lists new pairs: bulk add in Coin Manager → **Reload symbols on server** → **Sync now** on sitemap → watch IndexNow queue drain → spot-check one symbol in Schema validation.

**REST cache warmup** — `POST /wp-json/gate/v1/cache-warmup` lets the backend push refreshed precision and coin metadata into WordPress after a symbol reload.

## Search discovery (crawlers only)

Humans use **`/gate-app/`**. Separate **SEO URLs** under `/gate/{SYMBOL}/` serve crawler HTML + Schema.org JSON-LD from Node SSR — useful for search, not the interactive product. WordPress `template_redirect` routes crawlers; regular browsers stay on the interactive dashboard unless `?ssr=1` is used for testing.

Private code: [gate-live-stats-plugin](https://github.com/logicencoder/gate-live-stats-plugin) · live data [gate-live-stats-backend](https://github.com/logicencoder/gate-live-stats-backend)

Backend overview: [gate-live-stats-backend-overview](https://github.com/logicencoder/gate-live-stats-backend-overview)

Sibling product (same console patterns on MEXC): [mexc-live-stats-plugin-overview](https://github.com/logicencoder/mexc-live-stats-plugin-overview)

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
