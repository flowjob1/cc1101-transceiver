# cc1101-transceiver
ESPHome config for CC1101 RF Transceiver

A complete ESPHome configuration for the **CC1101 Sub-1 GHz RF Transceiver** with an **ESP32-WROOM**, featuring signal learning, replay, and Home Assistant integration.

Learn and replay 433 MHz RF signals from remotes, blinds, outlets, and more — directly from Home Assistant.

---

## Features

- **Learn RF signals** — Capture signals from any 433 MHz remote
- **Replay signals** — Retransmit learned signals on demand (single or x3)
- **Single-line log output** — Captured codes logged as copyable arrays
- **Dual pin wiring** — Simultaneous TX/RX without mode switching
- **Home Assistant integration** — Full control via buttons, switches, and sensors
- **Web interface** — Built-in web server on port 80 for standalone use

---

## Hardware Requirements

| Component | Description |
|-----------|-------------|
| ESP32-WROOM DevKit | Any ESP32 development board |
| CC1101 Module | 433 MHz version (common green or blue PCB modules) |
| Jumper Wires | 8 female-to-female dupont wires |
| Antenna | 433 MHz antenna (17.3 cm wire or SMA antenna) |

---

## Wiring Diagram

### Dual Pin Mode (Recommended)

Uses separate pins for TX and RX — no mode switching needed.

```
 ESP32-WROOM                CC1101 Module
 ┌──────────┐              ┌──────────────┐
 │          │              │              │
 │     3V3  ├──── Red ─────┤ VCC          │
 │     GND  ├──── Black ───┤ GND          │
 │          │              │              │
 │  GPIO23  ├──── Orange ──┤ MOSI (SI)    │
 │  GPIO19  ├──── Yellow ──┤ MISO (SO)    │
 │  GPIO18  ├──── Green ───┤ SCK (SCLK)   │
 │   GPIO5  ├──── Purple ──┤ CSN (CS)     │
 │          │              │              │
 │   GPIO4  ├──── Blue ────┤ GDO0 (TX)    │
 │   GPIO2  ├──── Cyan ────┤ GDO2 (RX)    │
 │          │              │              │
 └──────────┘              └──────────────┘

CC1101 Pin No. Item Direction Description
1 GND Ground
2 VCC Power supply, 1.8V - 3.6V
3 GDO0 Output Data output pin
4 CSN Input Module chip selection pin for starting SPI communication
5 SCK Input SPI clock pin
6 MOSI Input SPI data Input pin
7 MISO/GDO1 Output SPI data output pin
8 GDO2 Output Data output pin

<img width="725" height="353" alt="grafik" src="https://github.com/flowjob1/cc1101-transceiver/blob/f9a89d09006f6e9553189d2ca6a32f16fd605a35/cc1101-pin-view.png" />


```

| Wire | ESP32 Pin | CC1101 Pin | Function |
|------|-----------|------------|----------|
| Red | 3V3 | VCC | Power (3.3V only!) |
| Black | GND | GND | Ground |
| Orange | GPIO23 | MOSI | SPI Data Out |
| Yellow | GPIO19 | MISO | SPI Data In |
| Green | GPIO18 | SCK | SPI Clock |
| Purple | GPIO5 | CSN | SPI Chip Select |
| Blue | GPIO4 | GDO0 | TX Data |
| Cyan | GPIO2 | GDO2 | RX Data |

> ⚠️ The CC1101 is a **3.3V device**. Do not connect to 5V — it will damage the module.

> **Note:** If GPIO2 causes issues on your board (some have an onboard LED), use GPIO15, GPIO16, or GPIO17 instead.

---

Pin numbering may vary between modules. Always verify with your module's datasheet.

---

## Installation

### Prerequisites

- [ESPHome](https://esphome.io/) installed (via Home Assistant Add-on or standalone)
- ESP32 board with USB cable
- CC1101 module wired to ESP32


## Usage

### Learning a Signal

1. Press **"Learn Signal"** in Home Assistant (or toggle **"Learning Mode"** on)
2. Status shows *"Waiting for signal... Press remote now!"*
3. Press the button on your RF remote near the CC1101
4. Status updates to *"Signal learned! X pulses captured"*

### Replaying a Signal

- **"Replay Learned Signal"** — Transmit once
- **"Replay x3"** — Transmit 3 times with 50ms gaps (more reliable for some devices)

### Clearing the Signal

- Press **"Clear Learned Signal"** to reset and learn a new one

> **Note:** The learned signal is stored in RAM. It will be lost on reboot. See [Adding Permanent Buttons](#adding-permanent-buttons) to save signals permanently.

### Copying Raw Codes from Logs

Every received signal is logged as a single copyable line in the ESPHome console:

```
[I][raw_code:xxx]: Pulses: 122
[I][raw_code:xxx]: Copy this line:
[I][raw_code:xxx]: [592, -635, 266, -295, 617, -615, 268, -309, ...]
```

You can paste this array directly into the YAML config as a permanent button.

---

## Home Assistant Entities

### Buttons

| Entity | Icon | Description |
|--------|------|-------------|
| Learn Signal | `mdi:record-rec` | Start learning mode |
| Replay Learned Signal | `mdi:replay` | Transmit learned signal once |
| Replay x3 | `mdi:replay` | Transmit 3 times with 50ms gaps |
| Clear Learned Signal | `mdi:delete` | Clear signal from memory |
| TX Test | `mdi:access-point` | Send a test signal |
| Restart | | Reboot the ESP32 |

### Switch

| Entity | Icon | Description |
|--------|------|-------------|
| Learning Mode | `mdi:school` | Toggle learning on/off |

### Sensors

| Entity | Type | Description |
|--------|------|-------------|
| Learn Status | Text | Current status message |
| Last Received Signal | Text | Preview of last captured signal |
| Learned Signal Length | Number | Pulse count of learned signal |
| Signal Learned | Binary | Whether a signal is currently stored |

---

## Adding Permanent Buttons

Once you've captured a signal from the logs, add it to the `button:` section in your YAML to make it permanent:

```yaml
- platform: template
  name: "Living Room Blinds Up"
  icon: "mdi:blinds-open"
  on_press:
    - remote_transmitter.transmit_raw:
        carrier_frequency: 0Hz
        code: [592, -635, 266, -295, 617, -615, ...]

- platform: template
  name: "Living Room Blinds Down"
  icon: "mdi:blinds"
  on_press:
    - remote_transmitter.transmit_raw:
        carrier_frequency: 0Hz
        code: [605, -622, 280, -310, ...]
```

You can also wrap them in a `repeat` for reliability:

```yaml
- platform: template
  name: "Garage Light"
  icon: "mdi:lightbulb"
  on_press:
    - repeat:
        count: 3
        then:
          - remote_transmitter.transmit_raw:
              carrier_frequency: 0Hz
              code: [500, -500, 500, -500, ...]
          - delay: 50ms
```

---

## Automation Examples

### Close Blinds at Sunset

```yaml
automation:
  - alias: "Close Blinds at Sunset"
    trigger:
      - platform: sun
        event: sunset
    action:
      - button.press:
          entity_id: button.cc1101_rf_transceiver_living_room_blinds_down
```

### Toggle Lights via Motion Sensor

```yaml
automation:
  - alias: "RF Light on Motion"
    trigger:
      - platform: state
        entity_id: binary_sensor.hallway_motion
        to: "on"
    action:
      - button.press:
          entity_id: button.cc1101_rf_transceiver_hallway_light
```

---

## Configuration Reference

### CC1101 Settings

Adjust in the `cc1101:` section of the YAML:

| Setting | Value | Description |
|---------|-------|-------------|
| `frequency` | `433.92MHz` | Operating frequency (300–928 MHz) |
| `output_power` | `10` | TX power in dBm (-30 to +11) |
| `modulation_type` | `ASK/OOK` | Modulation scheme |
| `symbol_rate` | `5000` | Symbol rate in Baud |
| `filter_bandwidth` | `200kHz` | Receive filter bandwidth |

### Remote Receiver Tuning

| Setting | Value | Description |
|---------|-------|-------------|
| `tolerance` | `50%` | How much pulse timing can deviate |
| `filter` | `200us` | Ignore pulses shorter than this |
| `idle` | `10ms` | Gap that marks end of a signal |

**Tuning tips:**
- Too much noise → increase `filter` to `500us` or reduce `filter_bandwidth`
- Missing signals → increase `tolerance` or `idle`

---

## Troubleshooting

### CC1101 Not Detected

- **"FF0F was found" error** — SPI wiring issue. Double-check MISO, MOSI, SCK, and CSN connections.
- Verify you're using **3.3V**, not 5V.
- Use shorter jumper wires — long wires cause SPI failures.

### Not Receiving Signals

- Confirm the remote operates on **433.92 MHz** (some use 315 or 868 MHz).
- Attach a proper antenna — a **17.3 cm** straight wire works for 433 MHz.
- Check that GPIO2 isn't conflicting with your board's onboard LED.
- Try increasing `filter_bandwidth` to capture wider frequency deviations.

### Signals Captured But Replay Doesn't Work

- Use **"Replay x3"** — some devices need multiple transmissions.
- Move the ESP32 + CC1101 **closer** to the target device.
- Increase `output_power` to `11` (maximum).
- **Rolling code devices** (garage doors, car key fobs) cannot be replayed — each press generates a unique code.

### Too Much Noise in Logs

- Increase `filter` from `200us` to `500us`
- Reduce `filter_bandwidth` from `200kHz` to `100kHz`
- Move the antenna away from the ESP32 board and USB cable

---

## Compatible Devices

| ✅ Works (Fixed Code) | ❌ Won't Work (Rolling/Encrypted) |
|------------------------|-----------------------------------|
| Motorized blinds/shades | Car key fobs |
| RF power outlets | Most garage door openers |
| Ceiling fan remotes | Security alarm systems |
| Doorbells | |
| RF light switches | |
| Fixed-code gate remotes | |


---

## References

- [ESPHome CC1101 Component](https://esphome.io/components/cc1101/)
- [ESPHome Remote Receiver](https://esphome.io/components/remote_receiver/)
- [ESPHome Remote Transmitter](https://esphome.io/components/remote_transmitter/)
- [CC1101 Datasheet (TI)](https://www.ti.com/lit/ds/symlink/cc1101.pdf)


---

## License

MIT License — free to use, modify, and share.














