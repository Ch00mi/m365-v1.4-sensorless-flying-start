# Firmware Lineage

This file records the experiment sequence so future work does not repeat old dead ends.

| Stage | Primary purpose | Important result |
|---|---|---|
| `PHASE_VOLTAGE_TEST0_FIX2` | Passive Va/Vb/Vc measurement, speed/direction, safe dashboard power lifecycle | Bridge-off BEMF sensing worked; standstill rejected; direction/speed worked |
| `PASSIVE_PLL_TEST1` | Lock a PLL to passive BEMF angle | Stable lock in both directions; good confidence; clean signal loss |
| `FOC_ANGLE_MAPPING_TEST1` | Validate BEMF ↔ flux mapping and FOC Park sign convention | ~90° mapping confirmed; q sign consistent |
| `FOC_HANDOFF_PREP_TEST1` | Calculate handoff shadow state without enabling PWM | `pv_ho_ready` could become valid; shadow αβ reconstruction matched passive BEMF closely |
| `ACTIVE_HANDOFF_TEST1` | First manually armed zero-current takeover pulse | Real current buildup observed; hard abort worked |
| `FIX1–FIX4` | Power-button / failsafe lifecycle corrections | Normal dashboard power behavior restored |
| `FIX5` | ADC multimode / current-trigger bring-up | Progress toward injected-current operation |
| `FIX6_TIM1_TRGO` | Use TIM1 TRGO for injected-current trigger with MOE off | TIM1/current sampling started reliably while bridge remained disabled |
| `FIX7_CURRENT_NOISE_DIAG` | Characterize bridge-off current noise | Current-A showed systematic ~1 A equivalent artifact; current-B much cleaner |
| `FIX8_CURRENT_SETTLING_AB_TEST` | A/B sample-time experiment | ADC1 A-channel 1.5→28.5 cycles fixed most of the artifact |
| `FIX9_IDLE_QUALIFIER_TRIP_DIAG` | Deterministic idle qualification and detailed trip capture | Stable pre-arm readiness; 1.2 A hard abort retained |
| `FIX10_VOLTAGE_SCALE_TEST` | Short ~350 µs amplitude sweep | Larger preload reduced q-current tendency; true voltage mismatch suspected |
| `FIX11_ANGLE_OFFSET_CAL` | ± electrical-angle sweep | 0° became the best baseline; angle not dominant error |
| `FIX12_FINE_VOLTAGE_GAIN_CAL` | 1400/1500/1600 permille | Improvement continued toward ~1600 |
| `FIX13_FINE_GAIN_1650_1700` | 1650/1700 and six-sample trace analysis | 1625–1650 region best; 1700 introduced larger negative Id; trace showed real progressive current |
| `FIX14_PASSIVE_BEMF_CAL` | Absolute passive voltage calibration | **Next required task; not completed before hardware failure** |

## Important state semantics used late in the project

### PLL

```text
pv_pll_state = 0  SEARCH / no signal
pv_pll_state = 1  ACQUIRE
pv_pll_state = 2  LOCKED
pv_pll_state = 3  HOLDOVER / weak signal
```

### Active test

```text
pv_act_state = 0  IDLE / passive
pv_act_state = 1  ACTIVE
pv_act_state = 2  DONE / successful timeout completion
pv_act_state = 3  ABORTED
```

`pv_act_arm` was a one-shot Live Expressions command and was cleared by firmware.

`pv_act_reset = 1` was required between active attempts to return to the passive-ready path.

## Safety values that should not be casually relaxed

Late experimental values:

- active hard phase-current abort: ~1200 mA
- idle current qualification threshold: 450 mA peak / 256-sample window
- required clean idle windows: 8
- short active pulse: nominal ~350 µs, actual termination typically ~380 µs at next current callback
- active voltage command limit: 300 internal units
- first active speed envelope: approximately 45–65 mechanical rpm / 900–1300 eRPM

The project deliberately avoided "solving" bad behavior by increasing the current limit.
