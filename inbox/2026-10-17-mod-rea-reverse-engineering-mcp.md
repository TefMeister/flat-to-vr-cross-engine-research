# Tool lead: REA (Reverse Engineer Anything), an MCP for reverse engineering

From: modding session, home PC, 2026-10-17 (Tefa sent the link). `[reported]` -- read from the README and docs only,
not installed, not run.

- Repo: https://github.com/morluto/rea (MIT, npm package `rea-agents`, very active). Site: https://rea.tools
- What it is: one MCP server (plus a `rea` command line) that hands an agent findings with evidence. Native binaries go
  through an existing Hopper, Ghidra or IDA install (pseudocode, assembly, strings, symbols, calls, references, data
  types, call-target resolution). Also: .NET assemblies (metadata, CIL; static, no install needed), JavaScript and
  Electron apps, websites, Android APKs, WASM, SQLite, packet captures.
- Setup: `npx rea-agents setup` registers it with Claude Code (edits the agent's MCP config, with backups).
- Relevance to flat-to-VR work:
  - Overlaps what the estate already has for native code (ghidrust MCP for static work, x64dbg MCP for live work,
    the toolkit's `static-disasm.py`). Worth a side-by-side only where ghidrust has failed: on 2026-10-10 its
    decompiler hung on every re2.exe function (RE2 board).
  - Its .NET assembly analysis needs nothing installed: possibly useful for Unity (Mono) games, where the game's
    code ships as managed assemblies.
  - No game-engine-specific features (no RE Engine, Unreal or Unity type-database support).
- Status: not installed. Installing an outside tool needs Tefa's yes (the /gu rule).
