---
name: game-recon
description: Figure out how a specific installed game can be modded, before writing any mod code. Covers where it's installed, the engine and version, managed vs native code, anti-cheat, loaders already present, save and config folders, and the community's modding route. Also checks whether the game can run on this machine (the host's OS and CPU against the game's platform) and how to find notes for a cross-platform case. Use when the user asks "can I mod <game>?", "what engine is <game>", "how do people mod <game>", or at the start of any modding task. Produces MODDING_PLAN.md.
---

# Game recon

Goal: in about five minutes, know what you're dealing with and which route to take. Write the answer down
so every later step can rely on it.

## 0. Has another agent been here?
`um kb search "<game>"` (and the engine name). A field note in the shared knowledge base can hand you the
working versions, the route, and the gotchas before you touch anything. See the **share-field-notes** skill.
If the game can't run on this machine as it is (a Windows-only game on a Mac or Linux box, an x86 game on an
ARM CPU), step 1b says what else to search.

## 1. Find it and fingerprint it
```bash
um scan --list                 # Steam, Epic and Xbox installs (Windows, WSL, Linux, macOS)
um scan "<name or folder>"     # engine, version, exes (.NET?), anti-cheat, loaders, mod folders, saves, routes
um scan "<game>" --json        # the same, machine-readable
```
`um` lives at `bin/um` in the universal-modder repo (plugins and clones put it on PATH). Anywhere else:
`uv tool install git+https://github.com/rehan-remade/universal-modder`.

`um scan` reads files only. It indexes the install (bounded), sniffs PE headers, the Unity/Godot/GameMaker
headers and the Unreal version string, maps known games to their community loader, and points to the
playbook to read: `skills/mod-any-game/references/engines/<engine>.md`.

The scan can't see everything, so check these by hand:
- **Game not in a store library** (GOG, itch, a standalone folder): pass the folder path.
- **Several engines' signals:** launchers and web helpers are common. The top score wins, but read the
  "also" line.
- **Anti-cheat installed elsewhere:** kernel drivers (Vanguard's `vgk.sys`) and launcher-level protection
  don't live in the game folder. Search "<game> anti-cheat".
- **Online-only or live-service games:** treat them as protected even if nothing was detected.

## 1b. Does it run on this machine?
Do this when the host and the game's platform differ (a Windows-only game on a Mac or Linux box, an x86 game
on an ARM CPU), or when the OS build, GPU, driver or layer version could matter even on the same platform.
Fast path:
1. **Compare** host and game. `um scan` prints each exe's CPU (`[x64]`, `[x86]`, `[arm64]`); add the OS
   build, the GPU and the graphics API the exe imports.
2. **Search the notes** by situation and by the exact error text, before researching (spell the API out:
   `dx11` misses where `directx 11` hits):
   ```bash
   um kb search "windows game on mac"
   um kb search "directx apple silicon"
   um kb search "<error text>"
   ```
   A matching note: follow its setup and versions, then verify.
3. **No match?** Probe the layer on something tiny before the real game, and record exact versions on the
   `Host / compat` line below.

Anti-cheat is a stop sign (`skills/mod-any-game/references/safety.md`). The route ladder, the layers (CPU,
Windows API, graphics API, launcher), same-OS mismatches (OS build, GPU, driver, layer version, runtimes)
and the sources are in `skills/game-recon/references/compatibility.md`.

## 2. Research the living community (always; versions move)
Search, in order:
1. "<game> modding" / "<game> mod loader" / "<game> modding wiki". The game's wiki often has a modding page.
2. Nexus Mods (most popular mods show which frameworks they depend on), Thunderstore (Unity games: BepInEx
   packs), mod.io, the Steam Workshop (does the game have one? `um scan` shows installed Workshop content).
3. GitHub: "<game> mod", "<game> modding api", "<game> decompile", "<game> sdk", "<engine> mod loader".
4. Recent posts (X/Reddit/Discord announcements) for new frameworks. The loader might be days old.

Capture: the loader's name, repo, **current version and install steps**, the game versions it supports, a
"hello world" example mod, and where logs go.

## 3. Decide
- **Can it be modded safely?** Look at anti-cheat, online-only parts, the EULA or mod policy, and the
  ownership checks that loaders rely on (see `skills/mod-any-game/references/safety.md`). If not: say so,
  and offer what is possible (official tools, offline modes, a different game with the same idea).
- **Route:** loader API > data/asset mod > managed patching > native hooks > reimplementation/mashup. Pick
  the first that reaches the user's idea.

## 4. Write MODDING_PLAN.md
```markdown
# <Game> modding plan
- Install: <path> (<store> <appid>), version <x>
- Engine: <engine + version>, code: managed .NET / IL2CPP / native, 64-bit
- Anti-cheat / online: <none | what + verdict>
- Host / compat: <host OS+CPU> vs <game platform+API>; layer: <native | compat layer | VM>;
  notes read: <queries>; verified by: <probe, log or screenshot>; unverified: <list>
- Saves: <path>   Config: <path>   Logs: <path>
- Community route: <loader vX.Y (repo)>, install: <steps>, example mod: <link>
- Chosen route for "<idea>": <route> because <reason>
- Lab plan: backup <folders> (um backup), lab profile <how>, windowed <how>
- Unknowns to resolve first: <list>
```
Then continue with the mod-any-game loop (lab setup → source of truth → vertical slice).
