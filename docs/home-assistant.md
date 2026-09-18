# Home Assistant: discovery, calibration, and the automation

## 1. Discovery

Once the device has flashed and joined WiFi (see `flashing.md`), Home
Assistant should auto-discover it under **Settings > Devices & Services**
as a new ESPHome integration. Click **Configure** and paste in the API
encryption key from `esphome/secrets.yaml`. You'll then have two new
entities:

- `sensor.dartboard_impact_magnitude` — the live acceleration magnitude, in g
- `binary_sensor.dartboard_hit` — "on" briefly whenever a hit is detected

(Exact entity IDs may differ slightly depending on how Home Assistant
slugifies the device/sensor names — check under **Settings > Devices &
Services > Entities** and adjust the automation YAML in
`homeassistant/dartboard_light.yaml` to match.)

## 2. Calibrate the impact threshold

Open `sensor.dartboard_impact_magnitude` in Home Assistant and watch it
live (the entity's "history"/logbook graph works well for this) while you:

- Leave the room quiet for a bit, to see the resting noise floor (should
  hover around 1.0-1.5g)
- Throw a handful of darts and note how high the magnitude spikes on a hit

Pick a threshold clearly above the noise floor and clearly below a real
hit, and set it in `esphome/vibration-sensor.yaml`:

```yaml
lambda: |-
  return id(impact_magnitude).state > 3.0;  // <- adjust this number
```

Reflash (over OTA at this point, no USB needed) and confirm
`binary_sensor.dartboard_hit` toggles "on" reliably on real throws and
stays "off" otherwise. If it's too sensitive, raise the threshold; if it
misses real hits, lower it.

Once you're happy with it, you can optionally set `internal: true` on the
magnitude sensor in the YAML to hide it from your entity list — it's only
really needed during calibration.

## 3. The automation

The light logic itself is deliberately simple and lives entirely in Home
Assistant (not on the ESP), using a `timer` helper:

1. A hit turns the light on and (re)starts a 30-minute timer.
2. Every subsequent hit restarts that timer, so the light stays on as long
   as you keep playing.
3. If the timer ever finishes (30 minutes pass with no hits), the light
   turns off.

The ready-to-adapt version of this is in
`homeassistant/dartboard_light.yaml` — see the comments at the top of that
file for how to create the timer helper and where to swap in your real
entity IDs (the binary sensor, your Tapo switch entity, and the timer).
