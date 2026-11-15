# Research status

## Project state

**Status: Initial read-only research phase documented.**

The repository contains the device baseline, OEM-lock framework/HAL analysis, UMS9620 APB protection source analysis, and the methodology/evidence standard.

## Established findings

- **Observed:** The tested Moto G35 5G (XT2433-4, product `manila`) reported a locked bootloader, Verified Boot state `green`, and dm-verity `enforcing`.
- **Observed:** Fastboot reported `frp-state: Protection`. This alone does not establish that a setup-time FRP challenge is active.
- **Observed:** Android exposed the framework OEM-lock service and the vendor AIDL OEM-lock HAL. SELinux denied direct ADB-shell access to the vendor executable.
- **Source-confirmed:** AOSP defines separate carrier/device OEM-unlock authorization methods and framework permission checks.
- **Source-confirmed:** The examined UMS9620 source contains AON and PMU APB protection nodes and a matching driver.
- **Build-recipe evidence:** The inspected production fragment disables `SPRD_DEBUG`; the debug fragment enables it and configures `SPRD_APB_PROT=m`. The final installed kernel configuration is not available.

## Important unresolved questions

1. Motorola's proprietary OEM-lock HAL decision logic and its response for this handset.
2. The exact final merged kernel configuration used by the installed firmware.
3. Whether the APB protection driver binds and programs the hardware at runtime.
4. Whether APB protection has any relationship to FRP or OEM-unlock policy. No such relationship has been established.
5. Whether Motorola's published `MMI-VUOA35.116-100` kernel source is the exact source revision used for the lab device's observed `VUOA35.116-151` firmware. The shared release-family prefix is not proof of an exact match; see [Firmware and kernel source provenance](FIRMWARE-KERNEL-PROVENANCE.md).

## Scope boundary

The documented work is read-only. No FRP bypass, bootloader unlock, exploit, protected-partition write, or security-control circumvention was attempted.

Do not treat an unsupported command, empty output, access denial, source-code feature, or build fragment as proof beyond what it directly demonstrates. New claims should be added only when supported by safe, reproducible evidence.

## Source revision

- Kernel repository: [MotorolaMobilityLLC/kernel-sprd](https://github.com/MotorolaMobilityLLC/kernel-sprd)
- Branch: `andriod-15-release-vuoa35` (upstream spelling)
- Examined commit prefix: `eefd920cfb04`

This status describes the evidence collected so far; it does not claim complete reverse engineering of Motorola's proprietary bootloader.
