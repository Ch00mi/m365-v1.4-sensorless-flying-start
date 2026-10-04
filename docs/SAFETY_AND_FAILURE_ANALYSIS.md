# Safety and Hardware Failure Analysis

## Experimental safety architecture

The project intentionally used layered protections:

- bridge disabled during passive acquisition;
- active takeover required explicit manual `pv_act_arm = 1`;
- passive readiness / PLL / speed / voltage gates;
- bridge-off current qualification;
- hard phase-current abort around 1.2 A;
- short active window;
- voltage-command limit;
- fresh reset / requalification before another pulse.

The philosophy was to diagnose incorrect behavior instead of relaxing protection limits.

## ST-Link / ESC incident

Late in the project the debug connection used:

```text
GND
SWCLK
SWDIO
5 V
```

The ground connection became detached while other leads remained connected.

Observed symptoms included:

- audible buzzing from a small switching inductor / power-supply area;
- abnormal partial powering behavior;
- heating;
- dashboard abnormal behavior;
- ST-Link LED eventually no longer lit even with the target disconnected.

A later loose GND lead contacted a power-stage / MOSFET point and produced a spark.

Important chronology:

> The ESC had already stopped starting normally before the later spark.

Therefore the spark should not automatically be treated as the original cause.

Initial phase-bridge diode/continuity checks were fairly symmetric and did not immediately reveal one obvious shorted MOSFET.

The exact damaged ESC component was not conclusively localized.

## Future debug wiring

For a replacement controller, do not use the ST-Link 5 V output to power the target.

Use the ESC's normal power system and connect only the signals required for SWD:

```text
GND
SWDIO
SWCLK
(optional) NRST
```

If the programmer requires target-voltage reference, verify the exact ST-Link model/pin function before connecting it.

## Before resuming firmware experiments

1. Verify BAT supply and low-voltage rails.
2. Verify normal dashboard power on/off.
3. Verify the ST-Link by itself over USB.
4. Verify SWD without target back-powering.
5. Verify bridge remains off in passive firmware.
6. Verify phase-voltage baselines are symmetric at rest.
7. Verify current offsets with the known ADC1 settling fix.
8. Only then continue FIX14.

## Do not

- raise the 1.2 A hard limit merely to get through a bad handoff;
- resume gain tuning before phase-voltage scaling is validated;
- treat a correct PLL angle as proof that BEMF amplitude is correct;
- power the M365 ESC from an arbitrary ST-Link 5 V lead.
