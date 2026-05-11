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

## 2026-05-11 (evening) — Phase 2 + Phase 3 in one go

Decision to keep the Nano RP2040 hardware and serve all Concept-Bytes Kickstarter backers with the same setup. Big batch of changes upstreamed as [PR #11](https://github.com/Concept-Bytes/Open-Chess/pull/11) (+1107 / -282 lines, 9 files).

**Phase 2 — chess rules:**
- Check / checkmate / stalemate detection with on-board animations
- Legal-move filtering (no more pinned-piece moves that expose own king)
- Castling (kingside + queenside, full FIDE rules including king-not-in-check + king-doesn't-pass-through-attacked-square)
- En passant (pink LED hint on EP target, applyMove removes the right pawn)
- 50-move rule + insufficient material draw (K-vs-K, K+minor-vs-K)
- Promotion choice via 4 LEDs on player's back rank (a=Q, b=R, c=B, d=N) in Human-vs-Human; bot mode still auto-Q

**Phase 3 — UX polish:**
- Sensor debounce: 3 consecutive identical reads required (eliminates the piece-flicker on board setup we saw in serial)
- AP `OpenChessBoard` shut down when entering non-WiFi modes (saves ~100mA, removes orphan WiFi network)
- 10 self-tests run at every boot before WiFi setup — PASS/FAIL printed to serial, red flash on failure

**Engine architecture refactor:**
- New `GameState` struct (castling rights, EP target, halfmove clock, last move)
- `getLegalMoves`, `isInCheck`, `isSquareAttacked`, `hasAnyLegalMove`, `getGameResult`, `applyMove`, `runSelfTests`
- All board mutation now goes through `ChessEngine::applyMove` so castling/EP/state can never be missed

**Validation: 10/10 self-tests passing** at boot (initial pawn moves, knight moves, no false check, Fool's Mate, pinned rook, both castlings, castle-in-check forbidden, en-passant, K-vs-K draw, kingside castle layout). Compile: 150679 bytes (0% of 16MB), 44640 bytes RAM (16%). Tested on hardware: chess_moves opens + plays through check + castles correctly; chess_bot still connects via #9 and applies validated moves.

Known follow-ups (kept out of scope): 5-char promotion parsing for bot mode (`e7e8q`), 3-fold repetition (needs position hashing + history).

## 2026-05-11 (later, again) — v1.1.0-rp2040: column-mirror bug + UX overhaul

Released as [v1.1.0-rp2040](https://github.com/semichcsc-byte/Open-Chess/releases/tag/v1.1.0-rp2040). Two themes:

### 🐛 The column-mirror coordinate bug

During testing of the new 2-selector menu (D5 = HvsH, E4 = AI), the LEDs were showing in D4 / E5 instead. Built an interactive 4-corner calibration sketch to find out why.

Results:
```
h1 -> sensor (0, 0)   expected (0, 7)
a1 -> sensor (0, 7)   expected (0, 0)
a8 -> sensor (7, 7)   expected (7, 0)
h8 -> sensor (7, 0)   expected (7, 7)
```

Diagnosis: **the Concept-Bytes PCB v1 wires file `a` to the rightmost shift-register column** (and file `h` to the leftmost). The upstream firmware never accounted for this. Every chess move printed to serial had its file letter mirrored across the a-h axis. The board worked because the mirror was self-consistent, but:
- Serial debug logs were misleading (`Player moved P from a7 to a5` actually meant `h7-h5`)
- Stockfish FEN strings were technically wrong (engine got mirrored positions; replied with valid mirrored moves)
- Any future Web UI / PGN export would have shown the wrong notation

Fix: encapsulated at the driver layer in [`board_driver.cpp`](https://github.com/semichcsc-byte/Open-Chess/blob/main/board_driver.cpp) — both `readSensors()` and `getPixelIndex()` now mirror `col` to `7 - col`. **All 36 callers (engine, chess_moves, chess_bot, animations) untouched**.

### ✨ UX overhaul (also in v1.1)

- **Setup hint at boot**: white side glows soft white, black side glows red; LEDs go dark as pieces are placed.
- **Convergent rainbow explosion** when all 32 pieces are in place: 4 collapsing rainbow rings, then 4 diagonal beams from corners, then a white shockwave outward, ending with a triple pulse on the 2 selectors. ~3.5 s.
- **Simplified 2-option menu**: D5 = HvsH, E4 = AI. Sensor Test removed from menu (still in code).
- **Sticky menu**: lifting a piece to use as a selector no longer regresses to the setup hint.
- **Reset gesture**: while playing, all 32 pieces back to starting squares for >1.5 s drops back to the menu.
- **Boot banner**: now prints firmware version + fork name on serial.

### 📖 Manual rewrite — WiFi setup

Got bitten by my own `arduino_secrets.h` getting overwritten by the upstream placeholder `Kc iphone` / `12345678` (a copy-back accident during development). Rewrote the WiFi setup section in [docs/MANUAL.md](https://github.com/semichcsc-byte/Open-Chess/blob/main/docs/MANUAL.md#wifi--ai-mode-setup) of the firmware repo:

- Prerequisites (2.4 GHz only, special chars in passwords, Arduino CLI install)
- 6-step quick setup walkthrough
- Common failure modes (5 GHz, captive portals, WPA Enterprise, phone hotspots, hidden SSIDs)
- **Troubleshooting table by `WL_*` status code** so users can self-diagnose
- Security note: don't share compiled `.uf2` because it embeds the WiFi password in plaintext

Verified: 10/10 self-tests passing on hardware.

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
