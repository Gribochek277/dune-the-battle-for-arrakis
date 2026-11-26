> Sanitized mirror of Forgejo `serhii/dune-the-battle-for-arrakis`. Source code is not published here.
>
> Commit texts: `commits/`. Need the code? Email: sergeyalpatov1@gmail.com
> Source: Forgejo `serhii/dune-the-battle-for-arrakis` | Synced: 2026-11-26T22:09:03Z

---

# Dune: The Battle for Arrakis — Mega Drive reverse-engineering workspace

[Українською](docs/wiki/README-UKR.md)

A reproducible rebuild of a user-supplied Sega Mega Drive cartridge from readable
source, plus a native PC port of the game. One source produces the cartridge image
and `port/build/dune-port`.

**Status in one line:** the baseline rebuilds the dump byte for byte, the modified
build differs only in the header's own title and checksum, and the port plays the
game — {{fact:c_routines}} of its {{fact:routines}} routines are the port's own C,
{{fact:cartridge_routines}} are a declared cartridge remainder the final proof
measures, {{fact:pixel_exact}} of {{fact:checkpoints}} checkpoints are pixel for
pixel the reference's, and the music is the port's own YM2612.

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
```

The cartridge dump is not in Git: put it at `rom/original/DUNETHEBATTLEFORARRAK.BIN`
(`rom/README.md` says how). The analysis tools live in `tools/`, user-local.

## The port

`make -C port play` builds one binary that boots the cartridge and runs its game
logic as the port's C: {{fact:groups}} groups, {{fact:rows}} rows,
{{fact:implementations}} implementations. The machine has one execution path since
ticket 200, the interpreter, and the port's C is the replacement table consulted
before it; the {{fact:cartridge_routines}} routines no row owns are the
cartridge's, declared and counted (`analysis/final-proof.json`).

| what it does | the check that proves it |
|---|---|
| draws the reference's picture — {{fact:checkpoints}} committed checkpoints, title screen to campaign credits | `bash tests/test_fidelity.sh`: 0 of 71 680 pixels differ |
| plays the cartridge's music through the port's own YM2612 | `bash tests/test_ym_log.sh`, `bash tests/test_audio_compare.sh` |
| runs at the cartridge's rate: 59.92 frames a second | `bash tests/test_pacing.sh` |
| costs less than the cartridge's own code: 2.65 ms a frame against 6.22 | `bash tests/test_frame_cost.sh` |
| is measured against the cartridge on the whole route | `bash tests/test_final_proof.sh` |

`--renderer vk` draws the same scene on the GPU where the Vulkan headers and
`glslc` exist; the CPU `scene` path is the default and needs nothing. `dune-port
--help` lists the rest: the widescreen shell, the port's own settings and gamepad
screens, the save file, `--debug`.

## Layout

| path | contents |
|---|---|
| `src/` | the cartridge source: `main.asm`, hand-written `rom_head.asm` and `hardware.asm`, generated `segments.inc` |
| `port/` | the native port: `machine/` (the 68000, the Z80, the frame clock), `chips/` (VDP, PSG, YM2612, audio), `output/` (the renderers), `next/` (the port's C), `app/`, `platform/`, `state/`, `tests/` |
| `assets/` | the cartridge's content as editable files: graphics, music, the data tables, each with a manifest |
| `disassembly/` | the sega2asm split config and the generated per-segment `.asm`, committed because modifications are made there |
| `scripts/` | inspection, splitting, building, validation, comparison, diagnostics |
| `tests/` | the checks behind `tests/run_all.sh` |
| `analysis/` | code map, names, flag inventory, recordings, the final proof (derived files are git-ignored) |
| `patches/` | the pinned sega2asm fixes a byte-exact rebuild needs |
| `tools/` | the user-local toolchain and `REVISIONS.txt` (binaries are git-ignored) |
| `rom/` | the dump (git-ignored) and how to place it |
| `docs/wiki/` | the prose, as a submodule of the project's wiki |
| `docs/html/` | the generated documentation site, committed |
| `docs/tasks/` | the ticket store |

## Documentation

The prose lives in the project's wiki (D158) and `docs/wiki` is its submodule: the
same files are the wiki's pages and the site's document sources. Every page is in
English and, where the translation exists, in Ukrainian beside it (`-UKR`).

* [the wiki](http://100.110.77.11/serhii/dune-the-battle-for-arrakis/wiki) — start
  at `Home`, then read `project-status.md` for what works today.
* [the site](http://100.110.77.11/serhii/dune-the-battle-for-arrakis/src/branch/main/docs/html/index.html) —
  one page per routine, name and work-RAM address, generated and committed.
* [project-status.md](docs/wiki/project-status.md) — what the project is, what works
  and what is next, with the command behind every number.
* [goals.md](docs/wiki/goals.md) — the two goals and their definitions of done.
* [CONTEXT.md](CONTEXT.md) — the project's words.
* [Glossary-UKR.md](docs/wiki/Glossary-UKR.md) — the Ukrainian vocabulary.
* [next-engine.md](docs/wiki/next-engine.md) — the replacement seam and the recipe
  for one screen; [native-port.md](docs/wiki/native-port.md) — the port's subsystems.
* [modding.md](docs/wiki/modding.md) — three worked changes; [build-and-test.md](docs/wiki/build-and-test.md) —
  the commands and what each check proves.

## What this project does *not* contain

The cartridge dump, the extracted graphics/audio blobs, emulator save files, built
emulator binaries and the port's build products. `rom/original/` is the reference
input and is git-ignored; nothing here redistributes the game.

## Requirements

`docs/wiki/toolchain.md` has the install commands. The analysis is user-local:
sega2asm (patched), ClownAssembler, BlastEm 1.0.0, Ghidra 12.1.4, REA 6.4.0 — no
system packages, no `sudo`. The port needs a C99 compiler, `make` and SDL2;
`--renderer vk` additionally needs the Vulkan headers and `glslc`.
