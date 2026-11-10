# Supported devices

Pulsar talks to Hikvision access-control / time-attendance terminals over
**ISAPI**. There are two tiers of support:

1. **Profiled models** (table below) — Pulsar knows exactly what the
   terminal can do, so fingerprint and/or face enrollment is enabled
   automatically.
2. **Any other ISAPI-compliant Hikvision terminal** — Pulsar will still
   connect, poll events, and sync users/cards via card + PIN. Fingerprint
   and face push are *not* auto-enabled for an unprofiled model, since
   Pulsar can't confirm the hardware supports them without the device's
   spec. [Ask us to add your model](#requesting-a-new-model) and we'll
   profile it.

This list is current as of **v0.3.13** (2026-11-10).

## Profiled models

| Model | Fingerprint | Face | Card | PIN | Time attendance |
|---|:-:|:-:|:-:|:-:|:-:|
| DS-K1T804AMF | ✅ | — | ✅ | ✅ | ✅ |
| DS-K1T804AMW | ✅ | — | ✅ | ✅ | ✅ |
| DS-K1T804BMF | ✅ | — | ✅ | ✅ | ✅ |
| DS-K1T804AEF | ✅ | — | ✅ | ✅ | ✅ |
| DS-K1T8005EFX | ✅ | — | ✅ | ✅ | ✅ |
| DS-K1T8005EFWX-B | ✅ | — | ✅ | ✅ | ✅ |
| DS-K1A802AMF-B | ✅ | — | ✅ | ✅ | ✅ |
| DS-K1A802AEF | ✅ | — | ✅ | ✅ | ✅ |
| DS-K1A802AEF-B | ✅ | — | ✅ | ✅ | ✅ |
| DS-K1A8503EF | ✅ | — | ✅ | ✅ | ✅ |
| DS-K1A8503EF-B | ✅ | — | ✅ | ✅ | ✅ |
| DS-K1A8503MF | ✅ | — | ✅ | ✅ | ✅ |
| DS-K1A8503MF-B | ✅ | — | ✅ | ✅ | ✅ |
| DS-K1A802AF *(no card reader)* | ✅ | — | — | ✅ | ✅ |
| DS-K1A802AF-B *(no card reader)* | ✅ | — | — | ✅ | ✅ |
| DS-K1A8503F *(no card reader)* | ✅ | — | — | ✅ | ✅ |
| DS-K1T605MF | ✅ | ✅ | ✅ | ✅ | ✅ |
| DS-K1T605EF | ✅ | ✅ | ✅ | ✅ | ✅ |
| DS-K1T605MF-B | ✅ | ✅ | ✅ | ✅ | ✅ |
| DS-K1T671MFW | ✅ | ✅ | ✅ | ✅ | ✅ |
| DS-K1T671MF | ✅ | ✅ | ✅ | ✅ | ✅ |
| DS-K1T670MFWX | ✅ | ✅ | ✅ | ✅ | ✅ |
| DS-K1T320MFWX-B | ✅ | ✅ | ✅ | ✅ | ✅ |
| DS-K1T321MFWX | ✅ | ✅ | ✅ | ✅ | ✅ |
| DS-K1T341CMFW | ✅ | ✅ | ✅ | ✅ | ✅ |
| DS-K1T342MFWX | ✅ | ✅ | ✅ | ✅ | ✅ |
| DS-K1T343MFWX | ✅ | ✅ | ✅ | ✅ | ✅ |
| DS-K1T343EFWX | ✅ | ✅ | ✅ | ✅ | ✅ |
| DS-K1T344MFX-E1 | ✅ | ✅ | ✅ | ✅ | ✅ |
| DS-K1T642MF | ✅ | ✅ | ✅ | ✅ | ✅ |
| DS-K1T673TM-E1 | — | ✅ | ✅ | ✅ | ✅ |
| DS-K1T341AMF | — | ✅ | ✅ | ✅ | ✅ |
| DS-K1T342EWX | — | ✅ | ✅ | ✅ | ✅ |
| DS-K1T672E | — | ✅ | ✅ | ✅ | ✅ |
| DS-K1T642MW | — | ✅ | ✅ | ✅ | ✅ |
| DS-K1T331 *(no built-in card reader)* | — | ✅ | — | ✅ | ✅ |

Capabilities are cross-checked against official Hikvision datasheets and
user manuals.

## Known gaps

A few Hikvision terminals ship the fingerprint sensor as an **optional
hardware module**, so the same model number covers both fingerprint and
non-fingerprint units (e.g. `DS-K1T671MF-L`, `DS-K1T673DWX-PROE1`). Pulsar
can't tell the two apart from the model string alone, so these aren't
profiled yet — if you have one of these, tell us which hardware variant
you have (with/without the fingerprint module) and we'll add it.

## Requesting a new model

Open an issue on this repo with the exact model number printed on the
device label (and a link to its datasheet/manual if you have one) and
we'll add a capability profile for it.
