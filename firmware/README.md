# OpenChess Firmware

Arduino firmware for the OpenChess PCB v1.

## Setup

1. Install [Arduino IDE](https://www.arduino.cc/en/software)
2. Add Arduino Mbed OS Nano Boards via Board Manager
3. Select **Arduino Nano RP2040 Connect**
4. Upload firmware

## Files

- `sensor_test/` — Test Hall effect sensors (upload first to verify wiring)
- `main/` — Main chess firmware

## Notes

- Source code available via [Patreon](https://www.patreon.com/c/ConceptBytes) or included with PCB purchase
- Shift register: **74HC594** (NOT 74HC595)
- If using ESP32 instead of RP2040, pin mapping needs adjustment
