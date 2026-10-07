# DATA // RUN

A playable data-courier game built on the original `rec_tunnel` C11/raylib
flight sandbox. Fly through the twisting Data Vortex tunnel, collect amber data
gates, avoid security hazards and enter the cyan uplink before time runs out.
The tunnel is the default route; Grid City is selectable from the menu.

![DATA // RUN gameplay](docs/data-run-gameplay.png)

## Play

Collect at least **8 of 25 data gates** on a 3,000 metre route within **90 seconds**.
Three hazard hits disable the ship. Consecutive pickups increase your score
multiplier, and near misses earn bonuses. Hold boost to accelerate from
**32 to 65 m/s**; missed pickups do not interrupt boost.

| Key | Action |
| --- | --- |
| Enter | Start / retry |
| WASD / arrows | Steer |
| Shift / Space | Boost / afterburner |
| Q / E | Barrel roll |
| Escape | Pause / resume |
| R | Restart |
| Backspace | Return to menu |
| 1 / 2 in menu | Tunnel / Grid City |
| M | Mute / unmute |
| F11 | Fullscreen |
| F12 | Screenshot |

The game pauses when its window loses focus. This is a replayable arcade
vertical slice with one mission type; there is no campaign or saved progression.

## Features

- The original curved tunnel, rotating rings and helical ribbons, with smoother
  curves, detailed wall panels and faint peripheral speed streaks.
- A smooth courier ship with painted-metal texture, material lighting,
  articulated exhaust petals and blue/orange afterburner flames.
- Armored drones, moving sweepers, paired mines and laser walls with cyan safe
  openings. Clear target brackets and a minimal flight HUD.
- Original stereo techno synthesized live at 48 kHz, rising from **140 to
  166 BPM** with actual speed, plus procedural sound effects.
- Sustained boost and full elapsed-time simulation across long frames.
- A startup crash fix for zero-time first frames. Windows builds embed their
  textures, shaders, presets, font and license notices, so the EXE runs alone.

## Build

Requires CMake 3.20+ and a C11 compiler. CMake fetches pinned raylib 5.5 and
raygui 4.0. OpenGL 3.3 and an audio device are used by the game. On Windows,
MinGW builds statically link the compiler runtime and embed game resources.

From the repository root:

```sh
cmake -S sandbox/cybertunnel -B build -DCMAKE_BUILD_TYPE=Release -DCMAKE_POLICY_VERSION_MINIMUM=3.5
cmake --build build --parallel
ctest --test-dir build --output-on-failure
cmake --install build --prefix package --component Runtime
```

Launch `build/cybertunnel.exe` on Windows, or `build/cybertunnel` on Linux.
The installed package contains the executable and editable resource folders.
On Windows the executable also works without those folders. Linux builds keep
the `shaders`, `presets` and `assets` folders beside the executable.

## Verification and development

Six Windows CTest suites cover core behavior, mission rules, procedural music,
ship geometry, normal startup and standalone EXE startup. The startup checks
create hidden graphics windows and exercise the audio device while muted.
The four platform-independent suites do not require a graphics window.

```sh
build/cybertunnel --hidden --startup-check
build/cybertunnel --hidden --autoplay --audio-check --seed 83
build/cybertunnel --hidden --mission-shot 9 --w 1920 --h 1080 --out gameplay.png
build/cybertunnel --music-preview techno-preview.wav --seed 83
build/cybertunnel --sandbox --env 0
```

Use `cybertunnel.exe` in those commands on Windows. The music preview exports
24 seconds of the live synthesizer, moving from cruise to afterburner tempo.
`--sandbox` retains the original development environments and debug tools.

See [game details and verification](sandbox/cybertunnel/DATA_RUN.md) and the
[module integration guide](sandbox/cybertunnel/INTEGRATION.md).

## License

The original project is distributed under the [MIT License](LICENSE).
Third-party notices and the Rajdhani font's SIL Open Font License are included
in [assets](sandbox/cybertunnel/assets). The generated ship texture's native
resolution and production prompt are recorded with the asset.
