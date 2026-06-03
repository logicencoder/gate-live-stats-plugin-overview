# Gate.io Live Trading Statistics — WordPress Plugin (Public Overview)

How the **private WordPress plugin** presents Gate.io live stats on LogicEncoder.com.  
No PHP source, API keys, or hosting credentials in this repository.

**Live URL:** [logicencoder.com/gate-app/](https://logicencoder.com/gate-app/)  
**Private implementation:** [gate-live-stats-plugin](https://github.com/logicencoder/gate-live-stats-plugin)

---

## Purpose

WordPress is where visitors land. The plugin embeds a **multi-coin realtime dashboard** via shortcode, connects the browser to the private FastAPI backend on `ws2.logicencoder.com`, and serves **SSR pages** for SEO (`/gate-app/`, `/gate/{COIN}`) with Schema.org JSON-LD.

---

## System position

```
Visitor opens page with [gate_dashboard]
        │
        ├──► Browser ◄──WebSocket/REST──► FastAPI backend (private, ws2)
        │
        └──► Crawler ◄──SSR HTML────────► WordPress template_redirect → Node SSR
```

Backend overview: [gate-live-stats-backend-overview](https://github.com/logicencoder/gate-live-stats-backend-overview).

---

## Live dashboard shortcode

`[gate_dashboard symbol="ETHUSDT"]` — same feature set as MEXC: live trades (scroll/freeze), bot activity, analytics panels, coin manager, dead-symbols admin, Google validation dashboard, IndexNow sitemap, REST cache warmup (`gate/v1`).

---

## REPOS.md

| Repo | Visibility |
|------|------------|
| gate-live-stats-plugin | private (code) |
| gate-live-stats-plugin-overview | public (this repo) |
| gate-live-stats-backend | private |
| gate-live-stats-backend-overview | public |
