# OpenChess — Smart Chess Board Build

My build of the [OpenChess](https://concept-bytes.com/products/openchess-pcb) smart chess board by [Concept Bytes](https://www.youtube.com/@conceptbytes).

## Hardware

| Component | Status |
|-----------|--------|
| OpenChess PCB v1 | ✅ Have |
| Arduino Nano RP2040 Connect | ✅ Have |
| 3D printed board (bottom) | ✅ Printed |
| 3D printed board (top/tiles) | ⬜ TODO |
| 3D printed chess pieces (32) | ⬜ TODO |
| Neodymium magnets 10×2mm (36+) | ✅ Ordered |
| Steel discs 10×1mm (64+) | ✅ Ordered |
| Soldering (Arduino → PCB) | ⬜ TODO |

## Project Structure

```
firmware/       # Arduino firmware & sensor test code
stl/            # 3D print files (board, pieces, bases)
docs/           # Build notes, wiring, BOM
images/         # Build progress photos
```

## Tech Specs

- **PCB:** 64× Hall effect sensors + 64× RGB LEDs
- **MCU:** Arduino Nano RP2040 Connect
- **Shift register:** 74HC594 (⚠️ NOT 74HC595 — different pinout)
- **Board size:** <250×250mm
- **Magnets:** 10×2mm neodymium in each chess piece
- **Steel discs:** 10×1mm ferromagnetic under each square

## Build Notes

- Magnet polarity matters — flip if sensor doesn't trigger
- 10×2mm magnets may be too strong (pieces attract each other) — 8×2mm is an alternative
- Code is designed for Arduino Nano RP2040 — ESP32 needs pin changes
- STL files from [MakerWorld](https://makerworld.com/en/models/1256302-openchess-smart-chess-board) (Concept_Bytes)

## Links

- [OpenChess PCB (Concept Bytes)](https://concept-bytes.com/products/openchess-pcb)
- [Kickstarter Campaign](https://www.kickstarter.com/projects/conceptbytes/open-chess-a-3d-printable-smart-chess-board)
- [MakerWorld STLs](https://makerworld.com/en/models/1256302-openchess-smart-chess-board)
- [Patreon (firmware)](https://www.patreon.com/c/ConceptBytes)

## License

Build documentation: MIT. OpenChess design files: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) (Concept Bytes).
