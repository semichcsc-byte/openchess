# Firmware

## TL;DR — just give me a working binary

Grab the latest `.uf2` from the firmware fork:

➡️ **[github.com/semichcsc-byte/Open-Chess/releases/latest](https://github.com/semichcsc-byte/Open-Chess/releases/latest)**

Double-tap the white reset button on the Nano RP2040 → drag the `.uf2` file onto the `RPI-RP2` USB drive that appears. Done. See the [Quick Start in the main README](../README.md#quick-start).

## Sources

| Repo | Platform | Status | Notes |
|------|----------|--------|-------|
| [Concept-Bytes/Open-Chess](https://github.com/Concept-Bytes/Open-Chess) | Nano RP2040 | 🔴 Abandoned (last commit Aug 2025) | Has known bugs (AI mode hang, Stockfish parser, etc.) |
| [**semichcsc-byte/Open-Chess**](https://github.com/semichcsc-byte/Open-Chess) | Nano RP2040 | 🟢 Active — what you want | Patched fork with all fixes for the Concept-Bytes PCB + Nano RP2040 setup |
| [joojoooo/OpenChess](https://github.com/joojoooo/OpenChess) | **ESP32** | 🟢 Active | Different MCU, more features (Lichess, Web UI, OTA) but requires re-soldering with jumper wires |

→ Honest comparison in [`docs/COMPARISON.md`](../docs/COMPARISON.md).

## Compile from source (only needed for AI mode WiFi credentials)

```bash
# 1. Install board support and libraries (one-time)
arduino-cli core install arduino:mbed_nano
arduino-cli lib install "Adafruit NeoPixel"@1.14.0
arduino-cli lib install WiFiNINA

# 2. Clone the fork
git clone https://github.com/semichcsc-byte/Open-Chess.git
cd Open-Chess
git checkout v1.3.0-rp2040

# 3. Configure WiFi for AI mode (skip if you only want Human-vs-Human)
cp arduino_secrets_template.h arduino_secrets.h
# edit arduino_secrets.h with your SSID and password

# 4. Compile
arduino-cli compile --fqbn arduino:mbed_nano:nanorp2040connect .

# 5. Find your port
arduino-cli board list

# 6. Upload
arduino-cli upload --fqbn arduino:mbed_nano:nanorp2040connect -p /dev/cu.usbmodemXXX .
```

Alternatively, the standalone sensor test:

```bash
git clone https://github.com/Concept-Bytes/Open-Chess.git
arduino-cli compile --fqbn arduino:mbed_nano:nanorp2040connect Open-Chess/Sensor_Test/
arduino-cli upload  --fqbn arduino:mbed_nano:nanorp2040connect -p /dev/cu.usbmodemXXX Open-Chess/Sensor_Test/
```

## Pin mapping (Concept-Bytes PCB v1 + Nano RP2040)

| Function | Pin |
|----------|-----|
| NeoPixel `DIN` | D17 |
| Shift register `SER` | D2 |
| Shift register `SRCLK` | D3 |
| Shift register `RCLK` | D4 |
| Sensor columns (8×) | D6 D7 D8 D9 D10 D11 D12 D13 |

Shift register IC is **74HC594** — not 74HC595, the pinout is different. ESP32 needs different pins (see [`joojoooo` build guide](https://joojoooo.github.io/OpenChess/)).

## Library versions

| Library | Version | Notes |
|---------|---------|-------|
| Adafruit NeoPixel | **1.14.0** (pinned) | Required for RP2040 — newer versions had a regression |
| WiFiNINA | 2.0.1 (or newer) | For AI mode |
| arduino-cli core `arduino:mbed_nano` | 4.6.0 | Tested |

## Serial monitor

Open at **9600 baud**. By default the output is quiet — boot banner, the 10
self-test results (`PASS T1..T10`), a short "How to play" legend, then one line
per game event (moves, check, castling hints, result). Set `DEBUG_VERBOSE 1`
(and/or `WIFI_VERBOSE 1`) and re-flash for the full diagnostics:

- A versioned boot banner (`v1.3.0-rp2040`)
- 10 self-test results (`PASS T1..T10`)
- WiFi setup logs (verbose only)
- Every move parsed (`Player moved P from d2 to d4`, `Bot move: e7e5`, etc.)
- Game state changes (check, mate, draws)

If you see `WARNING: N engine self-tests FAILED` — the firmware is broken, re-flash a clean release.

## Bugs / improvements

File firmware bugs at [the fork repo](https://github.com/semichcsc-byte/Open-Chess/issues), not here.

## Where the hardware comes from

None of this firmware does anything useful without the matching PCB and 3D-printed parts:

- 🔗 **PCB:** [concept-bytes.com/products/openchess-pcb](https://concept-bytes.com/products/openchess-pcb), originally [Kickstarter](https://www.kickstarter.com/projects/conceptbytes/open-chess-a-3d-printable-smart-chess-board?ref=cc9981)
- 🔗 **3D files:** [MakerWorld — OpenChess Smart Chess Board](https://makerworld.com/en/models/1256302-openchess-smart-chess-board?from=search#profileId-1279762) by Concept_Bytes (CC BY 4.0)
