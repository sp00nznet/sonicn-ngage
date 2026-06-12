# SonicN · N-Gage → Native

**Static recompilation of _SonicN_ (Nokia N-Gage, 2003) into a native executable, using [NGageRecomp](https://github.com/sp00nznet/ngagerecomp).**

> In 2003 Sega put Sonic on Nokia's taco phone. *SonicN* is essentially *Sonic Advance* re-tuned for a 176×208 screen, a numeric keypad, and a 104 MHz ARM with no graphics chip. It's a great first recomp target precisely because it's small, software-rendered, and a known quantity. This repo tracks the journey of turning it into code that runs anywhere.

This is the **first game port** built on NGageRecomp — the proving ground for the framework. The reusable tooling lives in the [ngagerecomp](https://github.com/sp00nznet/ngagerecomp) repo; this repo is SonicN-specific: the per-game config, the binary analysis, and the running progress log.

## The target

| | |
|---|---|
| Game | SonicN (Sega / Sonic Team, 2003) |
| Platform | Nokia N-Gage (Symbian OS 6.1 / Series 60 v1) |
| CPU | ARM925T @ 104 MHz, ARMv4T, **no GPU** (software rendering) |
| Executable | `sonicn.app` — Symbian `E32Image`, **1,396,120 bytes, uncompressed** |
| App UID | `0x101FB882` |

The full layout (`system/apps/sonicn/`) is a normal Series 60 install: one `.app` executable, `.rsc` resources, `images.mbm`/`volume.mbm` bitmaps, and `*.bin` graphics/sound/level data. Technical details in [`docs/BINARY-NOTES.md`](docs/BINARY-NOTES.md).

## Why SonicN first

- **Uncompressed `E32Image`** → the recompiler's front end can skip the inflate path for v1.
- **Software-rendered 2D** → no GPU HLE; "draw a frame" means "blit a buffer."
- **A port of a known game** → expected behavior is well documented, so we can tell when it's *right*, not just *running*.
- **Small, bounded asset set** → the file/bitmap formats are standard Symbian, not bespoke.

## Status

🚧 **Bring-up.** Tracking toward *first frame on screen*. Live checklist in [`PROGRESS.md`](PROGRESS.md).

```
[✔] Identify & verify the binary (E32Image, ARMv4, uncompressed)
[ ] Dump import table  → the exact Symbian ordinals SonicN needs
[ ] Recover functions  → feed the lifter
[ ] First function lifted & round-tripped
[ ] HLE bring-up set    (heap, file read, framebuffer, input)
[ ] First frame
[ ] Playable
```

## Getting the game (you bring your own)

This repo contains **no game code or assets** — only analysis and tooling. Drop your own legally-dumped `sonicn.app` (and the rest of `system/apps/sonicn/`) into `game/`; it's `.gitignore`d and never committed. See [`game/PLACE_GAME_HERE.md`](game/PLACE_GAME_HERE.md).

## Legal

SonicN is © Sega. Nothing copyrighted is distributed here. The recompiled output is a derivative of *your* dump, produced on *your* machine.
