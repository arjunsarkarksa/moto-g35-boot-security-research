# UMS9620 kernel source: APB protection

## Source context

The kernel source examined was Motorola Mobility's public repository:

- Repository: [MotorolaMobilityLLC/kernel-sprd](https://github.com/MotorolaMobilityLLC/kernel-sprd)
- Branch: `andriod-15-release-vuoa35` (upstream branch spelling)
- Examined commit prefix: `eefd920cfb04`
- Device build examined: `VUOA35.116-151`
- Device product: `manila`

The build recipes and device-tree references connect the UMS9620 source configuration with the Manila device family. This is a source/build-recipe connection; it does not prove that the tested phone's installed kernel was built from this exact commit and configuration.

## Device-tree nodes

File:

`arch/arm64/boot/dts/sprd/ums9620/apb_prot.dtsi`

The file defines two nodes using compatible string `sprd,apb-prot`:

- AON APB protection node, using `aon_apb_regs`, interrupt 254.
- PMU APB protection node, using `pmu_apb_regs`, interrupt 255.

Both nodes configure:

- `sprd,lock_magic = <0x6896>`
- `sprd,lock_but_write_int`
- `sprd,record_last`

The following properties are commented out in the inspected nodes:

- `sprd,panic`
- `sprd,glb_lock`
- `sprd,lock_id`

The absence of those properties in these nodes means the driver does not receive those options from these particular device-tree entries. It does not establish the state of unrelated hardware protection mechanisms.

## Driver behavior

Driver:

`drivers/unisoc_platform/debug/apb_prot/sprd_apb_prot.c`

The driver matches device-tree nodes with:

```c
{ .compatible = "sprd,apb-prot", }
```

The probe path obtains the APB syscon regmap, reads configuration properties, requests a threaded interrupt, and calls the hardware-setup routine. The driver is registered using a post-core initcall.

The inspected interrupt handler:

1. Reads the command/status register at offset `0xff0`.
2. Reads the debug snapshot interface at offset `0xffc`.
3. Logs the illegal master and address-offset fields.
4. Clears interrupt status using offset `0xff8`.
5. Calls `BUG()` only when the driver's panic option is enabled.

Because `sprd,panic` is commented out in the inspected AON and PMU nodes, those nodes do not request the driver's panic option through that property.

The driver also provides an exported helper, `sprd_apb_prot_write()`, which attempts to acquire/release a direct-access lock around a register write when the global-lock status bit is set. This is an internal kernel helper, not a user-space interface.

## Kconfig and Makefile

The APB protection Kconfig entry is:

```makefile
config SPRD_APB_PROT
        tristate "Spreadtrum APB PROT Support"
        depends on SPRD_DEBUG
```

The Makefile builds the driver according to `CONFIG_SPRD_APB_PROT`:

```makefile
obj-$(CONFIG_SPRD_APB_PROT) += sprd_apb_prot.o
```

The parent debug Kconfig defines `SPRD_DEBUG` as a boolean option dependent on `ARCH_SPRD || COMPILE_TEST`.

## Build-fragment comparison

The inspected production fragment:

`arch/arm64/configs/sprd_gki_qogirn6pro.fragment`

contains:

```text
# CONFIG_SPRD_DEBUG is not set
# CONFIG_SPRD_DEBUG_GIC is not set
```

The inspected debug fragment:

`arch/arm64/configs/sprd_gki_qogirn6pro_debug.fragment`

contains:

```text
CONFIG_SPRD_APB_PROT=m
CONFIG_SPRD_DEBUG=y
```

The production build recipe referenced the production fragment, while the debug recipe included the debug fragment as an additional configuration input. Both recipes referenced the Manila overlay device-tree artifact:

`arch/arm64/boot/dts/sprd/ums9620-2h10_manila-overlay.dtbo`

### Configuration conclusion and limitation

The production fragment explicitly disables `SPRD_DEBUG`. Since `SPRD_APB_PROT` depends on `SPRD_DEBUG`, this is strong evidence that the production recipe is not enabling the APB protection driver through that fragment.

However, the fragment alone is not the final merged kernel `.config`. The exact final configuration depends on all inputs and build steps. The available evidence does not include the final merged `.config` from the exact firmware installed on the tested phone.

Therefore, this repository does **not** claim that APB protection is definitely absent or active at runtime on the tested device.

## Evidence summary

**Source-confirmed**
- UMS9620 APB device-tree source defines AON and PMU protection nodes.
- Both nodes request write-violation interrupt reporting and last-event recording.
- The driver contains a matching compatible string and a probe/interrupt-handling path.
- APB protection depends on `SPRD_DEBUG`.
- The inspected production fragment disables `SPRD_DEBUG`; the debug fragment enables it and sets APB protection to module mode.
- The build recipes reference a Manila overlay artifact.

**Not established**
- Whether the exact production firmware's final merged kernel configuration includes the driver.
- Whether either APB node successfully binds to the driver on the tested phone.
- Whether the hardware registers were successfully programmed at runtime.
- Whether APB protection controls FRP, OEM unlocking, or bootloader authorization.

APB protection is a peripheral-bus access-control mechanism in the inspected source. No evidence collected here establishes a direct relationship between this driver and FRP or bootloader unlock policy.


## Live-device runtime observations (Experiment 179–180)

On 15 November 2026, read-only ADB inspection of the tested Moto G35 exposed these live device-tree nodes:

- `/sys/firmware/devicetree/base/soc/aon/aon-apb-prot`
- `/sys/firmware/devicetree/base/soc/aon/pmu-apb-prot`

The corresponding platform devices were also listed under `/sys/bus/platform/devices` with these registered names:

- `soc:aon:aon-apb-prot`
- `soc:aon:pmu-apb-prot`

Both device paths resolved to sysfs device entries. For each entry, checking its `driver` child returned `No such file or directory`; therefore, no driver symlink was observed for either platform device at the time of inspection. The `uevent` files could not be read by the ADB shell because access was denied.

The platform-driver directory listing contained names such as `dw-apb-uart`, but no obvious `sprd-apb-prot` entry. `dw-apb-uart` is an APB UART driver and must not be confused with the APB protection driver.

### Runtime interpretation

**Observed:** Both APB protection nodes exist in the live device tree, and both corresponding platform devices are registered.

**Observed:** Neither platform device exposed a `driver` symlink during this inspection. This is evidence that neither device was bound to a platform driver at that time.

**Not established:** Whether the protection hardware has been initialized or configured by another component, firmware, boot stage, or differently named driver; whether a driver can bind later; or whether the absence of a sysfs driver link reflects all relevant protection state.

The runtime result is consistent with the extracted kernel configuration not containing the named `CONFIG_SPRD_APB_PROT` and `CONFIG_SPRD_DEBUG` options, but it does not by itself prove the complete hardware protection state. No device settings, registers, or files were modified.


## Device-to-device-tree association (Experiment 181)

Read-only ADB inspection of the two registered platform devices showed:

- `soc:aon:aon-apb-prot/of_node` resolves to `/sys/firmware/devicetree/base/soc/aon/aon-apb-prot`.
- `soc:aon:pmu-apb-prot/of_node` resolves to `/sys/firmware/devicetree/base/soc/aon/pmu-apb-prot`.

This confirms that each registered platform device is associated with its corresponding live device-tree node. Attempts to read each device's `modalias` returned `Permission denied`, so the alias value was not obtained.

Combined with the earlier observation that neither device exposed a `driver` symlink, the current evidence is: both platform devices are registered and associated with their device-tree nodes, but no bound driver was observed at the time of inspection. The denied `modalias` read does not establish whether a matching driver exists elsewhere or whether another component configures the hardware.


## Loaded-module and driver-name inspection (Experiment 182)

A read-only inspection of the handset's loaded-module list (/proc/modules), filtered for APB/Spreadtrum-related names, showed numerous loaded Unisoc/Spreadtrum modules, including modem, PMIC, GPIO, clock, storage, and platform components. No loaded module named sprd_apb_prot or other obvious APB-protection module name appeared in the returned output.

The platform-driver directory again showed dw-apb-uart, alongside unrelated entries, but no obvious sprd-apb-prot driver entry. dw-apb-uart is an APB UART controller driver, not evidence of the APB protection driver.

### Interpretation and limits

This adds runtime evidence that no obvious APB-protection module was loaded and no obvious APB-protection driver name was registered in the inspected driver directory at that time. /proc/modules lists loadable modules currently loaded; it does not list drivers built into the kernel. The inspection also does not establish whether an APB-protection module file exists on disk, whether another component configures the hardware, or whether hardware protection is active. The module-directory portion of the command output was not included in the captured excerpt, so no conclusion is drawn about module files on disk.

No modules were loaded or unloaded, and no device settings or registers were modified.


## On-disk module filename search (Experiment 183)

A read-only search for filenames matching APB/protection-related patterns was run with a maximum depth of three in the following locations:

- `/vendor_dlkm/lib/modules`
- `/vendor/lib/modules`
- `/system/lib/modules`
- `/odm/lib/modules`
- `/lib/modules`

The first two directories existed but returned no matching filenames. The latter three directories were reported as not present. No matching APB-protection module file was found in the searched locations.

### Interpretation and limits

This result is consistent with the absence of an obvious separately installed APB-protection module in the checked locations. It is not a complete filesystem or partition inventory, does not search outside those paths, and does not rule out a built-in driver, a differently named module, or configuration by boot firmware or another component. Combined with the lack of an observed platform-driver binding and the lack of an obvious loaded module name, the evidence strengthens—but does not prove—the hypothesis that the inspected production system is not running the source-tree APB protection driver. Hardware protection state remains undetermined.

No files or device settings were modified.
