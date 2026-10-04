# Test Results and Interpretation

Structured versions are in `../data/`.

## 1. Current noise: FIX7 → FIX8

### Before the sample-time fix

Representative bridge-off FIX7 behavior:

- A peak: ~912–1026 mA
- A average abs: ~223–239 mA
- B peak: ~152–190 mA
- B average abs: ~42–49 mA
- A had many >300 mA and >600 mA samples
- B had essentially none >300 mA

### After ADC1 A-channel sample time increased to 28.5 cycles

Representative FIX8 behavior:

- A peak: typically ~228–304 mA, occasional ~342 mA
- B peak: ~152–190 mA
- A average: ~58–77 mA
- B average: ~53–61 mA
- normally no >300 mA samples, none >600/900/1200 mA

Interpretation: the original ~1 A signal was mostly ADC acquisition settling, not real phase current.

## 2. Active takeover: first hard-abort behavior

Before the short-pulse calibration stage:

- preload around 50–60 mechanical rpm
- BEMF roughly ~1.5–1.7 V in firmware units
- active hard trip after roughly 0.55–0.75 ms
- phase-current peak ~1.25–1.29 A
- current rose across consecutive samples

Interpretation: real voltage mismatch, not just one bad ADC sample.

## 3. Voltage-scale sweep

See `../data/voltage_scale_sweep.csv`.

Trend:

- 800–1300 permille: substantial positive q-axis current ramp
- 1400–1600: generally improved
- 1625–1650: best observed region
- 1700: q-axis did not materially improve while negative d-axis increased

## 4. Angle sweep

See `../data/angle_offset_sweep.csv`.

Controlled 0° baseline:

- -19 mA avg Id at ~1035 eRPM
- +50 mA avg Id at ~1078 eRPM
- -38 mA avg Id at ~1119 eRPM

This was a key argument that angle was not the main remaining problem.

## 5. Six-sample trace at ~1185 eRPM

See `../data/trace_1185_erpm.csv`.

The phase currents satisfied approximately `Ia + Ib + Ic = 0` at every sample, and current magnitude increased progressively through the pulse.

## 6. Six-sample trace at ~1210 eRPM

See `../data/trace_1210_erpm.csv`.

All six samples were in `dyn_state = 2`; `TIM1_CNT` remained about 409–422.

Interpretation: current growth persisted with fixed reconstruction state and nearly fixed PWM sampling position.

## 7. Absolute BEMF check

At about 60 mechanical rpm:

```text
DMM:      ~5.4–5.5 Vrms line-line
firmware: ~1.711–1.811 V BEMF magnitude
```

Expected αβ peak from DMM:

```text
5.4 Vrms → ~4.41 V
5.5 Vrms → ~4.49 V
```

This was the strongest remaining lead and directly motivated FIX14.
