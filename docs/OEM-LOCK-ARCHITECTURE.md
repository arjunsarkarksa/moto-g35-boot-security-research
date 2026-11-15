# OEM-lock framework and vendor HAL

## Purpose

This document records the Android OEM-lock service architecture observed on the Moto G35 5G and distinguishes Android framework behavior from vendor implementation details.

## Device-side observations

The device exposed both the framework OEM-lock service and the AIDL HAL service:

- Framework service: `oem_lock`
- AIDL HAL instance: `android.hardware.oemlock.IOemLock/default`
- Vendor process: `/vendor/bin/hw/vendor.sprd.hardware.oemlock-service`
- Vendor service name: `vendor.oemlock-default`

The vendor VINTF manifest at `/vendor/etc/vintf/manifest/vendor-oemlock-default.xml` declared the AIDL HAL:

```xml
<manifest version="1.0" type="device">
    <hal format="aidl">
        <name>android.hardware.oemlock</name>
        <interface>
            <name>IOemLock</name>
            <instance>default</instance>
        </interface>
    </hal>
</manifest>
```

The vendor init file `/vendor/etc/init/vendor-oemlock-default.rc` declared:

```text
service vendor.oemlock-default /vendor/bin/hw/vendor.sprd.hardware.oemlock-service
    class hal
    user system
    group system
```

The process was observed running as UID/GID 1000, with SELinux context:

```text
u:r:hal_oemlock_default:s0
```

The device was in SELinux enforcing mode.

## ADB shell access boundary

Attempts by the unprivileged ADB shell to inspect the vendor executable directly were denied. Audit records showed SELinux access denials with:

- Source context: `u:r:shell:s0`
- Target context: `u:object_r:hal_oemlock_default_exec:s0`
- Target class: `file`
- `permissive=0`

This demonstrates a real access boundary for the shell domain. It does not demonstrate that the Android framework cannot communicate with the HAL through its authorized Binder interface. Direct executable-file access and framework-to-HAL IPC are different operations.

The OEM-lock process showed zero effective Linux capabilities in the observed process status. Capability bounding masks are not the same as effective capabilities. SELinux remains an independent mandatory access-control layer.

## AIDL interface

The Android OEM-lock AIDL interface defines these methods:

1. `getName()`
2. `isOemUnlockAllowedByCarrier()`
3. `isOemUnlockAllowedByDevice()`
4. `setOemUnlockAllowedByCarrier()`
5. `setOemUnlockAllowedByDevice()`

The interface separates carrier policy from device/user policy. The Android framework applies permission and user-restriction checks around the relevant operations.

## Framework authorization versus current lock state

These concepts must not be conflated:

- **OEM-unlock authorization**: whether the applicable user, carrier, and administrative policy permits an unlock request.
- **Current bootloader state**: whether the bootloader currently reports the device as locked or unlocked.

The device reported a locked bootloader and Verified Boot green state. A previously observed OEM-unlock-allowed property is not contradictory: permission to request unlocking is not the same as an unlocked bootloader.

## Permission observations

The Settings package was observed requesting/holding OEM-unlock-related privileged permissions, including:

- `MANAGE_USER_OEM_UNLOCK_STATE`
- `OEM_UNLOCK_STATE`
- `READ_OEM_UNLOCK_STATE`

The platform permission declarations inspected identified privileged/signature protection levels for these permissions. These grants to privileged platform components do not imply that the ADB shell has the same authority.

## Framework source interpretation

AOSP's OEM-lock framework design includes permission checks and checks for relevant user restrictions. It coordinates framework policy with vendor HAL responses and persistent-data-block state.

AOSP source is useful for understanding the Android architecture, but it is not proof that every implementation detail is identical in Motorola's Android 15 build. The exact vendor service implementation and proprietary bootloader behavior are not fully established by the available evidence.

A particularly important distinction in framework APIs is between aggregate authorization (whether unlocking is allowed) and the current device lock state. Some framework paths may synchronize persistent state while evaluating aggregate authorization; therefore, this research does not invoke privileged OEM-lock methods or attempt to change state.

## Conclusions

**Observed**
- Framework and AIDL OEM-lock services were registered.
- The vendor OEM-lock process was running.
- The vendor process used the `hal_oemlock_default` SELinux domain.
- SELinux denied direct ADB-shell access to the vendor executable.

**Source-confirmed**
- The Android OEM-lock AIDL interface separates carrier and device authorization methods.
- The Android framework applies permission and policy checks around OEM-lock operations.

**Not established**
- The exact internal decision logic of Motorola's proprietary OEM-lock HAL.
- Whether the vendor HAL would authorize an unlock request for this specific device.
- Whether Motorola's unlock portal currently accepts this model or software build.
- The exact internal communication and storage behavior of the proprietary bootloader.

No bypass or modification of OEM-lock state was attempted.
