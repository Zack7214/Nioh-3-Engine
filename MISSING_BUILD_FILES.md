# Source files missing from the public repo

## Status after `1b7c75c` (2026-11-23)

Thanks for the quick fix! With `1b7c75c` ("Add the assembly and assets a clean 1.8.39 build was missing") **all four
mods build** from a clean checkout: Nioh3EngineFix 1.8.39 links and passes every offline harness, BossRematch 1.5.6
(with `SoulCoreForms`), CamKiProbe (self-test passes) and Training.

| Item | Status |
|---|---|
| 1. EngineFix: 26 missing asm caves | **Fixed** in `1b7c75c` |
| 2. Training: `style/rank_atlas.png` | **Fixed** in `1b7c75c` |
| 2. Training: `data/boss-action-record.tsv` | **Fixed** in `1b7c75c` |
| 3. `re/scripts_re/savecrypt.py` | **Still missing.** `mod/Nioh3EngineFix/build.bat` builds the `.asi`, then exits 1 at `tools/saves/build_live_package.py` (`FileNotFoundError` on `re/scripts_re/savecrypt.py`). |
| 3. `tools/aob_audit.py` | **Still missing.** `gen_rvas.py` can't create a pattern for a new game address. |
| 3. Check/test inputs and unpublished generators | **Still missing.** Everything in section 3 apart from `data/boss-action-record.tsv`. |

The highest-value remaining ones are `savecrypt.py` (so `build.bat` finishes with exit 0) and `aob_audit.py` (so
contributors can add new addresses). The original report follows.

---

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

## 3. Other files that are missing

I checked every path the build, test and tool scripts refer to. These are missing from the repo (no commit has them):

### Needed to build or to add new features

| Missing file | Used by | Effect |
|---|---|---|
| `re/scripts_re/savecrypt.py` | `tools/saves/build_live_package.py`, run by `mod/Nioh3EngineFix/build.bat` (save helpers step, after linking) | Even once item 1 is fixed, `build.bat` still exits with 1 at the save helpers step. |
| `tools/aob_audit.py` | `mod/*/tools/gen_rvas.py` (`import aob_audit`), `tools/check_fixed_rvas.py` | Contributors can't generate a byte pattern for a new game address, so new `rvas.json` entries can't be added. |

### Needed by the checks and tests

| Missing file | Used by |
|---|---|
| `MODLOG.md` | `release/check_release.py` (crashes with `FileNotFoundError`) |
| Commit `d4518e85` (`RELEASE` pin) | `mod/settings/test_catalog.py` (`git show d4518e85:...` fails: that commit isn't in the public history) |
| `mod/Nioh3EngineFix/tests/vrr_floor_shim.asm` | `tests/run_vrr_floor_harness.py` |
| `re/scripts_load/ls_sites.py` | `tests/run_loadstats_harness.py`, `run_prefetch_harness.py`, `run_readfix_harness.py` |
| `re/symbols/symbols.tsv` (old path; the public copy is `symbols/symbols.tsv`) | `tests/run_fnprobe_harness.py` |
| `tools/testmatrix/padrec.py`, `regsuite.py`, `white_dot_walk.ps1` | `run_padrec_test.py`, `run_holdkeygap_harness.py`, BossRematch `test_white_dot.py` |
| `site/version.json`, `site/api/crash.py`, `site/tests/*` | `run_updatecheck.py`, `run_crashupload_test.py`, `run_perfsnap_test.py` |
| `feedback-api/report_api.py`, `gpu-shaders/sweep/build_sweep.py` | `tests/site_perf_check.py`, `tests/run_psoswap_harness.py` |
| `data/boss-action-record.tsv` | Training `build.bat`/`build.sh` (see item 2), `release/package_final.py` |

### Generators and tools the docs list as not yet published

These are only needed to rebuild the committed tables from game data, but the offline tests below look for them:

- `tools/rematch_catalog.py` (`run_rematch_harness.py`)
- `tools/auto_arena.py`, `tools/gen_boss_moves.py`, `tools/gen_move_motion.py`, `tools/gen_summon_rows.py` (Training `run_move_toggle_tests.py`, `run_move_toggle_available.py`)
- `tools/gen_fallback_rosters.py` (`fallback_roster_data_test.py`)
- `tools/gen_native_entry_gates.py` (BossRematch `test_native_entry_gates.py`)
- `tools/gen_actionsets.py`, `tools/n3lfm.py`
- `tools/rematch/verify_all.ps1`, `tools/testhub/hub.py`, `tools/export_public.py`, `tools/setup/`, `tools/updater/update-linux.sh`

## What does work

`generate_catalog.py --check`, `gen_header.py --check`, `generate_menu_defaults.py --check`,
`symbols/build_exports.py --check` and `check_fixed_rvas.py` all pass. BossRematch and CamKiProbe build cleanly
(including CamKiProbe's self-test), and EngineFix's offline harnesses all pass before the link step.

Thanks!
