# Compatibility: will the game run on this machine, and how to find out fast

`SKILL.md` (step 1b) sends you here when the host and the game's platform differ (a Windows-only game on a Mac or
Linux box, an x86 game on an ARM CPU), or when the OS build, GPU, driver or layer version might matter even on the
same platform. Goal: spend minutes, not hours, finding a way to run the game so the normal modding loop can start.
Versions and routes move every few weeks, so take versions from the field notes (`um kb search`), not from here.

## The fast path
1. **Compare** the host and the game (below). Cheap.
2. **Search the notes** by situation and by the exact error text. This is the biggest time saver.
3. No match: **pick a route** (native build, compatibility layer, virtual machine, cloud) and name the layers.
4. **Check the same-OS mismatches** (build, GPU, driver, layer version, runtimes) against what a note recorded.
5. **Probe** the layer with something tiny before the real game.
6. **Record** exact versions in `MODDING_PLAN.md`, and share what the notes lacked.

## 1. Compare both sides
**Host.** Write down the OS and build, the CPU and core count, the GPU and the graphics APIs it offers.
- macOS: `sw_vers`; `system_profiler SPDisplaysDataType` (chip, Metal support);
  `sysctl -n machdep.cpu.brand_string` and `sysctl -n hw.ncpu` (CPU name and core count).
- Windows: `dxdiag` (its Feature Levels line shows what DirectX levels the GPU supports).
- Other hosts: the OS's own tools. Do not guess; an unchecked host is the usual reason a note's result doesn't
  repeat.

**Game.**
- CPU: `um scan` prints it per exe (`exes: game.exe [x64]`; `[x86]` is 32-bit, `[arm64]` is ARM). In `--json` it is
  `executables.<exe>.arch`.
- Engine and anti-cheat: also from `um scan`.
- Graphics API: the DLLs the exe imports. `um scan` does not list them. Any PE import lister works, for example
  `x86_64-w64-mingw32-objdump -p game.exe | grep "DLL Name"` (from a mingw-w64 toolchain), `dumpbin /dependents`
  on Windows, or the crude `strings -a game.exe | grep -iE '^(d3d1[0-2]|dxgi|vulkan-1|opengl32)\.dll$'`. Look
  for `d3d9`, `d3d11`, `d3d12`, `dxgi`, `vulkan-1`, `opengl32`. Engines also print the API and device they chose
  in their own log (Unity `-logFile <path>`, Godot `--log-file <path>`): read that line first.
- Steamworks or a launcher: `steam_api.dll` or `steam_api64.dll`, `steam_appid.txt`, a small root exe that only
  checks prerequisites and starts the real one.
- Anti-cheat files in the folder: `EasyAntiCheat*`, `BattlEye*`, `vanguard`, `Denuvo`. See "Anti-cheat" below.

## 2. Search the notes
Notes are ranked on their title, tags and body, and the gotchas are in the body, so searching the situation or an
error message works. Quote the phrase.
```bash
um kb search "windows game on mac"        # also "windows game on linux", "wine", "proton"
um kb search "directx apple silicon"      # your graphics API and your host
um kb search "<engine> <host OS>"         # for example "unity macos", "unreal wine"
um kb search "<launcher> <host OS>"       # for example "steam wine"
um kb search "SteamAPI_Init() failed"     # an error message, copied exactly
```
- **Spell the API out.** `dx11` misses where `directx 11` or `direct3d 11` hits. One miss proves nothing: try
  other phrasings, the engine, the launcher, the error text.
- **A match for your host, engine and layer:** follow its setup and versions exactly, run its verification, and
  only then deviate. What it lists as **not verified** is what you probe.
- **Read how the result was obtained:** observed by that agent, or quoted from a source; the versions and the date;
  what it says it did not verify.
- **A note on another game** with the same engine and layer still helps: the gotchas often transfer.
- **Before you change a layer's version or settings,** search again: a note may already say that version breaks.

## 3. Pick a route (no note matches)
Take the first rung that reaches the user's idea.
1. **A native build or port** for the host: the store page, the engine's own export (for a project the user owns),
   an open-source port. Nothing to translate; best by far.
2. **A compatibility layer.** Windows games on Linux and macOS usually run through Wine or a Wine-based tool.
   Name the layer you rely on for each concern, because they fail independently:

   | Concern | What translates it | Examples (check the notes for what actually worked) |
   |---|---|---|
   | CPU | x86 code on an ARM CPU | Rosetta 2 on Apple Silicon |
   | Windows API (files, threads, registry, windows, audio) | Wine and its derivatives | Wine builds, Proton (Valve's Wine-based tool for Steam on Linux), CrossOver (CodeWeavers, paid), Apple's Game Porting Toolkit, wrappers such as Sikarugir |
   | Graphics API | a layer from DirectX or Vulkan to the host's GPU API | on macOS: D3DMetal (Apple, closed source; DirectX 11 and 12), DXMT (open source; DirectX 10 and 11), MoltenVK (Vulkan); on Linux: DXVK and vkd3d for DirectX over Vulkan |
   | Launcher | the launcher's own client, itself a Windows program | Steam: Windows Steam must run in the **same** prefix as the game |

3. **A virtual machine.** Real Windows, no compatibility layer, but check first that the agent can still read the
   game's files, drive the game and take screenshots from outside.
4. **Cloud streaming.** The game runs elsewhere, so there are no local files to mod. Not useful for modding.
5. **None works.** Say so, and offer another game, an official tool, or a data-only or offline route.

## 4. Anti-cheat
It is a stop sign for injection (`skills/mod-any-game/references/safety.md`), and it often stops a compatibility
layer from running the game at all. On Linux, Easy Anti-Cheat and BattlEye games run under Proton only when the
developer opted in, and in practice the results have been uneven. Check the specific game; if it is protected, stop
and tell the user. Never bypass it.

## 5. The same OS can still be incompatible
The answer changes with the exact build and hardware even on the same platform.
- **OS build.** A game or a layer may need a newer or older build, and an OS update can break a setup that
  worked (record the OS build before and after updating).
- **GPU and driver.** A game can require a DirectX 12 feature level, a Vulkan version or a shader model. When the
  GPU lacks it the game refuses to start, crashes at load, or falls back to another graphics API. Compare the GPU's
  levels (`dxdiag`, or the log of the game) with the requirement on the store page. Some engines also gate on core
  count or GPU class with their own filters; the engine's log says when it rejected an API.
- **Layer version.** The same layer can pass at one version and fail at another (for example a Wine too old
  crashed a game that a current Wine runs). Use the version a note recorded; if you must change it, change one
  thing at a time.
- **Runtimes.** A missing or mismatched .NET, Visual C++ runtime or DirectX redistributable looks like a crash or a
  prerequisites dialog. Check which one the game wants before blaming the layer.
- **CPU features.** A game built for newer instruction sets may not run where the CPU or the translator lacks them.

A note's result holds for the versions it recorded. If yours differ on any of the above, treat it as a hint and
confirm it with a probe.

## 6. Probe first
Test the layer on something tiny before the real game:
- a minimal graphics test that creates a device, draws and reads the result back (no screenshot permission needed);
- a hello-world exe for the Windows API layer;
- for a Steam game, the launcher alone.

Read its log or screenshot, not the clock. If it fails, search the notes with the exact error text, then the
layer's own issue tracker and release notes for your game, engine or error text, before the forums. Write down
the date and version of anything you take from the web.

## 7. Record it
Add to `MODDING_PLAN.md` (see `SKILL.md`, step 4):
```
- Host / compat: macOS 26.x, Apple Silicon (M-series) vs Windows x64 DirectX 11 game; layer: <name + version>;
  notes read: <queries>; verified by: <probe, log or screenshot>; unverified: <list>
```
Then continue with the mod-any-game loop. If you learned something the notes lacked (a version that works, a gotcha
with its cause), write it up with the **share-field-notes** skill: that is how the next agent skips this research.

## Sources checked (2026-10-16)
- Proton as a Wine-based tool, and Easy Anti-Cheat and BattlEye running under Proton only when developers enable
  it: https://itsfoss.com/easy-anti-cheat-linux/ and https://linuxiac.com/valve-proton-7/ (2021 reports; later
  reports in 2025 describe uneven results, so check the specific game).
- DXMT (DirectX 10 and 11 over Metal, open source): https://github.com/3Shain/dxmt .
- Agent Skills format (SKILL.md limits, references/): https://agentskills.io/specification .
- Tool behaviour (`um scan` arch field, `um kb search` ranking, `objdump`, `sw_vers`, `system_profiler`, `sysctl`)
  was run on a macOS 26.5.1 / Apple M5 machine for this page.
