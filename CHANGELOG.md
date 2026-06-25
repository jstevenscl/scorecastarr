# Changelog

All notable changes to ScorecastArr are documented here.

Versioning follows [Semantic Versioning](https://semver.org/):
- `MAJOR.MINOR.PATCH` for stable releases (e.g. `v1.0.0`)
- `-beta` suffix for pre-release builds (e.g. `v0.2.0-beta`)

Docker images are published to [GitHub Container Registry](https://ghcr.io):
```
ghcr.io/OWNER/scorecastarr-api:latest    # tracks main — what most users pull
ghcr.io/OWNER/scorecastarr-api:dev       # tracks dev branch — bleeding edge
ghcr.io/OWNER/scorecastarr-api:v0.3.0   # pinned version
```

---

## [v0.3.0-beta] — 2026-06-25

### Added
- **FIFA World Cup 2026** — added to Sports Library and Ticker Overlay under International Soccer. Shows all group stage results automatically (no "Show Ended" toggle required) by fetching ESPN day-by-day from June 11 onwards. Motor cache worker also backfills match history on each run. Live matches refresh every 30s.
- **Ticker UI overhaul** — group chips for bulk sport selection, active panel with live stream badges, per-group expand/collapse, and inline sport toggles.
- **Ticker stream end-to-end** — score ticker channel plays in TiviMate with scrolling scores; confirmed working with Dispatcharr.
- **Per-sport refresh rates** — each sport has a configurable auto-refresh interval (NASCAR/F1: 10s, MLB/NHL: 20s, NBA/PGA: 30s, most others: 60s). 5s master tick dispatches only sports whose due time has passed. Rates shown as badges in Sports Library; `*` suffix = global default.
- **Built-in setup walkthrough** — 7-step guided overlay auto-triggers on first run (no scoreboards + `sca_setup_done` not set); also accessible via the 🚀 Setup Guide sidebar button.
- **Interactive tour** — self-contained 6-chapter guide at `/tour`; also published to GitHub Pages.
- **Stream safe-area padding** — 0–10% overscan inset per scoreboard (Step 3 → Card Scale → Safe-Area Padding). Subtracts from available height/width for set-top box overscan compensation.
- **PGA cut players toggle** — defaults to hidden; user must opt-in to show cut players on the leaderboard.
- **Dispatcharr orphaned profile cleanup** — Settings → Ticker Overlay → Maintenance → 🧹 button deletes stream profiles ending in ` (Ticker)` that are no longer associated with any active ticker. Confirmed safe: active ticker profiles are always preserved.
- **Sports Library rate badge tooltip** — hovering the expand button shows a tooltip explaining the `*` suffix (global default) vs. custom rate or off.
- **Cache-worker failure alerting** — workflow opens a GitHub issue in `scorecastarr` on any failed cache run (deduplicated); auto-closes on next successful run.
- **CI/CD pipeline restructured** — three explicit workflows: `dev.yml` (push to `dev` → `:dev` tags), `beta.yml` (push to `main` → `:beta` tags), `release.yml` (version tag → `:latest`/versioned). All three files include a shared header documenting the full pipeline.

### Fixed
- **Channel group bleeding between scoreboards in Dispatcharr push** ([#2](https://github.com/jstevenscl/scorecastarr/issues/2)) — Three related bugs caused the wrong channel group to be sent when switching between scoreboards. Fixed with an explicit key-presence check, always-included group field in PATCH payloads, and capturing `_sbId` before the first `await` in both `executeQuickUpdate` and `executeWizardPush`.
- **F1/NASCAR/PGA card header height did not respond to Header/Status Size slider** ([#1](https://github.com/jstevenscl/scorecastarr/issues/1)) — `.card-header` had hardcoded padding; now uses `calc(var(--card-header-size, 11px) * 0.45)`.
- **F1/NASCAR/PGA sport label and Period/Time Size slider** ([#3](https://github.com/jstevenscl/scorecastarr/issues/3)) — sport labels now use `--card-header-size`; monospace stats data now uses `--card-period-size`.
- **Slider range caps** ([#3](https://github.com/jstevenscl/scorecastarr/issues/3)) — Title Size (22→36), Header/Status Size (24→36), Period/Time Size (24→36), Detail/Label Text Size (16→28), Driver/Player Name Size (28→40).
- **Logo gallery thumbnails** — changed from 36×36 square thumbnails to 140×27px landscape crops so logos are actually distinguishable.
- **NASCAR driver names** — NOAPS and Trucks series now scrape full names from Fox Sports standings after `cf.nascar.com/cacher/` started returning 403 from GitHub Actions.
- **NASCAR headshot cache format** — nascar-drivers cache now always writes clean slug-keyed entries, eliminating silent fallback to the legacy name-map path.
- **Tennis headshot 404 errors** — removed ESPN CDN fallback (it never hosted tennis images); tennis players now fetched via ESPN Core API.
- **`/api/stream/status` returning 404** — added proxy route in `api/app.py` forwarding to the stream manager.
- **NASCAR post-race detection** — `lapsDone` check, upcoming date guard, standings timestamp, and lap counter added to correctly identify post-race state.
- **F1 next-race pointer permanently stale** — was stuck on Miami GP; now fetches full Jolpica 2026 schedule on every cache run and advances automatically.
- **GHA build cache** — removed stale cache directives from web/nginx build in `beta.yml` that were causing failed builds.
- **Soccer `STATUS_FULL_TIME` not recognized as final** — `parseGame` only matched `STATUS_FINAL`; ESPN uses `STATUS_FULL_TIME` for completed soccer matches. Also added `STATUS_HALFTIME` → `isLive`.
- **`actions/checkout` deprecated** — upgraded cache-worker from v4.2.2 (Node.js 20) to v6.0.2.

### Changed
- Motor cache reseed no longer requires a container restart — call `POST /api/motor/reseed` to force an immediate re-read from the data branch.
- `:beta` Docker images now build on `main` push (previously built on every `dev` push, meaning untested code reached users).

---

## [v0.2.0-beta] — 2026-02-19

### Added
- **Channel numbering modes** — `auto` (sequential from base number) or `manual` (per-channel override in `config.json`)
- **Channel Profiles integration** — assign ScorecastArr channels to Dispatcharr Channel Profiles:
  - `all` — added to every profile (default, backward-compatible)
  - `none` — no profile assignment
  - `specific` — list explicit profile IDs; invalid IDs are warned and skipped
- **`config/config.json`** — persistent config file mounted into the API container; controls numbering mode, per-channel numbers/names/enabled flags, group name, and profile assignment; reloaded automatically on each 6-hour re-sync
- **GitHub Actions CI/CD** — two workflows:
  - `release.yml` — triggered by version tags (`v*.*.*`); builds multi-arch images (amd64 + arm64), pushes to ghcr.io with `:latest`/`:beta`/`:vX.Y.Z` tags, creates GitHub Release with notes
  - `beta.yml` — triggered on every push to `main`; auto-publishes `:beta` and `:beta-<sha>` images
- **nginx Dockerfile** — web service now has its own Dockerfile; scoreboard HTML baked in at build time with optional runtime volume override
- **Per-channel `enabled` flag** — individual channels can be disabled in `config.json` without removing them from the config

### Changed
- `docker-compose.yml` now pulls images from `ghcr.io` by default; local build retained as commented fallback
- `SCORECASTARR_TAG` env var controls which image tag to use (`latest`, `beta`, or pinned version)
- `GITHUB_OWNER` env var sets the ghcr.io namespace
- API container now mounts `./config` volume for persistent `config.json`
- Token refresh proactively runs every minute loop; refresh + re-auth fallback chain unchanged

### Fixed
- Config reload now happens on every 6-hour re-sync (not just startup)
- Profile ID validation warns and skips invalid IDs rather than crashing

---

## [v0.1.0-beta] — 2026-02-18 — Initial Release

### Added
- **Live sports scoreboard** — headless Chromium renders scoreboard HTML; FFmpeg encodes to HLS
- **7 channel variants** — All Sports, NFL, NBA, MLB, NHL, NCAA Basketball, NCAA Baseball
- **Dispatcharr auto-registration** — Python API authenticates via JWT, creates group + streams + channels on startup
- **6-hour re-sync** — channels re-registered automatically to survive Dispatcharr restarts
- **Docker Compose stack** — 4 containers: web (nginx), renderer (Chrome), ffmpeg, api
- **Named pipe architecture** — renderer feeds FFmpeg directly without temp files
- **HLS multi-output** — single FFmpeg process splits one video source to 7 simultaneous HLS streams
- **Mock ESPN data** — scoreboard falls back to rich mock data when ESPN API is blocked (CORS/sandbox)
- **Team browser** — sidebar modal with 2-column team grid, league tabs, live search, star toggles
- **Season awareness** — off-season dimming, preseason/postseason badges, empty off-season boards hidden
- **Per-sport ended games filters** — All / ⭐ First / ⭐ Only per league
- **3 layout modes** — Full, Grid, Ticker
- **Global filters** — All / Live / ⭐ Mine
- **Auto-refresh** — 60-second data refresh with live clock
- **Favorites system** — teams and leagues, persisted to localStorage with in-memory fallback

---

## Roadmap

### Planned for v0.3.0-beta
- Web-based config UI — edit channel numbers and profile assignment without touching JSON
- Off-season placeholder screens — custom "Season starts in X days" screen per sport
- Per-sport renderer instances — true isolated video per channel (opt-in, multi-Chrome mode)
- EPG injection — push XMLTV program guide data to Dispatcharr alongside channels

### Planned for v1.0.0 (stable)
- Full installation wizard
- Automated update detection and pull
- Health dashboard endpoint
- Unraid Community App template
- Portainer App template
