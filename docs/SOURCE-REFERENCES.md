# Source reference index

This index maps the principal source-level claims in this repository to the files inspected. Kernel links are pinned to the examined Motorola kernel revision prefix so readers can compare the same source snapshot.

## Motorola / Unisoc kernel source

Repository: [MotorolaMobilityLLC/kernel-sprd](https://github.com/MotorolaMobilityLLC/kernel-sprd)

Examined branch: `andriod-15-release-vuoa35` (upstream spelling)  
Examined commit prefix: `eefd920cfb04`

| Source file | Why it matters | Supports |
|---|---|---|
| [UMS9620 APB protection device tree](https://github.com/MotorolaMobilityLLC/kernel-sprd/blob/eefd920cfb04/arch/arm64/boot/dts/sprd/ums9620/apb_prot.dtsi) | AON and PMU APB protection nodes and their properties | Source configuration only; not runtime activation |
| [APB protection driver](https://github.com/MotorolaMobilityLLC/kernel-sprd/blob/eefd920cfb04/drivers/unisoc_platform/debug/apb_prot/sprd_apb_prot.c) | Compatible match, register handling, setup, interrupt path, helper function | Driver implementation in the examined source |
| [APB protection Kconfig](https://github.com/MotorolaMobilityLLC/kernel-sprd/blob/eefd920cfb04/drivers/unisoc_platform/debug/apb_prot/Kconfig) | Dependency on `SPRD_DEBUG` | Build-time dependency |
| [APB protection Makefile](https://github.com/MotorolaMobilityLLC/kernel-sprd/blob/eefd920cfb04/drivers/unisoc_platform/debug/apb_prot/Makefile) | Object selection using `CONFIG_SPRD_APB_PROT` | Build integration |
| [Spreadtrum debug Kconfig](https://github.com/MotorolaMobilityLLC/kernel-sprd/blob/eefd920cfb04/drivers/unisoc_platform/debug/Kconfig) | Definition of the parent debug option | Kconfig definition |
| [Production GKI fragment](https://github.com/MotorolaMobilityLLC/kernel-sprd/blob/eefd920cfb04/arch/arm64/configs/sprd_gki_qogirn6pro.fragment) | Production fragment's debug-option settings | One build input, not final merged config |
| [Debug GKI fragment](https://github.com/MotorolaMobilityLLC/kernel-sprd/blob/eefd920cfb04/arch/arm64/configs/sprd_gki_qogirn6pro_debug.fragment) | Debug fragment's APB/debug settings | One build input, not final merged config |
| [UMS9620 Manila overlay DTS](https://github.com/MotorolaMobilityLLC/kernel-sprd/blob/eefd920cfb04/arch/arm64/boot/dts/sprd/ums9620-2h10_manila-overlay.dts) | Device-family overlay source, where present in this revision | Build/device-family association, not proof of exact installed image |

If a pinned link does not resolve, inspect the repository branch and commit history; do not silently substitute a different revision.

## Android OEM-lock architecture

- [Android Open Source Project](https://source.android.com/)
- [Android Verified Boot documentation](https://source.android.com/docs/security/features/verifiedboot)

The OEM-lock discussion uses AOSP interface/framework design as an architectural reference. AOSP behavior is not treated as proof of Motorola's exact vendor implementation. Device-specific observations are separately recorded in [OEM-lock architecture](OEM-LOCK-ARCHITECTURE.md).

## Device-side evidence

The following observations came from the tested handset and are documented in [Device baseline](DEVICE-BASELINE.md):

- Android boot properties for lock state and Verified Boot.
- Fastboot-reported security and FRP indicators.
- Vendor VINTF declaration and init service definition for OEM-lock.
- Process SELinux context and ADB-shell access denials.
- Android service registration and package/permission observations.

Device-side outputs are not public source files. Reproducibility requires the same device model and software build, and should not require publishing unique device identifiers or private account data.

## Evidence limitations

A source file proves what the examined source contains. It does not by itself prove:

- That the tested phone runs a binary built from that exact revision.
- That a build fragment is the final merged kernel configuration.
- That a driver successfully binds or programs hardware at runtime.
- That an Android framework path matches every Motorola vendor modification.
- That APB protection controls FRP or OEM-unlock authorization.

See [Research status](RESEARCH-STATUS.md) for the current findings and unresolved questions.
