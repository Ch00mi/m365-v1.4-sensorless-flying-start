# Hardware and Signal Chain

## Controller

- Xiaomi M365 Classic ESC
- `SCO_DRV V1.4`
- STM32F103C8T6
- 36 V-class power stage

## Motor

Green Mover direct-drive bicycle hub motor:

- 36 V / 250 W
- no Hall sensors
- 40 magnets / 20 pole pairs

Measured / derived values:

```text
R line-line  ≈ 0.20 Ω
R phase      ≈ 0.10 Ω

L line-line  ≈ 1.21 mH
L phase      ≈ 0.605 mH

BEMF         ≈ 0.090 V RMS line-line / mechanical rpm
Flux linkage ≈ 0.035 Wb
Kv           ≈ 7.9 rpm/V
```

## Phase-voltage sensing

Board/code audit used:

```text
Phase A → PA6 / ADC1_IN6
Phase B → PA7 / ADC1_IN7
Phase C → PB1 / ADC1_IN9
```

Divider treated as approximately:

```text
22 kΩ high side
 6 kΩ low side
```

Earlier firmware used a nominal phase-voltage full-scale constant:

```text
PV_PHASE_FULL_SCALE_MV = 15400
```

FIX14 preserved that as:

```text
PV_PHASE_NOMINAL_FULL_SCALE_MV = 15400
PV_PHASE_CAL_GAIN_PERMILLE     = 2520
```

The 2520 factor was an **experimental calibration hypothesis** derived from DMM comparison. It was not yet validated across multiple speeds.

## Current sensing

Firmware mapping used during diagnostics:

```text
Current A → ADC3 / PA3
Current B → ADC4 / PA4
Current C → ADC5 / PA5
```

A critical hardware/ADC finding was that ADC1 phase-A injected-current sampling required more acquisition time under the combined regular/injected workload.

The A/B experiment:

```text
A: 1.5 ADC cycles  → 28.5 ADC cycles
B: remained 1.5 ADC cycles as control
```

caused A-channel noise to collapse toward B-channel levels.

## PWM / gate chain

Static audit from `FOC_ANGLE_MAPPING_TEST1`:

```text
TIM1 CH1/CH1N (PA8/PB13)   → gate chain → phase A
TIM1 CH2/CH2N (PA9/PB14)   → gate chain → phase B
TIM1 CH3/CH3N (PA10/PB15)  → gate chain → phase C
```

## Passive acquisition invariant

During passive acquisition:

- power PWM channels were not allowed to drive the bridge;
- MOE was kept low;
- no FOC current command was applied;
- BEMF was inferred only from open-circuit motor terminal voltage.

This is important when comparing the project to observer-based sensorless systems: the motor was used as a generator before takeover.
