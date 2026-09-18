# ESP32/ESP8266 Vibration Sensor for Home Assistant

A cheap ESP8266/ESP32 board plus an ADXL345 accelerometer, turned into a Home
Assistant vibration/impact sensor via ESPHome. Built here to switch a
dartboard light on the moment you start throwing and off again after 30
minutes of no activity, but the sensor itself is generic: anything that
needs an "impact just happened" trigger can use it.

## Why this exists

ESPHome has no official ADXL345 component, and the old "custom component"
trick (`platform: custom` + a hand-written `.h` file) was removed from
ESPHome in 2025. This project works around both by reading the ADXL345's
raw I2C registers directly inside an ESPHome template sensor lambda, using
ESPHome's built-in `i2c::I2CDevice` class. No external libraries, no
`external_components`, just the one YAML file in `esphome/`.

## How it works

```
 CJMCU-ADXL345 --I2C--> ESP8266/ESP32 --WiFi/ESPHome API--> Home Assistant --> Tapo smart plug (light)
```

1. The ESP polls the ADXL345 over I2C every 50ms and publishes the total
   acceleration magnitude (√(x²+y²+z²)) as a sensor. At rest it reads
   ~1.0g; a dart hitting the board causes a sharp spike above that.
2. A template binary sensor (`Dartboard Hit`) fires "on" whenever the
   magnitude crosses a threshold you calibrate, with a short debounce so
   Home Assistant sees one clean pulse per hit.
3. A Home Assistant automation turns the light on and (re)starts a 30
   minute timer whenever a hit comes in; a second automation turns the
   light off when that timer runs out with no further hits.

## Repo layout

- `esphome/vibration-sensor.yaml` — the ESPHome device config (impact
  magnitude sensor + debounced binary sensor)
- `esphome/secrets.yaml.example` — template for the secrets ESPHome needs;
  copy to `secrets.yaml`, fill in your own values, never commit the real
  file (it's gitignored)
- `docs/hardware.md` — parts list and wiring
- `docs/flashing.md` — how to build/compile and flash the firmware, first
  time over USB and every time after that over WiFi/OTA
- `docs/home-assistant.md` — connecting the device to Home Assistant,
  calibrating the impact threshold, and the automation logic
- `homeassistant/dartboard_light.yaml` — the timer helper + two
  automations, ready to adapt to your own entity IDs

## Hardware used

- AZ-Delivery ESP8266 (NodeMCU/D1 Mini) or ESP32 Dev Kit
- CJMCU-ADXL345 accelerometer breakout
- A Tapo smart plug (e.g. P100/P110) exposed to Home Assistant, powering
  the dartboard light

See `docs/hardware.md` for wiring and `docs/flashing.md` to get firmware
onto the board.

## Status / calibration note

The impact threshold in `esphome/vibration-sensor.yaml` (`> 3.0` g) is a
starting point, not a guarantee — it depends on where you mount the
accelerometer and how solid the board/cabinet is. Watch the
`Dartboard Impact Magnitude` entity live in Home Assistant while throwing a
few darts and while the room is quiet, then tune it. Details in
`docs/home-assistant.md`.

## License

MIT — see [LICENSE](LICENSE).
