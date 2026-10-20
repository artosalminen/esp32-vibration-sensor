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
automations in `homeassistant/` to match.)

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

## 3. The automations

The light logic lives entirely in Home Assistant, not on the ESP, and needs
no helpers:

1. **Lights on:** three hits within 30 seconds turn on the board light, the
   display and the mood light. One knock or bump isn't enough.
2. **Lights off:** after 15 minutes with no hits, the board light and the
   display turn off. The mood light stays on. This also applies if the
   lights were switched on by hand and nobody played.

Each is its own file, ready to paste into the automation editor's YAML
mode: `homeassistant/darts_lights_on.yaml` and
`homeassistant/darts_lights_off.yaml`. Replace the placeholder
`light.darts_mood_light` with your mood light's entity ID, and check the
other IDs against **Settings > Devices & Services > Entities**.
