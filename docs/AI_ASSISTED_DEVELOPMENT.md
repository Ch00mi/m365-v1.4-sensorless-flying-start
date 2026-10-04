# AI-Assisted Development

This project was developed through an iterative **AI-assisted, hardware-validated engineering process**.

## Roles

The human operator:

- defined the bicycle/controller/motor setup and desired behavior;
- selected the engineering goals and safety constraints;
- performed all physical wiring, measurements and bench tests;
- flashed firmware and operated STM32CubeIDE / Live Expressions;
- provided real test results and screenshots;
- decided whether proposed behavior was physically acceptable;
- validated conclusions against the actual controller and motor.

ChatGPT (OpenAI) assisted with:

- firmware modification and diagnostic instrumentation;
- motor-control reasoning and hypothesis generation;
- design of controlled A/B experiments;
- interpretation of ADC, current, PLL and handoff traces;
- organization of the experiment sequence;
- technical writing and documentation.

## Validation rule

No important result in this repository is presented as true merely because an AI suggested it.

The development loop was:

```text
hypothesis / firmware change
        ↓
real hardware experiment
        ↓
measurement
        ↓
hypothesis confirmed, rejected or refined
        ↓
next experiment
```

Two examples show why this mattered:

1. Apparent ~1 A bridge-off current was initially suspicious, but an A/B ADC sample-time experiment showed it was primarily ADC acquisition settling.
2. Active handoff current initially looked like an angle/tuning issue, but the angle sweep and later independent DMM measurement redirected the investigation toward absolute BEMF amplitude scaling.

The physical hardware remained the arbiter of truth.
