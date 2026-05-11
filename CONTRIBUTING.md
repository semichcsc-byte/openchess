# Contributing

Thanks for considering a contribution. This repo holds **build documentation** for my OpenChess assembly. The actual firmware source code lives in a separate repo:

➡️ **Firmware bug reports / PRs**: [`semichcsc-byte/Open-Chess`](https://github.com/semichcsc-byte/Open-Chess) (the fork with the patches)

➡️ **Build documentation issues** (this repo): [`semichcsc-byte/openchess/issues`](https://github.com/semichcsc-byte/openchess/issues)

## What's in scope here

- Improvements to [the user manual](docs/MANUAL.md), [BUILD_LOG](docs/BUILD_LOG.md), [BOM](docs/BOM.md), or [comparison doc](docs/COMPARISON.md)
- Better photos in `images/`
- Typo fixes
- Adding troubleshooting tips you discovered while building yours
- Translations (especially Portuguese — happy to merge a `README.pt.md`)

## What's out of scope

- Firmware source code changes — those go to [the fork](https://github.com/semichcsc-byte/Open-Chess)
- 3D model changes — STLs come from [MakerWorld](https://makerworld.com/en/models/1256302-openchess-smart-chess-board) (Concept Bytes, CC BY 4.0)
- Hardware schematic changes — see [`joojoooo` build guide](https://joojoooo.github.io/OpenChess/) for the canonical schematic

## How to file an issue

When reporting a build problem, please include:

- Your hardware: PCB version (v1?), MCU (Nano RP2040 / ESP32?), magnets size
- Firmware version (printed at boot in serial banner — `v1.0.0-rp2040`)
- The full **serial monitor output at 9600 baud**, from the boot banner to the moment things go wrong
- A photo if relevant

Bonus: paste the relevant section of the [troubleshooting table](docs/MANUAL.md#troubleshooting) you already tried.

## How to submit a PR

1. Fork the repo
2. Create a branch (`git checkout -b doc/your-improvement`)
3. Make your changes
4. Run `git commit` with a descriptive message
5. Push to your fork and open a PR against `main`

For doc changes, no special CI runs — just make sure markdown renders correctly on GitHub.

## Code of conduct

Be civil. The other forks (Concept-Bytes upstream, joojoooo) are not enemies; they're solving different problems with different trade-offs. Critique the work, never the people.

## Recognising contributors

If you make a meaningful contribution (more than a typo), I'll add you to the contributors list in the README. If you build the board following these docs and want me to link your build, open an issue with photos.
