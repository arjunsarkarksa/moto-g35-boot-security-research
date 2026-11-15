# Vendor OEM-lock source availability

## Question

Is the Motorola-specific implementation behind the Moto G35's vendor OEM-lock HAL publicly available in the Motorola source examined for this project?

## Device-side facts

The tested Moto G35 reports the AIDL HAL service:

`vendor.sprd.hardware.oemlock-service`

Its vendor init service definition launches:

`/vendor/bin/hw/vendor.sprd.hardware.oemlock-service`

The vendor VINTF manifest declares `android.hardware.oemlock`, interface `IOemLock`, instance `default`. The process was observed in SELinux domain `hal_oemlock_default`. The phone also contains `/vendor/lib64/oemlock_ca.so` and a 32-bit counterpart.

These facts establish the presence of the service and related vendor components on the tested firmware. They do not reveal their internal implementation or prove how the library is linked.

## Public-source search performed

A targeted search was made in the public Motorola repository:

- Repository: [MotorolaMobilityLLC/kernel-sprd](https://github.com/MotorolaMobilityLLC/kernel-sprd)
- Branch: `andriod-15-release-vuoa35` (upstream spelling)
- Examined commit prefix: `eefd920cfb04`
- Search terms: `vendor.sprd.hardware.oemlock-service`, `oemlock_ca.so`, and `IOemLock`

No matching vendor service implementation or OEM-lock client-library source was located in that repository search. The repository is a kernel-source tree; the absence of these matches there does not establish that no source exists elsewhere.

Additional global GitHub searches found third-party Unisoc device firmware-dump and proprietary-file lists that name `vendor.sprd.hardware.oemlock-service` and `oemlock_ca.so`. Those entries are useful evidence that these are distributed as vendor components on other Unisoc-based devices, but they are file manifests or proprietary blobs, not the source code for Motorola's Moto G35 implementation.

## What can and cannot be concluded

**Source-search result:** The Motorola-specific vendor OEM-lock implementation was not located in the inspected public Motorola kernel repository.

**Not established:**

- That Motorola has never published the implementation elsewhere.
- That the service binary is identical across Unisoc devices or firmware builds.
- That `oemlock_ca.so` is directly linked to the service.
- The internal carrier-authorization logic, device-authorization logic, persistent-storage behavior, or any device-specific policy.
- That the tested firmware was built from the examined kernel revision.

The AOSP Android 15 framework and AIDL contract remain useful for understanding the framework/HAL boundary, but they are not substitutes for Motorola's vendor source.

## Research boundary and next evidence

The next defensible source would be a matching Moto G35 firmware extraction with trustworthy build provenance, or a Motorola-published vendor source package that includes the service implementation. Until such evidence is available, the internal vendor behavior remains **Unknown**.

This project will not attempt to invoke privileged OEM-lock methods, issue raw Binder transactions, alter persistent data, or bypass SELinux to fill this source gap. The current finding is documented as a limitation rather than guessed.

## References

- [Motorola Mobility kernel source](https://github.com/MotorolaMobilityLLC/kernel-sprd)
- [AOSP Android 15 OemLockService.java](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/android15-release/services/core/java/com/android/server/oemlock/OemLockService.java)
- [AOSP Android 15 VendorLockAidl.java](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/android15-release/services/core/java/com/android/server/oemlock/VendorLockAidl.java)
- [AOSP OEM-lock AIDL interface](https://android.googlesource.com/platform/hardware/interfaces/+/master/oemlock/aidl/android/hardware/oemlock/IOemLock.aidl)
