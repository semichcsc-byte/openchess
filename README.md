# OpenChess — Smart Chess Board Build

My build of the [OpenChess](https://concept-bytes.com/products/openchess-pcb) smart chess board by [Concept Bytes](https://www.youtube.com/@conceptbytes).

## Hardware

| Component | Status |
|-----------|--------|
| OpenChess PCB v1 | ✅ Have |
| Arduino Nano RP2040 Connect | ✅ Soldered to PCB |
| 3D printed board (bottom) | ✅ Printed |
| 3D printed board (top/tiles) | ⬜ TODO |
| 3D printed chess pieces (32) | ⬜ TODO |
| Neodymium magnets 10×2mm (36+) | ✅ Have |
| Steel discs 10×1mm (64+) | ✅ Have |
| Soldering (Arduino → PCB) | ✅ Done |

## Software

| Step | Status |
|------|--------|
| Sensor_Test firmware | ✅ Uploaded & validated — sensors work! |
| OpenChess.ino (full game) | ✅ Compiled & uploaded |
| WiFi configured | ✅ Stockfish AI working |
| Libraries (NeoPixel v1.14, WiFiNINA) | ✅ Installed |
| AI mode WiFi AP→STA patch | ✅ Applied — see [PR #9](https://github.com/Concept-Bytes/Open-Chess/pull/9) upstream |

## Project Structure

```
firmware/       # Arduino firmware notes
stl/            # 3D print files info (board, pieces, bases)
docs/           # Build notes, wiring, BOM
images/         # Build progress photos
```

## Tech Specs

- **PCB:** 64× Hall effect sensors (A3144) + 64× RGB LEDs (NeoPixel GRBW)
- **MCU:** Arduino Nano RP2040 Connect (WiFi + BLE built-in)
- **Shift register:** 74HC594 (⚠️ NOT 74HC595 — different pinout)
- **Pin mapping:** NeoPixel=17, ShiftReg SER=2 SRCLK=3 RCLK=4, Sensors=6-13
- **Board size:** <250×250mm
- **Magnets:** 10×2mm neodymium in each chess piece
- **Steel discs:** 10×1mm ferromagnetic under each square

## Game Modes

1. **Human vs Human** — pick up piece (red LED) → valid moves (white LEDs) → place (green flash)
2. **Human vs AI** — Stockfish engine via WiFi (Easy/Medium/Hard/Expert)
3. **Sensor Test** — LEDs light up where magnets are detected

Mode selection: power on → 4 white LEDs in center → place piece on one to select.

## Build Notes

- Magnet polarity matters — only south pole triggers A3144 sensors, flip if no detection
- 10×2mm magnets may be too strong (pieces attract each other) — 8×2mm is an alternative
- Code is designed for Arduino Nano RP2040 — ESP32 needs pin changes
- STL files from [MakerWorld](https://makerworld.com/en/models/1256302-openchess-smart-chess-board) (Concept_Bytes)
- Serial Monitor at 9600 baud for debug

## Firmware Sources

| Source | URL | Platform | Status (May 2026) | Notes |
|--------|-----|----------|-------------------|-------|
| Official (Concept-Bytes) | [GitHub](https://github.com/Concept-Bytes/Open-Chess) | Arduino Nano RP2040 | 🟡 Abandoned — last commit Aug/2025, 0 PRs merged | What I'm using. Needs my [PR #9](https://github.com/Concept-Bytes/Open-Chess/pull/9) for AI mode to work. |
| My fork (with fix) | [GitHub](https://github.com/semichcsc-byte/Open-Chess) | Arduino Nano RP2040 | 🟢 Has WiFi AP→STA fix on `fix/wifi-ap-sta-transition` | Use until/unless upstream merges PR #9 |
| Fork (joojoooo) | [GitHub](https://github.com/joojoooo/OpenChess) | ESP32 | 🟢 Active — last commit today, 16 stars | Lichess, Web UI, OTA, calibration. Requires Nano ESP32 + jumper wires (PCB pinout doesn't match) |
| Build Guide (joojoooo) | [Website](https://joojoooo.github.io/OpenChess/) | — | 🟢 Maintained | Schematics, wiring, web flasher |

## Links

- [OpenChess PCB (Concept Bytes)](https://concept-bytes.com/products/openchess-pcb)
- [Official Firmware (GitHub)](https://github.com/Concept-Bytes/Open-Chess)
- [Kickstarter Campaign](https://www.kickstarter.com/projects/conceptbytes/open-chess-a-3d-printable-smart-chess-board)
- [MakerWorld STLs](https://makerworld.com/en/models/1256302-openchess-smart-chess-board)
- [Build Guide & Schematics](https://joojoooo.github.io/OpenChess/)

## License

Build documentation: MIT. OpenChess design files: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) (Concept Bytes).
