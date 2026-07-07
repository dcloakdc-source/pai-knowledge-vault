# PAI eGPU Dashboard Persistence

This module provides automated startup and monitoring for the PAI circular LCD dashboard on the SGWZONE eGPU.

## Features

- **Windows Auto-Start**: Automatically launches the API bridge and dashboard UI at logon.
- **Android Boot Persistence**: Hardware-level script to prepare the eGPU display on boot.
- **Watchdog Monitoring**: Automatically detects and recovers from crashes or disconnections.
- **Manual Control**: Single scripts for starting/stopping without automation.

---

## Setup Instructions

### 1. Windows Host (Automation)

To install the automatic startup tasks:

```powershell
cd \\nas6db92a\pai\scripts
.\install-egpu-autostart.ps1
```

This creates three Windows Scheduled Tasks:
- `PAI-API-Bridge`: Starts the metrics server.
- `PAI-eGPU-Dashboard`: Starts the fullscreen display (30s delay).
- `PAI-eGPU-Watchdog`: Health check every 5 minutes.

### 2. eGPU Device (Boot Persistence)

To ensure the display prepares itself on eGPU power-on:

```powershell
cd \\nas6db92a\pai\scripts
.\deploy-boot-script.ps1
```

*Note: This modifies `/system/init.rc` on the eGPU and requires root access.*

---

## Usage

### Start Manually
If automation is not installed, use:
```powershell
# Start API (leave open)
powershell -File .\scripts\start-api-bridge.ps1

# Start Dashboard (leave open)
powershell -File .\scripts\start-pai-fullscreen.ps1
```

### Health Check
Run the watchdog manually to verify systems:
```powershell
powershell -File .\scripts\egpu-dashboard-watchdog.ps1
```

### Uninstallation
```powershell
# Remove Windows tasks
.\scripts\install-egpu-autostart.ps1 -Uninstall

# Remove Android boot script
.\scripts\deploy-boot-script.ps1 -Uninstall
```

---

## Documentation

Comprehensive guides are available in `\\nas6db92a\pai\Packs\pai-wazuh-integration\`:

- **[Manual Startup Guide](Packs/pai-wazuh-integration/EGPU_MANUAL_START.md)**: Troubleshooting and step-by-step manual start.
- **[Technical Walkthrough](Packs/pai-wazuh-integration/walkthrough_egpu_autostart.md)**: Detailed evidence of implementation and architecture.

---

## Logs

Log files are stored in `\\nas6db92a\pai\logs\`:
- `api_bridge.log`: API server output.
- `dashboard_service.log`: Dashboard renderer output.
- `watchdog.log`: Health check and recovery events.
