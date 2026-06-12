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

## Tooling note

The dispatcher keeps a 32-entry call-trace ring (`ngage_trace`) so a fault prints the
last guest addresses dispatched — that's how each crash site is located. The current
frontier is inside `sub_10016990` (control construction) using a buffer that an upstream
stub hasn't populated yet.
