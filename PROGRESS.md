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
| 4b | Control flow + stack + writeback → whole functions lift | ✅ done (100% / compiles clean) |
| 4c | HLE foundation: IAT dispatch + first shims | ✅ done |
| 4d | Heap allocator + new/delete + leave/cleanup-stack model | ✅ done |
| 4e | Function registry + dispatch (game calls itself); EFSRV file reads | ✅ done |
| 4f | Framebuffer pipeline: CFbsBitmap + DataAddress + present (45/233) | ✅ done |
| 4g | Game code runs: image loader + virtual dispatch; NewApplication executes | ✅ done |
| 4h | App init chain traced (docs/BOOT-CHAIN.md); driving into ConstructL | ✅ done (66/233) |
| 5 | Finish construction → CPeriodic tick renders a frame | 🟡 in progress |
| 6 | **First frame on screen** | ⬜ |
| 7 | Controllable Sonic | ⬜ |
| 8 | Sound | ⬜ |
| 9 | Playable start→first level clear | ⬜ |

## Log

### 2026-06-12 — Day 0 (cont.): traced the app init chain, driving the bootstrap
- **Mapped the full S60 startup** from the binary (IDA + running it) — see
  `docs/BOOT-CHAIN.md`: NewApplication → CreateDocumentL → `CSonicNAppUi::ConstructL`
  (sub_100163FC) → control `ConstructL` (sub_10016990, creates the window + the
  **CPeriodic game-loop timer**) → the tick `sub_100170D8` → `sub_10017260` = the
  game's update+render. Found the screen-rect plumbing and the 538 KB control state.
- Built the machinery to drive it: `CPeriodic::NewL`/`Start` + a frame **pump**,
  `ApplicationRect` (176×208), and the **descriptor constructors** (TPtr/TPtrC/TBuf —
  the game builds these everywhere). **66/233 shims.**
- Driving `ConstructL` now **executes into the control construction**. Added a 32-entry
  call-trace ring to the dispatcher so each fault prints the last guest addresses — this
  is how the frontier is located. Current wall: inside sub_10016990 on a buffer an
  upstream stub hasn't filled. Bring-up from here is the usual crash → fix → repeat.
- **Next:** keep walking the construction path to the first `CPeriodic` tick that
  renders into the CFbsBitmap (the framebuffer pipeline downstream already works).

### 2026-06-12 — Day 0 (cont.): the game's own code runs 🎉
- **First contact:** invoked the app's single export `NewApplication()` (@0x100163bc) —
  SonicN's own (recompiled) code executed: `operator new` its app object, ran the ctor,
  installed the vtable, returned a valid object at 0x10800008. Only 1 stubbed call, benign.
- Two pieces unlocked it:
  - **Image loader** (`image.c` + `gen_image.py` → `segments.bin`): loads the image's data
    segments (vtables/const pools/jump tables) into guest memory — the lifted code reads
    these by address. Without it the vtable read returned 0.
  - **Virtual dispatch** (`ngage_vcall`): reads an object's vtable and calls the slot.
- Drove the next step: **`CreateDocumentL` dispatches correctly through the vtable** (+0x10
  → 0x100ed104). It lands on the framework thunk, so the real document/appui/control
  override chain is the next trace.
- **Next:** walk that chain + stand up the active scheduler and the game's `CPeriodic`
  tick (its render loop) → `CCoeControl::Draw`, which feeds the (working) framebuffer.

### 2026-06-12 — Day 0 (cont.): pixels flow — the framebuffer pipeline works
- Mapped SonicN's render path from its imports: it creates a **CFbsBitmap**, gets its
  raw buffer via **DataAddress()**, software-renders into it, then flips via **NOKIAFC**
  (BITGDI has no draw calls imported → the game does its own pixels).
- Built it: `hle/fbserv.c` (CFbsBitmap Create/DataAddress/SizeInPixels/DisplayMode/
  Header, BitGc stubs, NOKIAFC flip) + `framebuffer.c` (EColor4K/64K/Gray256/16MU →
  RGB888 presenter, DWORD-aligned scanlines). **45/233 shims.**
- Verified the pipeline end-to-end: created a 176×208 EColor64K bitmap via the HLE,
  software-rendered an RGB565 gradient into DataAddress(), flipped → a correct image
  (see ngagerecomp `docs/pixel-pipeline-proof.png`). Whole program still links.
- Honest scope: this proves the *pipeline*; SonicN's own draw code only runs after the
  **S60 app framework boots** (active scheduler + CONE/EIKCORE/AVKON) — the next block.

### 2026-06-12 — Day 0 (cont.): the game reads its own assets
- **Function registry** (`gen_register.py`): all 2,621 lifted functions register at their
  guest addresses; dispatch upgraded to lazily-sorted binary search. Verified the game
  calls its own function (memset) by address through dispatch.
- **EFSRV file reads** (`hle/efsrv.c` + `desc.c`): `RFs::Connect`, `RFile::Open`/`Read`/
  `Size`/`Seek`/`Close` over a host-mounted root, with full Symbian descriptor decode
  (TBufC/TPtrC/TPtr/TBuf). Verified by opening the real `volume.mbm`: size 1239 matches,
  and 64 read bytes match the file byte-for-byte. **32/233 shims.**
- Full integration: 2,621 game fns + runtime + HLE link into one 6.3 MB executable.
- **Next:** the framebuffer path (NOKIAFC full-screen + BITGDI/FBSCLI) and input →
  first frame.

### 2026-06-12 — Day 0 (cont.): heap + leave model; whole game links
- Implemented the EUSER **heap** (`heap.c`: guest allocator) and **new/delete** shims
  (`CBase::operator new` zeroed, `User::AllocL`, `operator new[]`/`delete`), plus the
  **leave / cleanup-stack** machinery (`kernel.c`: setjmp/longjmp trap + cleanup stack)
  and `LeaveIfError`/`Trap`/`UnTrap`/`CleanupStack::*`/`Exit`/`Panic`. **19/233 shims.**
- Verified: alloc/free/reuse; a `LeaveIfError(-4)` deep in a call unwinds through
  `ngage_run`, runs cleanup (frees the pushed block), returns the code.
- **Whole-program proof:** all 2,621 lifted functions + runtime + HLE link into one
  6.2 MB executable that runs its init path (heap + wiring all 233 imports).
- Honest note: nested guest `TRAP` recovery still routes leaves to the outermost
  handler (needs an inline setjmp at the TRAP call site, a lifter hook) — fine for the
  happy path. **Next:** EFSRV file reads to load SonicN's `*.bin` assets.

### 2026-06-12 — Day 0 (cont.): 100% lift + HLE foundation
- Closed the last stubs: register-amount shifts (`ngage_lsl/lsr/asr/ror`), ABI register
  aliases (ip/fp/sl/sb), `ldr pc` jump tables → dispatch, goto-into-chunk fix.
  **All 2,621 functions / 223,868 instructions lift; the whole 247k-line corpus
  compiles under `clang -Wall` with no errors/warnings.** (Compiling the corpus caught
  two bugs the coverage % missed — that's the real gate.)
- **Started HLE.** Dumped the 233-slot import address table (`extract_imports.py`),
  generated the wiring (`gen_hle.py`): each slot points to itself and dispatches to a
  shim; unimplemented imports log their real Symbian name. First real shims
  (`memcpy`/`memset`/`Mem::FillZ`) verified by calling `memcpy` through the game's own
  `ldr r12,[slot]; blx r12` path — it copied correctly.
- **Next:** EUSER heap + the `TRAP`/cleanup-stack leave model, then EFSRV reads to load
  the `*.bin` assets, then framebuffer → first frame.

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
