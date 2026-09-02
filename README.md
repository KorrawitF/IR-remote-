# IR-remote-

Wi-Fi to infrared bridge for an ESP8266: it receives commands over the network and replays captured IR codes at a TV and an air conditioner, turning two ordinary appliances into network-controllable ones.

Two independent sketches are included, differing only in how commands arrive.

| File | Transport |
| --- | --- |
| `ESP8266_IRremote.ino` | **Blynk** — commands come from the Blynk app on virtual pin V0 |
| `ESP8266_IRremote_2.ino` | **Bare HTTP** — the ESP runs a `WiFiServer` on port 80 and matches the requested path |

## How it works

Each remote button was captured once as a raw timing array (the microsecond on/off durations of the IR burst) and embedded in the sketch. On a matching command the sketch calls `remote.sendRaw(<array>, <length>, 38)` to replay it at 38 kHz through an IR LED on **GPIO4 (D2)**.

The TV codes are 75-element arrays; the air-conditioner codes are 199 elements, since AC remotes transmit their entire state — mode, temperature, fan — in every burst rather than a single button code.

Relative volume steps are implemented by repeating the volume burst in a loop with a one-second delay, which is why `Volume+3`, `+5`, and `+10` exist as separate commands.

## Commands

Both sketches accept the same names (as a Blynk string on V0, or as an HTTP path such as `http://<esp-ip>/Temp25`):

**TV** — `TurnOn`, `VolumeUp`, `VolumeDown`, `Volume+3`, `Volume+5`, `Volume+10`, `Volume-3`, `Volume-5`, `Volume-10`, `HDMI`

**Air conditioner** — `On`, `Off`, `Temp24`, `Temp25`, `Temp26`, `Temp27`, `Temp28`, `Swing`, `Mode_DRY`, `LED_Off`

## Hardware

- ESP8266 board (NodeMCU / Wemos D1 mini)
- IR LED on GPIO4 (D2), driven through a transistor and current-limiting resistor for usable range
- Line of sight to the appliances

## Requirements

Arduino IDE with the ESP8266 board package and:

- `IRremoteESP8266`
- `Blynk` (for the Blynk sketch only)

## Configuration

Fill in the blank credentials at the top of whichever sketch you use:

```cpp
char auth[] = "";   // Blynk auth token (Blynk sketch only)
char ssid[] = "";   // Wi-Fi SSID
char pass[] = "";   // Wi-Fi password
```

## Notes

- The IR codes are specific to the remotes they were captured from. For different appliances, recapture them with an IR receiver and an `IRrecvDumpV2`-style sketch and replace the arrays.
- In the HTTP sketch, `Run()` is called once from `setup()` and `loop()` is empty, so it handles a single client per boot — moving the call into `loop()` makes it serve continuously.
- The HTTP sketch has no authentication; keep it on a trusted LAN.
