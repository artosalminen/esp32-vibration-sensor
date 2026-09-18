# Building and flashing the firmware

Only the very first flash needs a USB cable — the chip is blank, so
ESPHome can't push firmware to it over WiFi yet. Every flash after that
(tuning the threshold, tweaking anything) can go over OTA straight from
the ESPHome dashboard, no cable required.

## 1. Pick your ESPHome tool

- If your Home Assistant is HA OS or Supervised (Settings > Add-ons >
  Add-on Store exists), install the official **ESPHome** add-on. It gives
  you a dashboard, manages `secrets.yaml` for you, and later lets you push
  updates over WiFi.
- If you run HA Container/Core (no Supervisor), or just want the quickest
  path, use **ESPHome Web** (esphome.io/web) in Chrome or Edge instead — it
  flashes straight from the browser over USB, no install required. Note
  that ESPHome Web has no secrets file, so you'd inline the real values
  directly into the YAML instead of using `esphome/secrets.yaml`.

## 2. Wire the hardware

Follow `hardware.md` in this folder before flashing, so you can verify the
sensor works right after the first flash.

## 3. Fill in your secrets

Copy `esphome/secrets.yaml.example` to `esphome/secrets.yaml` and fill in
your WiFi SSID/password, an API encryption key, and an OTA password (the
example file explains how to generate the key). If you're using the
ESPHome add-on/dashboard, its "New Device" wizard can generate the API key
for you automatically.

## 4. Create the device and load the config

In the ESPHome dashboard or ESPHome Web, choose "New Device", give it a
name, then discard the wizard's generated YAML and paste in the contents
of `esphome/vibration-sensor.yaml`. Uncomment whichever board block
matches your AZ-Delivery board — the `esp8266`/`nodemcuv2` block, or the
`esp32`/`esp32dev` block — and comment out the other one.

## 5. Connect via USB and flash

Plug the ESP board in with a proper data USB cable. If the serial port
doesn't show up, install the board's USB driver first — AZ-Delivery boards
typically use either a CP2102 or CH340 chip, so grab the matching Silicon
Labs CP210x or CH340 driver for your OS. Click **Install > "Plug into this
computer"**, pick the serial port, and wait roughly 1-2 minutes for it to
compile and flash. If it gets stuck at "Connecting...", hold the board's
FLASH/BOOT button down for a couple of seconds right as it starts
connecting.

## 6. Verify it boots and joins WiFi

Right after flashing, the dashboard shows live logs over the same USB
connection. Check for:

- The I2C scan line confirming address `0x53` was found
- A WiFi connected message
- The API/OTA services starting

Within a minute or two, Home Assistant should auto-discover it under
**Settings > Devices & Services** as a new ESPHome integration — click
Configure and paste in the encryption key from your secrets.

## 7. Calibrate, then go wireless

See `home-assistant.md` for calibrating the impact threshold once the
device is live in Home Assistant. From here on you can reflash over
WiFi/OTA straight from the ESPHome dashboard — the USB cable is only
needed again if the device loses its WiFi connection entirely.

## Troubleshooting

- **Serial port never appears**: missing USB driver (see step 5).
- **Stuck at "Connecting..."**: hold FLASH/BOOT during the connect attempt.
- **I2C scan doesn't show 0x53**: check wiring, especially SDA/SCL not
  swapped, and that SDO is tied to GND (address 0x53) or VCC (address
  0x1D, matching the address in the YAML).
- **Impact sensor never trips / trips constantly**: recalibrate the
  threshold in `esphome/vibration-sensor.yaml` — see `home-assistant.md`.
