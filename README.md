# AegisBanner

A lightweight, persistent security classification banner for Windows workstations in high-security and air-gapped environments. Displays hostname, classification label, and site identifier across all monitors. Fully managed via Active Directory Group Policy.

## Overview

AegisBanner renders a persistent strip at the top of every connected display showing three configurable zones:

```
┌─────────────────────────────────────────────────────────────────┐
│  WS-ALPHA-01           SECRET//NOFORN                SITE-BRAVO │
└─────────────────────────────────────────────────────────────────┘
```

| Zone | Content | Configurable |
|------|---------|--------------|
| Left | Workstation hostname (`%COMPUTERNAME%`) | Show/hide via GPO |
| Center | Primary classification label | Yes — via GPO |
| Right | Secondary label (site, building, system ID) | Yes — via GPO |

The banner cannot be closed, minimized, moved, or right-clicked by standard users. It reserves screen real estate via the Windows AppBar API so maximized application windows tile cleanly below it.

---

## Features

- **Multi-monitor** — spawns one AppBar window per display; handles monitors being added or removed at runtime
- **Zero dependencies** — links only against OS-provided DLLs (`user32`, `gdi32`, `shell32`, `advapi32`); no .NET runtime, no VC++ redistributable, no third-party libraries
- **Air-gap ready** — single self-contained `.exe`, no network calls, no external assets
- **GPO managed** — all visual settings driven from `HKLM\Software\Policies\AegisBanner`; ships with `.admx` / `.adml` templates for the Central Store
- **Live refresh** — registry watcher thread picks up `gpupdate` changes instantly without restarting
- **Anti-tamper** — `Alt+F4`, right-click, close/minimize/maximize buttons, and `Alt+Space` are all intercepted and suppressed for non-admin users
- **DPI aware** — Per-Monitor DPI v2; banner height scales correctly on mixed-DPI multi-monitor setups
- **Event logging** — start, stop, config reload, and error events written to the Windows Application Event Log

---

## Requirements

### Runtime
| Requirement | Detail |
|-------------|--------|
| OS | Windows 10 or Windows 11 (64-bit) |
| Dependencies | None beyond standard Windows DLLs |
| Privileges | Standard user (runs at logon via Run key) |

### Build
| Tool | Version |
|------|---------|
| Visual Studio / Build Tools | 2019 or 2022 |
| MSVC compiler (`cl.exe`) | 14.x or later |
| Windows SDK | 10.0.19041 or later |

---

## Deployment

### 1 — Place the binary

Copy `AegisBanner.exe` to each workstation. Recommended path:

```
%ProgramFiles%\AegisBanner\AegisBanner.exe
```

Push via GPO Files Preference, SCCM, or removable media.

### 2 — Register auto-start

Add a GPO Registry Preference (or import manually):

| Setting | Value |
|---------|-------|
| Hive | `HKEY_LOCAL_MACHINE` |
| Key | `SOFTWARE\Microsoft\Windows\CurrentVersion\Run` |
| Name | `AegisBanner` |
| Type | `REG_SZ` |
| Data | `%ProgramFiles%\AegisBanner\AegisBanner.exe` |

### 3 — Install GPO templates

Copy to your Central Store (or local `PolicyDefinitions`):

```
\\<DOMAIN>\SYSVOL\<DOMAIN>\Policies\PolicyDefinitions\AegisBanner.admx
\\<DOMAIN>\SYSVOL\<DOMAIN>\Policies\PolicyDefinitions\en-US\AegisBanner.adml
```

### 4 — Configure via GPMC

Navigate to:
```
Computer Configuration → Administrative Templates → Windows Components → AegisBanner
```

---

## Configuration

All settings live under `HKEY_LOCAL_MACHINE\Software\Policies\AegisBanner`.

| Registry Value | Type | Description | Default |
|----------------|------|-------------|---------|
| `ShowHostname` | `DWORD` | `1` = show hostname in left zone, `0` = hide | `1` |
| `ClassificationText` | `REG_SZ` | Centre zone label | `UNCLASSIFIED` |
| `SecondaryLabel` | `REG_SZ` | Right zone label. Empty string hides the zone | *(empty)* |
| `BgColor` | `REG_SZ` | Background colour as `#RRGGBB` | `#007A33` |
| `FgColor` | `REG_SZ` | Text colour as `#RRGGBB` | `#FFFFFF` |
| `FontSize` | `DWORD` | Font size in points (6–72). Banner height auto-adjusts | `11` |

Changes are detected by the registry watcher thread and applied within seconds — no restart required.

### Quick-start without GPO

Edit and import `GPO/AegisBanner_Config.reg`:

```bat
reg import AegisBanner_Config.reg
```

### Common classification colour schemes

| Classification | `BgColor` | `FgColor` |
|----------------|-----------|-----------|
| Unclassified | `#007A33` | `#FFFFFF` |
| CUI | `#502B85` | `#FFFFFF` |
| Confidential | `#0033A0` | `#FFFFFF` |
| Secret | `#C8102E` | `#FFFFFF` |
| Top Secret | `#FF8C00` | `#000000` |
| TS/SCI | `#FFFF00` | `#000000` |

> **Note:** Always confirm colour choices against your organisation's applicable marking guides and ISS guidance before deploying to production.

---

## File Reference

```
AegisBanner/
├── AegisBanner.adml # English language strings for ADMX
├── AegisBanner.admx     # Group Policy Administrative Template

---

## Event Log

AegisBanner writes to **Windows Logs → Application** under source `AegisBanner`.

| Type | Event |
|------|-------|
| Information | Application started successfully |
| Information | Configuration reloaded from registry |
| Information | Application stopped |
| Information | Rebuilt after display configuration change |
| Warning | Registry key not found — running with defaults |
| Warning | AppBar registration failed on a monitor |
| Error | Window creation failed |

---

## Uninstalling

```bat
taskkill /IM AegisBanner.exe /F
reg delete "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" /v AegisBanner /f
reg delete "HKLM\Software\Policies\AegisBanner" /f
rmdir /s /q "%ProgramFiles%\AegisBanner"
reg delete "HKLM\SYSTEM\CurrentControlSet\Services\EventLog\Application\AegisBanner" /f
```

If deployed via GPO, disable or delete the GPO first to prevent reinstallation on the next policy refresh.

---

## License

MIT License — see [LICENSE](LICENSE) for details.
