# Publication / License Notes

This repository is intentionally documentation-first.

## Current license finding

The broader `EBiCS_Firmware` repository explicitly states that it is distributed under the **GNU GPL version 3 or later**.

However, the separate `EBiCS_motor_FOC` repository — the motor-control library whose code structure closely matches the base used by this project — currently has **no LICENSE file and no explicit license statement in its README**.

This is not merely an unnoticed ambiguity: there is already an open upstream issue asking specifically for a license:

https://github.com/EBiCS/EBiCS_motor_FOC/issues/2

That issue has remained open since 2022. A 2024 comment in the same issue quotes GitHub's default-copyright rule: without a license, the author retains the normal exclusive rights and public users are not automatically granted general redistribution/derivative-work rights.

The current SmartESC_STM32_v3 master also references `EBiCS_motor_FOC` directly as a git submodule.

Therefore, while the surrounding EBiCS/SmartESC ecosystem is clearly community/open-source oriented, the safest reading for the specific `EBiCS_motor_FOC` source is:

> **publicly viewable and forkable on GitHub, but redistribution of a modified copy is not clearly licensed yet.**

This repository therefore does not attach the complete modified motor-control source at this time.

## What is intentionally NOT included

### Firmware source snapshots

The experimental `motor.c` lineage is derived from the public EBiCS / SmartESC M365 firmware family and contains substantial code corresponding to the `EBiCS_motor_FOC` motor-control base.

The last confirmed tested revision is fully identified by filename and cryptographic hashes in:

`docs/LAST_TESTED_FIRMWARE.md`

The source archive is preserved locally but not redistributed here until the applicable upstream license or explicit permission is confirmed.

If the upstream authors later clarify that `EBiCS_motor_FOC` is GPLv3-or-later (or another open-source license), the source can be added while preserving the applicable copyright/license notices, marking modifications, and following that license's redistribution requirements.

### Third-party hardware/schematic images

The original M365 V1.4 hardware/schematic image used during the investigation was found externally and its redistribution license has not been confirmed. Crops/annotations derived from that image are therefore not bundled here.

Prefer linking to the original source, obtaining permission, or replacing it with an original diagram drawn from factual measurements/pin mappings.

## What this repository is designed to publish now

- original project narrative and engineering conclusions;
- original measurements and CSV test data;
- original Live Expressions screenshots produced during the experiments;
- original tables and reconstructed traces;
- factual hardware pin/channel mappings;
- links and citations to prior art;
- a description of algorithms and experimental procedure;
- hashes and technical identity of the preserved last-tested firmware.

## Important

Calling the project "scientific", "educational", "non-commercial", or "experimental" does not by itself create permission to redistribute copyrighted source code or third-party images.

A research/quotation exception may exist in some jurisdictions for limited uses, but it should not be treated as blanket permission to post complete modified source files.

## Recommended path to public source release

1. Preserve the local FIX13/FIX14 archives and hashes.
2. Obtain a clear license statement for `EBiCS_motor_FOC`, ideally by resolution of the existing upstream license issue.
3. Preserve all upstream copyright/license notices.
4. Add a clear `MODIFICATIONS.md` describing this project's changes and dates.
5. If GPL-covered, release the covered modified source under the applicable GPL terms and include the license.
6. If another open-source license is selected, follow its attribution/redistribution conditions.
7. Until then, keep this public repository documentation/data-first.

This note is a practical publication precaution, not legal advice.
