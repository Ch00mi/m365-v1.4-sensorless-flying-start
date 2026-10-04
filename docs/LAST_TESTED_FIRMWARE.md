# Last Tested Firmware State

## Last confirmed hardware-tested build

The last firmware revision that was **actually run successfully on the physical controller before the hardware failure** was:

```text
GreenMover_ACTIVE_HANDOFF_TEST1_FIX13_FINE_GAIN_1650_1700.zip
```

SHA-256:

```text
a482c12123919b7a86366abf15becf7a48c377d1694feb65115e921dc7576043
```

The archive contains the two replacement files used in the experimental workflow:

```text
Core/Src/motor.c
Core/Inc/config.h
```

Their SHA-256 hashes are:

```text
motor.c
31bee049909b1333e2f4b8dc6946a4cf15b98638fb693e1d10eda47563eb73ee

config.h
bb40435e057d2fdc544dd639b841503cfd2355116d67f3cc631e48c763e8138e
```

## Full local project backup

A later recovered full STM32CubeIDE project archive is named:

```text
green mover firmwafe backup.zip
```

SHA-256:

```text
76534874f0f89cd7ede0c9e997f8c27ce046f3d0195b56cbc3b9345898c0904f
```

The project folder inside that archive still carries the older historical name:

```text
GreenMover_M365_Sensorless_Test_v0.3_TEST1
```

However, direct byte-for-byte comparison shows that its active:

```text
Core/Src/motor.c
Core/Inc/config.h
```

are **identical to FIX13**, including the same hashes listed above.

Therefore, despite the old project-folder name, this backup preserves the final confirmed FIX13 motor-control/configuration state inside a complete STM32CubeIDE project tree.

## FIX13 status

FIX13 was the last confirmed tested revision.

It included:

- passive three-phase BEMF sensing;
- passive PLL;
- BEMF-to-flux mapping;
- handoff shadow/preload logic;
- ADC1 current-sampling settling fix;
- bridge-off idle-current qualification;
- ~1.2 A hard active-current abort;
- short ~350–380 µs active takeover pulse;
- voltage-scale calibration up to 1700 permille;
- angle offset fixed at 0°;
- six-callback current tracing;
- trace of phase currents, reconstruction state and TIM1 count.

The last active tests were performed with this revision.

## FIX14 status

The next generated revision was:

```text
GreenMover_ACTIVE_HANDOFF_TEST1_FIX14_PASSIVE_BEMF_CAL.zip
```

SHA-256:

```text
63c3435c701afff65e21fb0bc87c8b2d73b4653085ed9670d27ba2b5d60bd3ae
```

FIX14 introduced the passive absolute BEMF calibration hypothesis:

```text
PV_PHASE_NOMINAL_FULL_SCALE_MV = 15400
PV_PHASE_CAL_GAIN_PERMILLE     = 2520
```

and new passive diagnostics such as:

```text
pv_dbg_phase_cal_permille
pv_dbg_mag_permille
pv_dbg_ll_rms_est_mv
```

FIX14 was intended to be tested **passively first**, with no active arm, against DMM line-line RMS measurements at several speeds.

The controller/ST-Link failure occurred before this calibration experiment was completed. Therefore:

> **FIX13 = last confirmed hardware-tested build.**  
> **FIX14 = next generated continuation build, not experimentally validated.**

## Why the source archive is not attached publicly yet

The last tested source is preserved locally and its identity is recorded above, but the complete source archive is not currently redistributed in this public repository because its motor-control base contains code derived from the EBiCS / SmartESC lineage and the separate `EBiCS_motor_FOC` repository does not currently state an explicit redistribution license.

See `../LEGAL_PUBLICATION_NOTES.md`.
