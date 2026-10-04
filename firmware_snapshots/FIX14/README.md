# FIX14 — Passive BEMF calibration

Status: **generated continuation, not hardware-validated**.

FIX14 follows the published, hardware-tested FIX13 snapshot. This directory contains the original FIX14 test notes plus exact unified diffs for the two modified source files:

- `FIX13_to_FIX14_motor.patch` — `Core/Src/motor.c`
- `FIX13_to_FIX14_config.patch` — `Core/Inc/config.h`
- `README_FIX14.txt` — original snapshot notes

Original archive identity:

- `GreenMover_ACTIVE_HANDOFF_TEST1_FIX14_PASSIVE_BEMF_CAL.zip`
- SHA256 `63c3435c701afff65e21fb0bc87c8b2d73b4653085ed9670d27ba2b5d60bd3ae`

Apply both patches to the FIX13 source tree to reproduce the FIX14 source changes. FIX14 should first be validated passively against DMM line-line RMS measurements before any active handoff testing.
