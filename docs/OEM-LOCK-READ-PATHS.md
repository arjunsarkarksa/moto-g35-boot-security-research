# OEM-lock read paths and state boundaries

## Source revision

This analysis uses the Android Open Source Project `android15-release` branch:

- [OemLockService.java](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/android15-release/services/core/java/com/android/server/oemlock/OemLockService.java)
- [VendorLockAidl.java](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/android15-release/services/core/java/com/android/server/oemlock/VendorLockAidl.java)
- [IOemLock.aidl](https://android.googlesource.com/platform/hardware/interfaces/+/master/oemlock/aidl/android/hardware/oemlock/IOemLock.aidl)

The Android 15 branch version of `OemLockService.java` was inspected directly. Its Gitiles blob is `d00268899a07c12b17e50abca294bcf7861b7b1d`. This is an AOSP reference baseline, not proof that Motorola's framework source is byte-for-byte identical.

## Distinct state questions

The framework keeps several questions separate:

| Question | Framework path | Meaning |
|---|---|---|
| Does carrier policy allow OEM unlock? | `isOemUnlockAllowedByCarrier()` | Reads the carrier authorization flag through the selected OEM-lock implementation. |
| Does device/user policy allow OEM unlock? | `isOemUnlockAllowedByUser()` | Reads the device-side authorization flag through the selected OEM-lock implementation. |
| Is OEM unlock allowed overall? | `isOemUnlockAllowed()` | Combines carrier and device authorization, then synchronizes the OEM-unlock bit in persistent data storage. |
| Is the bootloader currently unlocked? | `isDeviceOemUnlocked()` | Reads `ro.boot.flash.locked`; it is a current-state check, not an authorization check. |

The AIDL HAL methods `isOemUnlockAllowedByCarrier()` and `isOemUnlockAllowedByDevice()` are distinct read operations. The corresponding setters are state-changing operations and are outside this research.

## Important read-versus-write distinction

In the inspected Android 15 AOSP implementation, `isOemUnlockAllowed()` is **not a strictly read-only diagnostic**. After reading carrier and device authorization, it calls the persistent-data-block synchronization path. That path can update the OEM-unlock bit in the FRP/persistent-data partition.

For this reason, this project does not invoke `isOemUnlockAllowed()` or any setter.

The individual carrier and device/user getter paths forward to the HAL and return the result without calling the persistent-data-block synchronization helper in the inspected AOSP implementation. However, they are protected by privileged permissions:

- Carrier getter: `MANAGE_CARRIER_OEM_UNLOCK_STATE`.
- Device/user getter: `MANAGE_USER_OEM_UNLOCK_STATE`.
- Aggregate authorization and current unlocked-state getters: `READ_OEM_UNLOCK_STATE` or `OEM_UNLOCK_STATE`.

A normal ADB shell must not be assumed to hold these permissions. A permission grant observed for the Settings package does not transfer to the shell UID.

## What the Moto G35 evidence establishes

**Observed on the tested handset**
- The framework OEM-lock service and AIDL HAL service were registered.
- The vendor OEM-lock service process was running in `hal_oemlock_default`.
- The Settings package was observed with OEM-lock-related privileged permissions.
- The shell domain was denied direct file access to the vendor OEM-lock executable by SELinux.
- The bootloader reported locked state through `ro.boot.flash.locked=1`.

**Not established**
- The actual carrier authorization boolean returned by the vendor HAL.
- The actual device/user authorization boolean returned by the vendor HAL at the time of testing.
- The exact Motorola-specific framework implementation and any vendor modifications.
- Whether Motorola's proprietary bootloader would accept an unlock request.

The direct executable SELinux denial does not prove that framework Binder communication is blocked. Conversely, service registration does not prove that a particular getter returns successfully.

## Research decision

No direct OEM-lock Binder calls, raw Binder transactions, setters, aggregate authorization calls, or persistent-data-block operations are part of this phase.

The remaining carrier/device authorization values are classified as **Unknown** because the available unprivileged observations do not establish them. This is an evidence boundary, not a reason to bypass Android's permission model.
