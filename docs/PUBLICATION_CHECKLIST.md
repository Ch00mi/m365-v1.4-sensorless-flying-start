# Publication Checklist

## Endless Sphere

Use `forum/ENDLESS_SPHERE_POST.md` as the main thread.

Suggested title:

> **Passive Sensorless Flying-Start FOC on Xiaomi M365 V1.4 — 3-Phase BEMF + PLL + Handoff Measurements**

In the older EBiCS/M365 thread, post only a short pointer to the new thread.

## GitHub

Publish:

- `README.md`
- `docs/`
- `data/`
- selected original experiment screenshots
- firmware source only after license/provenance review

## VESC / motor-control communities

Use a shorter post focused on bridge-off BEMF acquisition, PLL synchronization, zero-current takeover, six-sample traces and the absolute BEMF scale discrepancy.

## Reddit

A short cross-post is useful for discovery, but link back to the GitHub repository and/or Endless Sphere thread rather than duplicating the entire report. Good targets include communities focused on e-bikes, motor control, embedded STM32 and DIY EV work.

## Claims to avoid

Do not claim:

- invention of flying start;
- invention of BEMF+PLL rotor estimation;
- completed sensorless FOC;
- proven final zero-current takeover.

## Claims supported by the experiment

It is reasonable to say:

- stock M365 V1.4 hardware was used successfully for passive three-terminal voltage sensing;
- passive direction/speed/electrical-angle estimation worked;
- passive PLL lock worked in both directions;
- active short-pulse current growth was measured cycle-by-cycle;
- ADC settling caused a large false current artifact and was corrected experimentally;
- independent DMM testing revealed a major discrepancy in absolute BEMF amplitude;
- the project stopped before validating the corrected amplitude scale.

## Community questions worth asking

- What exact phase-voltage divider/effective ADC scaling is expected on V1.4?
- Which αβ magnitude convention is used in comparable M365/FOC implementations?
- How is controller state preloaded before a zero-torque flying restart?
- Has anyone captured phase current during the first 0.5 ms of a successful takeover?
