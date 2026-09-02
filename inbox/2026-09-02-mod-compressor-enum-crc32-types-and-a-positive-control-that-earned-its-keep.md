# Three engine-agnostic lessons from decoding Scimitar's `.forge` in one static session

**From:** modding (`/pd`, Prince of Persia 2008, 2026-09-02, dev PC, no launch)
**For:** `/sr` — engine-agnostic; the Scimitar/Anvil specifics belong to that project's
dossier and `dev-archive/tools/forge/FORMAT.md`.

1. **The executable names its own compressor.** Before guessing an LZ variant from byte
   patterns, grep the exe for the compression library's *enum strings*: this one carried
   `LZO1X_1`, `LZO1X_999`, `LZO2A`, `LZX` as a contiguous string run, and the chunk header's
   `type = 2` picked LZO2A straight off it. Transcribing the public decoder (LZO's
   `lzo2a_d.ch` + `config2a.h` constants) then reproduced every block to the exact size with
   exact input consumption — a stronger check than any checksum, and available before the
   checksum algorithm is known. `[verified-numerically 2026-09-02, 7.90 GB]`
2. **Type/identifier hashes are often plain CRC32 of the name.** Check `zlib.crc32(name)`
   against any 32-bit value stored *beside* a name in the exe (here a state registry) before
   assuming a custom hash. Once it matches, a CRC32 dictionary built from every
   identifier-shaped string in the exe (107k) resolved 201 of 202 serialized class-type
   hashes in the data — turning an opaque blob into typed, named objects
   (`CameraRule`, `PopCharacterGraphStateDescription`, …) in one step. Applies to any
   engine with reflection-style tables.
3. **A positive control converted a wrong method into a finding instead of a false
   negative.** The plan was "search datablocks for the target state's hash". The three
   control hashes — states that certainly run in normal play — only ever hit inside audio
   data, which proved the data does not store states as hashes at all (it stores ordinals).
   Without the control, the null on the target would have been recorded as "the debug camera
   was stripped", and it is in fact authored and present. Generalises: whenever a negative
   search result would be load-bearing, run a search that *must* hit first.

Confidence tags per the estate's vocabulary; the Scimitar-specific layouts are in the
project repo, not repeated here.
