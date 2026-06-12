# Put your own SonicN dump here

This repo ships **no copyrighted game files**. To work on the port, drop your own
legally-obtained Series 60 install of SonicN into this folder:

```
game/
└── system/apps/sonicn/
    ├── sonicn.app          <- the E32Image the recompiler reads
    ├── etcdata.bin
    ├── action_char4.bin
    ├── action_char8.bin
    ├── images.mbm
    ├── volume.mbm
    ├── snd000.bin … snd099.bin
    ├── sonicn.rsc / sonicn_caption.rsc / sonicn.aif
    └── …
```

Everything in `game/` is `.gitignore`d and will never be committed. Sanity check:
`sonicn.app` should be **1,396,120 bytes** with app UID **`0x101FB882`** (v1.0.14).
