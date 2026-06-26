# Changelog

All notable changes to Pulsar Agent are documented here. Format based on
[Keep a Changelog](https://keepachangelog.com/), versioning follows
[SemVer](https://semver.org/).

## [v0.2.0] — 2026-06-26

Self-service Windows release. Replaces the NSSM + `install.bat` + hand-edited
`config.yaml` flow with a guided installer, a browser setup wizard, and a
system-tray app. A non-technical user can stand up a branch in ~10 minutes
without a terminal.

### Added

- **`PulsarSetup.exe` installer (Inno Setup).** Double-click + UAC → installs to
  `C:\Program Files\Pulsar`, registers the `PulsarAgent` service as a **native
  Windows service** (no more NSSM), adds the tray app to login autostart, and
  opens the setup wizard. Clean uninstall from Add/Remove Programs, with an
  opt-in prompt to keep or purge `C:\ProgramData\Pulsar` (default: keep, so a
  reinstall preserves pairing + devices).
- **Browser setup wizard** at `127.0.0.1:9090`: connect to the ERP with its
  existing **API-account credentials** (ERP URL + Client ID + Client Secret +
  Branch ID), validated inline by a live token **Test**; then scan the local
  network for Hikvision terminals (or add by IP) with a per-device **Test** that
  auto-detects the model, and review & finish. The same app is the ongoing
  dashboard — add/edit/remove devices and re-pair, applied live via config
  hot-reload (no restart).
- **System-tray app** (`pulsar-tray.exe`): green/amber/red status icon plus Open
  Dashboard, View logs, Restart agent (elevated), and Quit.

### Changed

- **All secrets are sealed at rest.** The ERP client secret *and* every Hikvision
  device password are encrypted with Windows DPAPI (machine scope) in
  `secret.dat`; `config.yaml` now contains **no secrets**.
- Runtime data moved to `C:\ProgramData\Pulsar` (service-writable); binaries live
  in `C:\Program Files\Pulsar`.

### Security

- The local web API is guarded against CSRF and DNS-rebinding (loopback `Host`
  allowlist, `Origin` check, and a JSON content-type requirement on all
  state-changing requests).

### Notes

- The wizard authenticates to the ERP with the **existing API-account**
  mechanism (OAuth2 client-credentials → JWT at `/api/v1/auth/api-account/token`)
  — the same one in use today. No new ERP-side endpoints are required.

## [v0.1.4] — 2026-05-27

Hardening release from the ACME workshop install — five Hikvision V1.4.x
firmware quirks discovered + fixed in the field, plus the cloud-side
`device_user_id` flow (Plan B) and a per-device clock-drift detector on
the health dashboard.

### Added

- **`attendance_only` mode for door-less terminals** *(default: true)*.
  When the terminal sits on a wall as an attendance scanner with no door
  behind it, the agent reclassifies "user identified but device denied"
  events as `granted`. Side-steps the entire access-control schedule /
  holiday-group / door-rights plumbing that V1.4.x firmware requires.
  Real fingerprint mismatches (no user identified) still surface as
  `denied` so error signal isn't lost. Set to `false` explicitly when
  the device fronts a real door and you've completed the iVMS-4200
  Door Configuration wizard.
- **Clock-drift detector on the health dashboard** (`localhost:9090`).
  Every 5 minutes Pulsar polls the device's `/ISAPI/System/time` and
  compares against the agent host clock + timezone. A green/amber/red
  pill flags drift (>30s warn, >2m error) and timezone mismatches —
  both silently corrupt event polling because Hikvision queries by
  local-time strings. Also exposed in `/health` JSON for monitoring.
- **Numeric employeeNo handling** (`internal/idmap`). Hikvision
  rejects non-numeric `employeeNo` with `illegalEmployeeNo`
  (0x60006036, max 8 chars on V1.4.x). The agent now strips non-digits
  before pushing to the device and reverse-maps incoming swipes so the
  ERP still sees its original ID. Reverse map persists in bbolt across
  restarts.
- **Adaptive back-off on consecutive device failures**
  (30s → 1m → 5m → 15m). Prevents the polling loop from re-triggering
  account lockouts when a device is offline / misconfigured.
- **Plan B provisioning payload** — `device_user_id` and `card_no`
  carried in the user-change JSON. When the ERP sets `device_user_id`
  on an employee, the agent pushes that directly (no client-side
  derivation). When `card_no` is set, the agent additionally enrols
  the card on devices that support it. Backwards-compatible: payload
  without these fields falls back to the v0.1.3 path.
- **Hikvision install playbook** in the source README — activate,
  NTP, attendance-only vs access-control decision, fingerprint
  enrolment, smoke test. Saves engineers the day-long debug we did
  on 2026-05-27.

### Fixed

- **`Valid.enable = false` denied every swipe** on V1.4.x. Counter to
  the field name, `enable=false` disables the user account — even when
  the fingerprint matches the device reports `denied` (major=5
  minor=38). The agent now always sends `enable=true` with a
  wide-open `beginTime`/`endTime` (1970-01-01 → 2037-12-31) when the
  ERP doesn't provide explicit validity.
- **`userVerifyMode = ""` denied every swipe.** Empty string is
  treated as "no method allowed" on V1.4.x, not "use device default".
  Agent now defaults to `cardOrFpOrPw` — broadest accepted value per
  device capabilities, works on fingerprint-only, card-only, and
  multi-method terminals. Note: `cardOrFp` is rejected by V1.4.x
  (`badParameters`); the valid combinations are in
  `GET /ISAPI/AccessControl/UserInfo/capabilities`.
- **Phantom "granted" rows with empty employee_no.** `mapResult` was
  treating every Major=5 swipe minor not in the explicit denied-list
  as `granted`. But Major=5 covers door-open/close sensor events
  (minor=21/22) too — they have no employee attribution and were
  flooding the ERP as quarantined "anonymous user" rows. Now uses an
  explicit `grantedMinors = {1}` allowlist; everything else under
  Major=5 that isn't in the denied set classifies as `unknown` and
  is correctly ignored for attendance.

### Compatibility

- Standalone: drop-in upgrade, no config changes required. Existing
  `config.yaml` still works; `attendance_only` defaults to `true`.
- Full Plan B benefit (ERP-driven `device_user_id` in the queue,
  card-only swipe attribution) requires the matching ERP-side
  migrations (`V792` + `V793`) and service changes. Without them,
  the agent's idmap fallback covers the gap.

### Notes

- Bundled macOS binary is **not Apple-signed or notarized** —
  Gatekeeper will warn on first run; right-click → Open once to
  approve.
- NSSM is **not bundled** with the Windows zip. Download separately
  from <https://nssm.cc/download> and drop `win64\nssm.exe` next to
  `install.bat` before running (see README install step 3).

## [v0.1.3] — 2026-05-27

### Added
- **macOS install bundle** — universal binary (Intel + Apple Silicon, fused with
  `lipo`), LaunchDaemon plist, `install.sh` / `uninstall.sh` scripts. Mirrors
  the Windows service install flow. Files land under `/usr/local/bin`,
  `/usr/local/etc/pulsar-agent`, and `/Library/LaunchDaemons`. See the bundled
  `README.txt` for the full sequence.
- Cross-compile output for `darwin/amd64` and `darwin/arm64` in `build.sh`.

### Fixed
- **Intermittent `hikvision returned 401` on the poll loop** even when
  credentials were correct. The HTTP client kept TCP keep-alives enabled;
  Hikvision DS-K1T804AMF (V1.4.1, build 240318) closes idle keep-alive
  sockets aggressively, and on the next reuse the digest nonce state on the
  device no longer matches the client's cache, so it returns 401. Re-testing
  with `curl --digest` at the same 30s cadence never reproduced the failure,
  isolating the bug to Go's connection reuse rather than the device. The
  Hikvision client now sets `DisableKeepAlives: true` on its transport, so
  every request opens a fresh TCP connection and re-negotiates digest from
  scratch. Adds ~50ms per call; negligible at poll cadence.

### Notes
- Bundled macOS binary is **not Apple-signed or notarized**. On first run
  Gatekeeper may show "cannot verify developer" — right-click → Open once,
  or run from Terminal. For a public release a Developer ID Application
  certificate and notarization would be needed.

### Source
Built from `tachyoniclabs/pulsar@4b6bac7`.

[v0.1.3]: https://github.com/tachyoniclabs/pulsar-dist/releases/tag/v0.1.3

## [v0.1.2] — 2026-05-27

### Fixed
- **Events flush to ERP rejected with HTTP 400 `Unrecognized field "device_id"`**.
  The agent was POSTing `{ "device_id": "...", "events": [...] }` to
  `/api/v1/integrations/access-control/events`. The ERP's
  `IngestEventBatchRequest` accepts only `events` (each event already carries
  its own `device_id` inside). Jackson's strict-unknown-fields rejected the
  batch. `EventBatch` now serializes as `{ "events": [...] }` only. The dead
  `deviceID` field on the ERP `Client` and its constructor parameter were
  removed at the same time — they were used solely for this redundant
  marshaling.

### Source
Built from `tachyoniclabs/pulsar@4825d76`.

[v0.1.2]: https://github.com/tachyoniclabs/pulsar-dist/releases/tag/v0.1.2

## [v0.1.1] — 2026-05-27

### Fixed
- **Hikvision `AcsEvent` polling rejected with HTTP 400 `badParameters`** on
  DS-K1T804AMF (V1.4.1, build 240318). The firmware silently caps
  `AcsEventCond.searchID` at ~20 characters; Pulsar was generating
  `erp-agent-<19-digit nanoseconds>` = 29 chars and every poll failed.
  `searchID` now uses Unix seconds (20 chars). Collision risk is zero in
  practice — polls are `poll_interval` apart (default 30s).
  Diagnosed empirically against a live unit because the device returns no
  hint about the length limit. Code comment documents the constraint.

### Notes
- **Operators should set the device timezone to match the agent host.** The
  same firmware also rejects `startTime`/`endTime` whose offset differs from
  the device's configured `timeZone`. Pulsar formats timestamps using the
  Go host's local TZ via `time.RFC3339`. If the device is on China time
  (factory default `+08:00`) and the agent host is on Sri Lanka time
  (`+05:30`), the device returns `400 badParameters` until the device is
  reconfigured. Set via the device touchscreen (System → Time) or ISAPI:
  ```
  PUT /ISAPI/System/time
  <Time><timeMode>manual</timeMode><localTime>YYYY-MM-DDThh:mm:ss</localTime><timeZone>CST-5:30:00</timeZone></Time>
  ```

### Source
Built from `tachyoniclabs/pulsar@5f113f1`.

[v0.1.1]: https://github.com/tachyoniclabs/pulsar-dist/releases/tag/v0.1.1

## [v0.1.0] — 2026-05-26

Initial public release.

### Added
- **Hikvision adapter** via ISAPI (HTTP, Digest auth). Implements `Probe`,
  `FetchEvents`, `UpsertUser`, `DeleteUser`, `AddCard`.
- **Per-device poller** — each configured device gets its own goroutine and its
  own BoltDB buffer file under `agent.buffer_dir`.
- **Agent-level provisioner** with **branch-scoped ERP queue** — one HTTP poll
  per agent drains the branch's user-change queue and fans each change out to
  every locally configured device. Changes are acked back to the ERP only if
  *all* devices applied successfully.
- **Local health dashboard** at `http://127.0.0.1:9090` — auto-refreshing HTML
  plus JSON at `/health` for monitoring tools.
- **Connectivity check mode** — `pulsar-agent.exe --check --config <path>`
  validates device reachability, device auth, and ERP token issuance before
  the service is started.
- **Windows service installer** via NSSM — installs to `C:\pulsar-agent\`,
  registers `PulsarAgent` as an auto-start service, log rotation at 10 MB.
- **Per-model capability profiles** — Hikvision adapter keys feature support
  (fingerprint, face, card, PIN, attendance status) off the device `model`
  field; unknown models fall back to a conservative profile.
- **Vendor-neutral event shape** pushed to ERP regardless of source vendor.
  Optional `agent.include_raw_events` appends the original vendor-native event
  for forensics.

### Notes
- User sync is **cloud → device only**. Local enrollments (e.g., a fingerprint
  enrolled directly on the terminal) do not propagate back to the ERP.
- Single supported vendor in this release: **Hikvision**. The
  `DeviceAdapter` interface is designed for additional vendors (Suprema,
  ZKTeco, Idemia) to be added without changing the poller, provisioner,
  buffer, or ERP client.

### Source
Built from `tachyoniclabs/pulsar@d9523dd`.

[v0.1.0]: https://github.com/tachyoniclabs/pulsar-dist/releases/tag/v0.1.0
