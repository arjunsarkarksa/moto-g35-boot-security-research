# Firmware and Kernel Source Provenance

## Purpose

This note checks whether the public Motorola kernel source can be tied to the exact software build observed on the lab Moto G35 5G. It distinguishes a related device-family source release from an exact firmware match.

## Device build observed in the lab

The tested handset reported:

- Product: `manila`
- Model/SKU: XT2433-4
- Android: 15
- Build: `VUOA35.116-151`
- Incremental build identifier: `a43dc2`
- Fingerprint: `motorola/manila_gn/manila:15/VUOA35.116-151/a43dc2:user/release-keys`

These observations identify the installed software build, but do not independently identify the exact kernel source commit used to build its kernel.

## Motorola-published kernel source release

Motorola's public readme repository contains a Moto G35 5G build note named `MMI-VUOA35.116-100.txt`. It directs readers to `MotorolaMobilityLLC/kernel-sprd`, branch `MMI-VUOA35.116-100`, and gives a kernel build procedure using the UMS9620 GKI defconfig.

Sources:

- [Motorola G35 build note: MMI-VUOA35.116-100.txt](https://github.com/MotorolaMobilityLLC/readme/blob/master/MMI-VUOA35.116-100.txt)
- [Motorola kernel source branch: MMI-VUOA35.116-100](https://github.com/MotorolaMobilityLLC/kernel-sprd/tree/MMI-VUOA35.116-100)
- [Motorola kernel repository issue #13: source release confirmation](https://github.com/MotorolaMobilityLLC/kernel-sprd/issues/13)

Motorola's response on 24 April 2026 stated that Moto G35 kernel source had been published at the `MMI-VUOA35.116-100` branch.

## Running-kernel identity observed on the lab handset

Read-only ADB commands were used to collect `uname -a`, `/proc/version`, selected kernel sysctl values, Android build fingerprints, and the bootloader identifier.

The handset returned:

- `uname -a`: `Linux localhost 5.15.197-android13-8-00005-g2d8ad9139b89-ab15063902 #1 SMP PREEMPT Fri Mar 20 11:14:06 UTC 2026 aarch64 Toybox`
- `/proc/version`: `Linux version 5.15.197-android13-8-00005-g2d8ad9139b89-ab15063902 (build-user@build-host) (Android (8508608, based on r450784e) clang version 14.0.7 (https://android.googlesource.com/toolchain/llvm-project 4c603efb0cca074e9238af4b4106c30add4418f6), LLD 14.0.7) #1 SMP PREEMPT Fri Mar 20 11:14:06 UTC 2026`
- `ro.bootimage.build.fingerprint`: `motorola/manila_gn/manila:15/VUOA35.116-151/a43dc2:user/release-keys`
- `ro.build.fingerprint`: `motorola/manila_gn/manila:15/VUOA35.116-151/a43dc2:user/release-keys`
- `ro.vendor.build.fingerprint`: `motorola/manila_g/manila:13/VUOA35.116-151/a43dc2:user/release-keys`
- `ro.bootloader`: `manila_g-151_2026_202607300508_user`

### Interpretation

- The running kernel identifies itself as Linux `5.15.197`, with Android common-kernel lineage string `android13-8-00005`.
- The kernel release suffix includes `g2d8ad9139b89`, which resembles an abbreviated Git commit identifier, and build identifier `ab15063902`. These are useful correlation keys, but their meaning and mapping to Motorola's published source must be independently verified.
- The kernel build reports Clang `14.0.7`, Android LLVM build `8508608`, base revision `r450784e`, LLVM source revision `4c603efb0cca074e9238af4b4106c30add4418f6`, and LLD `14.0.7`.
- The kernel build timestamp is 20 March 2026 at 11:14:06 UTC.
- The `android13` kernel lineage string does not mean the handset's Android system is Android 13. The system and boot-image fingerprints identify Android 15; the vendor fingerprint separately reports Android 13.
- Reads of `/proc/sys/kernel/osrelease` and `/proc/sys/kernel/version` were denied to the ADB shell. This is an access restriction and does not imply those kernel values are absent.

A GitHub code search of the Motorola kernel repository for `g2d8ad9139b89`, `ab15063902`, and the full kernel release string returned no indexed matches. This negative search result is inconclusive: it does not prove the commit or build metadata is absent from the source history or build artifacts.

## Comparison and interpretation

| Evidence | Value | Interpretation |
|---|---|---|
| Lab device build | `VUOA35.116-151` | Exact installed build reported by the handset |
| Motorola kernel source release | `MMI-VUOA35.116-100` | Officially published Moto G35 kernel source release |
| Shared family prefix | `VUOA35.116` | Same broad software release family |
| Build suffix | `151` versus `100` | Not an exact build-number match |
| Running kernel release | `5.15.197-android13-8-00005-g2d8ad9139b89-ab15063902` | Runtime kernel identity observed on device |
| Candidate source commit suffix | `g2d8ad9139b89` | Potential correlation key; mapping unverified |

**Conclusion — Source-confirmed:** Motorola publicly released Moto G35 kernel source associated with `MMI-VUOA35.116-100`.

**Conclusion — Observed:** The lab handset reports firmware `VUOA35.116-151` and the running-kernel identity recorded above.

**Conclusion — Unknown:** The inspected public evidence does not establish that the `MMI-VUOA35.116-100` source branch is the exact kernel source revision used for the lab handset's `VUOA35.116-151` firmware.

A shared prefix, device codename, board family, kernel version, or compatible device-tree overlay is not sufficient to prove an exact source-to-binary match.

## Evidence needed for an exact match

1. A Motorola-published kernel release or build manifest explicitly associated with `VUOA35.116-151`.
2. A signed firmware package manifest or build metadata identifying the kernel source revision/commit.
3. Corroboration that the runtime kernel suffix `g2d8ad9139b89` maps to a specific commit in Motorola's published source.
4. A Motorola source release note explicitly mapping the later firmware build to a specific kernel branch or commit.

Until such evidence is found, use this wording: **“Motorola-published Moto G35 kernel source for build family VUOA35.116-100; exact correspondence to lab firmware VUOA35.116-151 remains unverified.”**

## Safety and scope

This is a source-provenance investigation only. It does not require flashing firmware, unlocking the bootloader, modifying partitions, or extracting private identifiers. Do not publish IMEI, serial number, account data, or other unique device identifiers while collecting additional evidence.
