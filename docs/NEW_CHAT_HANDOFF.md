# New Chat / Developer Handoff

Continue sensorless flying-start FOC work for a Xiaomi M365 Classic controller (`SCO_DRV V1.4`, STM32F103C8T6) driving a Green Mover 36 V / 250 W direct-drive hub motor without Hall sensors.

## Motor

```text
20 pole pairs / 40 magnets
R line-line ≈ 0.20 Ω → phase model ≈ 0.10 Ω
L line-line ≈ 1.21 mH → phase model ≈ 0.605 mH
BEMF ≈ 0.090 V RMS line-line / mechanical rpm
flux linkage ≈ 0.035 Wb
Kv ≈ 7.9 rpm/V
```

## Goal

```text
rider pedals first
→ bridge OFF
→ passive Va/Vb/Vc sensing
→ common-mode removal
→ Clarke BEMF αβ
→ PLL direction/speed/electrical angle
→ preload a BEMF-matched voltage vector
→ PWM on with Id≈0/Iq≈0
→ continuous FOC
→ PAS ramps +Iq
```

## Proven mapping

```text
Phase voltage:
A → PA6 / ADC1_IN6
B → PA7 / ADC1_IN7
C → PB1 / ADC1_IN9

Current firmware mapping:
A → ADC3 / PA3
B → ADC4 / PA4
C → ADC5 / PA5
```

Passive BEMF sensing and PLL worked in both directions. BEMF-to-flux mapping ~90° was validated. Shadow handoff could become ready.

## Current-sense finding

ADC1 A-channel at 1.5-cycle acquisition produced false ~1 A equivalent noise. Changing A injected-current sampling to 28.5 ADC cycles while B remained 1.5 cycles reduced A noise toward B-channel levels. Keep this fix.

## Safety

```text
idle current qualifier ≈ 450 mA peak per 256-sample window
8 clean windows required
active hard abort ≈ 1.2 A
short active pulse ≈ 350 µs nominal, ~380 µs actual / six callbacks
```

Do not increase the hard limit casually.

## Active handoff

Real progressive current buildup occurred.

Angle-offset sweep showed 0° was a strong baseline, so gross angle error was not the main problem.

Voltage-gain sweep found the best empirical region around 1625–1650 permille; 1700 introduced more negative Id.

Key ~1210 eRPM trace, all `dyn_state=2`:

```text
t(us) Ia  Ib  Ic   pmax TIM1_CNT
54     38  38 -76    76   422
117   114  38 -152  152   422
181   266  38 -304  304   410
244   266 152 -418  418   411
308   342 152 -494  494   410
371   304 190 -494  494   409
```

This showed real current growth, not merely an ADC glitch, reconstruction-state transition or large PWM sample-position jitter.

## Most important unresolved finding

At ~60 mechanical rpm:

```text
DMM line-line RMS ≈ 5.4–5.5 VAC
firmware BEMF magnitude ≈ 1.711–1.811 V
expected αβ peak ≈ 4.41–4.49 V
```

Passive amplitude therefore appeared low by roughly 2.5×.

## Resume point

**FIX14_PASSIVE_BEMF_CAL**, not more gain tuning.

```text
PV_PHASE_NOMINAL_FULL_SCALE_MV = 15400
PV_PHASE_CAL_GAIN_PERMILLE = 2520
active voltage scale returned to 1000

new diagnostics:
pv_dbg_phase_cal_permille
pv_dbg_mag_permille
pv_dbg_ll_rms_est_mv
```

Test passively at ~40/60/80 rpm against a DMM before arming.

## Hardware incident

The test ESC and ST-Link were damaged after SWD wiring/back-powering trouble. Replacement hardware is needed. Future SWD should not use ST-Link 5 V as target power.
