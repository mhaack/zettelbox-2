# zettelbox-2

A DIY e-ink notepad box: a bigger Waveshare 2.7" display, a CNC-cut plywood case, and more Home Assistant data on the desk.

ESP32 with a [Waveshare 2.70" e-ink display](https://www.waveshare.com/2.7inch-e-paper-hat.htm), cycling through 9 pages every 30 seconds:

| Page | Content |
|------|---------|
| 1 | Weather — condition icon, temperature, wind |
| 2 | Energy — solar, battery, consumption, grid import/export |
| 3 | Pool — pump/solar valve state, water & heater temps |
| 4 | Climate — indoor/outdoor temp, humidity, particulates |
| 5 | Car — Polestar 4 odometer, charge level, range, charging |
| 6 | Waste — next bin collection |
| 7 | Website — [markus-haack.com](https://markus-haack.com) visitor & page view stats via Pirsch |
| 8 | Claude — session & weekly usage, reset timers |
| 9 | System — WiFi signal, IP, uptime, time |

**Hardware:** ESP32dev · Waveshare 2.70in e-paper (264×176px, rotated 90°) · SPI

## Project Structure

```
zettelbox-2/
  zettelbox-2.yaml        # Main device config
  pirsch-analytics.yaml   # Package: Pirsch API auth + stats fetch
secrets.yaml              # All credentials (not committed)
```

## Setup

**Prerequisites:** [ESPHome CLI](https://esphome.io/guides/getting_started_command_line.html) and a `secrets.yaml` at the repo root.

```yaml
# secrets.yaml
wifi_ssid: "..."
wifi_password: "..."
ap_ssid: "..."
ap_password: "..."
api_encryption_key: "..."
pirsch_domain_id: "..."
pirsch_client_id: "..."
pirsch_client_secret: "..."
```

Each device folder has a `secrets.yaml` symlink pointing to the root file:

```bash
ln -s ../secrets.yaml zettelbox-2/secrets.yaml
```

**Compile & flash:**

```bash
esphome run zettelbox-2/zettelbox-2.yaml
```

**Flash only** (after initial compile):

```bash
esphome upload zettelbox-2/zettelbox-2.yaml
```

**Monitor logs:**

```bash
esphome logs zettelbox-2/zettelbox-2.yaml
```

## Pirsch Analytics Integration

`pirsch-analytics.yaml` is a reusable ESPHome package that handles OAuth2 authentication and stats fetching from the [Pirsch API](https://docs.pirsch.io/api-sdks/api).

- Token auto-refreshes when less than 5 minutes remaining
- 401 responses clear the token to force re-authentication
- Stats update every 30 minutes and on boot
- Fetches both all-time stats (from 2020-01-01) and today's stats separately
