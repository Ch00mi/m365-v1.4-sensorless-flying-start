# Firmware Snapshot Archive

This directory contains the preserved experimental firmware lineage used during the M365 V1.4 flying-start investigation.

## Reference states

- `GreenMover_ACTIVE_HANDOFF_TEST1_FIX13_FINE_GAIN_1650_1700.zip` — **last hardware-tested state**.
- `GreenMover_ACTIVE_HANDOFF_TEST1_FIX14_PASSIVE_BEMF_CAL.zip` — **next generated continuation; not hardware-validated**.

FIX13 is the correct reference when comparing against measured active-handoff results. FIX14 contains the passive BEMF absolute-amplitude calibration work that should be tested first on replacement hardware.

The historical progression is documented in `../docs/FIRMWARE_LINEAGE.md`, and the exact hashes/status of FIX13 and FIX14 are documented in `../docs/LAST_TESTED_FIRMWARE.md`.
