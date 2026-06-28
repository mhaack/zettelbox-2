# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ESPHome Commands

```bash
# Compile and flash (preferred — compile must run before upload on first flash)
esphome run <device-folder>/<device>.yaml

# Flash only (requires prior compile)
esphome upload <device-folder>/<device>.yaml

# Monitor device logs
esphome logs <device-folder>/<device>.yaml

# Validate config without compiling
esphome config <device-folder>/<device>.yaml
```

## Project Structure

Each device lives in its own subfolder:

```
zettelbox-2/
  zettelbox-2.yaml       # Main device config (ESP32dev, waveshare 2.70in e-ink)
  pirsch-analytics.yaml  # Reusable package: Pirsch API auth + stat fetch logic
secrets.yaml             # All secrets (WiFi, API passwords, Pirsch credentials)
dist/                    # Build output (generated, gitignore this)
```

## Architecture Patterns

**Packages**: `pirsch-analytics.yaml` is an ESPHome package included via `!include`. It defines globals, sensors, and scripts. The consuming device config (`zettelbox-2.yaml`) declares substitution variables (`pirsch_domain_id`, `pirsch_client_id`, `pirsch_client_secret`) that the package references via `${var}`.


**Build paths**: Each config sets `build_path: ../dist`. ESPHome resolves this relative to its internal `.esphome/` dir, so output lands in `dist/` inside each device folder (e.g. `zettelbox-2/dist/`).

**Secrets**: All credentials live in `secrets.yaml` at repo root, referenced via `!secret <key>`. Each device subfolder has a symlink `secrets.yaml → ../secrets.yaml`. API uses encryption key (`api_encryption_key`), not password auth (removed in ESPHome 2026.1.0).

**Pirsch auth flow** (`pirsch-analytics.yaml`):
1. `pirsch_fetch_stats` (orchestrator) — checks token expiry, re-auths if needed
2. `pirsch_authenticate` — POSTs to Pirsch API, stores bearer token + expiry timestamp
3. `pirsch_get_stats` — fetches all-time stats (from 2020-01-01 to today)
4. `pirsch_get_stats_today` — fetches today's stats only
Token auto-refreshes when <5 minutes remaining. 401 responses clear the token to force re-auth.

**zettelbox-2 layout**: Waveshare 2.70in rotated 90° = 264×176px canvas. Multi-page display cycling every 30s — weather, energy, pool, climate, car, waste, website stats, Claude usage, system info.
