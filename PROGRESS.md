# SonicN recomp — progress log

Newest entries on top. Milestones tracked toward **first frame**, then **playable**.

## Milestones

| # | Milestone | State |
|---|---|---|
| 0 | Binary identified & header verified (`E32Image`, ARMv4, uncompressed) | ✅ done |
| 1 | Full header parse (sections, imports, relocs, entry point) | ✅ done (IDA EPOC loader) |
| 2 | Import table dumped → exact Symbian symbol list (233 / 11 DLLs) | ✅ done |
| 3 | Function recovery over the code section (2,621 funcs) | ✅ done (IDA) |
| 4 | First function lifted ARM→C and round-trip verified | ✅ done |
| 4b | Control flow + stack + writeback → whole functions lift (97.8% / 99.94%) | ✅ done |
| 5 | HLE bring-up set (heap, file read, framebuffer present, key input) | ⬜ (unblocked) |
| 6 | **First frame on screen** | ⬜ |
| 7 | Controllable Sonic | ⬜ |
| 8 | Sound | ⬜ |
| 9 | Playable start→first level clear | ⬜ |

## Log

### 2026-06-12 — Day 0 (cont.): whole functions lift; firmware in hand
- Added **control flow** to the lifter: intra-function branches → labels/`goto`,
  conditional branches → `if (cond) goto`, calls → `ngage_call()` dispatch, returns.
  Plus PUSH/POP/LDM/STM, pre/post-index writeback, MUL/MLA/SMULL/UMULL, standalone
  shifts, signed loads, and condition/`S`-suffix stripping.
- **Whole-binary coverage: 97.8% of all 2,621 functions, 99.94% of 223,868
  instructions, zero exceptions.** Residual 141 stubs = register-amount shifts.
- Proof: SonicN's `memset` (`sub_100E76E4`) and the dispatch path both compile under
  `clang -Wall` and **execute correctly** (fills N bytes, `r0` post-increments, the
  `count==0`/`bxle` early-return works; `ngage_call` resolves via the table).
- Firmware arrived: an EKA2L1 N-Gage (rh-4) drive with all 10 core Symbian DLLs
  (`euser`/`efsrv`/`fbscli`/`bitgdi`/`cone`/`eikcore`/`avkon`/`apparc`/`estlib`/`ws32`)
  and the S60v1 N-Gage ROM — so the **reference oracle** can boot SonicN and the **HLE
  ordinal map** is now sourced directly. See `docs/ASSETS.md`.
- **Next:** register-amount shifts to close the last 141; then HLE bring-up
  (heap/file/framebuffer/input) toward first frame.

### 2026-06-12 — Day 0 (cont.): first ARM→C lift round-trips ✅
- Built the lifter in [ngagerecomp](https://github.com/sp00nznet/ngagerecomp): `extract.py`
  (IDA bridge → functions + segment bytes) and `lift.py` (Capstone ARM→C, no license).
- Lifted SonicN's `sub_1000BD34` (ldr-literal / mov / strh / bx lr) to C; it compiles
  under `clang -Wall` and executes correctly — the halfword store lands at `r0+0x29a`.
  PC-relative literal-pool loads are folded to constants from the image. **Milestone 4 done.**
- Coverage on the ≤200B function set (1,270 funcs): **70% of instructions**, **27% of
  functions** lift with zero stubs. Top gaps = branches/calls, then push/pop/ldm/stm.
- **Next:** control flow (intra-function branches as labels, calls via dispatch) so
  whole non-leaf functions lift; then trace the framebuffer path.

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
