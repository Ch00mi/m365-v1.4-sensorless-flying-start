# Publication / License Notes

This repository is intentionally documentation-first.

## What is intentionally NOT included

### Firmware source snapshots

The experimental `motor.c` lineage appears to be derived from the public EBiCS / SmartESC M365 firmware family. The exact upstream licensing/provenance of the specific `EBiCS_motor_FOC` base used for these experiments has not yet been established with enough confidence for public redistribution.

Do not publish the local firmware ZIP archive until the applicable upstream license or explicit permission is confirmed.

The broader EBiCS firmware lineage contains GPLv3-or-later licensing statements, but the separate current `EBiCS_motor_FOC` repository does not presently expose an unambiguous license file in the material reviewed for this project. Do not assume the license of a related repository automatically applies to this exact source lineage.

If the applicable upstream is confirmed as GPLv3-or-later, redistribution can be done while preserving copyright/license notices, including the license, marking modifications, and licensing covered modified source under compatible GPL terms.

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
- a description of algorithms and experimental procedure.

## Important

Calling the project "scientific", "educational", "non-commercial", or "experimental" does not by itself create permission to redistribute copyrighted source code or third-party images.

A research/quotation exception may exist in some jurisdictions for limited uses, but it should not be treated as blanket permission to post complete modified source files.

## Recommended path to public source release

1. Identify the exact upstream repository/commit used as the firmware base.
2. Confirm the license applicable to `EBiCS_motor_FOC`.
3. Preserve all copyright/license notices.
4. Add a clear `MODIFICATIONS.md` describing this project's changes and dates.
5. If GPL-covered, release the covered modified source under the applicable GPL terms and include the license.
6. If no license can be established, ask the copyright holder/maintainer for explicit permission before redistributing the derived files.
7. Until then, publish this documentation/data repository and provide pseudocode or independently written explanations instead of the complete derived source.

This note is a practical publication precaution, not legal advice.
