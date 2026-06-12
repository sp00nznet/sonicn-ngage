# SonicN — binary notes

Verified facts about the dump, gathered before any recompilation. Source: the No-Intro
`SonicN (USA, Europe) (En,Fr,De,Es,It) (v1.0.14)` set.

## Install layout (`system/apps/sonicn/`)

| File | Size | What it is |
|---|---:|---|
| `sonicn.app` | 1,396,120 B | **The executable** — Symbian `E32Image`, ARMv4. The recomp target. |
| `etcdata.bin` | ~1.9 MB | bulk game data (levels / tables) |
| `action_char4.bin` | ~3.6 MB | character/animation graphics (4bpp) |
| `action_char8.bin` | ~108 KB | character/animation graphics (8bpp) |
| `images.mbm` | ~49 KB | Symbian Multi-BitMap (UI / static images) |
| `volume.mbm` | ~1.2 KB | volume UI bitmap |
| `snd000.bin`…`snd099.bin` | ~180–330 KB ea | sound/music streams |
| `sonicn.rsc`, `sonicn_caption.rsc` | tiny | Series 60 resource files |
| `sonicn.aif` | ~2 KB | application info file (icons/caption) |
| `game.id`, `nokia.dat` | 0 B | N-Gage card markers |

## `E32Image` header (verified)

Read directly from the first bytes of `sonicn.app`:

| Field | Offset | Value | Meaning |
|---|---|---|---|
| `iUid1` | 0x00 | `0x10000079` | `KDynamicLibraryUid` — a Series 60 `.app` is a polymorphic DLL |
| `iUid2` | 0x04 | `0x100039CE` | `KUidApp` |
| `iUid3` | 0x08 | `0x101FB882` | SonicN's application UID |
| `iUidCheck` | 0x0C | `0x1D3282A4` | UID checksum |
| `iSignature` | 0x10 | `'EPOC'` (`45 50 4F 43`) | image magic |
| `iCpu` | 0x14 | `0x00002000` | **`ECpuArmV4`** |

First 64 bytes (hex):
```
79 00 00 10 ce 39 00 10 82 b8 1f 10 a4 82 32 1d
45 50 4f 43 00 20 00 00 de 0b 1c 9a 00 00 00 00
01 00 af 00 40 d7 c2 f3 92 a4 e0 00 03 00 00 00
e8 f7 14 00 00 00 00 00 00 10 00 00 00 00 10 00
```

Implications:
- **Uncompressed** image (no inflate step needed for v1 of the recompiler).
- **ARMv4** — the lifter's baseline ISA. Watch for **Thumb** (ARMv4T) regions.
- Because it's a DLL-form `.app`, the real entry is the exported `NewApplication()` /
  `E32Dll` path, not a flat `main` — the recompiler/HLE must model the Series 60 app
  framework startup, not just jump to an entry point.

## Next data to gather

- [ ] Full header parse: code size, data size, export count, **import table**, relocation tables, entry point.
- [ ] Import table dump → `(DLL, ordinal)` list → maps to the [Symbian HLE](https://github.com/sp00nznet/ngagerecomp/blob/main/docs/SYMBIAN-HLE.md) surface.
- [ ] Thumb vs ARM region map.
- [ ] `images.mbm` / `volume.mbm` decode (standard Symbian MBM).
