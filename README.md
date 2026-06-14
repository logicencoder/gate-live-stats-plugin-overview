# Gate.io Live Stats

Public **Gate.io spot trading statistics** on logicencoder.com — live trade tape, bot-activity views, analytics panels, and indexable per-coin pages with Schema.org JSON-LD.

Private plugin: [logicencoder/gate-live-stats-plugin](https://github.com/logicencoder/gate-live-stats-plugin). Data pipeline: [gate-live-stats-backend](https://github.com/logicencoder/gate-live-stats-backend) — [backend overview](https://github.com/logicencoder/gate-live-stats-backend-overview).

## Live dashboard

**[logicencoder.com/gate-app/](https://logicencoder.com/gate-app/)**

Shortcode **`[gate_dashboard symbol="ETHUSDT"]`** — optional default symbol; conditional asset load only on pages that use the shortcode.

## Per-coin SEO pages

Pretty URLs **[/gate/{SYMBOL}/](https://logicencoder.com/gate/)** — WordPress `template_redirect` serves Node SSR HTML with live stats embedded. Legacy query form: `/gate-app/?coin=SYMBOL`. SEO hub breadcrumb at [/gate/](https://logicencoder.com/gate/).

Open Graph default image: logicencoder-gate-app.png on logicencoder.com.

## Visitor experience

- Live WebSocket from **`wss://ws2.logicencoder.com/ws`**
- Scroll or freeze trade tape; bot-activity indicators
- Pair-level charts and 24h stats from backend PostgreSQL history
- Breadcrumb links to site home and applications catalogue

Excluded from sitemap (hardcoded): `BRISEUSDT`, `MOGUSDT`.

## WordPress admin

**Gate.io Trading Dashboard** settings: WebSocket URL, SSR base URL (defaults point at ws2 tunnel). Coin manager, dead-symbol admin list, Google validation dashboard, IndexNow sitemap queue (15-minute cron), coin sync from backend API.

**AJAX (admin + public where wired):** API key proxy, server stats, monitoring status, visitor logs, coin full name lookup, health check.

**REST:** `POST /wp-json/gate/v1/cache-warmup` — backend pushes schema/cache sync after symbol reload.

## Data flow

Gate.io spot WebSocket → Python ingest → PostgreSQL `gateio_trades` → `/ws` to browser + `/ssr_m1/{symbol}` for SSR. Unlike MEXC Live Stats, Gate SEO uses **live SSR through WordPress routing**, not disk snapshot POSTs.

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
