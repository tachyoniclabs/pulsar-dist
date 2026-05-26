# Changelog

All notable changes to Pulsar Agent are documented here. Format based on
[Keep a Changelog](https://keepachangelog.com/), versioning follows
[SemVer](https://semver.org/).

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
