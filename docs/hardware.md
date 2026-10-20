# Hardware & wiring

## Parts list

- **diymore ESP32-C3 dev board with 0.42" OLED** (Amazon B0DYV1TJQS). An
  ESP32-C3 with a built-in 72x40 SSD1306 OLED, a BOOT and a RST button, and
  native USB-C (no separate USB-serial chip, so no driver is needed).
- **Hailege SW-420 vibration sensor module** (Amazon B07XJTNRD8). A spring
  vibration switch plus an LM393 comparator, a sensitivity potentiometer
  and a digital `DO` output.
- 3 female-to-female Dupont jumper wires
- A USB-C cable that carries data (not a charge-only cable)
- A Tapo smart plug (e.g. P100/P110) already exposed to Home Assistant,
  powering the dartboard light

## Wiring: SW-420 → ESP32-C3

<img width="536" height="326" alt="Screenshot 2026-10-20 at 9 39 05" src="https://github.com/user-attachments/assets/1437a8e8-64e0-495d-b326-1b858e2224f8" />

| SW-420 pin | ESP32-C3 pin | Notes |
|------------|--------------|-------|
| VCC        | 3V3          | The module works from 3.3V to 5V. Use 3.3V so `DO` stays at a safe level for the ESP32-C3's GPIOs. |
| GND        | GND          | |
| DO         | GPIO3        | Set as `sw420_pin` under `substitutions:` in `esphome/vibration-sensor.yaml` |

Already wired on the board, so leave these pins free:

| Function    | GPIO |
|-------------|------|
| OLED SDA    | GPIO5 |
| OLED SCL    | GPIO6 |
| BOOT button | GPIO9 (used as the threshold button) |

To move `DO` to another pin, pick a plain GPIO such as 0, 1, 4, 7, 10, 20 or
21. Avoid GPIO2, GPIO8 and GPIO9, which are strapping pins.

## The display and the BOOT button

The status screen shows:

```
+------------------------+
| WiFi ●       HA ●      |  filled dot = connected, ring = not connected
|        hits: 12        |  shows an inverted "HIT!" for 2s on each hit,
|                        |  or a blinking "connecting" while WiFi is down
| T3 [██████       ]     |  threshold + live pulse level (full = would trigger)
+------------------------+
```

The board has two buttons, **BOOT** and **RST**, and they're easy to mix up.
RST reboots the board, which looks like the WiFi reconnecting because the
screen shows "connecting" again. If a press doesn't open the threshold
screen, you pressed RST, so use the other button.

The BOOT button sets the hit threshold:

- **First press** opens the threshold screen and changes nothing.
- **Short press** while the screen is open: threshold +1. Past the maximum
  (20) it wraps back to 1.
- **Long press** (1 second or more) while the screen is open: threshold −1.
- The display returns to the status screen 3 seconds after the last press.

The threshold is saved across reboots. It is also exposed in Home Assistant
as `number.dartboard_impact_sensor_hit_threshold` (on the device page under
Configuration), so you can change it from there as well.

The BOOT button also selects download mode when held during power-up or
reset. That only matters at boot, so normal presses are safe.

## The two sensitivity controls

1. **The SW-420 potentiometer (hardware, coarse).** It sets the comparator
   level. Turn it until the module's `DO` LED stays steady while the board
   is still and flickers when you tap the dartboard.
2. **The hit threshold (software, fine).** This is the number of SW-420
   pulses within a 100ms window that counts as a hit. A real dart makes the
   spring chatter for a while and gives several pulses. A small bump or
   noise gives only one or two. Raise the threshold to ignore light knocks
   and lower it if real hits are missed.

The detection counts pulses, so it works the same whether the module's
`DO` idles high or low.

## Mounting

The SW-420's spring switch is most sensitive along the length of its
metal can. Mount the module firmly, using a screw or hot glue rather than
hanging wires:

- directly on the back of the dartboard, or
- on the surround or cabinet frame, if the board is mounted solidly enough
  for hits to carry through to the frame

The ESP32-C3 board can sit a short wire length away, somewhere you can see
the display and reach the BOOT button. Test on jumper wires first. Placement
noticeably affects sensitivity.
