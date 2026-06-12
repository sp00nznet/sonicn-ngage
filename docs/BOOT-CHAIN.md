# SonicN — app init chain (traced)

The S60 application startup, mapped from the binary (IDA + driving the recompiled code).
Game functions are `sub_…`; framework calls are Symbian imports (HLE).

```
NewApplication()                         0x100163BC  (the .app's single export)
  └─ operator new(556) + CEikApplication ctor + install vtable @0x1014EF30
     vtable: [4]+0x10 = CreateDocumentL, [5]+0x14 = AppDllUid, [2]+0x08 = dtor

CreateDocumentL(CApaProcess*)            vtable +0x10        → CEikDocument-derived
  └─ CreateAppUiL()                                          → CSonicNAppUi

CSonicNAppUi::ConstructL()               sub_100163FC   ◄── the heart of init
  ├─ NOKIAFC_1(&cfg, langData)           full-screen mode init (language-selected)
  ├─ CAknAppUi::BaseConstructL(0)
  ├─ op new(551200) + CCoeControl ctor   the main game control (≈538 KB of game state)
  │    └─ sub_10002BD4 / sub_10009080 / sub_10018D4C   (sub-object constructors)
  ├─ store control @ appui+96
  ├─ CONE_315(control, appui+28)
  ├─ ApplicationRect(&rect)              → 0,0,176,208
  ├─ control->ConstructL(rect)           sub_10016990   ◄── window + game loop
  │    ├─ CCoeControl::CreateWindowL()
  │    ├─ CONE_318(control, rect)
  │    ├─ vcall control.vtable[+0x20](control)           (activate/size)
  │    ├─ control[+100] = CPeriodic::NewL(0)             the game-loop timer
  │    ├─ sub_10017024(control)  → CPeriodic::Start(1000us, cb=sub_100170D8)
  │    └─ … initialise ≈538 KB of game state (offset +64 = 176 = screen width) …
  ├─ AddToStackL(control)
  └─ SetKeyBlockMode(1)

— then the active scheduler would run; the CPeriodic fires each tick —

game tick                                sub_100170D8  → sub_10017260   (update + render)
  renders into a CFbsBitmap, flips via NOKIAFC → the screen
```

## Where the runtime is

- `NewApplication()` runs end-to-end; `CreateDocumentL` dispatches through the vtable.
- Driving `ConstructL` directly (over a fresh AppUi + the [`ngagerecomp`] `CPeriodic`
  pump) gets **into the control construction**; bring-up is now the usual crash-driven
  grind — each missing framework detail surfaces as a fault, gets implemented, repeats.
- Implemented along the way: `CPeriodic::NewL`/`Start` + a pump, `ApplicationRect`,
  the descriptor constructors (TPtr/TPtrC/TBuf), and the active-scheduler no-ops.

## Frontier: an active-object lifecycle (sub_10018D4C)

One of the control's sub-constructors builds an **active object** (a `CActive`-derived
43488-byte object) inside a `TRAP`-guarded retry loop and operates on it. With the
scheduler stubbed, its state machine doesn't advance and a teardown path reaches `RunL`
with the wrong state, hitting `User::Panic` (reason 1). True call chain at the panic:

```
ConstructL → sub_10018D4C → sub_100E7A10 → sub_100E7A58 → sub_100EB28C
  → sub_100ED7B0 → sub_100EB760 → sub_100E876C   (TRAP retry loop)
  → sub_100EB1F8 (active-object dtor) → sub_100ED7A8 → sub_100EB58C (RunL) → Panic
```

`sub_100E876C` is `TRAPD(err, { obj = new …; }); if(!err) TRAPD(err, sub_100EB28C(obj)); while(err);`
— a construct-then-operate retry. `sub_100EB58C` is the active object's `RunL`: it
asserts `*(this+32) == 1` (the request was issued) before advancing the state to 2.

Reaching this needs a **real active-object runtime**: `CActive::Cancel`→`DoCancel`,
request issue/complete, and a run loop that dispatches `RunL` only on completion — plus
the nested-`TRAP` lifter hook (the retry loop relies on its own `TRAP` catching leaves).
That's the next substantial block; the rest of the pipeline below it (the `CPeriodic`
tick → render → framebuffer) is already in place.

## Tooling note

Bring-up uses two debug facilities in the runtime (`-DNGAGE_MEM_GUARD`): a guest-memory
bounds reporter, and a real **call stack** (`ngage_stack`/`ngage_calldepth` in
`dispatch.c`) that prints the caller→callee chain at a fault or panic — how the chain
above was recovered. A 32-entry trace ring and a runaway-recursion guard round it out.
