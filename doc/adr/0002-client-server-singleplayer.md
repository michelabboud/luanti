# ADR-0002: Client-server architecture for singleplayer

Date: 2011-01-01
Status: Accepted

## Context

Most voxel games of the era (2010-2012) used a split codebase: one code path for
singleplayer (direct, no network layer) and another for multiplayer (networked).
This approach is simpler to implement initially but creates a persistent maintenance
burden as features must be duplicated or conditionally compiled.

Luanti needed to support:
- Offline singleplayer with full mod API access
- Local area network (LAN) play with friends joining easily
- Dedicated server mode for public servers
- A consistent modding API that works identically in all play modes

## Decision

Even in singleplayer mode, Luanti always runs a local server (`Server` class) and a
local client (`Client` class) that communicate over the same network protocol, using
loopback or in-process message passing. There is no "offline-only" code path in the
game logic layer.

## Consequences

**Positive:**
- Mod API is identical across singleplayer, LAN, and dedicated server — mods written for
  one mode work in all modes without modification
- LAN multiplayer is trivially enabled: singleplayer worlds are already hosted on a server
- Bug fixes to the server-side game logic automatically fix both singleplayer and multiplayer
- Simplifies testing: unittests can spin up a server without a display or network stack

**Negative / trade-offs:**
- Higher overhead for singleplayer compared to a direct code path (loopback serialization,
  thread synchronisation between client and server threads)
- Longer startup time in singleplayer because a full server must initialise
- More complex state to reason about: client state and server state are separate, requiring
  careful synchronisation via the network protocol

**Neutral / notes:**
- The `Server` class lives in `src/server.h`; `Client` lives in `src/client/client.h`
- Network protocol opcodes are defined in `src/network/networkprotocol.h`
- The in-process connection uses a shared-memory loopback transport to reduce latency
