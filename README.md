# Pulsar — Distribution

Signed Windows builds of **Pulsar Agent**, the on-premise edge agent that bridges
access-control terminals (fingerprint, card, face) to a cloud ERP.

> Source code lives in a separate repository. This repo only ships binaries and
> the installer.

## What's new in v0.2.0

Pulsar now ships a **guided installer** — no more NSSM, no hand-edited YAML.
Double-click `PulsarSetup.exe`, then a browser-based setup wizard walks you
through connecting to your ERP and adding your terminals. A system-tray app
shows status at a glance.

## Download

Grab the latest release from the [Releases page](https://github.com/tachyoniclabs/pulsar-dist/releases/latest).

Each release contains:

| File | Purpose |
|---|---|
| `PulsarSetup-vX.Y.Z-<sha>.exe` | The installer (everything is inside it) |
| `SHA256SUMS` | SHA-256 checksums for verification |

## Verify the download (Windows PowerShell)

```powershell
# Replace with the file you downloaded
$exe = "PulsarSetup-v0.2.0-3df3779.exe"

$expected = (Get-Content SHA256SUMS | Select-String $exe).ToString().Split(" ")[0]
$actual   = (Get-FileHash -Algorithm SHA256 $exe).Hash.ToLower()

if ($expected -eq $actual) { "OK" } else { "MISMATCH — do not install" }
```

The installer is also code-signed. If Windows SmartScreen still warns on first
run, that's expected for a freshly-signed publisher — click **More info → Run
anyway**.

## Install

1. **Download and verify** `PulsarSetup.exe` (see above).
2. **Double-click it.** Approve the User Account Control (admin) prompt. The
   installer copies the files, registers the `PulsarAgent` Windows service
   (auto-start), adds the tray app to your login, and opens the **setup wizard**
   in your browser.
3. **Complete the wizard** (`http://127.0.0.1:9090`):
   1. **Connect to your ERP** — generate an **API account** in your ERP's
      integration settings, then enter its **Client ID** and **Client Secret**,
      your **ERP URL**, and your **Branch ID** into the wizard. Click **Test &
      continue** — the wizard verifies the credentials against your ERP before
      moving on. The secret is sealed on disk and never shown again.
   2. **Find your terminals** — the wizard scans your local network for Hikvision
      devices. Tick the ones you want, enter the device admin password (one field
      applies to all, with per-device override), and hit **Test** to confirm each
      one (it auto-detects the model). No terminals found? Add one by IP.
   3. **Review & finish** — the agent saves its config, starts polling, and the
      dashboard shows your devices with live status.

That's it. No NSSM, no config file to edit, no terminal commands.

## Day-to-day (system tray)

After install, the Pulsar icon sits in your system tray (a coloured dot shows
health: green = running, amber = cloud unreachable, red = stopped):

| Menu item | What it does |
|---|---|
| **Open Dashboard** | Opens `http://127.0.0.1:9090` — status + device management |
| **Open status** | Same dashboard, focused on ERP/device connectivity |
| **View logs** | Opens the agent log file |
| **Restart agent** | Restarts the service (asks for admin) |
| **Quit** | Closes the tray only — the agent keeps running |

**Add, edit, or remove terminals later** from the dashboard's *Devices* section —
changes apply immediately without a reinstall or restart. You can also
**re-pair** with your ERP there if credentials are rotated.

## Uninstall

Uninstall from **Settings → Apps** (or Control Panel → Programs). The uninstaller
stops and removes the `PulsarAgent` service and the tray autostart, then asks
whether to also delete your configuration and data in `C:\ProgramData\Pulsar`:

- **No** (default) — keeps your ERP pairing and device list, so a reinstall picks
  up where you left off.
- **Yes** — removes everything.

## Where things live

| Path | Contents |
|---|---|
| `C:\Program Files\Pulsar\` | The binaries (read-only at runtime) |
| `C:\ProgramData\Pulsar\config.yaml` | Configuration (contains **no secrets**) |
| `C:\ProgramData\Pulsar\secret.dat` | ERP secret + device passwords, sealed at rest (Windows DPAPI) |
| `C:\ProgramData\Pulsar\pulsar-agent.log` | Service log |
| `C:\ProgramData\Pulsar\data\` | Per-device event buffers |

The local dashboard/API listens on `127.0.0.1:9090` only and rejects
cross-origin browser requests. **Don't repoint the port** — the tray and the
installed shortcuts assume `9090`.

## Operations

| Task | How |
|---|---|
| Start / stop / restart the service | `services.msc` → *Pulsar Access-Control Agent*, or `sc start/stop PulsarAgent` (admin), or the tray's **Restart agent** |
| Check service state | `sc query PulsarAgent` |
| View logs | Tray → **View logs**, or open `C:\ProgramData\Pulsar\pulsar-agent.log` |
| Add / change / remove terminals | Dashboard → **Devices** |
| Re-pair with the ERP | Dashboard → **Re-pair** |

## Versioning

Releases follow [Semantic Versioning](https://semver.org/).

- **Major** (`v1.0.0 → v2.0.0`): breaking changes to the config schema or the
  ERP HTTP contract.
- **Minor** (`v0.1.0 → v0.2.0`): new features — backwards-compatible.
- **Patch** (`v0.1.0 → v0.1.1`): bug fixes and security patches.

Each release filename embeds both the version and the short source SHA, e.g.,
`PulsarSetup-v0.2.0-3df3779.exe` — `3df3779` is the source commit it was built from.

## Supported platforms

- **Windows x86-64**: Windows 10/11, Windows Server 2016+. Officially supported.
- **Linux / macOS**: the agent builds and runs (service management and the tray
  are Windows-only); not shipped here.

## License

Distributed by Tachyonic Labs. All rights reserved unless a `LICENSE` file is
shipped with a release.
