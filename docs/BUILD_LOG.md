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

## 2026-05-11 — AI mode debug & upstream PR

- Symptom: selecting AI mode "did nothing" — board appeared frozen.
- Step 1 — wrong selector: serial showed I was placing the piece on `(4,3) = D4 = Game Mode 3 (Coming Soon)`, a placeholder. AI mode is `(3,4) = E5`.
- Step 2 — real bug found: even on the right square, `chess_bot.cpp::connectToWiFi()` calls `WiFi.begin()` directly without ever shutting down the AP that `WiFiManager::begin()` started in `setup()`. WiFiNINA on the Nano RP2040 Connect (and Nano 33 IoT / MKR WiFi 1010) cannot run AP and STA simultaneously → `WiFi.begin()` silently never returns `WL_CONNECTED`.
- Patch: `WiFi.disconnect(); WiFi.end(); delay(2000);` before `WiFi.begin()`.
- Verified: bot connects on attempt 1, gets IP `192.168.0.99`, Stockfish API responds (`bestmove d7d5`), full game playable.
- Also confirmed Stockfish API is up: `curl https://stockfish.online/api/s/v2.php?...` returns `{"success":true,...}`.
- Submitted upstream as **PR [Concept-Bytes/Open-Chess#9](https://github.com/Concept-Bytes/Open-Chess/pull/9)** (`fix/wifi-ap-sta-transition`, closes #5).
- Surveyed forks: 24 total. Only **[joojoooo/OpenChess](https://github.com/joojoooo/OpenChess)** is actively maintained (16 stars, last commit today, ESP32-based, Lichess + Web UI + OTA). [KhachDavid/smart-chessboard](https://github.com/KhachDavid/smart-chessboard) is a stockfish-only fix-fork, abandoned since Sep/2025.
- Upstream `Concept-Bytes/Open-Chess`: 0 PRs ever merged, last commit Aug/2025, 7 open issues (incl. #5) → effectively abandoned. PR #9 will sit in queue but is searchable for the next person hitting the bug.

## 2026-05-11 (later) — RP2040 quality pass (Phase 1)

Kept hardware as-is (Nano RP2040 + Concept-Bytes PCB) and focused on fixes that benefit every user with the same setup. 5 patches in [PR #10](https://github.com/Concept-Bytes/Open-Chess/pull/10):

1. **`stockfish_settings.h`**: `medium()` depth 6 → 10 (Easy and Medium were identical despite different banner)
2. **`chess_bot.cpp::parseStockfishResponse`**: split HTTP headers from body before scanning for JSON — Cloudflare's `Nel:` and `Report-To:` headers contain JSON-shaped values that the old `indexOf("{")` was grabbing
3. **`chess_bot.cpp::makeBotMove`**: validate API move locally before mutating state, return turn to player on any failure path (no more frozen `botThinking`)
4. **`OpenChess.ino` MODE_GAME_3**: replace 3s blind loop with wait-until-piece-lifted gate; eliminates serial spam and lets user recover without reboot
5. **`chess_engine.h`**: introduce `MAX_MOVES_PER_PIECE` constant; was inconsistent (`moves[27]` in chess_bot, `moves[28]` everywhere else — off-by-one buffer overflow risk)

Compile: `Sketch uses 144717 bytes (0%) of program storage space, +337 bytes vs PR #9 baseline`. Verified by upload + serial monitor: boot clean, mode selection works, no regressions on working paths. Local belt-and-suspenders defence in `.gitignore` against `arduino_secrets.h` even if `upstream-firmware/` ignore is ever removed.

Next (Phase 2, optional): chess rules completeness — check detection, pinned pieces, checkmate, castling, en passant. All RP2040-friendly, all in pure C++.

## Next Steps

- [ ] Print top tiles (64 squares)
- [ ] Print frame & base plates
- [ ] Print chess pieces (10×2mm magnet version)
- [ ] Glue steel discs under each tile (64 pcs)
- [ ] Glue magnets into chess pieces (32 pcs — south pole facing down!)
- [x] Full game test: Human vs Human
- [x] Full game test: Human vs AI (Stockfish) — works after WiFi AP→STA patch
- [ ] Optional: fix `MODE_GAME_3` infinite re-selection loop while piece sits on the placeholder square
- [ ] Consider ESP32 migration for [joojoooo](https://github.com/joojoooo/OpenChess) fork features (Lichess, Web UI, OTA)
