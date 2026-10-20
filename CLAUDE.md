# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A C11 / raylib 5.5 procedural "cyberspace tunnel" flight sandbox (`sandbox/cybertunnel`), built as a reusable module (`cybertunnel_lib`) with a harness executable. A collaborator later layered a game, **DATA // RUN**, on top (`src/game/`). The executable now launches the game by default; `--sandbox` gives the original development harness with environments and debug tools. The sandbox environments are the primary work; game changes should not alter how the sandbox environments look.

All code lives under `sandbox/cybertunnel/`. Paths below are relative to it.

## Build and test

```sh
cmake -S . -B build && cmake --build build -j      # RelWithDebInfo by default; raylib/raygui fetched (pinned)
ctest --test-dir build --output-on-failure          # all suites
ctest --test-dir build -R data_run_mission          # one suite (names: cybertunnel_core, data_run_mission, data_run_techno, data_run_ship)
./build/cybertunnel_tests                           # core determinism/continuity tests, no window or GL needed
```

The suites have no per-case filter; each binary runs all its cases from `main()`. The graphics-dependent ctests (`data_run_endless`, `_startup`, `_live_menu`, `_standalone`) are registered on Windows only. Run them by hand on Linux:

```sh
./build/cybertunnel --hidden --startup-check
./build/cybertunnel --hidden --menu-check --seed 83
./build/cybertunnel --hidden --autoplay --missions 6 --audio-check --seed 83
```

## Running the sandbox

```sh
./build/cybertunnel --sandbox --env N        # 0 vortex, 1 template, 2 grid city, 3 server corridor, 4 greybox
  [--seed N] [--preset NAME] [--set name=value ...] [--shot SECONDS --out FILE] [--bench SECONDS]
  [--list-params] [--write-preset NAME] [--boost] [--noship] [--hdr8]
```

`--shot` runs a fixed 1/60 s step by frame count, so the same command produces a byte-identical PNG. Use it for before/after regression checks of a visual refactor. Debug renders go in `shots/` (git-ignored). Runtime keys: F1 param panel, F2 free cam, F3 overlay, T switch environment, 1–9 layer toggles, R reseed, F12 screenshot.

The executable loads shaders, presets and assets from the copies next to it (`build/shaders`, `build/presets`, `build/assets`), which the post-build step copies from `src/render/shaders/`, `presets/` and `assets/`. After editing a shader or preset, rebuild (or copy the file into `build/`) before taking a shot. Editing the source file alone changes nothing at run time. Hot reload (file mtime polling) watches the `build/` copies. Presets are `key=value`; section headers are ignored on load.

On this machine the command sandbox blocks the display. Window, `--shot` and `--hidden` runs need to run outside the sandbox with `DISPLAY=:1` and `XAUTHORITY` set to `/run/user/1000/xauth_*`. Without them GLFW fails and the run segfaults (exit 139).

## Architecture

- **Core / environment split** (`src/env/env.h`). The core owns the path, chunk streaming, ship/flight, camera and HDR post. An `Environment` is a table of callbacks: `pathStyle`, `registerParams`, `buildChunk`, `uploadChunk`, `drawBackground/Opaque/Emissive`, `energyAt`, `atmosphere`. Registry: `ENV_VORTEX=0, ENV_TEMPLATE=1, ENV_CITY=2, ENV_CORRIDOR=3, ENV_GREYBOX=4` (the greybox is a plain movement-tuning course, `--env 4 --preset greybox`; registry ids are not the "Env #N" names in the docs). To add one, follow INTEGRATION.md §11 (copy `env_template.c`, register in `env.h`, `cybertunnel.h`, `cybertunnel.c` and `CMakeLists.txt`).
- **Determinism is a hard rule.** `buildChunk` is pure CPU with no GL calls. Its output depends only on (seed, params, chunkIndex, path). Seed RNGs with `ct_hash64_3(seed, ENV_ID, chunkIndex)` and per-feature sub-streams; never use `rand()` or `GetRandomValue()`. Tests check build-order independence (`build_order_independent()` in `tests/test_main.c`). `init` is called with `shaders == NULL` in tests, so skip GL work then.
- **Path** (`core/path.c`): centripetal Catmull-Rom from noise-driven control points, arc-length resampled into a ring buffer, with rotation-minimizing frames. Open environments (city, corridor) set `floorDepth`, `maxPitchDeg=0` and `twistRateMax=0` to get a flat path with world-up normals.
- **Precision**: world positions are double (`DVec3`). Rendering uses "render space" = world − `renderOrigin` (floating origin), and chunk geometry is stored relative to `chunk->origin`.
- **Rendering**: an HDR scene target, then bloom, then the composite pass (`render/pipeline.c`). Neon lines are camera-facing ribbon geometry from `render/linegeo.c`. Shared fog/fade/time uniforms come from `env_set_common_uniforms()` and `shaders/common.glsl`, which is `#include`d by most shaders. Building blocks shared across environments: `render/streaks`, `render/shapes`, `render/gridfloor`, `env/lattice_place` (single-owner lattice cells for open environments), `env/trace_route`, `core/pacing` (archetype Markov chain).
- **Shared shaders are shared state.** `ring.fs`, `tower.fs`, `sky.fs`, `grid.fs`, `streak.vs` and `common.glsl` are each used by several environments; for example, `ring.fs` serves both the vortex and the corridor. A new uniform that only one environment sets is left at zero in the others. Make zero mean the old behaviour, or set the uniform from every user. Game-only looks are opt-in parameters enabled in `presets/mission*.ini`, for example `cityGameLook`, which turns on the muted city palette, the lit tower windows (`uWindows` in `tower.fs`) and the skyline and moon (`uSkyline` in `sky.fs`).
- **Params**: every tunable is registered with a globally unique name. Group names become debug-panel and ini sections. Editing a path-affecting group triggers a full rebuild (`group_rebuilds_path` in `debug/debug_ui.c`).
- **Game layer** (`src/game/`, `src/main.c`): uses only the public API in `src/cybertunnel.h`. It draws game objects through `ct_set_scene_draw`. Mission and difficulty rules are in `mission.c`; the live-synthesized music is `techno.c`; the game presets are `presets/mission*.ini`. On Windows, shaders, presets and assets are embedded as RC resources (`core/resources.c`).

## Docs in the repo

- `BRIEF.md`: original spec, palette and pitfalls checklist.
- `NOTES.md`: per-phase implementation log with acceptance results and user feedback decisions for Env #1–#3.
- `PLAN_ENV2_GRID_CITY.md`, `PLAN_ENV3_SERVER_CORRIDOR.md`: environment plans.
- `INTEGRATION.md`: public API, GL ownership and the full parameter table.
- `DATA_RUN.md`: the game.

Record new environment work in `NOTES.md` in the same style.
