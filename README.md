# OpenChess — Smart Chess Board (Nano RP2040 fix-fork build)

[![Firmware release](https://img.shields.io/github/v/release/semichcsc-byte/Open-Chess?label=firmware&color=brightgreen)](https://github.com/semichcsc-byte/Open-Chess/releases/latest)
[![Build doc release](https://img.shields.io/github/v/release/semichcsc-byte/openchess?label=build%20doc&color=blue)](https://github.com/semichcsc-byte/openchess/releases/latest)
[![Self-tests](https://img.shields.io/badge/self--tests-10%2F10-brightgreen)](docs/MANUAL.md#self-tests)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

My build of the [OpenChess](https://concept-bytes.com/products/openchess-pcb) smart chess board by [Concept Bytes](https://www.youtube.com/@conceptbytes), with **bug-fix patches** for the Arduino Nano RP2040 Connect firmware.

> ## 🎯 Are you a Kickstarter backer with the Concept-Bytes PCB?
>
> **AI mode hangs?** **Sensors flicker?** **Stockfish API failed?** **Pieces appear/disappear when you slide them?**
>
> Use my patched firmware — it fixes those issues. The original Concept-Bytes repo has been unmaintained since Aug/2025 (0 PRs ever merged, 7 open issues including the AI mode hang).
>
> 🔥 **[Download v1.0.0-rp2040 firmware (.uf2 drag-and-drop)](https://github.com/semichcsc-byte/Open-Chess/releases/latest)**
>
> Backed the [OpenChess Kickstarter](https://www.kickstarter.com/projects/conceptbytes/open-chess-a-3d-printable-smart-chess-board?ref=cc9981)? Same hardware — these fixes apply to your board too.

---

## Table of contents

- [What this repo is](#what-this-repo-is)
- [Quick start (flash patched firmware in 60 seconds)](#quick-start)
- [User manual](docs/MANUAL.md) — game modes, moves, troubleshooting
- [Comparison with other forks](docs/COMPARISON.md) — what we have, what we don't
- [Hardware & build status](#hardware--build-status)
- [Firmware sources](#firmware-sources)
- [Build documentation](docs/BUILD_LOG.md) — chronological log of the build
- [Bill of materials](docs/BOM.md)
- [Contributing](CONTRIBUTING.md)
- [License](#license)

---

## What this repo is

This repository has **two parts**:

1. **Build documentation** for my personal OpenChess assembly (this repo, [`semichcsc-byte/openchess`](https://github.com/semichcsc-byte/openchess)) — BOM, photos, lessons learned.
2. **Patched firmware** for the Concept-Bytes PCB + Arduino Nano RP2040 Connect ([`semichcsc-byte/Open-Chess`](https://github.com/semichcsc-byte/Open-Chess)) — fixes the broken AI mode, sensor flicker, missing chess rules, and packs everything into a one-file `.uf2` drop-in.

If you just want a working board, scroll to **[Quick start](#quick-start)**. If you want to understand the build, see **[BUILD_LOG.md](docs/BUILD_LOG.md)**.

---

## Quick start

> Goal: get the patched firmware running on your board in under 60 seconds, without installing the Arduino IDE.

### 1. Download the firmware

[![Latest release](https://img.shields.io/github/v/release/semichcsc-byte/Open-Chess?label=v1.0.0-rp2040)](https://github.com/semichcsc-byte/Open-Chess/releases/latest)

Grab `OpenChess-v1.0.0-rp2040.uf2` (302 KB).

### 2. Put the board in bootloader mode

**Double-tap** the white reset button on the Arduino Nano RP2040 Connect within ~500 ms. The board mounts as a USB drive named **`RPI-RP2`**.

### 3. Drag-and-drop

Drag the `.uf2` file onto the `RPI-RP2` drive. The board reboots automatically into the new firmware.

### 4. Open the Serial Monitor (optional but recommended)

At **9600 baud**. You should see:

```
================================================
         OpenChess Starting Up
         Firmware: v1.0.0-rp2040
         Fork:    semichcsc-byte/Open-Chess
================================================
=== ChessEngine self-tests ===
PASS T1..T10
=== Self-tests complete: 10/10 passed ===
```

If you see fewer than 10 passes, the board flashes red 5 times — the firmware is broken, **don't use it for a game**.

### 5. Pick a game mode

Four white LEDs light up in the centre of the board. See [the user manual](docs/MANUAL.md#mode-selection) for what each one does. **TL;DR**: AI mode is the second LED (E5 square in chess notation).

### 6. (Only for AI mode) Configure WiFi

The pre-built `.uf2` does **not** include WiFi credentials. To use AI mode, clone the source, edit `arduino_secrets.h`, and recompile. Full instructions in [the manual](docs/MANUAL.md#wifi--ai-mode-setup).

---

## What's fixed vs upstream Concept-Bytes

| Bug / missing feature in upstream | Fixed in v1.0.0-rp2040 |
|---|---|
| 🔴 AI mode hangs at "Connecting to WiFi…" | ✅ AP→STA tear-down before `WiFi.begin()` ([PR #9](https://github.com/Concept-Bytes/Open-Chess/pull/9)) |
| 🔴 "Stockfish API failed" on success | ✅ Parser splits HTTP body from headers ([PR #10](https://github.com/Concept-Bytes/Open-Chess/pull/10)) |
| 🔴 Easy and Medium AI are identical | ✅ `medium()` now sends depth=10 ([PR #10](https://github.com/Concept-Bytes/Open-Chess/pull/10)) |
| 🔴 No check / checkmate / stalemate | ✅ Full detection + on-board animation ([PR #11](https://github.com/Concept-Bytes/Open-Chess/pull/11)) |
| 🔴 Pinned pieces could move (expose own king) | ✅ Legal-move filtering |
| 🔴 No castling | ✅ Kingside + queenside, FIDE legal |
| 🔴 No en-passant | ✅ Pink LED hint, correct capture |
| 🔴 No 50-move rule / insufficient material | ✅ Auto-detected, draw animation |
| 🔴 Pawn always auto-promoted to Queen | ✅ Q/R/B/N choice via 4 LEDs (Human-vs-Human) |
| 🟡 Sensor flicker when sliding pieces | ✅ Debounce: 3 consecutive scans required |
| 🟡 `OpenChessBoard` AP stays up forever (~100 mA) | ✅ Shut down when not needed |
| 🟡 No way to know if firmware is broken | ✅ 10 self-tests at every boot |

Plus assorted code-quality fixes (buffer sizes, off-by-one, MODE_GAME_3 spam loop, etc.).

See **[docs/COMPARISON.md](docs/COMPARISON.md)** for an honest comparison with the more advanced [`joojoooo/OpenChess`](https://github.com/joojoooo/OpenChess) ESP32 fork.

---

## Hardware & build status

| Component | Status |
|-----------|--------|
| OpenChess PCB v1 | ✅ Have |
| Arduino Nano RP2040 Connect | ✅ Soldered to PCB |
| 3D printed board (bottom) | ✅ Printed |
| 3D printed board (top/tiles) | ⬜ TODO |
| 3D printed chess pieces (32) | ⬜ TODO |
| Neodymium magnets 10×2 mm (36+) | ✅ Have |
| Steel discs 10×1 mm (64+) | ✅ Have |
| Soldering (Arduino → PCB) | ✅ Done |

| Software step | Status |
|------|--------|
| `Sensor_Test` firmware | ✅ Sensors validated |
| `OpenChess.ino` (full game) | ✅ Compiled & uploaded |
| WiFi + Stockfish AI | ✅ Working |
| 10 engine self-tests | ✅ 10/10 passing |
| Patched firmware released | ✅ [v1.0.0-rp2040](https://github.com/semichcsc-byte/Open-Chess/releases/latest) |
| 3 upstream PRs filed | ✅ [#9](https://github.com/Concept-Bytes/Open-Chess/pull/9), [#10](https://github.com/Concept-Bytes/Open-Chess/pull/10), [#11](https://github.com/Concept-Bytes/Open-Chess/pull/11) |

---

## Tech specs

- **PCB:** 64× Hall effect sensors (A3144) + 64× RGB LEDs (NeoPixel GRBW)
- **MCU:** Arduino Nano RP2040 Connect (WiFi + BLE built-in via WiFiNINA)
- **Shift register:** 74HC594 (⚠️ NOT 74HC595 — different pinout)
- **Pin mapping:** NeoPixel = D17, ShiftReg SER = D2 SRCLK = D3 RCLK = D4, Sensor cols = D6–D13
- **Board size:** < 250×250 mm
- **Magnets:** 10×2 mm neodymium in each chess piece (south pole down for A3144)
- **Steel discs:** 10×1 mm ferromagnetic under each square
- **Serial:** 9600 baud for debug output
- **Sketch size:** 150 799 bytes (≈ 1 % of 16 MB flash)
- **RAM:** 44 640 bytes (16 % of 270 KB)

---

## Game modes

1. **Human vs Human** — pick up piece (red LED) → valid moves shown (white) → place (green flash). Includes castling, en passant, promotion choice.
2. **Human vs AI** — Stockfish via WiFi (Easy / Medium / Hard / Expert). Bot move validated locally before applying.
3. **Sensor Test** — LEDs light up where a magnet is detected; useful for hardware verification.

Mode selection: power on → 4 white LEDs in the centre → place any chess piece on one to select. Full layout in [the manual](docs/MANUAL.md#mode-selection).

---

## Firmware sources

| Source | Platform | Status (May 2026) | Notes |
|--------|----------|-------------------|-------|
| [Concept-Bytes/Open-Chess](https://github.com/Concept-Bytes/Open-Chess) (official) | Nano RP2040 | 🔴 Abandoned — last commit Aug 2025, 0 PRs merged | Original. Several known bugs unfixed for 9+ months. |
| [**semichcsc-byte/Open-Chess**](https://github.com/semichcsc-byte/Open-Chess) (this fork) | Nano RP2040 | 🟢 Active — v1.0.0-rp2040 released | What you want if you have the Concept-Bytes PCB + Nano RP2040. |
| [joojoooo/OpenChess](https://github.com/joojoooo/OpenChess) | **ESP32** (different MCU!) | 🟢 Active — 16★, last commit May 2026 | More features (Lichess online, Web UI, OTA, calibration), but requires re-soldering with jumper wires. |
| [joojoooo build guide](https://joojoooo.github.io/OpenChess/) | — | 🟢 Maintained | Best schematic + wiring documentation for both platforms. |

→ **Detailed comparison: [docs/COMPARISON.md](docs/COMPARISON.md)**

---

## Repo layout

```
openchess/
├── README.md                      ← you are here
├── LICENSE                        ← MIT
├── CONTRIBUTING.md                ← how to file issues / PRs
├── docs/
│   ├── MANUAL.md                  ← full user manual
│   ├── COMPARISON.md              ← honest gap analysis vs other forks
│   ├── BUILD_LOG.md               ← chronological build journal
│   └── BOM.md                     ← bill of materials with prices & links
├── firmware/                      ← README pointing at the fork
├── stl/                           ← 3D-print sources & notes
├── images/                        ← build photos (TODO)
└── upstream-firmware/             ← gitignored local clone, do not commit
```

---

## Links

- 🔗 [OpenChess PCB (Concept Bytes)](https://concept-bytes.com/products/openchess-pcb)
- 🔗 [Kickstarter campaign](https://www.kickstarter.com/projects/conceptbytes/open-chess-a-3d-printable-smart-chess-board?ref=cc9981) — the original crowdfunding source for the PCB + Arduino kit
- 🔗 [MakerWorld STLs](https://makerworld.com/en/models/1256302-openchess-smart-chess-board?from=search#profileId-1279762) — official 3D-printable parts (board, tiles, pieces) by Concept_Bytes
- 🔗 [Official firmware (GitHub)](https://github.com/Concept-Bytes/Open-Chess) — abandoned, see fork below
- 🔗 [**My patched firmware fork**](https://github.com/semichcsc-byte/Open-Chess) — Nano RP2040 fixes
- 🔗 [`joojoooo` build guide & schematics](https://joojoooo.github.io/OpenChess/) — best wiring docs
- 🔗 [`joojoooo` ESP32 fork](https://github.com/joojoooo/OpenChess) — alternative if you want Lichess + Web UI

---

## License

Build documentation in this repo: [MIT](LICENSE).

OpenChess design files (PCB, STLs): [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) by Concept Bytes.

Firmware fork: MIT (inherited from upstream Concept-Bytes/Open-Chess).
