# Architecture Decision Records

This directory contains Architecture Decision Records (ADRs) for the Luanti engine.
ADRs document significant architectural and design decisions, capturing the context,
the decision made, and its consequences. They serve as institutional memory so future
contributors understand *why* the codebase is built the way it is.

## Format

We use [MADR](https://adr.github.io/madr/) (Markdown Any Decision Records) — a
lightweight format that is easy to write and read in plain text.

See [TEMPLATE.md](TEMPLATE.md) for the template to use when creating a new ADR.

## Creating an ADR

1. Copy `TEMPLATE.md` to a new file named `NNNN-short-title.md`, where `NNNN` is the
   next sequential number (zero-padded to 4 digits).
2. Fill in all sections. Keep each section concise.
3. Set status to `Proposed` initially; it becomes `Accepted` after contributor review.
4. Open a PR — the PR discussion is part of the decision record.

## Numbering

ADRs are numbered sequentially and never renumbered. If a decision is superseded, mark
the old ADR `Superseded by ADR-NNNN` and create a new one.

## Index

| ADR | Title | Status |
|-----|-------|--------|
| [0001](0001-cpp17-engine-language.md) | C++17 as engine language standard | Accepted |
| [0002](0002-client-server-singleplayer.md) | Client-server architecture for singleplayer | Accepted |
| [0003](0003-lua-scripting-api.md) | Lua as the modding scripting language | Accepted |
| [0004](0004-mapblock-16x16x16.md) | 16×16×16 node MapBlock chunk size | Accepted |
