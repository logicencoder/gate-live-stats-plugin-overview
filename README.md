# Gate.io Live Stats

Public **Gate.io spot trading statistics** on logicencoder.com — live trades, analytics panels, and indexable per-coin SEO pages with Schema.org JSON-LD.

Private plugin: [logicencoder/gate-live-stats-plugin](https://github.com/logicencoder/gate-live-stats-plugin). Data pipeline: [gate-live-stats-backend](https://github.com/logicencoder/gate-live-stats-backend) — [backend overview](https://github.com/logicencoder/gate-live-stats-backend-overview).

## Live dashboard

**[logicencoder.com/gate-app/](https://logicencoder.com/gate-app/)**

Shortcode **`[gate_dashboard symbol="ETHUSDT"]`** — optional default symbol; same live tape and analytics as the hub page.

## Per-coin SEO pages

Pretty URLs **[/gate/{SYMBOL}/](https://logicencoder.com/gate/)** served via WordPress `template_redirect` + Node SSR HTML (legacy query form: `/gate-app/?coin=`). SEO hub linked from UI breadcrumbs at [/gate/](https://logicencoder.com/gate/).

## Visitor experience

Live WebSocket from `wss://ws2.logicencoder.com/ws` — scroll/freeze trade tape, bot-activity views, pair stats from backend PostgreSQL history.

## WordPress admin

Settings: WebSocket URL, SSR base URL, coin manager, dead-symbol list, Google validation dashboard, IndexNow sitemap queue (15-minute cron), coin sync from API.

## Data flow

Gate.io spot WebSocket → Python ingest → PostgreSQL → `/ws` to browsers + SSR endpoint for crawlers and pretty URLs.

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
