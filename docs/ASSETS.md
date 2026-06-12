# Reference assets (local, not committed)

These support the port but are **never committed** (firmware/ROM/game data). Paths are
on the working machine; bring your own equivalents.

## EKA2L1 N-Gage device drive — `Ngage.zip`

An EKA2L1 N-Gage (hardware model **rh-4**) data drive: `Data/drives/z/rh-4/...`. Its
`system/libs/` holds the real Symbian system DLLs we link against:

| DLL | Size | Our imports |
|---|---:|---:|
| `euser.dll` | 210 KB | 78 |
| `avkon.dll` | 314 KB | 19 |
| `estlib.dll` | 71 KB | 3 |
| `bitgdi.dll` | 59 KB | 6 |
| `fbscli.dll` | 34 KB | 7 |
| `eikcore.dll` | 32 KB | 55 |
| `cone.dll` | 29 KB | 46 |
| `efsrv.dll` | 25 KB | 13 |
| `apparc.dll` | 13 KB | 4 |
| `ws32.dll` | 21 KB | — |

**Use:** these `.dll`s are themselves `E32Image` files with export tables, so loading
them in IDA gives the authoritative **ordinal → SDK signature** map for every import in
[`HLE-IMPORTS.md`](HLE-IMPORTS.md), plus reference ARM implementations to mirror in the
native HLE.

## Symbian ROMs — `Symbian OS rom.zip`

A set of EKA2L1 ROM/RPKG dumps. The relevant one is
`s60v1/Nokia N-Gage & N-Gage QD (S60v1)` — the firmware that lets **EKA2L1 boot the
N-Gage** and run SonicN as a behavioral oracle (compare framebuffer/timing against the
recomp). Set it up via `ngagerecomp/harness/run-eka2l1.ps1`.

## Why this matters

The lifter already turns SonicN's ARM into native C. The remaining work is HLE, and
these assets de-risk it: the DLLs pin down exactly what each of the 233 imports is, and
the ROM gives a runnable ground truth.
