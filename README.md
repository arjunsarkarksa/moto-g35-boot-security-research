# Moto G35 5G Boot Security Research

**Project motto: Build Shields, Not Weapons.**

This repository documents read-only technical research into the boot-security architecture of the Motorola Moto G35 5G (XT2433-4, product name `manila`). It combines observations from the device with analysis of publicly available Android and Motorola/Unisoc kernel source.

## Research goals

- Record the device's reported bootloader and Verified Boot state.
- Understand the Android OEM-lock framework and vendor HAL boundary.
- Document FRP-related indicators without attempting to bypass Factory Reset Protection.
- Analyze relevant UMS9620 device-tree and APB protection source.
- Separate directly observed facts from source-level indications and unanswered questions.

## Scope and safety

This is a **read-only research and documentation project**. It does not provide instructions to bypass FRP, defeat bootloader security, exploit privileged services, modify protected partitions, or circumvent SELinux. No unlocking, flashing, or persistent-data modification is part of the documented experiments.

Only test devices that you own or are explicitly authorized to examine. Redact serial numbers, IMEIs, account identifiers, tokens, and other personal data before publishing logs.

## Device baseline

The tested device reported:

- Device: Motorola Moto G35 5G
- Model/SKU: XT2433-4
- Product/codename: `manila`
- Android: 15 (API 35)
- Build: `VUOA35.116-151`
- Build type: `user`
- Build tags: `release-keys`
- Reported bootloader state: locked
- Verified Boot state: green
- dm-verity mode: enforcing

See [Device baseline and observations](docs/DEVICE-BASELINE.md).

## Main findings

### Device observations

The device reported `ro.boot.flash.locked=1`, `ro.boot.vbmeta.device_state=locked`, `ro.boot.verifiedbootstate=green`, and `ro.boot.veritymode=enforcing`. Fastboot also reported `secure=true` and `frp-state: Protection`.

These are observations of reported state. In particular, the Fastboot FRP indicator does **not**, by itself, establish that the device is currently blocked by FRP or identify the contents of FRP storage.

### OEM-lock architecture

The device exposes the Android OEM-lock AIDL interface and a vendor service named `vendor.sprd.hardware.oemlock-service`. The service runs in the SELinux domain `hal_oemlock_default`. ADB shell access to the vendor executable was denied by SELinux; that denial does not prove that Android's privileged framework-to-HAL communication is unavailable.

See [OEM-lock architecture](docs/OEM-LOCK-ARCHITECTURE.md).

### UMS9620 APB protection source

The public kernel source contains two APB protection device-tree nodes (AON and PMU) and a matching Unisoc driver. The nodes configure write-violation interrupt reporting and last-event recording.

The production GKI fragment explicitly disables `SPRD_DEBUG`; the debug fragment enables `SPRD_DEBUG` and configures `SPRD_APB_PROT=m`. This is build-recipe evidence, not confirmation of the exact merged configuration or runtime state of the tested phone.

See [Kernel and APB protection analysis](docs/KERNEL-APB-PROTECTION.md).

## Repository structure

- `docs/DEVICE-BASELINE.md` — observed device properties and interpretation.
- `docs/OEM-LOCK-ARCHITECTURE.md` — Android framework, vendor HAL, permissions and SELinux observations.
- `docs/OEM-LOCK-READ-PATHS.md` — Android 15 read-path analysis and state-changing API boundary.
- `docs/VENDOR-OEMLOCK-SOURCE-AVAILABILITY.md` — targeted public-source search and unresolved vendor implementation boundary.
- `docs/KERNEL-APB-PROTECTION.md` — UMS9620 device-tree, driver and build-fragment analysis.
- `docs/METHOD.md` — evidence labels, methodology and limitations.
- `docs/SOURCE-REFERENCES.md` — pinned kernel-source links and evidence mapping.
- `docs/RESEARCH-STATUS.md` — consolidated findings and unresolved questions.

## Source references

- [Motorola Mobility kernel source repository](https://github.com/MotorolaMobilityLLC/kernel-sprd)
- [Android Open Source Project](https://source.android.com/)
- [Android Verified Boot](https://source.android.com/docs/security/features/verifiedboot)

The Motorola kernel source examined for this work was the Android 15 branch `andriod-15-release-vuoa35` (branch spelling as published by the upstream repository), at a commit beginning `eefd920cfb04`.

## Evidence standard

Every conclusion should be labeled as one of:

- **Observed** — directly returned by the tested device or tool.
- **Source-confirmed** — directly supported by inspected source code or configuration.
- **Inference** — a reasoned interpretation that is not directly verified on the device.
- **Unknown** — evidence is insufficient to make a reliable claim.

An empty command result, denied shell access, or unimplemented Fastboot command is not proof that a feature or capability does not exist.

## Status

The initial read-only research phase is documented. See [Research status and unresolved questions](docs/RESEARCH-STATUS.md) for the consolidated findings and remaining unknowns. This repository is a technical record, not a claim that every aspect of Motorola's proprietary bootloader implementation has been reverse-engineered.
