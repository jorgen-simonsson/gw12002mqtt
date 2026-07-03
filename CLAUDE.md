# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`gw12002mqtt` is a Go application that polls an ECOWITT GW1200 weather gateway over its local HTTP API and publishes all sensor readings as a JSON payload to a local MQTT broker. No cloud connection required.

**Stack:** Go 1.24, `paho.mqtt.golang`, `godotenv`. Single-file implementation in `main.go`.

## Commands

```sh
go build ./...          # compile
go run .                # run locally (reads .env if present)
go test ./...           # run tests
docker compose up -d    # build image and run in container
docker compose logs -f  # follow logs
```

## Architecture

Everything lives in `main.go`. The data flow is:

1. `main()` loads config, connects to MQTT, fires an immediate poll, then loops on a ticker.
2. `poll()` calls `fetchLiveData()` (HTTP GET to `http://<GW1200_HOST>/get_livedata_info`) then `publishJSON()`.
3. `buildReadings()` maps raw API fields to named `reading{Value, UnitOfMeasure}` structs via `parseReading()`.
4. The full readings map is marshalled to JSON and published to `<MQTT_TOPIC>/json`.

### Key domain detail: field ID mapping

The GW1200 HTTP API returns sensor values keyed by hex IDs (e.g. `"0x02"`) from the Ecowitt HTTP API Interface Protocol. The `fieldNames` map in `main.go` translates these to human-readable MQTT sub-topic names. Decimal string IDs (`"3"`, `"5"`) are calculated values the gateway computes itself (feel-like, VPD), not raw sensor reads.

`parseReading()` handles two formats: values with an embedded unit (`"54%"`, `"0.5 m/s"`) and values with a separate `unit` field.

### Sensor sources

- `common_list` / `rain` arrays — standard sensors keyed by hex ID
- `wh25` array — indoor temp/humidity and barometric pressure from the gateway's built-in sensor
- `piezoRain` array — WS90 ultrasonic/piezo sensor data including battery/voltage status

## Configuration

Copy `.env.example` to `.env`:

```env
GW1200_HOST=192.168.x.x   # required
MQTT_HOST=192.168.x.y     # required
MQTT_TOPIC=weather/gw1200  # required; /json is appended automatically
MQTT_PORT=1883             # default 1883
MQTT_USER=
MQTT_PASSWORD=
POLL_INTERVAL=5            # seconds, default 5
```

`GW1200_HOST`, `MQTT_HOST`, and `MQTT_TOPIC` are required — the app exits immediately if any are missing.

## API Reference

- [HTTP API Interface Protocol v1.0.6](https://oss.ecowitt.net/uploads/20260114/HTTP%20API%20interface%20Protocol%20(Generic)-(V1.0.6-2026-1-14).pdf) — source of truth for field IDs used in `fieldNames`
- [TCP API Interface Protocol v1.7.0](https://oss.ecowitt.net/uploads/20260112/TCP%20API%20Interface%20Communication%20Protocol%20V1.7.0.pdf) — binary protocol on port 45000 (not currently used)
