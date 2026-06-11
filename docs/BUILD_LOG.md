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

## 2026-05-11 (night) — v1.1.1-rp2040: bot-thinking pulse + SSL stability

Released as [v1.1.1-rp2040](https://github.com/semichcsc-byte/Open-Chess/releases/tag/v1.1.1-rp2040).

### The bug I introduced and then fixed

User asked for a 'bot is thinking' light show. First attempt: bouncing comet on rank 8, animated from inside the SSL request loop, ~20 fps. Result: **"Failed to connect to Stockfish API" on every move**. Took me a serial capture to figure out what happened.

Root cause: each NeoPixel `show()` call disables interrupts for ~2.5 ms (bit-banging the WS2812 protocol). Calling it 20 times per second from inside an active SSL handshake / read corrupts enough TLS records that the connection dies. The animation killed the network it was decorating.

Fix: slow sine-wave breathing on rank 8 with a ~1.5 s period, discretised into 8 brightness buckets, only redraw when the bucket changes. Average ~5 fps, ~95% reduction in NeoPixel writes during the request.

Verified end-to-end on the physical board: e2-e4 → breathing pulse plays for ~2 s → Stockfish responds with `bestmove e7e5` → bot move displayed correctly. Played several more moves including a capture and an invalid-move recovery, all worked.

### Known issue (deferred to v1.2)

Serial debug log still has a **row-axis mirror** in chess notation: a move from `e2 to e4` is logged as `e7 to e5`. Internally the firmware is consistent (Stockfish receives the right FEN, the bot's response is applied to the right squares), but the human-readable print is wrong. Same kind of bug as the column mirror fixed in v1.1, fixable in `chess_bot.cpp::processPlayerMove` by changing `Serial.print(8 - fromRow)` to `Serial.print(fromRow + 1)`. Will batch into v1.2 with a couple of other small print/notation cleanups.

## 2026-05-11 (very late) — v1.1.2-rp2040: bot survives Stockfish timeout

Released as [v1.1.2-rp2040](https://github.com/semichcsc-byte/Open-Chess/releases/tag/v1.1.2-rp2040).

User report: "left the board for a few minutes, came back, asks to set up the pieces from scratch". Diagnosis: API timeout + accidental reset gesture.

When `makeStockfishRequest` returned an empty response (network blip, `stockfish.online` slow, transient TLS failure) or `parseStockfishResponse` / `parseMove` failed, `botThinking` was correctly cleared but `isWhiteTurn` stayed `false`. The next loop iteration of `chess_bot::update()` saw `isWhiteTurn == false && botThinking == false` and just sat idle, frozen, with no LEDs and no serial activity. The user fiddled with pieces trying to figure out what was wrong, accidentally satisfied the reset gesture (32 pieces in starting position for >1.5 s), and ended up back at the menu requiring a full re-setup.

Fix: all three failure paths in `makeBotMove` now reset `isWhiteTurn = true` so the user can retry the move. Added a brief red flash on rank 8 in the API-no-response case for visual feedback.

Verified by killing the router mid-move; board flashed red on rank 8 and gave turn back to the player so the move could be retried after the network recovered.

## 2026-05-11 (end of day) — v1.2.0-rp2040: consolidation

Released as **[v1.2.0-rp2040](https://github.com/semichcsc-byte/Open-Chess/releases/tag/v1.2.0-rp2040) — marked Latest** on the firmware repo.

No source code changes vs v1.1.2. This release is purely versioning + docs hygiene:

- CHANGELOG.md rewritten with a single canonical 'first stable release' section listing all features + the 9 bug fixes vs upstream Concept-Bytes
- 'Detailed iteration history' section preserves links to v1.0.0 → v1.1.2 for back-story readers
- The four iteration releases marked as **pre-release** on GitHub so a new visitor lands on v1.2.0 directly

Rationale: shipping 4 patch releases on the same day was noise on the releases page. v1.2.0 is the canonical install target; the iteration history stays reachable for anyone who wants to read the bug-by-bug story.

## 2026-05-11 (very late) — v1.2.1 castling hint, then v1.2.2 reliability crisis

Played a few games of AI mode to validate v1.2.0. Hit one polish issue (castling silently expects you to move the rook too — easy to miss if you don't play tournament chess) and one *huge* one (red flash on Stockfish API failure way more often than expected).

### v1.2.1: castling visual hint (HvsH only)

When the king moves two squares, Human-vs-Human mode now flashes the rook source square in blinking blue 4 times and lights the destination solid blue, then holds both for an extra 2 seconds. Serial also prints `Castling: move the rook from h1 to f1`. AI mode still does direct board mutation so the hint isn't drawn there yet — deferred to v1.3 with the bot refactor.

Released as **[v1.2.1-rp2040](https://github.com/semichcsc-byte/Open-Chess/releases/tag/v1.2.1-rp2040)** (now demoted to pre-release after v1.2.2 superseded it).

### v1.2.2 first attempt: defense in depth (didn't fix the real bug)

User feedback: "API timed out, board flashed red, gave the turn back". Played another game, same thing happened. Built defense in depth:

- 5-attempt retry loop with exponential backoff (was 1 attempt, then 3)
- Pre-flight `WiFi.status()` check before each attempt
- `ensureWiFiConnected()` quick reconnect (~8s) if link drops between attempts
- `reinitWiFiModule()` full `WiFi.end() + WiFi.begin()` cycle after 3 consecutive fails
- Bounded read loop with 8s inter-byte timeout (no `client.readString()` hangs)
- Hard 4096-byte response cap
- Detailed failure classification logs
- Pre-call breathing pulse (5 forced frames, ~600ms blue) so the user always sees something happening
- Amber retry pulse during backoff (different colour from blue thinking pulse)
- Red flash only after all 5 attempts exhausted (was: after 1)

Built a debug-instrumented build with per-step latency telemetry: `[DBG] Calling client.connect(...)`, `[DBG] connect() returned TRUE/FALSE after Xms`, `[DBG] First byte after Xms`, `[DBG] Total request time: Xms`, RSSI per request, etc.

Played another game. **Smoking gun captured**:

```
[DBG] Pre-flight: WiFi status=3 RSSI=-41 dBm IP=192.168.0.99
[DBG] Calling client.connect(stockfish.online:443)...
[DBG] connect() returned FALSE after 9312ms      ← TLS handshake times out

[DBG] connect() returned FALSE after 9515ms      ← attempt 2 also fails (same)
[DBG] connect() returned FALSE after 9508ms      ← attempt 3 also fails
[DBG] connect() returned FALSE after 30015ms     ← attempt 4 (after WiFi reset) ALSO fails
[DBG] connect() returned FALSE after    13ms     ← attempt 5 (NINA wedged now)
```

5 attempts in a row at -38 to -41 dBm (perfect signal), all hit a ~9.5s TLS handshake timeout. Even the full WiFi module reset doesn't help. Verified `curl https://stockfish.online/...` from the laptop in 0.7s, so the server is fine. Conclusion: the **NINA-W102 firmware 3.0.1 fundamentally cannot complete TLS 1.3 handshakes with Cloudflare-fronted endpoints**. The retry loop just makes it fail more thoroughly.

### v1.2.2 real fix: Cloudflare Worker proxy

Solution: stop trying to do TLS on the constrained device. Put a tiny proxy on Cloudflare that:

1. Accepts plain HTTP from the board (no TLS at all on the NINA)
2. Forwards to `stockfish.online` over HTTPS server-side
3. Returns just the JSON response

Wrote a 76-line JavaScript Worker (`worker/proxy.js`), deployed to `https://openchess-proxy.semichcsc.workers.dev` on the free Cloudflare tier (100k requests/day — plenty for a few thousand active boards). Switched `chess_bot.h` from `WiFiSSLClient` to plain `WiFiClient` and pointed at the proxy.

For users who don't trust our public proxy: `worker/README.md` documents the 30-second self-host: `cd worker && npx wrangler deploy`, then change one `#define`. Code is MIT-licensed, runs anywhere.

Verified on hardware — 8 consecutive Stockfish API calls all succeeded on attempt 1:

```
[DBG] connect() returned TRUE after 304ms     →  Total: 1943ms  →  bestmove e7e5
[DBG] connect() returned TRUE after  40ms     →  Total: 2049ms  →  bestmove b8c6
[DBG] connect() returned TRUE after 314ms     →  Total: 2082ms  →  bestmove g8f6
[DBG] connect() returned TRUE after 700ms     →  Total: 2625ms  →  bestmove h7h6
[DBG] connect() returned TRUE after  48ms     →  Total: 2646ms  →  bestmove d7d5
[DBG] connect() returned TRUE after  67ms     →  Total: 2036ms  →  bestmove f6d5
[DBG] connect() returned TRUE after 145ms     →  Total: 2088ms  →  bestmove c8e6
```

Was previously **0/5** with the direct HTTPS path. Now **8/8** consecutive. Total request time 1.9-2.6s, indistinguishable from native curl latency.

Released as **[v1.2.2-rp2040](https://github.com/semichcsc-byte/Open-Chess/releases/tag/v1.2.2-rp2040) — marked Latest**. v1.2.1 demoted to pre-release.

The defense-in-depth retry loop is still there, just almost never fires now. Belt + braces.

## 2026-06-11 — v1.3.0-rp2040: play-testing fixes + UX

Hooked the assembled board up over USB and played full games against the bot.
Several issues only show up once you actually play; fixed them all and shipped
[v1.3.0-rp2040](https://github.com/semichcsc-byte/Open-Chess/releases/tag/v1.3.0-rp2040).

- **Runtime AI difficulty**: after picking AI mode, 4 centre squares light up as buttons (c4 Easy/green, d4 Medium/blue, e4 Hard/amber, f4 Expert/red). No more recompiling — was the #1 roadmap item.
- **"Whose turn" indicator**: the side-to-move's back rank gently breathes (white = you, red = opponent in HvH), mirroring the bot's blue thinking pulse. Asked for after losing track of turns mid-game.
- **Blue AI selector LED** in the menu (HvH stays white) for at-a-glance distinction.
- **AI mode castling fixed**: the player literally could not castle in AI mode — `getPossibleMoves` (used there) never generates castling, so the 2-square king move was rejected as illegal. Switched AI mode to `getLegalMoves`, made it move the rook internally for both player and bot, and added the single blue rook-destination hint. Found by play-testing: serial showed `picked up WHITE piece 'K' at f1` → `Invalid move`.
- **Invalid-FEN-after-castling fixed**: `boardToFEN()` hard-coded `KQkq`. Once anyone castled, that made the FEN illegal and `stockfish.online` replied `{"success":false,"data":"Invalid FEN."}`, which the firmware (correctly) treated as a failure and bounced the turn back to the player — looked like "it's my turn again" after a capture. Now derives castling rights / en-passant / halfmove clock from live `GameState` (the bot side now tracks state too, via a new `updateCastlingRights`).
- **Stockfish parser hardened**: split HTTP body from headers before scanning for `{`, so Cloudflare's JSON-shaped `Report-To:` header is no longer mistaken for the payload (it recovered before, but fragile).
- **Mirrored rank notation** in AI-mode serial logs fixed (`8 - row` → `row + 1`) — the long-standing row-axis mirror noted back in v1.1.1.
- **Light-bleed fix**: lifting a piece now clears the turn-indicator row before drawing legal-move dots (otherwise the king's whole rank stayed lit).
- **Clean serial by default**: the 10s `uptime` heartbeat and the wall of boot/WiFi debug are gated behind `DEBUG_VERBOSE` / `WIFI_VERBOSE` (both off); a short "How to play" legend prints at boot instead.

Verified on hardware: full AI game including kingside castling (`Castling: move the rook from h1 to f1`, FEN `…/RNBQ1RK1`), bot responses ~2s end-to-end, turn indicator visible. Published to fork `main`, cut the release with the `.uf2`, and opened cumulative PR [Concept-Bytes/Open-Chess#12](https://github.com/Concept-Bytes/Open-Chess/pull/12) on top of #9/#10/#11.

> Toolchain note: built locally with `arduino-cli` + `arduino:mbed_nano@4.6.0`, Adafruit NeoPixel 1.14.0, WiFiNINA. `arduino_secrets.h` stays gitignored — the WiFi password never leaves the machine (gitleaks pre-commit confirms clean).

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
