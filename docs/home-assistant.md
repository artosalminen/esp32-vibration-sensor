# Home Assistant: discovery, calibration, and the automation

## 1. Discovery

Once the device has flashed and joined WiFi (see `flashing.md`), Home
Assistant should auto-discover it under **Settings > Devices & Services**
as a new ESPHome integration. Click **Configure** and paste in the API
encryption key from `esphome/secrets.yaml`. Paste only the value, without
the surrounding quotes. It is 44 characters and ends in `=`. On macOS you
can copy it straight to the clipboard with:

```sh
sed -nE 's/^dartboard_api_key: *"?([^"]+)"?.*/\1/p' esphome/secrets.yaml | tr -d '\n' | pbcopy
```

You'll then have these entities:

- `binary_sensor.dartboard_hit`: "on" briefly whenever a hit is detected
- `sensor.vibration_pulses`: the highest SW-420 pulse count seen in any
  100ms window during the last second (used for calibration)
- `number.hit_threshold`: the hit threshold, also adjustable from the
  board's BOOT button

(Home Assistant usually prefixes these with the device name, e.g.
`binary_sensor.dartboard_impact_sensor_dartboard_hit`. Check the real IDs
under **Settings > Devices & Services > Entities** and adjust the
automation YAML in `homeassistant/dartboard_light.yaml` to match.)

## 2. Calibrate

Set the hardware first. Turn the SW-420 potentiometer until the module's
`DO` LED is steady while everything is still and flickers when you tap
the dartboard.

Next, open the **Vibration Pulses** sensor's history in Home Assistant, or
watch the pulse bar at the bottom of the board's display. Then:

- Leave the room quiet for a bit, and walk past or close a door, to see
  the background level
- Throw a handful of darts and note how many pulses a real hit gives

Choose a threshold above the background level and below a typical hit.
Set it with the BOOT button (press once to open the threshold screen, short
press for +1, long press for −1) or with the Hit Threshold entity in Home
Assistant. It takes effect immediately and is kept across reboots, so no
reflash is needed. On the display, the pulse bar fills completely whenever
a window reaches the threshold.

If even a gentle hit fills the bar many times over, the potentiometer is
set too sensitive. If real hits barely register, it isn't sensitive
enough.

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
