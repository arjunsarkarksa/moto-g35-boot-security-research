# Research status

## Project state

**Status: Initial read-only research phase documented.**

The repository contains the device baseline, OEM-lock framework/HAL analysis, UMS9620 APB protection source analysis, firmware/kernel provenance evidence, and the methodology/evidence standard.

## Established findings

- **Observed:** The tested Moto G35 5G (XT2433-4, product `manila`) reported a locked bootloader, Verified Boot state `green`, and dm-verity `enforcing`.
- **Observed:** Fastboot reported `frp-state: Protection`. This alone does not establish that a setup-time FRP challenge is active.
- **Observed:** Android exposed the framework OEM-lock service and the vendor AIDL OEM-lock HAL. SELinux denied direct ADB-shell access to the vendor executable.
- **Source-confirmed:** AOSP defines separate carrier/device OEM-unlock authorization methods and framework permission checks.
- **Source-confirmed:** The examined UMS9620 source contains AON and PMU APB protection nodes and a matching driver.
- **Observed running-kernel configuration:** `/proc/config.gz` was pulled from the handset and inspected locally. It reports `CONFIG_IKCONFIG=y`, `CONFIG_IKCONFIG_PROC=y`, `CONFIG_SECURITY_SELINUX=y`, and `CONFIG_MODULES=y`. Neither `CONFIG_SPRD_APB_PROT` nor `CONFIG_SPRD_DEBUG` appears in the extracted configuration. This is evidence those named options are not enabled in the running configuration; it does not rule out differently named or firmware-level protection.
- **Observed:** The running handset kernel identifies itself as `5.15.197-android13-8-00005-g2d8ad9139b89-ab15063902`, built with Clang 14.0.7 and LLD 14.0.7, timestamped 20 March 2026 11:14:06 UTC.
- **Observed:** System and boot-image fingerprints identify Android 15, while the vendor fingerprint reports Android 13. The kernel's `android13` lineage string is not evidence that the system OS is Android 13.
- **Unverified correlation:** The kernel release contains `g2d8ad9139b89`, a potential source-commit correlation key. Its mapping to Motorola's published source has not been established.
- **Source-provenance status:** Motorola's published `MMI-VUOA35.116-100` source is in the same broad `VUOA35.116` family as lab firmware `VUOA35.116-151`; exact source-to-binary correspondence remains unknown. See [Firmware and kernel source provenance](FIRMWARE-KERNEL-PROVENANCE.md).

- **Observed live APB device-tree/platform state:** The handset exposes AON and PMU APB protection nodes, and corresponding platform devices `soc:aon:aon-apb-prot` and `soc:aon:pmu-apb-prot` are registered. Neither device exposed a `driver` symlink during read-only inspection, so no platform-driver binding was observed at that time. This does not prove the protection hardware is inactive or exclude initialization by another component. See [UMS9620 APB protection](KERNEL-APB-PROTECTION.md).

- **Observed APB device-tree association:** Each registered APB platform device's `of_node` symlink resolves to its corresponding live AON or PMU APB protection node. Reading `modalias` was denied, so the alias value remains unknown. No driver binding was observed for either device. See [UMS9620 APB protection](KERNEL-APB-PROTECTION.md).

## Important unresolved questions

1. Motorola's proprietary OEM-lock HAL decision logic and its response for this handset.
2. The mapping between the running kernel configuration and the exact Motorola source revision used by the installed firmware.
3. Whether the APB protection driver binds and programs the hardware at runtime.
4. Whether APB protection has any relationship to FRP or OEM-unlock policy. No such relationship has been established.
5. Whether the runtime kernel suffix `g2d8ad9139b89` maps to a commit in Motorola's published source or another upstream source tree.

## Scope boundary

The documented work is read-only. No FRP bypass, bootloader unlock, exploit, protected-partition write, or security-control circumvention was attempted.

Do not treat an unsupported command, empty output, access denial, source-code feature, or build fragment as proof beyond what it directly demonstrates. New claims should be added only when supported by safe, reproducible evidence.

## Source revision

- Kernel repository: [MotorolaMobilityLLC/kernel-sprd](https://github.com/MotorolaMobilityLLC/kernel-sprd)
- Branch: `andriod-15-release-vuoa35` (upstream spelling)
- Examined commit prefix: `eefd920cfb04`

This status describes the evidence collected so far; it does not claim complete reverse engineering of Motorola's proprietary bootloader.


- **Observed loaded-module/driver listing (Experiment 182):** The inspected /proc/modules output contained numerous Spreadtrum/Unisoc modules but no obvious sprd_apb_prot module name. The platform-driver listing showed dw-apb-uart but no obvious APB-protection driver name. This does not rule out a built-in driver, an on-disk but unloaded module, or hardware configuration by another component. The captured output did not include the module-directory listing, so on-disk module availability remains unassessed. See [UMS9620 APB protection](KERNEL-APB-PROTECTION.md).


- **Observed module-file search (Experiment 183):** A read-only filename search found no APB/protection-pattern matches in the existing `/vendor_dlkm/lib/modules` or `/vendor/lib/modules` directories; `/system/lib/modules`, `/odm/lib/modules`, and `/lib/modules` were not present. This limited search does not rule out built-in or differently named drivers, other module locations, or firmware configuration. Combined with Experiments 180–182, it strengthens but does not prove the hypothesis that the inspected production system is not running the source-tree APB protection driver. Hardware protection state remains undetermined. See [UMS9620 APB protection](KERNEL-APB-PROTECTION.md).
