# gw12002mqtt

A Go application that polls an ECOWITT GW1200 weather gateway over the local HTTP API and publishes all sensor readings as a JSON payload to a local MQTT broker. No cloud connection required.

## How it works

The app polls `http://<GW1200_HOST>/get_livedata_info` at a configurable interval (default 5 seconds) and publishes a single JSON message to `<MQTT_TOPIC>/json`.

Each property in the JSON has the form:

```json
{
  "outdoor_temp":  { "value": 22.4, "unitOfMeasure": "C" },
  "wind_speed":    { "value": 0.5,  "unitOfMeasure": "m/s" },
  "rain_daily":    { "value": 0.2,  "unitOfMeasure": "mm" }
}
```

### Published properties

| Property | Source | Unit |
|---|---|---|
| `outdoor_temp`, `dew_point`, `feels_like` | Main station | C |
| `outdoor_humidity` | Main station | % |
| `vapor_pressure_deficit` | Calculated by gateway | kPa |
| `indoor_temp`, `indoor_humidity` | Gateway built-in sensor | C / % |
| `pressure_absolute`, `pressure_relative` | Gateway built-in sensor | hPa |
| `wind_direction`, `wind_speed`, `wind_gust`, `wind_speed_10min_avg` | WS90 ultrasonic | ° / m/s |
| `solar_radiation` | WS90 | W/m² |
| `uv_index` | WS90 | — |
| `rain_event`, `rain_rate`, `rain_daily`, `rain_weekly`, `rain_monthly`, `rain_yearly` | Tipping bucket | mm |
| `piezo_rain_*` | WS90 piezo sensor | mm |
| `piezo_raining` | WS90 piezo sensor | 0/1 |
| `lightning_bearing` | WS90 lightning detector | ° |
| `ws90_battery`, `ws90_voltage`, `ws90_cap_volt` | WS90 power status | — / V |

## Configuration

Copy `.env.example` to `.env` and fill in your values:

```env
GW1200_HOST=192.168.1.x      # IP or hostname of the GW1200
MQTT_HOST=192.168.1.y        # IP or hostname of the MQTT broker
MQTT_PORT=1883               # default 1883
MQTT_USER=
MQTT_PASSWORD=
MQTT_TOPIC=weather/gw1200    # base topic; /json is appended
POLL_INTERVAL=5              # seconds, default 5
```

## Running

```sh
docker compose up -d
```

Check logs:

```sh
docker compose logs -f
```

Verify data is arriving:

```sh
mosquitto_sub -h <MQTT_HOST> -u <MQTT_USER> -P <MQTT_PASSWORD> \
  -t "weather/gw1200/json" -v
```

## ECOWITT GW1200 Resources

### Device

- [GW1200 Product Page](https://shop.ecowitt.com/products/gw1200)
- [GW1200 Manual (PDF)](https://oss.ecowitt.net/uploads/20250331/GW1200.pdf) — updated 2025-03-31
- [Quick Start Guide (PDF)](https://oss.ecowitt.net/uploads/20250411/Quick%20start.pdf)
- [WS View Plus & Web UI Manual (PDF)](https://oss.ecowitt.net/uploads/20250408/WS%20View%20Plus%20%26%20Web%20UI%20Manual%20(Generic).pdf)
- [Firmware Upgrade Manual (PDF)](https://oss.ecowitt.net/uploads/20241211/Firmware%20Upgrade%20Manual(Generic).pdf)

### API Documentation

- [HTTP API Interface Protocol v1.0.6 (PDF)](https://oss.ecowitt.net/uploads/20260114/HTTP%20API%20interface%20Protocol%20(Generic)-(V1.0.6-2026-1-14).pdf) — LAN poll via `get_livedata_info`
- [TCP API Interface Protocol v1.7.0 (PDF)](https://oss.ecowitt.net/uploads/20260112/TCP%20API%20Interface%20Communication%20Protocol%20V1.7.0.pdf) — binary protocol on port 45000
- [Ecowitt Cloud API Documentation](https://doc.ecowitt.net/web/#/)
