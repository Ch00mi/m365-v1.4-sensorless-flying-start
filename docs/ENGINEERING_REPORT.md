# Engineering Report

## 1. Problem statement

The target was a sensorless pedal-assist controller using a Xiaomi M365 V1.4 ESC and a direct-drive hub motor with no Hall sensors.

A conventional sensorless startup from zero speed was unnecessary for the application: the rider can start rotating the wheel. The desired controller behavior was therefore a **flying start**.

The key requirement was to avoid the unpleasant and potentially unsafe behavior seen in early active-observer tests: magnetic braking and multi-amp current while rotor angle was not yet trustworthy.

The guiding rule became:

> **Do not energize the motor until rotor state has already been measured passively.**

## 2. Passive acquisition architecture

With the bridge disabled, the spinning PM motor acts as a generator.

The controller measures three terminal voltages and removes the common mode:

```text
VCM = (VA + VB + VC) / 3
VA' = VA - VCM
VB' = VB - VCM
VC' = VC - VCM
```

The centered phases are transformed into a stationary αβ vector using Clarke. Electrical angle comes from `atan2(eβ,eα)`, and electrical velocity comes from unwrapped angle change over time.

Because the motor has 20 pole pairs:

```text
eRPM = mechanical RPM × 20
```

so even modest bicycle wheel speed produces strong electrical frequency.

## 3. Initial passive validation

At standstill, the phase channels were nearly equal and common-mode removal collapsed the residual vector to only a few millivolts.

A typical stationary baseline was approximately:

- phase terminal baseline: ~120 mV
- common mode: ~117 mV
- residual BEMF magnitude: ~6 mV
- `vector_valid = 0`
- reported speed = 0
- bridge / MOE = off

With rotation, direction and eRPM were stable in both directions.

Approximate observed relationship:

- 40–60 mechanical rpm → ~800–1200 eRPM
- 80–100 mechanical rpm → ~1600–2000 eRPM

This matched the expected 20-pole-pair motor ratio.

## 4. PLL

A passive PLL was added to avoid using noisy instantaneous `atan2` angle directly as the FOC rotor reference.

Observed behavior:

- state 2 = locked
- state 3 = weak-signal holdover
- state 0 = no signal
- confidence reached 1000 during good rotation
- direction remained stable
- lock worked in both directions
- signal loss correctly returned the estimator to no-signal state

The BEMF-to-flux mapping test showed the expected approximately 90 electrical-degree relationship and correct q-axis sign in both directions.

## 5. Passive shadow handoff

`FOC_HANDOFF_PREP_TEST1` calculated the entire proposed takeover without writing PWM:

- measured BEMF αβ
- candidate flux angle
- exact board-specific command scaling
- candidate `Ud/Uq`
- candidate PI integral preload
- shadow SVPWM duties
- reconstructed physical αβ voltage
- difference between reconstructed voltage and measured BEMF

A readiness envelope rejected low signal, bad PLL state, bad battery range, excessive requested voltage, unsafe duty margin and other invalid conditions.

At suitable speeds the shadow model frequently reached `pv_ho_ready = 1`.

## 6. Current-sense bring-up

The first bridge-off injected-current tests reported unexpectedly large current on channel A.

FIX7 characterized the asymmetry using 256-sample windows.

Representative stationary result before correction:

- A peak ~0.91–1.03 A
- A average absolute ~0.22–0.24 A
- B peak ~0.15–0.19 A
- B average absolute ~0.04–0.05 A

The distribution contained many samples above 300 and 600 mA on A, while B remained quiet.

FIX8 changed only ADC1 phase-A injected sample time:

- A: 1.5 cycles → 28.5 cycles
- B remained 1.5 cycles as control

After the change:

- A peak typically ~0.23–0.34 A
- B peak ~0.15–0.19 A
- A average ~0.06–0.08 A
- B average ~0.05–0.06 A
- almost no samples above 300 mA
- none above 600/900/1200 mA

This strongly identified ADC acquisition settling/source impedance as the cause.

## 7. Idle qualification and protection

FIX9 increased the bridge-off readiness threshold to 450 mA but required eight consecutive clean 256-sample windows.

Any bad window reset readiness.

The active hard phase-current abort remained ~1.2 A.

This distinction matters: the threshold was not increased to hide the old 1 A artifact. The measurement problem had already been fixed in FIX8.

## 8. First active pulse

The first active handoff was deliberately short and heavily protected.

Initial tests showed real current buildup reaching the hard abort around 1.25–1.30 A in roughly 0.55–0.75 ms.

The abort shut MOE down quickly.

The pulse was then shortened to approximately 350 µs, which produced six injected-current callbacks before shutdown.

## 9. Voltage amplitude sweep

FIX10 introduced a runtime voltage scale.

The current controller remained open-loop for this experiment; `Id` and `Iq` targets were zero. The intent was to identify the applied voltage magnitude that best matched the already-existing motor BEMF.

The trend was that larger voltage scale generally reduced the q-axis mismatch until approximately 1.6×.

The best empirical region became ~1625–1650 permille. At 1700 permille, q-axis improvement no longer justified the increased negative d-axis current.

## 10. Angle-offset sweep

FIX11 tested voltage-vector angle offsets while keeping d/q diagnostics referenced to the original passive rotor-flux frame.

Offsets tested included approximately -4°, -2°, 0°, +2°, +4°.

Controlled 0° results included:

- ~1035 eRPM → avg Id ≈ -19 mA
- ~1078 eRPM → avg Id ≈ +50 mA
- ~1119 eRPM → avg Id ≈ -38 mA

The average was effectively near zero, strongly suggesting that the large remaining mismatch was not a gross rotor-angle error.

Angle correction was therefore locked to 0° for later voltage-gain tests.

## 11. Cycle-by-cycle current trace

A six-sample trace at ~1185 eRPM showed:

| t (µs) | dyn | Ia (mA) | Ib (mA) | Ic (mA) |
|---:|---:|---:|---:|---:|
| 54 | 1 | -38 | -114 | +152 |
| 117 | 1 | +228 | -190 | -38 |
| 181 | 1 | +228 | -342 | +114 |
| 244 | 1 | +456 | -380 | -76 |
| 308 | 2 | +684 | -532 | -152 |
| 371 | 2 | +684 | -570 | -114 |

For every sample, `Ia + Ib + Ic ≈ 0`.

The phase-current magnitude grew progressively. This was inconsistent with a single isolated ADC glitch.

A later ~1210 eRPM trace kept `dyn_state = 2` for all six samples:

| t (µs) | dyn | Ia (mA) | Ib (mA) | Ic (mA) | pmax (mA) | TIM1_CNT |
|---:|---:|---:|---:|---:|---:|---:|
| 54 | 2 | +38 | +38 | -76 | 76 | 422 |
| 117 | 2 | +114 | +38 | -152 | 152 | 422 |
| 181 | 2 | +266 | +38 | -304 | 304 | 410 |
| 244 | 2 | +266 | +152 | -418 | 418 | 411 |
| 308 | 2 | +342 | +152 | -494 | 494 | 410 |
| 371 | 2 | +304 | +190 | -494 | 494 | 409 |

This removed two major suspects:

1. the current buildup was not caused primarily by switching between reconstruction states;
2. ADC sampling was not moving wildly around the PWM cycle.

The test therefore pointed toward a real physical voltage mismatch.

## 12. Absolute BEMF discrepancy

At approximately 60 mechanical rpm, a handheld DMM measured roughly 5.4–5.5 VAC RMS line-line.

This agrees with the separately measured BEMF constant:

```text
0.090 V(LL,RMS)/rpm × 60 rpm ≈ 5.4 V(LL,RMS)
```

But the firmware reported only ~1.71–1.81 V BEMF magnitude.

For a balanced sinusoidal three-phase system:

```text
V(alpha-beta,peak) = V(LL,RMS) × sqrt(2/3)
```

so 5.4–5.5 Vrms line-line implies about 4.41–4.49 V αβ peak.

The passive firmware amplitude was therefore low by roughly 2.5×.

This explains how the PLL and angle could be correct while the voltage preload remained too small: uniform amplitude error preserves angle.

## 13. FIX14

FIX14 stopped further active tuning and introduced a dedicated passive calibration stage.

Changes included:

- keep the schematic-derived nominal ADC full scale visible: 15.4 V
- `PV_PHASE_CAL_GAIN_PERMILLE = 2520`
- effective full scale ~38.81 V
- add:
  - `pv_dbg_phase_cal_permille`
  - `pv_dbg_mag_permille`
  - `pv_dbg_ll_rms_est_mv`
- return active scale to 1000 permille
- do not arm during calibration

The intended test was to compare firmware-estimated line-line RMS against a DMM at approximately 40, 60 and 80 mechanical rpm.

That test was never completed because the controller was damaged.

## 14. Hardware incident

During later debug work the ST-Link had GND, SWCLK, SWDIO and 5 V connected. The ground lead became detached while other wires remained connected.

The system then showed abnormal back-powering symptoms:

- audible buzzing from the switching supply area
- abnormal dashboard behavior
- heating
- ST-Link eventually stopped lighting even with no target attached

Later, a loose GND lead contacted a power-stage point and sparked. However, the controller had already stopped starting normally before that spark, so the spark cannot confidently be identified as the original cause.

Initial phase-bridge continuity/diode checks did not reveal an obvious single shorted MOSFET.

The exact failed ESC component remains unknown.

## 15. Project conclusion

The project did not reach a rideable sensorless controller.

However, the passive-acquisition part was experimentally successful, and the active-handoff diagnostics narrowed the remaining problem significantly.

The continuation point is not "try a different observer."

It is:

> **validate and correct the absolute passive phase-voltage / BEMF scale, then repeat the same short protected handoff from a physically calibrated baseline.**

That is the most important preserved result of this work.
