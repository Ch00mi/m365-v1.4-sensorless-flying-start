# Firmware Snapshot Archive

The documentation repository intentionally separates public documentation from source publication.

Local experiment snapshots existed through:

```text
PHASE_VOLTAGE_TEST0_FIX2
PASSIVE_PLL_TEST1
FOC_ANGLE_MAPPING_TEST1
FOC_HANDOFF_PREP_TEST1
ACTIVE_HANDOFF_TEST1
FIX1 ... FIX14_PASSIVE_BEMF_CAL
```

The complete local snapshot archive is intentionally **not** committed here yet.

## Before publishing firmware publicly

The motor-control source lineage is derived from EBiCS / SmartESC work. The license applicable to the exact `EBiCS_motor_FOC` lineage should be confirmed before redistributing the modified full source.

A safe workflow is:

1. identify the exact upstream base;
2. confirm the applicable license;
3. preserve copyright/license notices;
4. clearly mark modifications;
5. include attribution;
6. if GPL-covered, release the covered modified source under the applicable GPL terms.

The documentation and measured data remain useful while source-license clarification is pending.
