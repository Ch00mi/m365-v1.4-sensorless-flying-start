# Exact Continuation Plan

Resume from **FIX14_PASSIVE_BEMF_CAL**. Do not return to the old observer path and do not continue increasing handoff voltage gain until passive voltage scale is validated.

## Phase 0 — replacement hardware qualification

1. Obtain a known-good M365 V1.4 controller or repair the damaged one.
2. Use a known-good ST-Link.
3. Verify normal dashboard power lifecycle.
4. Verify 12 V / 5 V / 3.3 V rails.
5. Verify SWD without using ST-Link 5 V as target power.
6. Confirm bridge-off current and phase-voltage baselines.

## Phase 1 — rebuild the passive baseline

Reconfirm:

- A/B/C phase sensing;
- no unexpected MOE;
- standstill rejection;
- forward/reverse direction;
- eRPM scaling;
- PLL lock;
- BEMF-to-flux mapping.

## Phase 2 — validate FIX14 absolute BEMF calibration

With bridge OFF and no active arm, at approximately 40, 60 and 80 mechanical rpm record:

```text
DMM line-line AC RMS
pv_pll_mrpm
pv_dbg_bemf_mag_mv
pv_dbg_expected_mag_mv
pv_dbg_mag_permille
pv_dbg_ll_rms_est_mv
```

Targets:

- firmware line-line RMS should track the DMM;
- ratio should remain roughly constant with speed;
- `pv_dbg_mag_permille` should be near 1000.

Do not overfit one speed point.

If the ratio is not constant with speed, investigate ADC offset/common-mode handling, clipping, DMM bandwidth/waveform assumptions, phase-divider network, ADC reference/scaling and αβ normalization convention.

## Phase 3 — only after passive calibration passes

Return active scale to a physically meaningful baseline close to 1000 permille.

Use the same short protected pulse:

- ~350–380 µs;
- hard current abort 1.2 A;
- no current PI tuning change;
- angle offset 0°;
- speed about 900–1300 eRPM.

Compare:

```text
avg Id
avg Iq
peak phase current
first/last Id/Iq
trace Ia/Ib/Ic
```

## Phase 4 — extend takeover gradually

Only after repeated short pulses show small Id/Iq:

1. confirm at several rotor electrical angles;
2. confirm both directions if reverse support is desired;
3. increase active duration gradually;
4. transition from frozen/free-running captured speed to live closed-loop estimator;
5. verify no current discontinuity at estimator/controller state transition;
6. close current PI with zero targets;
7. prove stable zero-torque catch.

## Phase 5 — PAS

Only after zero-torque takeover is stable:

- introduce a slow positive `Iq` ramp;
- preserve `Id ≈ 0`;
- apply PAS policy / torque limits;
- validate shutdown, loss-of-signal and re-catch behavior.

## Definition of "finished"

```text
bridge OFF, rider spins wheel
→ passive PLL LOCKED
→ PWM takeover
→ sustained Id ≈ 0, Iq ≈ 0
→ no perceptible braking or kick
→ continuous sensorless FOC
→ commanded +Iq ramp
→ smooth assist torque
```
