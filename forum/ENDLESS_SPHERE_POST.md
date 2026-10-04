# Passive Sensorless Flying-Start FOC on Xiaomi M365 V1.4 (STM32F103)
## Three-phase terminal-voltage sensing, BEMF PLL, and near-zero-current handoff experiments on a direct-drive e-bike hub motor

Hi everyone,

I am documenting an experimental project that unfortunately had to be paused because the test controller was damaged during debugging.

I still want to publish the work because several measurements and failure analyses may be useful to people working on M365 hardware, EBiCS/SmartESC, sensorless FOC, or flying-start/catch-on-the-fly control.

This is **not a claim of inventing sensorless flying start**. Measuring induced motor terminal voltage before PWM and using a PLL for synchronization is established motor-control work.

The question I wanted to investigate was more specific:

> **Can the stock Xiaomi M365 V1.4 hardware passively observe a spinning direct-drive hub motor with the power bridge disabled, obtain a trustworthy rotor electrical angle and speed from the three terminal voltages, lock a PLL, and then enable FOC with approximately zero Id and zero Iq?**

## Intended bicycle behavior

```text
Rider starts pedaling
        ↓
direct-drive hub begins rotating
        ↓
MOSFET bridge remains OFF
        ↓
measure all three motor terminal voltages
        ↓
remove common mode → BEMF α/β
        ↓
PLL locks to direction, speed and electrical angle
        ↓
predict the voltage vector already present at the terminals
        ↓
enable PWM synchronized to that vector
        ↓
ideally Id ≈ 0 and Iq ≈ 0
        ↓
continuous FOC
        ↓
only then ramp +Iq for pedal assist
```

The important design choice is that sensorless zero-speed startup is deliberately avoided. The rider provides initial motion.

## Hardware

Controller:

```text
Xiaomi M365 Classic ESC
PCB: SCO_DRV V1.4
MCU: STM32F103C8T6
Battery: nominal 36 V
```

Motor:

```text
Green Mover direct-drive hub
36 V / 250 W
no Hall sensors
40 magnets / 20 pole pairs
```

Measured motor parameters:

```text
R line-line      ≈ 0.20 Ω
R phase          ≈ 0.10 Ω
L line-line      ≈ 1.21 mH
L phase          ≈ 0.605 mH
BEMF constant    ≈ 0.090 V RMS line-line / mechanical rpm
flux linkage     ≈ 0.035 Wb
Kv               ≈ 7.9 rpm/V
```

At 50 mechanical rpm this 20-pole-pair motor is already at roughly 1000 eRPM.

## Why active observer acquisition was abandoned

Early observer/FOC acquisition attempts caused magnetic braking and roughly 2–3 A current while rotor angle was not yet trustworthy.

For a bicycle that is already being pedaled, this is exactly the wrong behavior.

So the architecture changed from:

```text
enable inverter → estimate rotor
```

to:

```text
estimate rotor while inverter is completely disabled → enable only after lock
```

## M365 V1.4 phase-voltage sensing

The V1.4 board/code audit identified three separate terminal-voltage ADC inputs:

```text
Phase A → PA6 / ADC1_IN6
Phase B → PA7 / ADC1_IN7
Phase C → PB1 / ADC1_IN9
```

The divider was treated as approximately 22k / 6k.

The passive firmware measured Va/Vb/Vc, calculated common mode, subtracted it and used a Clarke transform to obtain BEMF αβ. Raw electrical angle came from `atan2(eβ,eα)`.

## Passive results

At standstill, the three phase readings were nearly equal. After common-mode removal, residual BEMF magnitude was only a few millivolts and the motor was correctly classified as stationary.

When the wheel was rotated:

```text
40–60 mechanical rpm → ~800–1200 eRPM
80–100 mechanical rpm → ~1600–2000 eRPM
```

Both directions were detected correctly.

There was no magnetic braking because the bridge remained disabled.

## Passive PLL

A PLL was added on top of the BEMF angle.

It successfully provided stable electrical angle, eRPM, direction and lock/confidence in both directions.

The BEMF-to-flux relationship was also checked passively and produced the expected roughly 90° quadrature relationship with the correct q-axis sign.

## Shadow handoff

Before enabling PWM, the firmware calculated a complete "shadow" takeover:

- candidate rotor-flux angle;
- measured BEMF αβ;
- command-to-physical-voltage mapping;
- candidate Ud/Uq;
- candidate PI integral preload;
- shadow SVPWM duties;
- reconstructed physical αβ voltage.

No PWM registers or PI state were actually modified.

Within a suitable speed envelope, the shadow reconstruction matched measured BEMF closely and a handoff-ready state could be achieved.

## Current-sensing problem

Before active testing, one injected-current channel reported apparent ~0.9–1.0 A peaks even with the bridge off.

Detailed A/B statistics showed:

```text
A peak ~0.91–1.03 A, avg ~0.22–0.24 A
B peak ~0.15–0.19 A, avg ~0.04–0.05 A
```

This turned out not to be real motor current.

ADC1 current-A used a 1.5-cycle acquisition time. Changing only that channel to 28.5 cycles while keeping current-B at 1.5 cycles reduced A-channel noise dramatically toward B-channel levels.

After the change:

```text
A typical peak ~0.23–0.34 A
B peak         ~0.15–0.19 A
A average      ~0.06–0.08 A
B average      ~0.05–0.06 A
```

This was one of the most useful lessons: the correct fix was not to increase a threshold; it was to fix ADC acquisition.

## Protection and first active handoff

An idle qualifier required eight consecutive clean bridge-off windows before arming.

The active hard phase-current abort remained around 1.2 A.

The first real handoff produced genuine current buildup and protection disabled the bridge in less than about a millisecond when phase current approached the limit.

The test window was then shortened to about 350–380 µs, only six current callbacks.

## Voltage scale sweep

Runtime voltage scale was tested at approximately:

```text
800
1000
1200
1300
1400
1500
1600
1625
1650
1700 permille
```

Increasing voltage generally reduced the q-axis mismatch until around 1.6×.

Representative good points:

```text
1625 @ 1201 eRPM:
avg Id ≈ -63 mA
avg Iq ≈ +183 mA
phase peak ≈ 532 mA

1650 @ 1198 eRPM:
avg Id ≈ 0 mA
avg Iq ≈ +209 mA
phase peak ≈ 532 mA
```

At 1700 @ 1198 eRPM:

```text
avg Id ≈ -278 mA
avg Iq ≈ +209 mA
phase peak ≈ 608 mA
```

so I stopped increasing gain.

## Angle sweep

A separate electrical-angle offset sweep tested roughly -4°, -2°, 0°, +2°, +4°.

The controlled 0° baseline produced several very small average d-axis currents:

```text
~1035 eRPM → avg Id ≈ -19 mA
~1078 eRPM → avg Id ≈ +50 mA
~1119 eRPM → avg Id ≈ -38 mA
```

This strongly suggested that the main remaining mismatch was not a large rotor-angle error.

## Six-sample current traces

A trace at ~1185 eRPM:

```text
t [µs]   dyn     Ia     Ib      Ic
 54       1     -38   -114    +152
117       1    +228   -190     -38
181       1    +228   -342    +114
244       1    +456   -380     -76
308       2    +684   -532    -152
371       2    +684   -570    -114
```

For each sample, `Ia + Ib + Ic ≈ 0`.

The current grew progressively rather than appearing as one ADC glitch.

A second trace at ~1210 eRPM kept `dyn_state = 2` for all six samples:

```text
t [µs]   Ia     Ib      Ic    pmax   TIM1_CNT
 54      +38    +38     -76     76      422
117     +114    +38    -152    152      422
181     +266    +38    -304    304      410
244     +266   +152    -418    418      411
308     +342   +152    -494    494      410
371     +304   +190    -494    494      409
```

This mattered because current continued to grow without a reconstruction-state transition, while ADC sampling occurred at nearly the same TIM1 position each cycle.

At this point the current looked like a real physical voltage mismatch.

## The biggest discovery: BEMF amplitude scale

At about 60 mechanical rpm, a DMM connected line-to-line measured:

```text
~5.4–5.5 VAC RMS
```

This agrees almost perfectly with the independently measured motor BEMF constant:

```text
0.090 V/rpm × 60 rpm ≈ 5.4 V
```

But the firmware simultaneously reported only:

```text
pv_dbg_bemf_mag_mv ≈ 1711–1811 mV
```

For balanced sinusoidal three-phase BEMF:

```text
V(alpha-beta,peak) = V(LL,RMS) × sqrt(2/3)
```

Therefore 5.4–5.5 Vrms line-line corresponds to about 4.41–4.49 V αβ peak.

The firmware was seeing only ~1.7–1.8 V.

So the passive absolute BEMF amplitude appeared low by roughly 2.5×.

This can explain why direction, angle and PLL still looked good: a common amplitude scale factor does not significantly change `atan2`.

It may also explain why the active handoff empirically wanted much more voltage than the passive estimator predicted.

## Where the project stopped

The next firmware, `FIX14_PASSIVE_BEMF_CAL`, deliberately stopped active testing.

The planned calibration introduced:

```text
PV_PHASE_NOMINAL_FULL_SCALE_MV = 15400
PV_PHASE_CAL_GAIN_PERMILLE = 2520
```

and diagnostics:

```text
pv_dbg_phase_cal_permille
pv_dbg_mag_permille
pv_dbg_ll_rms_est_mv
```

The plan was to compare firmware-estimated line-line RMS against a DMM at ~40, ~60 and ~80 rpm and require a stable ratio before any more PWM handoff pulses.

Unfortunately the controller was damaged before that experiment was completed.

## Hardware failure

During later SWD work the ST-Link had GND, SWCLK, SWDIO and 5 V connected. The ground lead became detached while other leads remained present.

The setup showed abnormal back-powering symptoms, audible buzzing, heating and partial-power behavior. The ST-Link eventually stopped lighting even when connected by itself.

A later loose GND lead also contacted a power-stage point and sparked, although the controller had already stopped starting normally before that later spark.

Initial phase-bridge diode/continuity checks did not reveal one obvious shorted MOSFET. The exact failed ESC component was not localized.

Future debug will not use ST-Link 5 V as target power.

## What was actually demonstrated

```text
✓ passive three-phase terminal-voltage sensing
✓ standstill rejection
✓ forward/reverse direction detection
✓ correct approximate electrical speed
✓ passive electrical-angle extraction
✓ PLL synchronization with inverter off
✓ BEMF/flux quadrature mapping
✓ passive handoff qualification
✓ current-sense characterization
✓ ADC settling fix
✓ idle-current qualification
✓ fast over-current shutdown
✓ short active PWM handoff
✓ Id/Iq diagnostics
✓ cycle-by-cycle phase-current trace
✓ reconstruction-state and sample-timing checks
✓ angle sweep
✓ voltage-amplitude sweep
✓ independent BEMF scale discrepancy measurement
```

What was **not** demonstrated:

```text
passive lock
→ PWM takeover
→ continuous stable Id≈0/Iq≈0
→ continuous sensorless FOC
→ PAS torque ramp
```

So I do not consider the firmware complete.

## Why publish this unfinished?

The most useful result may be the chain of failed hypotheses:

- early active observer acquisition caused braking;
- passive sensing solved observability during bicycle motion;
- the apparent 1 A current problem was ADC settling, not motor current;
- the short-pulse current rise was real, not an isolated ADC glitch;
- angle sweep suggested angle was not the main error;
- gain sweep improved the symptom but did not explain it;
- a physical DMM measurement then exposed a major passive-voltage scale discrepancy.

That is more useful to future developers than simply reporting a final gain value.

## Prior art / related work

This project should be viewed as an implementation/measurement study, not a new flying-start theory.

Relevant work:

- Tin Bariša, Damir Sumina, Luka Pravica, Igor Čolović, *Flying start and sensorless control of permanent magnet wind power generator using induced voltage measurement and phase-locked loop*, Electric Power Systems Research 152 (2017), 457–465. DOI `10.1016/j.epsr.2017.08.002`.
- EBiCS on M365 discussion: https://endless-sphere.com/sphere/threads/ebics-firmware-on-a-m365-stm32-controller.111834/
- SmartESC_STM32_v2: https://github.com/Koxx3/SmartESC_STM32_v2
- oxifoc flying-restart / safety notes: https://github.com/okhsunrog/oxifoc/blob/main/docs/safety.md

## AI-assisted development

This was an AI-assisted, hardware-validated engineering project. The human operator defined the setup and goals, performed the physical measurements and controller tests, and validated results on the real M365 hardware. ChatGPT assisted with firmware changes, diagnostic instrumentation, hypothesis generation, analysis and technical documentation.

## Feedback I would especially like

1. Has anyone already implemented a complete bridge-off `Va/Vb/Vc → αβ → PLL → active FOC takeover` on stock M365 V1.4 hardware?
2. Is there an M365 V1.4 analog-front-end scaling detail that could explain the ~2.5× discrepancy?
3. Are there coordinate / αβ normalization conventions here that could explain part of the discrepancy?
4. How do other implementations preload voltage/current-controller state for open-circuit BEMF → active FOC transition?
5. Does anyone have cycle-by-cycle current captures from a successful zero-torque flying start for comparison?

When replacement hardware is available, the project should continue from **FIX14 passive absolute BEMF calibration**, not from further empirical gain tuning.
