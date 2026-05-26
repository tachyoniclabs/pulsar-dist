# Changelog

All notable changes to Pulsar Agent are documented here. Format based on
[Keep a Changelog](https://keepachangelog.com/), versioning follows
[SemVer](https://semver.org/).

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
