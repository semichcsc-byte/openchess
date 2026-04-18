# Build Log

## 2026-04-18 — Project Start

- PCB v1 in hand
- Arduino Nano RP2040 Connect in hand, soldered to PCB
- Bottom board printed
- Ordered neodymium magnets 10×2mm from Amazon.de (Magenesis B06X977K8L, 52pcs)
- Ordered steel discs 10×1mm from Amazon.de (B097C4C3SG, 25pcs × 3)
- Created GitHub repo
- Found official firmware: https://github.com/Concept-Bytes/Open-Chess
- Found community fork with more features: https://github.com/joojoooo/OpenChess
- Found build guide with schematics: https://joojoooo.github.io/OpenChess/
- Installed arduino-cli v1.4.1 + board package arduino:mbed_nano v4.5.0
- Installed Adafruit NeoPixel v1.14.0 + WiFiNINA v2.0.1
- **Compiled & uploaded Sensor_Test** → ✅ sensors work! LEDs light up with magnet
- **Compiled & uploaded OpenChess.ino** (full firmware with WiFi + Stockfish AI)
- All 3 game modes available: Human vs Human, Human vs AI, Sensor Test

## Next Steps

- [ ] Print top tiles (64 squares)
- [ ] Print frame & base plates
- [ ] Print chess pieces (10×2mm magnet version)
- [ ] Glue steel discs under each tile (64 pcs)
- [ ] Glue magnets into chess pieces (32 pcs — south pole facing down!)
- [ ] Full game test: Human vs Human
- [ ] Full game test: Human vs AI (Stockfish)
- [ ] Consider ESP32 migration for jojodio fork features (Lichess, Web UI, OTA)
