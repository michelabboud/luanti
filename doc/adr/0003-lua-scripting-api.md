# ADR-0003: Lua as the modding scripting language

Date: 2011-01-01
Status: Accepted

## Context

Luanti needed an embedded scripting language to expose a modding API without
requiring modders to write or compile C++. Key requirements:

- Easy to embed in a C++ application with a small footprint
- Simple enough for non-professional game developers and content creators to learn
- Fast enough for game logic running every server tick (~0.1 s intervals)
- Sandboxable to limit what mods can do (no raw filesystem or network access by default)
- Permissive license compatible with LGPL-2.1

Alternatives considered: Python, JavaScript (V8), Squirrel, AngelScript.

## Decision

Use Lua 5.1 as the modding scripting language, with optional LuaJIT as a drop-in
accelerator. A vendored copy of Lua 5.1 is kept in `lib/lua/` as a fallback when
system Lua is unavailable. LuaJIT is preferred at runtime when available
(`-DENABLE_LUAJIT=ON`, the default).

Client-side Lua (SSCSM — Server-Sent Client-Side Mods) runs in a separate restricted
environment on the client; the main mod API runs on the server.

## Consequences

**Positive:**
- Lua is tiny: the full interpreter compiles to ~200 KB, negligible compared to engine size
- LuaJIT delivers near-C performance for compute-heavy mod code via JIT compilation
- Lua's coroutine model maps well to game entity and event loop patterns
- The sandboxed environment (`src/script/`) can expose exactly the API surface we choose
- Large existing Lua ecosystem; modders familiar with Love2D, ComputerCraft, etc. transfer skills
- MIT-licensed Lua is compatible with Luanti's LGPL-2.1 license

**Negative / trade-offs:**
- Lua 5.1 is an older standard; some modern Lua idioms (integers, bitwise ops) require
  compatibility shims (`lib/bitop/`) or LuaJIT extensions
- No static typing makes large mod codebases harder to maintain; no built-in type checker
- Bridging C++ ↔ Lua has non-trivial overhead for frequently-called callbacks
- Two scripting environments (server + client SSCSM) increase API surface complexity

**Neutral / notes:**
- The Lua API is documented in `doc/lua_api.md` (~500 KB)
- `src/script/cpp_api/` contains the C++ side of all Lua bindings
- `src/script/lua_api/` defines all Lua-facing modules
- Separate scripting environments exist for: server mods, client SSCSM, main menu, pause menu
