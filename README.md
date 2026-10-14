> Sanitized mirror of Forgejo `serhii/dune-the-battle-for-arrakis`. Source code is not published here.
>
> Commit texts: `commits/`. Need the code? Email: sergeyalpatov1@gmail.com
> Source: Forgejo `serhii/dune-the-battle-for-arrakis` | Synced: 2026-10-20T11:00:35Z

---

# Dune: The Battle for Arrakis — Mega Drive reverse-engineering workspace

A reproducible, byte-exact rebuild of a user-supplied Sega Mega Drive cartridge,
plus the analysis tooling and the integration decisions behind it.

**Status in one line:** the workspace rebuilds the dump byte for byte from
source, the rebuilt image runs identically in the emulator, and the first source
modification is applied and verified.

```
original : rom/original/DUNETHEBATTLEFORARRAK.BIN  2097152 bytes  sha256 1f0f56cd…c7ec
baseline : build/dune.bin (from src/)              2097152 bytes  sha256 1f0f56cd…c7ec
modified : build/dune.bin after M1                 42 bytes differ, all in the header
```

## Quick start

```bash
# rebuild the cartridge from source
scripts/build.sh

# verify everything (validation, per-line byte check, byte comparison)
tests/run_all.sh

# boot the result headless and log every executed address
mkdir -p work/emu/home
HOME=$PWD/work/emu/home tools/blastem/blastem -b 900 -l build/dune.bin
```

## Layout

| path | contents |
|---|---|
| `src/` | the cartridge source: `main.asm`, hand-written `rom_head.asm` and `hardware.asm`, generated `segments.inc` |
| `disassembly/` | sega2asm split config, and the generated per-segment `.asm` (committed: it is what modifications are made in) |
| `scripts/` | inspection, splitting, building, validation, comparison, diagnostics, and a Ghidra export script |
| `tests/` | the automated checks behind `tests/run_all.sh` |
| `analysis/` | forensic report, code map, Ghidra inventory, coverage logs, EA truth table, work-RAM access map (derived, git-ignored) |
| `patches/` | the pinned sega2asm fixes required for a byte-exact rebuild |
| `tools/` | user-local toolchain and `REVISIONS.txt` (binaries are git-ignored) |
| `rom/` | your dump (git-ignored) and how to place it |
| `docs/` | environment, toolchain, ROM analysis, disassembly, build/test, decisions, RE notes, and the committed generated site under `docs/html/` |

## Documentation

* [the HTML documentation site](docs/html/index.html) — every routine (named or
  not), every curated name, the work-RAM map, the game's subsystems and the
  port's zones as generated pages and diagrams. Committed, so it reads from a
  `file://` URL; `python3 scripts/build_docs.py` rebuilds it and
  `tests/test_docs.sh` is the gate that keeps it current.
* [`docs/goals.md`](docs/goals.md) — the two goals (playable PC binary; ROM to source + assets), their definitions of done and the roadmap
* [`docs/collecting-coverage.md`](docs/collecting-coverage.md) — how to play the game so the analysis grows

* [`docs/environment.md`](docs/environment.md) — host, tool inventory, how the ROM was found
* [`docs/rom-analysis.md`](docs/rom-analysis.md) — what the dump is, header, vectors, checksum
* [`docs/disassembly.md`](docs/disassembly.md) — the split, the 14 decoder fixes, why each was needed
* [`docs/build-and-test.md`](docs/build-and-test.md) — commands and what each check proves
* [`docs/toolchain.md`](docs/toolchain.md) — install commands, revisions, REA integration and its limits
* [`docs/decisions.md`](docs/decisions.md) — autonomous decisions and reasoning
* [`docs/reverse-engineering-notes.md`](docs/reverse-engineering-notes.md) — findings, the applied modification, and the plans

## What this project does *not* contain

The cartridge dump, the extracted graphics/audio blobs, emulator save files, and
built emulator binaries. `rom/original/` is the reference input and is
git-ignored; nothing here redistributes the game.

## Requirements

See `docs/toolchain.md`. Everything is user-local: sega2asm (patched),
ClownAssembler, BlastEm 1.0.0, Ghidra 12.1.4, REA 6.4.0. No system packages and
no `sudo`.
