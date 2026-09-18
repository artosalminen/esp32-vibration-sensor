# Hardware & wiring

## Parts list

- An ESP8266 (AZ-Delivery NodeMCU or Wemos D1 Mini) **or** an ESP32
  (AZ-Delivery ESP32 Dev Kit C) board
- A CJMCU-ADXL345 accelerometer breakout board
- Dupont jumper wires (female-to-female if going board-to-board, or
  male-to-female depending on your headers) and a small breadboard for
  prototyping
- A USB cable capable of data transfer (not a charge-only cable) to match
  your board's port (micro-USB on most NodeMCU/D1 Mini boards, USB-C on
  many ESP32 dev kits)
- A Tapo smart plug (e.g. P100/P110) already exposed to Home Assistant,
  powering the dartboard light

## Wiring: CJMCU-ADXL345 → ESP board

| ADXL345 pin | Connects to                              | Notes |
|-------------|-------------------------------------------|-------|
| VCC         | 3V3                                        | The breakout has an onboard regulator but feed it 3.3V from the ESP board |
| GND         | GND                                        | |
| SDA         | An I2C-capable GPIO (see table below)      | Configured as `sda:` in `esphome/vibration-sensor.yaml` |
| SCL         | An I2C-capable GPIO (see table below)      | Configured as `scl:` in `esphome/vibration-sensor.yaml` |
| SDO         | GND                                        | Sets the I2C address to `0x53` (the address used in the config). Tying it to VCC instead gives address `0x1D` — if you do that, change the address in the YAML too. |
| CS          | VCC                                        | Keeps the chip in I2C mode rather than SPI mode |

### Default GPIO pins used in the config

| Board                      | SDA    | SCL    |
|-----------------------------|--------|--------|
| NodeMCU (ESP8266)           | GPIO4 (labelled "D2") | GPIO5 (labelled "D1") |
| ESP32 Dev Kit C              | GPIO21 (default SDA)  | GPIO22 (default SCL)  |

If you wire it to different pins, just update the `sda:`/`scl:` values under
the `i2c:` block in `esphome/vibration-sensor.yaml`.

## Mounting

Because the sensor works off total acceleration magnitude rather than a
specific axis, orientation doesn't matter much — mount it wherever it will
reliably pick up the impact vibration:

- Directly on the back of the dartboard itself, or
- On the surround/cabinet frame, if the board is mounted solidly enough
  that hits transmit through to the frame

Use a breadboard and jumper wires first and confirm reliable detection
before permanently mounting or soldering anything — placement noticeably
affects sensitivity, and you may need to move the sensor or retune the
threshold in `esphome/vibration-sensor.yaml` after testing.
