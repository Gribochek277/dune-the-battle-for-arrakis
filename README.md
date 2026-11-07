> Sanitized mirror of Forgejo `serhii/dune-the-battle-for-arrakis`. Source code is not published here.
>
> Commit texts: `commits/`. Need the code? Email: sergeyalpatov1@gmail.com
> Source: Forgejo `serhii/dune-the-battle-for-arrakis` | Synced: 2026-11-10T07:37:44Z

---

# Dune: The Battle for Arrakis — Mega Drive reverse-engineering workspace

A reproducible rebuild of a user-supplied Sega Mega Drive cartridge from readable
source, plus a native PC port of the game. The same source produces the cartridge
image and `port/build/dune-port`.

**Status in one line:** the baseline header rebuilds the dump byte for byte, the
modified build differs only in the header's own title and checksum, and the port
plays the game — {{fact:c_routines}} of its {{fact:routines}} routines are the
port's own C, {{fact:guest_routines}} are a declared guest remainder the final
proof measures, {{fact:pixel_exact}} of {{fact:checkpoints}} checkpoints are
pixel for pixel the reference's, and the music is the port's own YM2612 through
the port's audio engine.

```
original : rom/original/DUNETHEBATTLEFORARRAK.BIN  2097152 bytes  sha256 1f0f56cd…c7ec
baseline : the header at tag baseline-byte-exact   2097152 bytes  sha256 1f0f56cd…c7ec
modified : build/dune.bin (from src/)              2097152 bytes  42 bytes differ, all in the header
```

## Quick start

```bash
# rebuild the cartridge from source
scripts/build.sh

# build and play the native port: the window, the audio, every group
make -C port play
port/build/dune-port

# verify everything: {{fact:gate_all}} checks, {{fact:gate_fast}} of them needing
# no emulator and no display
tests/run_all.sh
tests/run_all.sh --tier all

# boot the result headless and log every executed address
mkdir -p work/emu/home
HOME=$PWD/work/emu/home tools/blastem/blastem -b 900 -l build/dune.bin
```

## The port

`make -C port play` builds one binary that boots the cartridge and runs its game
logic as the port's C: {{fact:groups}} groups, {{fact:rows}} rows,
{{fact:implementations}} implementations, and no address handed back to the
guest. The machine has one execution path since ticket 200 — the interpreter —
and the port's C is the replacement table consulted before it; the
{{fact:guest_routines}} routines no row owns are the guest's, declared and
counted (`analysis/final-proof.json`). The picture, the clock and the RAM are the
reference's because the C leaves the state the guest's own instructions would
have; the one measured exception is the played build's audio engine, whose
timing moves the mission's state a frame apart
(`docs/project-status.md`, "What is still the guest's").

| what it does | the check that proves it |
|---|---|
| **draws the reference's picture** — {{fact:checkpoints}} committed checkpoints, from the title screen to the campaign credits | `bash tests/test_fidelity.sh`: 0 of 71 680 pixels differ on every one of the {{fact:pixel_exact}} |
| **plays the cartridge's music**, through the port's own YM2612 and the game's own audio API | `bash tests/test_ym_log.sh`, `bash tests/test_audio_compare.sh` |
| **runs at the cartridge's rate**: 59.92 guest frames a second, paced by the wall clock | `bash tests/test_pacing.sh` |
| **costs less than the guest's own code**: 2.65 ms a frame against 6.22 ms with `--no-replace` | `bash tests/test_frame_cost.sh` |
| **is measured against the cartridge on the whole route**: what is still the guest's, and the reference arrangement beside it | `bash tests/test_final_proof.sh` (`analysis/final-proof.json`) |

`--renderer vk` draws the same scene on the GPU (0.84 ms a frame against the CPU
path's 6.52 ms on this machine) where the Vulkan headers and `glslc` are
installed; the CPU `scene` path is the default and needs nothing. `dune-port
--help` lists the rest: the widescreen shell, the port's own settings and
gamepad screens, the save file, and `--debug`.

## Layout

| path | contents |
|---|---|
| `src/` | the cartridge source: `main.asm`, hand-written `rom_head.asm` and `hardware.asm`, generated `segments.inc` |
| `port/` | the native port: `machine/` (the 68000, the Z80, the frame clock), `chips/` (VDP, PSG, YM2612, audio), `output/` (the renderers), `next/` (the port's C), `app/`, `platform/`, `state/`, `tests/` |
| `assets/` | the cartridge's content as editable files — graphics, music and the data tables, each with its generated manifest |
| `disassembly/` | sega2asm split config, and the generated per-segment `.asm` (committed: it is what modifications are made in) |
| `scripts/` | inspection, splitting, building, validation, comparison, diagnostics, and a Ghidra export script |
| `tests/` | the automated checks behind `tests/run_all.sh` |
| `analysis/` | forensic report, code map, Ghidra inventory, coverage logs, EA truth table, work-RAM access map (derived, git-ignored) |
| `patches/` | the pinned sega2asm fixes required for a byte-exact rebuild |
| `tools/` | user-local toolchain and `REVISIONS.txt` (binaries are git-ignored) |
| `rom/` | your dump (git-ignored) and how to place it |
| `build/`, `work/`, `out/` | the rebuilt cartridge, the checks' artifacts, the emulator's home and the Ghidra projects (git-ignored) |
| `docs/` | environment, toolchain, ROM analysis, disassembly, build/test, decisions, RE notes, and the committed generated site under `docs/html/` |

## Documentation

* [the HTML documentation site](docs/html/index.html) — every routine (named or
  not), every curated name, the work-RAM map, the game's subsystems and the
  port's zones as generated pages and diagrams. Committed, so it reads from a
  `file://` URL; `python3 scripts/build_docs.py` rebuilds it and
  `tests/test_docs.sh` is the gate that keeps it current.
* [`docs/project-status.md`](docs/project-status.md) — **what the project is,
  what works today and what is being built next**, with the command that
  measures every number on the page.
* [`docs/game-rules.md`](docs/game-rules.md) — **the game's rules**: what a
  side can see, what a hit does, where credits and spice come from, how a
  structure is placed, built, repaired and lost, how a mission decides it is
  over, and what a selection and an order are. Every rule cites the instruction
  that implements it or the committed measurement that established it, and the
  page's §9 is the honest list of what is not known yet.
* [`docs/native-port.md`](docs/native-port.md) — the port: every subsystem, its
  measurement and the decisions behind it, including the frame's cost and the
  audio's tolerance.
* [`docs/port-architecture.md`](docs/port-architecture.md) — the port's zones and
  the allowed include directions, and [`docs/next-engine.md`](docs/next-engine.md)
  — the replacement seam, the dispatch's price and the group's own C.
* [`docs/game-architecture.md`](docs/game-architecture.md) — the game's own
  subsystems, from the boot to the mission's loop.
* [`docs/display.md`](docs/display.md) — the window, the integer scales, the
  widescreen shell and `display.ini`.
* [`docs/widescreen.md`](docs/widescreen.md) — the widescreen shell: a target
  that is the window's pixels, regions and anchors, the fit-height and
  wide-world modes, and the integer-scale policy that keeps the 4:3 picture's
  shape (`--aspect 16:9`).
* [`docs/data.md`](docs/data.md) and [`docs/assets.md`](docs/assets.md) — the
  unit and structure tables, the missions and the text as editable source, and
  the extracted graphics and music with their manifests.
* [`docs/saves.md`](docs/saves.md), [`docs/locale.md`](docs/locale.md),
  [`docs/keyboard.md`](docs/keyboard.md), [`docs/mouse.md`](docs/mouse.md) and
  [`docs/gamepad.md`](docs/gamepad.md) — the player's side: the save, the
  language, and the three input devices with their files.
* [`docs/goals.md`](docs/goals.md) — the two goals (playable PC binary; ROM to
  source + assets), their definitions of done and the roadmap
* [`docs/modding.md`](docs/modding.md) — **three worked changes**: a gameplay, a
  balance and a cosmetic change, each with the file that is edited and the check
  that proves it took effect
* [`docs/collecting-coverage.md`](docs/collecting-coverage.md) — how to play the game so the analysis grows
* [`docs/environment.md`](docs/environment.md) — host, tool inventory, how the ROM was found
* [`docs/rom-analysis.md`](docs/rom-analysis.md) — what the dump is, header, vectors, checksum
* [`docs/disassembly.md`](docs/disassembly.md) — the split, the 14 decoder fixes, why each was needed
* [`docs/build-and-test.md`](docs/build-and-test.md) — commands and what each check proves
* [`docs/debugging.md`](docs/debugging.md) — `--debug`, the run's session directory, and the machine snapshot (`--debug-load`) that restores a run exactly
* [`docs/toolchain.md`](docs/toolchain.md) — install commands, revisions, REA integration and its limits
* [`docs/decisions.md`](docs/decisions.md) — autonomous decisions and reasoning
* [`docs/reverse-engineering-notes.md`](docs/reverse-engineering-notes.md) — findings, the applied modification, and the plans

## What this project does *not* contain

The cartridge dump, the extracted graphics/audio blobs, emulator save files,
built emulator binaries and the port's own build products (`build/`, `port/build/`).
`rom/original/` is the reference input and is git-ignored; nothing here
redistributes the game.

## Requirements

See `docs/toolchain.md`. The analysis is user-local: sega2asm (patched),
ClownAssembler, BlastEm 1.0.0, Ghidra 12.1.4, REA 6.4.0 — no system packages and
no `sudo`. The port needs a C99 compiler, `make` and SDL2; `--renderer vk`
additionally needs the Vulkan headers and `glslc`, and is built only where they
exist, so a host without them still gets a window.
