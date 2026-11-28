> Sanitized mirror of Forgejo `serhii/dune-the-battle-for-arrakis`. Source code is not published here.
>
> Commit texts: `commits/`. Need the code? Email: sergeyalpatov1@gmail.com
> Source: Forgejo `serhii/dune-the-battle-for-arrakis` | Synced: 2026-11-28T04:10:36Z

---

# Dune: The Battle for Arrakis — Mega Drive reverse-engineering workspace

[Українською](docs/wiki/README-UKR.md)

This project rebuilds a user-supplied Sega Mega Drive cartridge from readable
source. It also builds a native port of the game for the PC. One source gives the
cartridge image and `port/build/dune-port`.

**Status:** The baseline rebuilds the dump byte for byte. The modified build
differs only in the title and the checksum in the header. The port plays the game.
{{fact:c_routines}} of its {{fact:routines}} routines are the port's own C.
{{fact:cartridge_routines}} routines are a declared cartridge remainder, and the
final proof measures them. {{fact:pixel_exact}} of
{{fact:checkpoints}} checkpoints match the reference pixel for pixel. The music
is the port's own YM2612.

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

# verify everything: {{fact:gate_all}} checks, {{fact:gate_fast}} of them need
# no emulator and no display
tests/run_all.sh
tests/run_all.sh --tier all
```

The cartridge dump is not in Git. Put it at `rom/original/DUNETHEBATTLEFORARRAK.BIN`
(`rom/README.md` says how). The analysis tools live in `tools/`. They stay local to
the user.

## The port

`make -C port play` builds one binary. The binary boots the cartridge and runs the
game logic as the port's C. The port has {{fact:groups}} groups,
{{fact:rows}} rows and {{fact:implementations}} implementations.

Since ticket 200 the machine has one execution path, the interpreter. The port's C
is the replacement table, and the machine reads that table before the interpreter.
{{fact:cartridge_routines}} routines have no row. They stay the cartridge's
code, and the project declares and counts them (`analysis/final-proof.json`).

| what it does | the check that proves it |
|---|---|
| draws the reference's picture: {{fact:checkpoints}} committed checkpoints, from the title screen to the campaign credits | `bash tests/test_fidelity.sh`: 0 of 71 680 pixels differ |
| plays the cartridge's music through the port's own YM2612 | `bash tests/test_ym_log.sh`, `bash tests/test_audio_compare.sh` |
| runs at the cartridge's rate: 59.92 frames a second | `bash tests/test_pacing.sh` |
| costs less than the cartridge's own code: 2.65 ms a frame against 6.22 | `bash tests/test_frame_cost.sh` |
| the measurement compares it with the cartridge on the whole route | `bash tests/test_final_proof.sh` |

If the Vulkan headers and `glslc` are present, `--renderer vk` draws the same scene
on the GPU. The CPU `scene` path is the default and needs nothing.
`dune-port --help` lists the rest: the widescreen shell, the port's own settings
and gamepad screens, the save file and `--debug`.

## Layout

| path | contents |
|---|---|
| `src/` | the cartridge source: `main.asm`, hand-written `rom_head.asm` and `hardware.asm`, and generated `segments.inc` |
| `port/` | the native port: `machine/` (the 68000, the Z80, the frame clock), `chips/` (VDP, PSG, YM2612, audio), `output/` (the renderers), `next/` (the port's C), `app/`, `platform/`, `state/`, `tests/` |
| `assets/` | the cartridge's content as editable files: graphics, music and the data tables, each with a manifest |
| `disassembly/` | the sega2asm split config and the generated per-segment `.asm`, committed because the changes go there |
| `scripts/` | the tools that inspect, split, build, validate, compare and diagnose |
| `tests/` | the checks behind `tests/run_all.sh` |
| `analysis/` | code map, names, flag inventory, recordings and the final proof (git ignores the derived files) |
| `patches/` | the pinned sega2asm fixes that a byte-exact rebuild needs |
| `tools/` | the toolchain, local to the user, and `REVISIONS.txt` (git ignores the binaries) |
| `rom/` | the dump (git-ignored) and how to place it |
| `docs/wiki/` | the prose, as a submodule of the project's wiki |
| `docs/html/` | the generated documentation site, committed |
| `docs/tasks/` | the ticket store |

## Documentation

The prose lives in the project's wiki (D158), and `docs/wiki` is its submodule. The
same files are the wiki's pages and the site's document sources. Every page is in
English. Where a translation exists, the Ukrainian page is beside it (`-UKR`).

* [the wiki](http://100.110.77.11/serhii/dune-the-battle-for-arrakis/wiki) — start at
  `Home`, then read `project-status.md` for what works today.
* [the site](http://100.110.77.11/serhii/dune-the-battle-for-arrakis/src/branch/main/docs/html/index.html) —
  one page for each routine, with the name and the work-RAM address, generated and
  committed.
* [project-status.md](docs/wiki/project-status.md) — what the project is, what works and
  what is next, with the command behind every number.
* [goals.md](docs/wiki/goals.md) — the two goals and their definitions of done.
* [CONTEXT.md](docs/wiki/CONTEXT.md) — the project's words.
* [Glossary-UKR.md](docs/wiki/Glossary-UKR.md) — the Ukrainian vocabulary.
* [next-engine.md](docs/wiki/next-engine.md) — the replacement seam and the recipe for one
  screen.
* [native-port.md](docs/wiki/native-port.md) — the port's subsystems.
* [modding.md](docs/wiki/modding.md) — three worked changes.
* [build-and-test.md](docs/wiki/build-and-test.md) — the commands and what each check proves.

## What this project does *not* contain

The project does not contain the cartridge dump, the extracted graphics and audio
blobs, or the emulator's save files. It also leaves out the built emulator binaries
and the port's build products. `rom/original/` holds the reference input and is
git-ignored. Nothing here redistributes the game.

## Requirements

`docs/wiki/toolchain.md` gives the install commands. The analysis tools are local to
the user: sega2asm (patched), ClownAssembler, BlastEm 1.0.0, Ghidra 12.1.4 and
REA 6.4.0. They need no system packages and no `sudo`. The port needs a C99
compiler, `make` and SDL2. `--renderer vk` also needs the Vulkan headers and
`glslc`.
