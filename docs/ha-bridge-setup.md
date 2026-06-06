# HA Bridge Integration Notes

## Overview

This document describes how HA Bridge integrates with this system when used to trigger automation scripts that rely on SSH.

It is not part of the core `habrunasuser` logic, but explains required system configuration for stable operation.

---

## Critical Requirement: Startup Order

HA Bridge runs as a system service and may start before the user desktop session is fully initialised.

This can cause SSH-based automation to fail because:

- SSH agent is not available at boot time
- No authenticated SSH session exists yet
- SSH falls back to direct key authentication
- Passphrase prompts occur during automation

---

## Required System Behaviour

HA Bridge must only start after:

1. LXDE desktop session is running  
2. User SSH agent is available and unlocked  
3. Autostart scripts have completed  

---

## Systemd Service Configuration

HA Bridge is installed as a systemd service:

```bash
/etc/systemd/system/ha-bridge.service
```

### Current Service File

```ini
[Unit]
Description=HA Bridge
Wants=network.target
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/home/pi/ha-bridge

ExecStart=/home/pi/sbin/start-habridge.sh

Environment=PATH=/home/pi/sbin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
Environment=DISPLAY=:0
Environment=XAUTHORITY=/home/pi/.Xauthority

SupplementaryGroups=dialout

[Install]
WantedBy=multi-user.target
```

### Enable Service

```bash
sudo systemctl enable ha-bridge
```

---

## Systemd Override Configuration (Testing History)

During testing, a systemd override was used via:

```bash
sudo systemctl edit ha-bridge
```

### Example override used during debugging

```ini
[Service]
ExecStart=
ExecStart=/bin/true
```

This was used temporarily to prevent HA Bridge starting too early during boot while debugging SSH timing issues.

### Important Note

This override is NOT required in the final working configuration.

It was only used during troubleshooting of SSH authentication timing issues.

---

## Final Working Startup Method

Instead of relying on systemd boot timing, HA Bridge is started after the desktop session is ready.

Example:

```bash
sleep 10
sudo systemctl start ha-bridge
```

This ensures:

- LXDE session is fully initialised
- SSH agent is running and unlocked
- User environment variables are available

---

## Why This Is Necessary

Although HA Bridge is a system service, its execution timing is critical when used with SSH-based automation.

Starting it too early results in:

- Missing SSH agent
- Passphrase prompts
- Broken automation flow

---

## Summary

- HA Bridge must not start before user session is ready
- SSH agent must already be active
- systemd alone is not sufficient for correct timing
- startup delay ensures stable SSH automation behaviour

