# Research methodology and evidence standard

## Research approach

The project uses read-only inspection of the owned Moto G35 5G and analysis of publicly available Android and Motorola/Unisoc source code.

The primary device-side tools used included ADB and Fastboot from PowerShell. Public kernel source was inspected through GitHub's repository/tree/blob interfaces. The work avoided broad source-tree downloads and repeated scans when targeted source files were sufficient.

## Evidence labels

| Label | Meaning |
|---|---|
| Observed | Direct output from the test device or a tool. |
| Source-confirmed | Directly supported by inspected source code, a manifest, device tree, or build fragment. |
| Inference | A plausible interpretation based on observations or source, not independently verified. |
| Unknown | Available evidence is insufficient for a reliable conclusion. |

A source-code feature is not automatically proof that the feature is enabled in the installed firmware. A build fragment is not the same as the final merged kernel configuration. A registered service is not proof that every method call succeeds. A permission declaration is not proof that an unprivileged caller holds that permission.

## Negative and ambiguous results

The following interpretation rules are used:

- Empty output is inconclusive unless the command's semantics make absence definitive.
- An unsupported Fastboot command proves only that the tested interface did not implement that command.
- A SELinux denial proves that the attempted access was denied in that context; it does not prove all other access paths are denied.
- A service registration proves registration, not successful operation of every method.
- A reported FRP protection indicator does not prove that a setup-time FRP challenge is active.
- A property describing unlock permission must not be confused with current bootloader lock state.

## Safety boundaries

The research is intentionally non-invasive. It does not attempt to:

- Bypass Factory Reset Protection.
- Unlock or exploit the bootloader.
- Write to persistent-data-block or other protected partitions.
- Modify boot, vbmeta, recovery, or system partitions.
- Disable SELinux or evade access controls.
- Extract credentials, tokens, or private user data.

## Reproducibility and privacy

Record the device model, Android build, date, command, and relevant output when adding a new experiment. Redact serial numbers, IMEIs, account identifiers, email addresses, tokens, and other personal information before committing logs.

When source references are updated, record the exact repository, branch/ref, and commit SHA. Re-check conclusions if the firmware or kernel source revision changes.

## Open questions

The following remain outside what the current evidence can prove:

1. The complete proprietary Motorola bootloader decision logic.
2. The exact carrier/device authorization response returned by the vendor OEM-lock HAL for this handset.
3. The final merged kernel configuration used for the exact installed firmware.
4. Runtime binding and successful hardware programming of the APB protection nodes.
5. Any direct relationship between APB protection and FRP or OEM-unlock policy.

These questions should remain explicitly unresolved unless new, safe evidence is obtained.
