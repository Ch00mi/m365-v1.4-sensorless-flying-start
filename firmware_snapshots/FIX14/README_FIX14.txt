GreenMover ACTIVE_HANDOFF TEST1 - FIX14 PASSIVE BEMF ABSOLUTE CALIBRATION

Purpose
-------
Validate the absolute phase-voltage/BEMF scale before any more active handoff pulses.
A direct DMM measurement at about 60 mechanical rpm showed 5.4..5.5 Vrms line-line,
while the firmware reported only about 1.71..1.81 V alpha/beta magnitude.
For balanced sinusoidal three-phase BEMF, 5.4..5.5 Vrms line-line corresponds to
about 4.41..4.49 V alpha/beta phase-peak magnitude, so the previous firmware scale
was low by about 2.52x.

Changes
-------
- Keeps the schematic-derived nominal 15.4 V ADC full-scale visible.
- Adds PV_PHASE_CAL_GAIN_PERMILLE = 2520.
- Effective phase-voltage full scale becomes ~38.81 V.
- Adds diagnostics:
    pv_dbg_phase_cal_permille
    pv_dbg_mag_permille
    pv_dbg_ll_rms_est_mv
- Active handoff voltage scale is locked to 1000 permille for this calibration stage.
- No current-limit increase and no pulse-duration increase.

Test
----
Do NOT arm. Bridge remains off. Spin the wheel at several steady speeds (suggested
40, 60 and 80 mechanical rpm) and compare:
    pv_pll_mrpm
    pv_dbg_bemf_mag_mv
    pv_dbg_expected_mag_mv
    pv_dbg_mag_permille
    pv_dbg_ll_rms_est_mv
with a DMM line-line AC reading.

Targets
-------
- pv_dbg_mag_permille should be near 1000 (roughly +/-5..10% initially).
- pv_dbg_ll_rms_est_mv should track the DMM line-line RMS value.
- The ratio should stay approximately constant with speed.

Do not resume active handoff pulses until this passive calibration is validated.
