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
| 4h | App init chain traced (docs/BOOT-CHAIN.md); driving into ConstructL | ✅ done |
| 4i | Drive ConstructL deep: LDM bug fix, soft-float, descriptors, panic | ✅ done (88/233) |
| 5 | App+control ConstructL completes; game loop runs; render code executes | ✅ done (90/233) |
| 6 | Real asset (MBM) loading + game progression → gameplay graphics | 🟡 next |
| 6 | **First frame on screen** | ⬜ |
| 7 | Controllable Sonic | ⬜ |
| 8 | Sound | ⬜ |
| 9 | Playable start→first level clear | ⬜ |

## Log

### 2026-06-12 — Day 0 (cont.): game runs 1400 frames; frontier is the sprite engine
- Added a **soft memory-guard mode** (OOB reads → 0, writes dropped, both counted) to run
  *past* the sprite-engine null derefs instead of crashing. The game then runs **1400+
  frames, stable**.
- But the screen stays blank, with a burst of **~880k OOB reads** in the sprite blitter
  (`sub_1000C760` → `sub_100EBCC4`), all from a **null source pointer** — the sprite
  descriptors have null pixel-data pointers, i.e. the loaded `action_char*.bin` assets
  aren't linked into the sprites.
- That linkage is deep, SonicN-specific RE (its sprite/tile data format + asset→descriptor
  mapping); the real screen surface may also be the NOKIAFC framebuffer, not the bitmap we
  capture. Everything generalizable — CPU, boot, game loop, timing, asset I/O, pixel
  pipeline — is proven with real game code. **Gameplay graphics are now a per-game effort.**

### 2026-06-12 — Day 0 (cont.): the game advances + loads its real assets
- Found the timing was frozen: **`User::TickCount` was a constant stub**, so the game's
  per-frame time deltas were always 0 — animations/state transitions never advanced.
  Implemented `TickCount` (+`User::After`) to advance one tick/frame.
- The game now **progresses through its init states and loads its real graphics assets**:
  `action_char4.bin` (3.6 MB chars), `action_char8.bin`, `etcdata.bin` (levels), sound —
  all read fine through EFSRV — then runs its sprite/tile-setup code (`sub_1000B188`).
- **Next:** the sprite-setup path hits a null-deref (an HLE gap deep in the gameplay data
  path); fixing it is the road to recognizable graphics. Everything before it — boot, game
  loop, asset I/O, timing, the pixel pipeline — is proven with real game code.

### 2026-06-12 — Day 0 (cont.): the recompiled game runs its render loop 🎉🎉
- Gave the graphics objects (CFbsBitmapDevice via BITGDI ord 170, CFbsBitGc via
  CreateContext, and CFbsBitmap itself) **synthetic no-op vtables** so their virtual calls
  and deletes resolve instead of derefing null vtables.
- Result: the tick now runs cleanly for **300 frames with no crash**, and the game
  **renders into a 256×256 CFbsBitmap backbuffer** (118 DataAddress calls) which the
  present pipeline captures as a real frame.
- **Honest state:** the frame is currently **blank (white)** — early/loading state; sprite
  bitmaps are empty because CFbsBitmap::Load doesn't decode the real `.mbm`/`.bin` assets
  yet, and the game hasn't advanced past init (gated by the still-stubbed active-object
  machinery). The pixel pipeline is proven end-to-end with real game code.
- **Next:** real asset loading (MBM decode) + active-object/RTimer completion so the game
  advances to actual gameplay rendering.

### 2026-06-12 — Day 0 (cont.): ConstructL COMPLETES; the game tick runs 🎉
- Mapped `FBSCLI ord 156` = `CFbsBitmap::Load` (had mis-labeled it Connect). Implemented it
  to set the bitmap up as EColor4K → the bitmap-load path stops crashing and **the control's
  `ConstructL` runs to completion (`rc=0`)** and registers the **`CPeriodic` game-loop timer**.
- Pumping the tick surfaced the need for **jump-table lowering**: `ldr pc,[pc,rN,lsl#2]`
  switches were routed through dispatch with a stray `return` (skipping the epilogue → ABI
  violation). Implemented proper lowering — read the constant target table from the image,
  emit a C `switch` with local `goto`s. **85 tables across the binary**; also fixed the
  conditional `bxne` indirect-call case.
- Result: the whole app + control construction completes and **the game tick executes**,
  now faulting on a null-deref deeper in the render path (HLE grind continues, inside real
  game rendering). An **ABI-invariant check** (`-DNGAGE_ABI_CHECK`) drove all of this.

### 2026-06-12 — Day 0 (cont.): blew through the audio AO into control ConstructL
- The active object was a **`CMdaAudioOutputStream`** (audio). Implemented it as an HLE
  object with a synthetic no-op vtable (`ngagerecomp` `hle/media.c`) → past the panic.
- Then found **two more real lifter bugs** by running real code (both affect any game):
  - **scaled index dropped**: `ldr r0,[r6,r5,lsl #2]` emitted `r6+r5` (Capstone puts the
    shift on the operand, not `mem.lshift`). Hit on array indexing.
  - **ARMv4 indirect call as tail-return**: `mov lr,pc; bx ip` was lifted as `…;return;`,
    skipping the epilogue that restores r4–r11. Added an **ABI-invariant check** to the
    dispatcher (flags any callee that alters r4–r11/sp) which pinpointed it instantly.
- Result: the boot now runs through the entire audio AO + the 538 KB sub-object construction
  and into the **control's `ConstructL` (`sub_10016990`)**, which creates the game's
  `CFbsBitmap`s. Next fault there is an HLE gap (a null graphics object), not a lifter bug.
  **89/233 shims.**

### 2026-06-12 — Day 0 (cont.): isolated the active-object frontier
- Fixed another real bug: **`TTrap::Trap` wasn't writing `*aResult`**, so `TRAPD` loops saw
  a non-zero error and retried forever. Now sets `KErrNone`.
- Upgraded the debugger from a recency ring to a **true call stack** (`ngage_stack` /
  `ngage_calldepth`), and made `User::Panic` print the caller→callee chain.
- With it, isolated the exact frontier: a `CActive`-derived **active object** built in
  `sub_10018D4C` inside a `TRAP` retry loop; its teardown reaches `RunL` (`sub_100EB58C`)
  with the wrong state → panic. Full chain documented in `docs/BOOT-CHAIN.md`.
- Verdict: this needs a **real active-object runtime** (Cancel→DoCancel, request/complete,
  RunL-on-completion) + the nested-`TRAP` lifter hook — the next substantial, design-heavy
  block. Everything downstream (CPeriodic tick → render → framebuffer) is already built.

### 2026-06-12 — Day 0 (cont.): driving ConstructL deep — found a real lifter bug
- Ran `AppUi::ConstructL` under a memory-bounds guard. It surfaced a wild read at
  `0x3FE00004`, which a 32-entry dispatch call-trace pinned to **a lifter bug**:
  `ldm r9,{r9,r10}` loaded `r9` first, clobbering the base before the 2nd address —
  so it read `[loaded_value+4]`. Fixed in the lifter (snapshot the base into a temp);
  affects any binary, re-lifted the whole corpus.
- Implemented the **soft-float / integer-divide runtime** (`hle/softfloat.c`, 22 libgcc
  helpers — the constructor does double math) and the **descriptor constructors**.
- Found a runaway recursion: **`User::Panic` was returning** (it's fatal in Symbian),
  so the active-object retry loop spun forever → fixed Panic to unwind.
- Result: ConstructL now runs deep (alloc → descriptors → sub-object construction →
  float math) and **panics cleanly at the active-object state check**, unwinding via
  `ngage_run`. **88/233 shims.** Frontier = the active-object/async machinery (stubbed
  scheduler never advances `CActive` state) + the nested-`TRAP` hook.
- **Next:** real `CActive` request/complete + `RTimer` firing + run loop so construction
  completes and the `CPeriodic` tick renders.

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
