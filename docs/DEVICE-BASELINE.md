# Device baseline and observations

## Test device

| Property | Observed value |
|---|---|
| Device | Motorola Moto G35 5G |
| Model/SKU | XT2433-4 |
| Product name | `manila` |
| Android version | 15 |
| API level | 35 |
| Build fingerprint | `motorola/manila_gn/manila:15/VUOA35.116-151/a43dc2:user/release-keys` |
| Build ID | `VUOA35.116-151` |
| Incremental build | `a43dc2` |
| Build type | `user` |
| Build tags | `release-keys` |
| Security patch reported | 2026-08-05 |
| Vendor security patch reported | 2026-04-05 |

These values were collected from the test device during the research session. They identify the software build examined, not every Moto G35 firmware release.

## Boot and Verified Boot observations

The following Android properties were observed:

| Property | Value | What it supports |
|---|---|---|
| `ro.boot.flash.locked` | `1` | Bootloader reports locked state |
| `ro.boot.vbmeta.device_state` | `locked` | Verified Boot metadata reports locked device state |
| `ro.boot.verifiedbootstate` | `green` | Verified Boot reports the green state |
| `ro.boot.veritymode` | `enforcing` | dm-verity reports enforcing mode |
| `ro.boot.veritymode.managed` | `yes` | Device reports managed verity mode |

Fastboot observations included:

- `secure=true`
- `is-userspace=no`
- Current slot: `a`
- Slot count: `2`
- `frp-state: Protection`

### Interpretation limits

- These properties describe the state reported by the running software/bootloader at the time of testing.
- `frp-state: Protection` is an FRP-related bootloader indicator. It does not prove that a setup-time FRP challenge is currently active, that a particular Google account is required, or that FRP storage is damaged.
- `fdr-allowed: yes` does not establish that FRP has been cleared.
- `ro.frp.pst=/dev/block/by-name/persist` identifies a configured storage path; it does not reveal the contents of that storage.
- A reported OEM-unlock permission is distinct from the current bootloader lock state. Authorization to request unlocking and the device's current locked/unlocked state are different concepts.

## OEM-unlock and Fastboot command observations

The device previously reported `sys.oem_unlock_allowed=1`. This is a permission-related property and does not mean the bootloader is currently unlocked.

The command `fastboot flashing get_unlock_ability` returned:

```text
FAILED (remote: 'Not implement.')
```

This means the command is not implemented by the tested Fastboot interface. It does **not** prove that unlocking is impossible or that the device is eligible for unlocking.

The tested interface did not recognize `fastboot oem get_unlock_data` or `fastboot oem device-info`. Unsupported commands should not be treated as definitive security-state evidence.

## Android user and device-policy observations

During the documented checks:

- User 0 was the primary system user and was reported as running unlocked.
- User 0's displayed restrictions section contained no listed restrictions.
- The device reported `device_provisioned=1` and `user_setup_complete=1`.
- The Device Policy dump reported Device Owner Type `-1` and no displayed Profile Owner.
- The account dump for User 0 reported zero accounts at the time of that check.

These observations describe the device at the time they were collected. They do not prove that no enterprise-management components are installed, nor do they establish the historical state of removed users or accounts.

## Persistent data block and FRP service observations

The Persistent Data Block service dump reported:

- Block device path: `/dev/block/by-name/persist`
- File-backed: `false`
- Reported block-device size: 2,097,152 bytes
- Reported writable flag: `true`
- A configured allowed UID was displayed.

An ADB shell attempt to access protected data was denied. The service's reported writable flag is not evidence that the unprivileged ADB shell can write to the block device. No FRP data was read, changed, or cleared as part of this research.

## Safe reporting

Before publishing additional logs, remove serial numbers, IMEIs, account names, email addresses, unique device identifiers, access tokens, and other personal information.
