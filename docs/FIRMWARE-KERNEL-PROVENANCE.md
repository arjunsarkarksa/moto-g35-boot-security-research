# Firmware and Kernel Source Provenance

## Purpose

This note checks whether the public Motorola kernel source can be tied to the exact software build observed on the lab Moto G35 5G. It deliberately distinguishes a related device-family source release from an exact firmware match.

## Device build observed in the lab

The tested handset reported:

- Product: `manila`
- Model/SKU: XT2433-4
- Android: 15
- Build: `VUOA35.116-151`
- Incremental build identifier: `a43dc2`
- Fingerprint: `motorola/manila_gn/manila:15/VUOA35.116-151/a43dc2:user/release-keys`

These are device observations recorded in [DEVICE-BASELINE.md](DEVICE-BASELINE.md). They identify the installed software build, but do not independently identify the exact kernel source commit used to build its kernel.

## Motorola-published kernel source release

Motorola's public readme repository contains a Moto G35 5G build note named `MMI-VUOA35.116-100.txt`. It explicitly directs readers to the `MotorolaMobilityLLC/kernel-sprd` repository at branch `MMI-VUOA35.116-100` and gives a kernel build procedure using the UMS9620 GKI defconfig.

Sources:

- [Motorola G35 build note: MMI-VUOA35.116-100.txt](https://github.com/MotorolaMobilityLLC/readme/blob/master/MMI-VUOA35.116-100.txt)
- [Motorola kernel source branch: MMI-VUOA35.116-100](https://github.com/MotorolaMobilityLLC/kernel-sprd/tree/MMI-VUOA35.116-100)
- [Motorola kernel repository issue #13: source release confirmation](https://github.com/MotorolaMobilityLLC/kernel-sprd/issues/13)

The issue was opened to request Moto G35 source. Motorola's response on 24 April 2026 stated that the Moto G35 kernel source had been published at the `MMI-VUOA35.116-100` branch and closed the request as completed.

## Comparison and interpretation

| Evidence | Value | Interpretation |
|---|---|---|
| Lab device build | `VUOA35.116-151` | Exact installed build reported by the handset |
| Motorola kernel source release | `MMI-VUOA35.116-100` | Officially published Moto G35 kernel source release |
| Shared family prefix | `VUOA35.116` | Both identify the same broad software release family |
| Build suffix | `151` versus `100` | Not an exact build-number match |

**Conclusion — Source-confirmed:** Motorola publicly released Moto G35 kernel source associated with `MMI-VUOA35.116-100`.

**Conclusion — Observed:** The lab handset reports `VUOA35.116-151`.

**Conclusion — Unknown:** The inspected public evidence does not establish that the `MMI-VUOA35.116-100` source branch is the exact kernel source revision used for the lab handset's `VUOA35.116-151` firmware.

A shared prefix, device codename, board family, or compatible device-tree overlay is not sufficient to prove an exact source-to-binary match. Nor does a later firmware build necessarily imply that its kernel source changed; that would require build metadata or a matching source release to establish.

## Evidence needed for an exact match

Any of the following could strengthen the provenance chain:

1. A Motorola-published kernel release or build manifest explicitly associated with `VUOA35.116-151`.
2. A signed firmware package manifest or build metadata that identifies the kernel source revision/commit.
3. A matching kernel build identifier or source commit embedded in the installed kernel, corroborated against Motorola's published source.
4. A Motorola source release note explicitly mapping the later firmware build to a specific kernel branch or commit.

Until such evidence is found, the correct wording is **“Motorola-published Moto G35 kernel source for build family VUOA35.116-100; exact correspondence to lab firmware VUOA35.116-151 is unverified.”**

## Safety and scope

This is a source-provenance investigation only. It does not require flashing firmware, unlocking the bootloader, modifying partitions, or extracting private identifiers. Do not publish IMEI, serial number, account data, or other unique device identifiers while collecting additional evidence.
