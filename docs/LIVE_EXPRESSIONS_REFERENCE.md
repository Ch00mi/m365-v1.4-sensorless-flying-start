# Live Expressions Reference

This is a practical glossary of variables used during the later experiment stages.

## Core passive / PLL

```text
pv_dbg_tim1_moe
pv_dbg_adc_error_count

pv_pll_state
pv_pll_locked
pv_pll_confidence
pv_pll_erpm
pv_pll_mrpm
pv_pll_dir
pv_pll_phase_error_deg_x100
pv_pll_phase_error_abs_deg_x100
pv_pll_lock_count
pv_pll_loss_count
```

Late screenshots often showed PLL state 0 after the wheel stopped. That is normal; post-run active diagnostics remained latched.

## Handoff-prep

```text
pv_ho_ready
pv_ho_reject_mask
```

Late observed examples:

- `pv_ho_ready = 1`, reject mask `0` in suitable speed range.
- `reject_mask = 136 (0x88)` around ~207 mechanical rpm was consistent with speed / voltage-command protections.
- after the wheel stopped, a reject value around `79` was normal because PLL/BEMF/speed readiness disappeared.

## Active one-shot controls

```text
pv_act_idle_current_ok
pv_act_state
pv_act_arm
pv_act_reset

pv_act_arm_count
pv_act_run_count
pv_act_success_count
pv_act_abort_count
pv_act_abort_reason
```

Do **not** edit a compile-time source constant to arm. `pv_act_arm` was intentionally a runtime one-shot Live Expressions variable.

## Voltage / angle calibration

```text
pv_act_voltage_scale_permille
pv_act_scale_applied_permille

pv_act_angle_offset_deg_x100
pv_act_angle_applied_deg_x100
```

FIX12/13 ultimately hard-locked the angle correction to zero for gain calibration.

## Active-run summary

```text
pv_act_elapsed_us
pv_act_injected_callbacks
pv_act_pwm_was_enabled

pv_act_start_erpm
pv_act_start_ud
pv_act_start_uq
pv_act_start_bemf_mv
pv_act_start_flux_angle_q31

pv_act_peak_phase_ma
pv_act_peak_id_ma
pv_act_peak_iq_ma
pv_act_peak_u_abs
```

## d/q diagnostics

```text
pv_act_diag_samples

pv_act_diag_first_id_inst_ma
pv_act_diag_first_iq_inst_ma

pv_act_diag_last_id_inst_ma
pv_act_diag_last_iq_inst_ma

pv_act_diag_avg_id_inst_ma
pv_act_diag_avg_iq_inst_ma

pv_act_diag_peak_abs_id_inst_ma
pv_act_diag_peak_abs_iq_inst_ma

pv_act_diag_iq_delta_ma
```

`pv_act_diag_iq_delta_ma = last Iq - first Iq`.

Because the short pulse contained only ~6 samples, `Iq delta` was useful but quantized/noisy. Average d/q current and phase peak became more robust comparison metrics.

## Trace arrays

Late firmware contained a circular trace with arrays including:

```text
pv_act_trace_callback[]
pv_act_trace_elapsed_us[]
pv_act_trace_dyn_state[]
pv_act_trace_raw1_counts[]
pv_act_trace_raw2_counts[]
pv_act_trace_ia_ma[]
pv_act_trace_ib_ma[]
pv_act_trace_ic_ma[]
pv_act_trace_pmax_ma[]
pv_act_trace_tim1_cnt[]
```

A useful post-run view was indices `[0]` through `[5]` because a ~380 µs pulse generated six injected-current callbacks.

## FIX14 passive calibration diagnostics

```text
pv_dbg_phase_cal_permille
pv_dbg_mag_permille
pv_dbg_ll_rms_est_mv
```

The intended target was:

```text
DMM line-line RMS ≈ pv_dbg_ll_rms_est_mv / 1000
pv_dbg_mag_permille ≈ 1000
```

across several speeds before active handoff resumed.
