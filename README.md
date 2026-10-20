# ESP32-C3 Vibration Sensor for Home Assistant

A diymore ESP32-C3 board with a built-in 0.42" OLED plus an SW-420
vibration switch, turned into a Home Assistant vibration/impact sensor
with ESPHome. Built here to switch a dartboard light on the moment you
start throwing and off again after 30 minutes of no activity, but the
sensor itself is generic. Anything that needs an "impact just happened"
trigger can use it.

The on-board display shows whether the device is connected to WiFi and
Home Assistant, flashes **HIT!** when vibration is detected, and lets you
set the detection threshold with the board's BOOT button.

## How it works

```
 SW-420 --DO pulses--> ESP32-C3 (+OLED) --WiFi/ESPHome API--> Home Assistant --> Tapo smart plug (light)
```

1. The SW-420's spring switch chatters when the board is hit and puts out
   a burst of pulses on its `DO` pin. The ESP counts them in 100ms
   windows.
2. A window with at least **Hit Threshold** pulses counts as a hit. The
   `Dartboard Hit` binary sensor goes "on" for 500ms, so Home Assistant
   sees one clean pulse per hit.
3. The threshold can be set from the BOOT button and the display, or from
   Home Assistant, and it is kept across reboots. The module's
   potentiometer is a second, coarser sensitivity control.
4. Home Assistant automations turn the board light, display and mood light
   on after three hits within 30 seconds, and turn the board light and
   display off again after 15 minutes without a hit.

## Repo layout

- `esphome/vibration-sensor.yaml`: the ESPHome device config (pulse
  counting, hit detection, threshold, display and button UI)
- `esphome/secrets.yaml.example` — template for the secrets ESPHome needs;
  copy to `secrets.yaml`, fill in your own values, never commit the real
  file (it's gitignored)
- `docs/hardware.md` — parts list and wiring
- `docs/flashing.md` — how to build/compile and flash the firmware, first
  time over USB and every time after that over WiFi/OTA
- `docs/home-assistant.md` — connecting the device to Home Assistant,
  calibrating the impact threshold, and the automation logic
- `homeassistant/dartboard_light.yaml`: the two light automations, ready to
  paste into Home Assistant

## Hardware used

- [diymore ESP32-C3 dev board with 0.42" OLED](https://www.amazon.de/dp/B0DYV1TJQS)
- [Hailege SW-420 vibration sensor module](https://www.amazon.de/dp/B07XJTNRD8)
- A Tapo smart plug (e.g. P100/P110) exposed to Home Assistant, powering
  the dartboard light

See `docs/hardware.md` for wiring and the display/button UI, and
`docs/flashing.md` to get firmware onto the board.

## Status / calibration note

The default threshold (3 pulses per 100ms) is a starting point, not a
guarantee. The right value depends on the potentiometer setting, where
the module is mounted, and how solid the board or cabinet is. Watch the
pulse bar on the display, or the `Vibration Pulses` entity in Home
Assistant, while throwing a few darts and while the room is quiet, then
adjust it with the BOOT button. Details are in `docs/home-assistant.md`.

## License

MIT — see [LICENSE](LICENSE).
