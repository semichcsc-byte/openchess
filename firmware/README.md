# OpenChess Firmware

Arduino firmware for the OpenChess PCB v1.

## Source Code

- **Official repo:** https://github.com/Concept-Bytes/Open-Chess (MIT license)
- **Community fork (ESP32):** https://github.com/joojoooo/OpenChess (more features)
- **Build guide & schematics:** https://joojoooo.github.io/OpenChess/

## Setup (arduino-cli)

```bash
# Install board support
arduino-cli core install arduino:mbed_nano

# Install libraries
arduino-cli lib install "Adafruit NeoPixel@1.14.0"
arduino-cli lib install WiFiNINA

# Clone firmware
git clone https://github.com/Concept-Bytes/Open-Chess.git

# Configure WiFi (edit arduino_secrets.h)

# Compile sensor test
arduino-cli compile --fqbn arduino:mbed_nano:nanorp2040connect Open-Chess/Sensor_Test/

# Upload sensor test
arduino-cli upload --fqbn arduino:mbed_nano:nanorp2040connect --port /dev/cu.usbmodemXXXXXX Open-Chess/Sensor_Test/

# Compile & upload full firmware (copy all .ino/.h/.cpp to one folder named OpenChess/)
arduino-cli compile --fqbn arduino:mbed_nano:nanorp2040connect OpenChess/
arduino-cli upload --fqbn arduino:mbed_nano:nanorp2040connect --port /dev/cu.usbmodemXXXXXX OpenChess/
```

## Pin Mapping

| Function | Pin(s) |
|----------|--------|
| NeoPixel LED data | 17 |
| Shift register SER | 2 |
| Shift register SRCLK | 3 |
| Shift register RCLK | 4 |
| Column sensors | 6, 7, 8, 9, 10, 11, 12, 13 |

## Dependencies

| Library | Version | Notes |
|---------|---------|-------|
| Adafruit NeoPixel | **1.14.0** | ⚠️ Specific version required for RP2040 |
| WiFiNINA | 2.0.1 | For Stockfish AI via WiFi |

## Notes

- Shift register: **74HC594** (NOT 74HC595 — different pinout!)
- If using ESP32 instead of RP2040, pin mapping needs adjustment
- Serial Monitor: 9600 baud for debug
- Game mode selection on boot: 4 center LEDs, place piece to choose
