# Source files missing from the public repo (Nioh3EngineFix doesn't link; Training needs one extra file)

Hi! I'm building the mods from a clean clone of `main` (commit `b5f718a`, source v1.8.39) on Windows 11 with
VS Build Tools 2022 (MSVC 14.44, Windows SDK 10.0.26100) and zig 0.17.1. Three of the four mods build. Two things
are missing from the repo:

## 1. Nioh3EngineFix: 26 unresolved externals at link time

`mod/Nioh3EngineFix/build.bat` passes every offline harness and compiles every `.cpp`/`.asm`, then fails at the link:

```
bin\Nioh3EngineFix.asi : fatal error LNK1120: 26 unresolved externals
```

These symbols are declared/used in `src/dllmain.cpp`, `src/osd.cpp` and `src/gpuopt.cpp` (and referenced as
targets in `patches.json`), but no `.asm` file in any commit defines them:

| Missing symbol(s) | Used by (patches.json id / group) |
|---|---|
| `caveGov`, `gov_fix`, `gov_fps`, `gov_getFps`, `gov_obj` | `P8d` / VRR |
| `caveHold` | `P10` / VRRHOLD |
| `vrr_floorNs`, `vrr_floorHoldNs`, `vrr_floorProbeNs`, `vrr_floorOn`, `vrr_floorEnters`, `vrr_teArm`, `vrr_teFrames`, `vrr_ovrFrames`, `vrr_ovrVal`, `vrr_clampFrames`, `vrr_clampLostNs`, `vrr_dtShift` | VRR data (Tier120Floor GH #98, DtSmooth, GH #155) |
| `caveFwp`, `fwp_resume` | `FWP` / FWP (#101247) |
| `caveCharLod` | `RDC1` / RDC |
| `cavePropDist` | `RDF1` / RDF |
| `cavePlaceDist` | `RDP1`-`RDP10` / RDP |
| `caveDrawLimit` | `RDD1`-`RDD5` / RDD |
| `caveRenderDims` | `XRS1`-`XRS4` / XRS |
| `gpuopt_vrs` (`volatile uint8_t[96]`, `src/gpuopt.cpp:57`) | gpuopt |

The VRR ones look like they were dropped from `src/caves.asm`. The rest probably come from an `.asm` file that the
public export left out (and that would also need adding to the `ml64` lines in `build.bat`). Could you commit
the missing assembly?

## 2. Nioh3Training: `style/rank_atlas.png` is not committed

`style_meter_game.h` uses `.incbin "style/rank_atlas.png"`, but the PNG isn't in the repo, so the build fails with
`Could not find incbin file 'style/rank_atlas.png'`. (I got it working locally by extracting the 2048x1156 atlas
embedded in the v1.8.39 release `Nioh3Training.asi`.) Committing the PNG would fix it.

Minor: `mod/Nioh3Training/build.bat` ends with `copy ..\..\data\boss-action-record.tsv`, and that file isn't in
the repo either, so the script exits with 1 after a successful compile. The same table is already committed as
`mod/Nioh3Training/Nioh3Training_action_records.tsv`.

Thanks!
