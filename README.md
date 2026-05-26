# Pulsar — Distribution

Signed Windows builds of **Pulsar Agent**, the on-premise edge agent that bridges
access-control terminals (fingerprint, card, face) to a cloud ERP.

> Source code lives in a separate repository. This repo only ships binaries and
> install bundles.

## Download

Grab the latest release from the [Releases page](https://github.com/tachyoniclabs/pulsar-dist/releases/latest).

Each release contains:

| File | Purpose |
|---|---|
| `pulsar-agent-windows-vX.Y.Z-<sha>.zip` | Install bundle (binary + scripts + config template) |
| `SHA256SUMS` | SHA-256 checksums for verification |

## Verify the download (Windows PowerShell)

```powershell
# Replace VERSION with the version you downloaded
$zip = "pulsar-agent-windows-v0.1.0-d9523dd.zip"

# Expected hash from SHA256SUMS
$expected = (Get-Content SHA256SUMS | Select-String $zip).ToString().Split(" ")[0]

# Actual hash of the downloaded file
$actual = (Get-FileHash -Algorithm SHA256 $zip).Hash.ToLower()

if ($expected -eq $actual) { "OK" } else { "MISMATCH — do not install" }
```

## Install

1. **Download and verify** the zip (see above).
2. **Extract** to a working folder, e.g., `C:\pulsar-install\`.
3. **Download NSSM** from <https://nssm.cc/download>, extract `win64\nssm.exe`,
   and drop it into the same folder as `install.bat`.
4. **Edit `config.yaml`** — set device IP, credentials, ERP URL/client, and
   `provisioning.branch_id` (UUID from your ERP).
5. **Run as Administrator** from an elevated `cmd.exe`:
   ```cmd
   cd C:\pulsar-install
   install.bat
   ```
   This copies files to `C:\pulsar-agent\` and registers Windows service `PulsarAgent`.
6. **Verify connectivity** before starting the service:
   ```cmd
   C:\pulsar-agent\pulsar-agent.exe --check --config C:\pulsar-agent\config.yaml
   ```
   Expect `All checks passed.` Exit code 0 means safe to start.
7. **Start the service**:
   ```cmd
   nssm start PulsarAgent
   ```
8. **Open the dashboard**: <http://localhost:9090>

## Operations

| Command | Purpose |
|---|---|
| `nssm start PulsarAgent` | Start service |
| `nssm stop PulsarAgent` | Stop service |
| `nssm restart PulsarAgent` | Restart after config change |
| `nssm status PulsarAgent` | Check service state |
| `type C:\pulsar-agent\pulsar-agent.log` | View logs |
| `uninstall.bat` (as Administrator) | Remove the service |

Logs auto-rotate at 10 MB.

## Configuration

The shipped `config.yaml` is a template with placeholder values:

```yaml
devices:
  - device_id: workshop-front-door
    vendor: hikvision
    model: DS-K1T804AMF
    poll_interval: 30s
    http_timeout: 10s
    hikvision:
      url: http://192.168.x.x          # device IP
      username: admin
      password: REPLACE_ME              # admin password set during activation
      max_events_per_poll: 30
      default_door_right: "1"

erp:
  url: https://your-erp.example.com
  client_id: REPLACE_WITH_CLIENT_ID
  client_secret: REPLACE_WITH_CLIENT_SECRET

provisioning:
  enabled: true
  branch_id: 00000000-0000-0000-0000-000000000000  # branch UUID from ERP
  poll_interval: 60s
```

Multiple devices per agent are supported — add more entries under `devices:`.

## Versioning

Releases follow [Semantic Versioning](https://semver.org/).

- **Major** (`v1.0.0 → v2.0.0`): breaking changes to the config schema or the
  ERP HTTP contract.
- **Minor** (`v0.1.0 → v0.2.0`): new features (additional vendors, new ERP
  endpoints, new adapter capabilities) — backwards-compatible.
- **Patch** (`v0.1.0 → v0.1.1`): bug fixes and security patches.

Each release filename embeds both the version and the short source SHA, e.g.,
`pulsar-agent-windows-v0.1.0-d9523dd.zip` — `d9523dd` is the source commit it
was built from.

## Supported platforms

- **Windows x86-64**: Windows 10/11, Windows Server 2016+. Officially supported.
- **Linux x86-64**: testing only; not yet shipped here.

## License

Distributed by Tachyonic Labs. All rights reserved unless a `LICENSE` file is
shipped with a release.
