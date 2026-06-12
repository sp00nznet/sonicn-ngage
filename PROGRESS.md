# SonicN recomp — progress log

Newest entries on top. Milestones tracked toward **first frame**, then **playable**.

## Milestones

| # | Milestone | State |
|---|---|---|
| 0 | Binary identified & header verified (`E32Image`, ARMv4, uncompressed) | ✅ done |
| 1 | Full header parse (sections, imports, relocs, entry point) | ✅ done (IDA EPOC loader) |
| 2 | Import table dumped → exact Symbian symbol list (233 / 11 DLLs) | ✅ done |
| 3 | Function recovery over the code section (2,621 funcs) | ✅ done (IDA) |
| 4 | First function lifted ARM→C and round-trip verified | ⬜ |
| 5 | HLE bring-up set (heap, file read, framebuffer present, key input) | ⬜ |
| 6 | **First frame on screen** | ⬜ |
| 7 | Controllable Sonic | ⬜ |
| 8 | Sound | ⬜ |
| 9 | Playable start→first level clear | ⬜ |

## Log

### 2026-06-12 — Day 0 (cont.): IDA does the front end for free
- IDA Professional 9.1 (headless idalib) loads `sonicn.app` via its EPOC loader:
  **2,621 functions**, entry `0x100163bc` (`NewApplication()`), `start` `0x10000000`.
- **233 imports across 11 DLLs** dumped, ordinals demangled to SDK signatures →
  this is the complete HLE worklist, recorded in `docs/HLE-IMPORTS.md`.
- Hex-Rays decompiles the ARM cleanly (verified on `NewApplication()`), so IDA
  doubles as a recomp oracle. Milestones 1–3 done in one sitting.
- Pulled EKA2L1 (N-Gage emulator) as a reference oracle + HLE spec source.
- **Next:** lift the first real function ARM→C and round-trip it (Milestone 4),
  and trace the framebuffer path (NOKIAFC/BITGDI/FBSCLI).

### 2026-06-12 — Day 0: it's real
- Pulled `SonicN (…) (v1.0.14)` from the N-Gage set; extracted the Series 60 install.
- Verified `sonicn.app` is a Symbian `E32Image`: UID1 `0x10000079`, UID3 `0x101FB882`,
  signature `EPOC`, **`iCpu = ECpuArmV4`**, 1,396,120 bytes, **uncompressed**.
- Confirmed the friendly-case assumptions hold: ARMv4, no GPU, standard Symbian asset
  formats. Logged full notes in `docs/BINARY-NOTES.md`.
- Framework scaffolded in [ngagerecomp](https://github.com/sp00nznet/ngagerecomp).
- **Next:** parse the rest of the header and dump the import table (Milestones 1–2).
