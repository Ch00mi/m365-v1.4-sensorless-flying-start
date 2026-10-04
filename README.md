# GreenMover / Xiaomi M365 V1.4 — Passive Sensorless Flying-Start FOC

> **Status:** experimental research project, paused after test-controller / ST-Link hardware damage.  
> **Last confirmed hardware-tested firmware:** `FIX13_FINE_GAIN_1650_1700`. See `docs/LAST_TESTED_FIRMWARE.md`.  
> **Last technical milestone:** passive BEMF amplitude calibration was identified as the next blocking task (`FIX14_PASSIVE_BEMF_CAL`).  
> **Not yet demonstrated:** continuous zero-current sensorless takeover into stable FOC and PAS torque ramp.

## What this repository documents

This repository is an engineering record of experiments performed on a Xiaomi M365 Classic motor controller (`SCO_DRV V1.4`, STM32F103C8T6) driving a 36 V / 250 W direct-drive bicycle hub motor **without Hall sensors**.

The project goal was not generic sensorless startup from standstill. The intended application was a bicycle-style **flying start / catch-on-the-fly**:

```text
rider starts pedaling
        ↓
motor already rotates
        ↓
inverter bridge stays OFF
        ↓
measure Va / Vb / Vc passively
        ↓
remove common mode → Clarke → BEMF αβ
        ↓
PLL locks to direction / speed / electrical angle
        ↓
calculate a BEMF-matched initial voltage vector
        ↓
enable PWM with Id ≈ 0 and Iq ≈ 0
        ↓
transition into continuous FOC
        ↓
only then ramp +Iq for PAS
```

The flying-start principle itself is established prior art. The value of this work is the **specific M365 V1.4 implementation, measurements, failure analysis and raw test evidence**.

## Major results

### Demonstrated

- Passive measurement of all three motor terminal voltages while the bridge remained disabled.
- Reliable standstill rejection.
- Correct forward/reverse direction detection.
- Electrical speed tracking consistent with a 20-pole-pair motor.
- Passive electrical-angle extraction and PLL lock.
- BEMF-to-flux quadrature mapping in both directions.
- Passive "shadow handoff" calculations before enabling PWM.
- Current-sense bring-up and identification of a real ADC acquisition-settling problem.
- Fixing current-A sampling by increasing ADC1 sample time from 1.5 to 28.5 ADC cycles.
- Multi-window idle-current qualification.
- Fast hard-current abort around 1.2 A.
- Short active handoff tests of ~350–380 µs.
- Id/Iq diagnostics during takeover.
- Six-sample phase-current traces showing real progressive current buildup.
- Evidence that the current buildup was not primarily an isolated ADC spike, current-reconstruction state transition, or large PWM sampling jitter.
- Angle-offset sweep showing 0 electrical degrees was a strong baseline.
- Voltage-amplitude sweep locating the best empirical region around 1625–1650 permille.
- Independent DMM measurement revealing a large BEMF amplitude-scale discrepancy.

### Not yet demonstrated

- Continuous sensorless FOC takeover with sustained `Id ≈ 0`, `Iq ≈ 0`.
- Seamless long-duration flying-start operation.
- PAS torque ramp after sensorless takeover.
- Final absolute calibration of M365 phase-voltage sensing.

## The most important unresolved result

At approximately 60 mechanical rpm:

- DMM line-line voltage: **~5.4–5.5 VAC RMS**
- Firmware BEMF magnitude: **~1.71–1.81 V**

For balanced sinusoidal three-phase BEMF:

```text
V(alpha-beta, peak) = V(LL,RMS) * sqrt(2/3)
```

So 5.4–5.5 Vrms line-line corresponds to roughly **4.41–4.49 V αβ peak**, not 1.7–1.8 V.

That suggests the firmware's absolute passive BEMF amplitude was low by about **2.5×**. This can coexist with good angle and PLL behavior because uniform amplitude scaling does not substantially change `atan2(beta, alpha)`.

This became the next planned task: **FIX14 passive absolute BEMF calibration before any further active handoff tuning.**

## Hardware

Controller:

- Xiaomi M365 Classic ESC
- PCB `SCO_DRV V1.4`
- MCU `STM32F103C8T6`
- nominal 36 V battery

Motor:

- Green Mover direct-drive hub motor
- 36 V / 250 W
- no Hall sensors
- 40 rotor magnets / 20 pole pairs

Measured / derived motor parameters:

| Parameter | Value |
|---|---:|
| pole pairs | 20 |
| line-line resistance | ~0.20 Ω |
| equivalent phase resistance | ~0.10 Ω |
| line-line inductance | ~1.21 mH |
| equivalent phase inductance | ~0.605 mH |
| BEMF constant | ~0.090 V RMS line-line / mechanical rpm |
| flux linkage | ~0.035 Wb |
| Kv | ~7.9 rpm/V |

## M365 V1.4 passive phase-voltage channels

From the board/schematic audit used by the test firmware:

```text
Phase A → PA6 / ADC1_IN6
Phase B → PA7 / ADC1_IN7
Phase C → PB1 / ADC1_IN9
```

The phase divider was treated as approximately `22 kΩ / 6 kΩ`.

Current-channel mapping used by the firmware audit:

```text
Current A → ADC3 / PA3
Current B → ADC4 / PA4
Current C → ADC5 / PA5
```

PWM chain audit from `FOC_ANGLE_MAPPING_TEST1`:

```text
TIM1 CH1/CH1N (PA8/PB13)  → phase A gate driver
TIM1 CH2/CH2N (PA9/PB14)  → phase B gate driver
TIM1 CH3/CH3N (PA10/PB15) → phase C gate driver
```

See `docs/HARDWARE_AND_SIGNAL_CHAIN.md`.

## Repository map

- `docs/ENGINEERING_REPORT.md` — full project narrative and conclusions.
- `docs/FIRMWARE_LINEAGE.md` — evolution from passive voltage test to FIX14.
- `docs/LAST_TESTED_FIRMWARE.md` — exact identity, hashes and status of the final confirmed FIX13 build and untested FIX14 continuation.
- `docs/HARDWARE_AND_SIGNAL_CHAIN.md` — motor, ADC and PWM chain.
- `docs/LIVE_EXPRESSIONS_REFERENCE.md` — diagnostic variable glossary.
- `docs/TEST_RESULTS.md` — interpreted experimental results.
- `docs/SAFETY_AND_FAILURE_ANALYSIS.md` — protections, ST-Link incident, lessons.
- `docs/NEXT_STEPS.md` — exact continuation plan for replacement hardware.
- `docs/NEW_CHAT_HANDOFF.md` — compact context block for continuing development.
- `docs/AI_ASSISTED_DEVELOPMENT.md` — transparent development-method note.
- `data/*.csv` — structured test data reconstructed from the experiment log.
- `forum/ENDLESS_SPHERE_POST.md` — long-form forum post ready to edit/publish.
- `firmware_snapshots/README.md` — firmware snapshot lineage and archive guide.

## Firmware lineage at a glance

```text
PHASE_VOLTAGE_TEST0_FIX2
        ↓
PASSIVE_PLL_TEST1
        ↓
FOC_ANGLE_MAPPING_TEST1
        ↓
FOC_HANDOFF_PREP_TEST1
        ↓
ACTIVE_HANDOFF_TEST1
        ↓
FIX6 TIM1 TRGO current sampling
        ↓
FIX7 current-noise diagnostics
        ↓
FIX8 ADC1 settling A/B test
        ↓
FIX9 idle qualifier + trip diagnostics
        ↓
FIX10 voltage-scale test
        ↓
FIX11 angle-offset calibration
        ↓
FIX12 fine voltage-gain calibration
        ↓
FIX13 1650/1700 fine gain + trace analysis  ← last hardware-tested
        ↓
FIX14 PASSIVE BEMF ABSOLUTE CALIBRATION   ← generated continuation; resume here
```

## Safety note for future work

The experimental controller and ST-Link were damaged during debug wiring problems. A loose/disconnected ground combined with other SWD/5 V connections caused abnormal back-powering symptoms; a later loose ground wire also contacted a power-stage point and sparked.

For replacement hardware, do **not** use ST-Link 5 V to power the ESC. Use the target's normal supply and connect:

```text
GND
SWDIO
SWCLK
(optional) NRST
```

Verify target/reference requirements for the exact ST-Link variant before wiring.

## AI-assisted development note

This was an AI-assisted, hardware-validated engineering project. The human operator defined the hardware setup and goals, performed the physical measurements and controller tests, and validated behavior on the real M365 hardware. ChatGPT (OpenAI) assisted with firmware modifications, diagnostic instrumentation, hypothesis generation, data interpretation and technical documentation. Experimental claims in this repository are based on measurements from the physical controller/motor setup, not simulation alone.

See `docs/AI_ASSISTED_DEVELOPMENT.md`.

## Prior art / context

This project does **not** claim invention of flying start.

Relevant references:

1. Tin Bariša, Damir Sumina, Luka Pravica, Igor Čolović,  
   *Flying start and sensorless control of permanent magnet wind power generator using induced voltage measurement and phase-locked loop*,  
   Electric Power Systems Research 152 (2017), 457–465.  
   DOI: `10.1016/j.epsr.2017.08.002`

2. Endless Sphere — *EBiCS Firmware on a M365 STM32 Controller*  
   https://endless-sphere.com/sphere/threads/ebics-firmware-on-a-m365-stm32-controller.111834/

3. SmartESC_STM32_v2  
   https://github.com/Koxx3/SmartESC_STM32_v2

4. oxifoc safety/flying-restart notes  
   https://github.com/okhsunrog/oxifoc/blob/main/docs/safety.md

The interesting contribution here is applying passive three-phase terminal-voltage sensing and PLL acquisition specifically to stock M365 V1.4 hardware, then instrumenting the first few hundred microseconds of active takeover.

## Community / publication

The repository is intended to be a complete engineering record: documentation, measured data, firmware lineage and preserved experimental firmware states are published together so others can reproduce the reasoning and continue the work.

The most useful part of this project is not a claim that "it works"; it is the sequence of measurements showing **which hypotheses were eliminated and why**.
