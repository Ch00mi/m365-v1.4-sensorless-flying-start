# FIX14 — Passive BEMF calibration

Status: **generated continuation, not hardware-validated**.

FIX14 was generated after FIX13 to test the suspected absolute BEMF amplitude scaling error before attempting another active handoff.

Reference archive identity:

- Archive: `GreenMover_ACTIVE_HANDOFF_TEST1_FIX14_PASSIVE_BEMF_CAL.zip`
- SHA256: `63c3435c701afff65e21fb0bc87c8b2d73b4653085ed9670d27ba2b5d60bd3ae`

The original ZIP is preserved locally. In this repository FIX14 is represented as the exact source delta from the published, hash-verified FIX13 snapshot, so the control-code changes are directly reviewable.

Main changes:

- `PV_PHASE_NOMINAL_FULL_SCALE_MV = 15400`
- `PV_PHASE_CAL_GAIN_PERMILLE = 2520`
- active handoff voltage-scale sweep locked to `1000` permille
- added diagnostics:
  - `pv_dbg_phase_cal_permille`
  - `pv_dbg_mag_permille`
  - `pv_dbg_ll_rms_est_mv`
- passive phase-voltage reconstruction now applies the calibration gain before common-mode removal / Clarke transform
- intended validation: compare DMM line-line RMS against `pv_dbg_ll_rms_est_mv` around 40, 60 and 80 mechanical rpm

Apply both patches in this directory to the FIX13 source tree to reproduce the FIX14 source changes.
