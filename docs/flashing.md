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
of `esphome/vibration-sensor.yaml`. It already targets the diymore
ESP32-C3 board (`esp32-c3-devkitm-1`), so no board selection is needed.
The first compile needs internet access, because ESPHome downloads the
Roboto font from Google Fonts for the display.

## 5. Connect via USB and flash

Plug the board in with a USB-C data cable. The ESP32-C3 has USB built
in, so no driver is needed. It shows up as a "USB JTAG/serial debug unit"
(on macOS `/dev/cu.usbmodem…`). Click **Install > "Plug into this
computer"**, pick the serial port, and wait a few minutes for it to compile
and flash.

If no port shows up or flashing gets stuck at "Connecting...", put the
chip into download mode: hold **BOOT**, tap **RST**, then release
**BOOT**. Flash again, then press **RST** once it's done to start the new
firmware. A blank chip, or one running firmware that turned off USB, often
needs this on the first flash.

## 6. Verify it boots and joins WiFi

Right after flashing, the dashboard shows live logs over the same USB
connection. Check for:

- The OLED lighting up. The top row shows a WiFi ring that becomes a
  filled dot once connected, and the same for HA once Home Assistant
  connects.
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

- **Serial port never appears**: try another cable, since many USB-C
  cables are charge-only. Then use the BOOT + RST download-mode sequence
  from step 5.
- **Display stays dark**: power-cycle the board by unplugging and
  replugging it. Some of these OLEDs only initialise cleanly from a cold
  start.
- **Text is cut off or shifted**: a few batches of these boards map the
  72x40 window differently. Try `offset_x:` / `offset_y:` on the `display:`
  block, or as a last resort `model: "SH1106 128x64"` and draw inside the
  visible window.
- **Impact sensor never trips / trips constantly**: adjust the SW-420
  potentiometer, then the hit threshold. See `hardware.md` and
  `home-assistant.md`.
