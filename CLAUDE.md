# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Luanti (formerly Minetest) is a free, open-source voxel game engine written in C++17 with Lua scripting support. It supports client and server modes, cross-platform builds (Linux, Windows, macOS, Android), and extensive modding via Lua APIs.

## Build Commands

```bash
# Configure and build (debug, run-in-place)
cmake -B build -DCMAKE_BUILD_TYPE=Debug -DRUN_IN_PLACE=TRUE -DBUILD_SERVER=TRUE
cmake --build build --parallel $(nproc)

# Or use the CI script (sets Debug, RUN_IN_PLACE=TRUE, BUILD_SERVER=TRUE)
./util/ci/build.sh

# Run the client
./bin/luanti

# CMake presets available: Debug, Release, RelWithDebInfo, MinSizeRel
cmake -B build --preset Debug -DRUN_IN_PLACE=TRUE
```

## Testing

```bash
# C++ unit tests (Catch2-based, built by default with BUILD_UNITTESTS=TRUE)
./bin/luanti --run-unittests

# Run a specific test module
./bin/luanti --run-unittests --test-module <module_name>

# Benchmarks (must enable with -DBUILD_BENCHMARKS=TRUE)
./bin/luanti --run-benchmarks

# Integration tests (require built binary + devtest game)
./util/test_multiplayer.sh
./util/test_singleplayer.sh
./util/test_error_cases.sh
```

## Linting

```bash
# C++ linting (clang-tidy) - checks modernize-*, performance-*, misc-* rules
./util/ci/clang-tidy.sh

# Lua linting
luacheck builtin/
```

## Code Style

- **Indentation**: tabs (size 4) for C++, Lua, GLSL, CMake files; spaces (size 4) for Markdown
- **Charset**: UTF-8, LF line endings
- **C++ standard**: C++17 (GCC 7.5+ or Clang 7.0.1+)
- **C++ style guide**: https://docs.luanti.org/for-engine-devs/code-style-guidelines/
- **Lua style guide**: https://docs.luanti.org/for-engine-devs/lua-code-style-guidelines/
- **clang-tidy** enforces: `modernize-use-emplace`, `modernize-avoid-bind`, `performance-*`, `misc-throw-by-value-catch-by-reference`

## Architecture

### Core Engine (src/)

The engine has a client-server architecture, even in singleplayer (where a local server runs):

- **Client** (`src/client/`): Rendering, input, GUI, camera, HUD. Main class: `Client` in `src/client/client.h`
- **Server** (`src/server/`): Game logic, entity management via Server Active Objects (SAO), ban/rollback. Main class: `Server` in `src/server.h`
- **Network** (`src/network/`): Custom binary protocol with packet opcodes. Client/server packet handlers in `clientpackethandler.cpp`/`serverpackethandler.cpp`
- **Map/World** (`src/`): Voxel world using MapBlocks (16x16x16 node chunks). Key files: `map.h`, `mapnode.h`, `mapblock.h`, `servermap.h`
- **Map generation** (`src/mapgen/`): Multiple algorithms (v5, v6, v7, flat, fractal, carpathian, valleys, singlenode) plus cave/dungeon/tree/biome/ore/decoration generators
- **Scripting** (`src/script/`): Lua integration layer. `cpp_api/` implements C++ side, `lua_api/` defines Lua-facing modules. Separate scripting environments for server, client, main menu, and pause menu
- **GUI** (`src/gui/`): FormSpec-based UI system for in-game menus and HUD
- **Database** (`src/database/`): Pluggable storage backends (SQLite3, LevelDB, PostgreSQL, Redis, files)
- **Content** (`src/content/`): Mod and game (subgame) loading system

### Lua Side

- **builtin/** (`builtin/`): Core Lua code that implements game logic, API helpers, main menu. Split into `game/`, `client/`, `mainmenu/`, `common/`
- **Mod API**: Documented extensively in `doc/lua_api.md` (~500KB). Mods go in `clientmods/` or `mods/`

### Rendering

- **IrrlichtMt** (`irr/`): Custom fork of Irrlicht 3D engine used for rendering
- **Shaders** (`client/shaders/`): GLSL shaders for client rendering

### External Libraries

Vendored in `lib/`: Catch2 (testing), jsoncpp, lua/luajit, gmp, sha256, tiniergltf, bitop

## Key CMake Options

| Option | Default | Purpose |
|--------|---------|---------|
| `BUILD_CLIENT` | TRUE | Build the game client |
| `BUILD_SERVER` | FALSE | Build dedicated server |
| `BUILD_UNITTESTS` | TRUE | Build unit tests |
| `BUILD_BENCHMARKS` | FALSE | Build benchmarks |
| `RUN_IN_PLACE` | FALSE | Run from source directory (set TRUE for development) |
| `ENABLE_LTO` | varies | Link-time optimization |
| `ENABLE_LUAJIT` | ON | Use LuaJIT instead of bundled Lua 5.1 |

## Commit Conventions

- Present tense, descriptive messages
- First line: capitalized, <70 chars, no trailing period
- Second line: empty
- Subsequent lines: detailed explanation
- Non-bugfix PRs need concept approval from core developers
- Linear history via rebase (not merge commits)
- Do NOT manually update translation files (`updatepo.sh`, `luanti.pot`) or `minetest.conf.example`
