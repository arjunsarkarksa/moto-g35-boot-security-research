# Public Research, Private Identifiers

**Project motto: Build Shields, Not Weapons.**

The purpose of this project is to publish careful, reproducible, defensive research that helps other people understand Android boot security. A useful explanation, a corrected misconception, or one person learning how to protect a device is a meaningful outcome.

The Moto G35 5G used in this research has been designated as a dedicated lab device rather than a personal-use phone. This commitment does not change the safety boundary: experiments remain authorized, read-only unless a future change is explicitly reviewed and documented, and never aimed at bypassing FRP or defeating device protections.

## Public-versus-private data rule

**Real secrets and unique device identifiers do not belong in this public repository.** This includes:

- IMEI, MEID, serial number, Android ID, phone numbers, SIM identifiers, and account identifiers.
- Passwords, passcodes, recovery codes, API keys, access tokens, private keys, certificates containing private material, and session cookies.
- Personal account names, email addresses, Wi-Fi credentials, location history, and unredacted logs or screenshots that expose them.

Do not publish these values even if the device is dedicated to the lab. Lab ownership is not a reason to expose identifiers publicly.

## How to document values

Use explicit placeholders that cannot be mistaken for real observations:

| Data category | Public representation |
|---|---|
| IMEI / MEID | `[REDACTED_IMEI]` / `[REDACTED_MEID]` |
| Serial number | `[REDACTED_SERIAL]` |
| Account identifier | `[REDACTED_ACCOUNT]` |
| Password / token / key | `[REDACTED_SECRET]` |
| Lab device label | `LAB-DEVICE-01` |

Never invent random values and present them as device output. If synthetic data is useful for an example, label it clearly as **synthetic example — not collected from the device**.

Do not publish hashes of IMEIs, serial numbers, or other stable identifiers as a workaround. Hashes can still enable correlation or identification.

## Private lab records

When a real identifier is genuinely needed for local troubleshooting, keep it in a separate encrypted local record with access limited to the lab operator. Keep that record outside the Git repository, outside Git history, and out of cloud-synced folders where possible. Do not paste secrets into public issues, discussions, commit messages, screenshots, or chat excerpts intended for publication.

Before every commit or upload:

1. Review changed files and command output for identifiers and secrets.
2. Redact sensitive values at the source of the public copy.
3. Check screenshots, filenames, metadata, and logs—not only the visible text.
4. If a secret is accidentally committed, treat it as exposed: revoke or rotate it where possible, then remove it from the repository history as appropriate. A later deletion does not undo exposure.

## Evidence integrity

Redaction must not alter the technical meaning of an observation. Preserve relevant property names, permission names, error messages, version details, and non-identifying evidence. Mark redacted fields explicitly. Keep the original private evidence separately if it is needed for reproducibility.

Use the evidence labels defined in [METHOD.md](METHOD.md): **Observed**, **Source-confirmed**, **Inference**, and **Unknown**. Never replace an unknown value with a fabricated one.

## Commitment

We publish to help defenders and learners, not to expose a device or its owner. The goal is not measured by an unlock, exploit, or bypass. If this work helps even one person understand a security boundary or avoid a mistake, it has achieved something worthwhile.

**Build Shields, Not Weapons — one smile at a time.**
