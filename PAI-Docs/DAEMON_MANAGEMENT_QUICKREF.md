# PAI Daemon Management Quick Reference

**Cheat sheet for systemd user services**

---

## Common Commands

### Service Status

```bash
# All PAI services
systemctl --user list-units 'pai-*'

# Active services only
systemctl --user list-units 'pai-*' --state=active

# Failed services
systemctl --user list-units 'pai-*' --state=failed
systemctl --user --failed  # All failed units

# Specific service status
systemctl --user status pai-pulse.service
```

### Start/Stop/Restart

```bash
# Start
systemctl --user start pai-pulse.service

# Stop
systemctl --user stop pai-pulse.service

# Restart
systemctl --user restart pai-pulse.service

# Reload config (without restart)
systemctl --user reload pai-pulse.service
```

### Enable/Disable (Auto-start)

```bash
# Enable (auto-start on login)
systemctl --user enable pai-pulse.service

# Enable and start now
systemctl --user enable --now pai-pulse.service

# Disable (no auto-start)
systemctl --user disable pai-pulse.service

# Disable and stop now
systemctl --user disable --now pai-pulse.service
```

### Logs

```bash
# Last 100 lines
journalctl --user -u pai-pulse.service -n 100

# Follow live
journalctl --user -u pai-pulse.service -f

# Since boot
journalctl --user -u pai-pulse.service -b

# Time range
journalctl --user -u pai-pulse.service \
  --since "2026-06-24 00:00" \
  --until "2026-06-24 12:00"

# All PAI services logs
journalctl --user -u 'pai-*' -f
```

### Reload Systemd

```bash
# After editing .service files
systemctl --user daemon-reload
```

---

## Timers

### List Timers

```bash
# All timers
systemctl --user list-timers

# PAI timers only
systemctl --user list-timers 'pai-*'

# Show next run times
systemctl --user list-timers --all
```

### Timer Control

```bash
# Enable timer
systemctl --user enable --now pai-deadman.timer

# Disable timer
systemctl --user disable --now pai-deadman.timer

# Trigger timer manually (run now)
systemctl --user start pai-deadman.service
```

---

## Resource Monitoring

### CPU/Memory Usage

```bash
# Interactive top-like view
systemd-cgtop --user

# Specific service resource usage
systemctl --user status pai-pulse.service | grep -E "(Memory|CPU)"
```

### Journal Disk Usage

```bash
# Show journal size
journalctl --user --disk-usage

# Vacuum old logs (keep last 7 days)
journalctl --user --vacuum-time=7d

# Vacuum by size (keep last 500M)
journalctl --user --vacuum-size=500M
```

---

## Troubleshooting

### Service Won't Start

```bash
# 1. Check status for error messages
systemctl --user status pai-pulse.service

# 2. Check journal for details
journalctl --user -u pai-pulse.service -n 50

# 3. Verify unit file syntax
systemd-analyze verify ~/.config/systemd/user/pai-pulse.service

# 4. Check if port is in use
ss -tlnp | grep 31337

# 5. Test command manually
cd /home/duane/.claude/PAI/PULSE
/home/duane/.bun/bin/bun run pulse.ts
```

### Service Crashed/Failed

```bash
# View failure details
systemctl --user status pai-pulse.service

# Check recent logs
journalctl --user -u pai-pulse.service -n 100

# Restart
systemctl --user restart pai-pulse.service

# Check if it stays up
sleep 5 && systemctl --user status pai-pulse.service
```

### Service Zombie/Stuck

```bash
# Find PID
systemctl --user show -p MainPID pai-pulse.service

# Kill manually
kill -9 <PID>

# Restart
systemctl --user restart pai-pulse.service
```

### Timer Not Running

```bash
# Check timer status
systemctl --user list-timers pai-deadman.timer

# Check timer unit itself
systemctl --user status pai-deadman.timer

# Check service unit
systemctl --user status pai-deadman.service

# Manually trigger
systemctl --user start pai-deadman.service

# Check logs
journalctl --user -u pai-deadman.service -n 50
```

---

## Creating New Services

### Template: Simple Daemon

```bash
cat > ~/.config/systemd/user/pai-my-service.service <<'EOF'
[Unit]
Description=PAI My Service — what it does
After=network.target
Wants=network.target

[Service]
Type=simple
WorkingDirectory=/home/duane/.claude
ExecStart=/home/duane/.bun/bin/bun PAI/Tools/MyService.ts
Restart=on-failure
RestartSec=30
Environment=PATH=/home/duane/.bun/bin:/usr/local/bin:/usr/bin:/bin
Environment=PAI_DIR=/home/duane/.claude
EnvironmentFile=/home/duane/.config/PAI/secrets.env
StandardOutput=journal
StandardError=journal

# Resource limits
MemoryMax=1G
CPUQuota=50%

[Install]
WantedBy=default.target
EOF

systemctl --user daemon-reload
systemctl --user enable --now pai-my-service.service
systemctl --user status pai-my-service.service
```

### Template: HTTP Service with Health Check

```bash
cat > ~/.config/systemd/user/pai-my-api.service <<'EOF'
[Unit]
Description=PAI My API — HTTP service on :8999
After=network.target

[Service]
Type=simple
WorkingDirectory=/home/duane/.claude/PAI/Tools/MyAPI
ExecStart=/home/duane/.bun/bin/bun server.ts
Restart=always
RestartSec=10
Environment=PATH=/home/duane/.bun/bin:/usr/local/bin:/usr/bin:/bin
Environment=PAI_DIR=/home/duane/.claude
Environment=PORT=8999
EnvironmentFile=/home/duane/.config/PAI/secrets.env
StandardOutput=journal
StandardError=journal

# Health check after start
ExecStartPost=/bin/sleep 2
ExecStartPost=/usr/bin/curl -f http://localhost:8999/health || exit 1

# Resource limits
MemoryMax=512M
CPUQuota=25%

[Install]
WantedBy=default.target
EOF
```

### Template: Scheduled Task (Timer)

```bash
# Service unit (oneshot)
cat > ~/.config/systemd/user/pai-my-task.service <<'EOF'
[Unit]
Description=PAI My Task — scheduled job

[Service]
Type=oneshot
WorkingDirectory=/home/duane/.claude
ExecStart=/home/duane/.bun/bin/bun PAI/Tools/MyTask.ts
Environment=PAI_DIR=/home/duane/.claude
EnvironmentFile=/home/duane/.config/PAI/secrets.env
StandardOutput=journal
StandardError=journal
EOF

# Timer unit
cat > ~/.config/systemd/user/pai-my-task.timer <<'EOF'
[Unit]
Description=Run PAI My Task daily at 4am

[Timer]
OnCalendar=*-*-* 04:00:00
Persistent=true
RandomizedDelaySec=30m

[Install]
WantedBy=timers.target
EOF

systemctl --user daemon-reload
systemctl --user enable --now pai-my-task.timer
systemctl --user list-timers pai-my-task.timer
```

### Template: Templated Service (Multiple Instances)

```bash
# Template unit (@.service)
cat > ~/.config/systemd/user/pai-worker@.service <<'EOF'
[Unit]
Description=PAI Worker %i
After=network.target

[Service]
Type=simple
WorkingDirectory=/home/duane/.claude
ExecStart=/home/duane/.bun/bin/bun PAI/Tools/Worker.ts --id %i
Restart=on-failure
Environment=WORKER_ID=%i
Environment=PAI_DIR=/home/duane/.claude
EnvironmentFile=/home/duane/.config/PAI/secrets.env
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=default.target
EOF

# Start instances
systemctl --user daemon-reload
systemctl --user enable --now pai-worker@1.service
systemctl --user enable --now pai-worker@2.service
systemctl --user enable --now pai-worker@3.service

# Status all instances
systemctl --user status 'pai-worker@*'
```

---

## Health Checks

### PAI-Specific Health Probe

```bash
# Run full HA status check
bash ~/.claude/PAI/Tools/HA/ha-status.sh --summary

# Write status to file
bash ~/.claude/PAI/Tools/HA/ha-status.sh --write

# Check specific service tier
grep -A 10 "Tier 1" ~/.claude/MEMORY/STATE.claude/ha/ha-status-latest.json
```

### Individual Service Checks

```bash
# HTTP health endpoints
curl http://localhost:31337/health  # OmniPulse
curl http://localhost:8890/health   # Functions API
curl http://localhost:8891/health   # Memory SSE (if exists)
curl http://localhost:8888/health   # Voice Server (if exists)

# Process checks
systemctl --user is-active pai-pulse.service
systemctl --user is-enabled pai-pulse.service
systemctl --user is-failed pai-pulse.service
```

### Deadman Check

The `pai-deadman.service` runs every 30 minutes and checks all scheduled jobs:

```bash
# Trigger deadman check manually
systemctl --user start pai-deadman.service

# Check deadman logs
journalctl --user -u pai-deadman.service -n 50

# View deadman report
cat ~/MEMORY/STATE.claude/deadman-report.jsonl | tail -1 | jq .
```

---

## Best Practices

### 1. Always use `--user` for PAI services

```bash
# ✅ Correct
systemctl --user status pai-pulse.service

# ❌ Wrong (requires root, won't find user units)
systemctl status pai-pulse.service
```

### 2. Reload after editing service files

```bash
# Edit service
vim ~/.config/systemd/user/pai-pulse.service

# Reload systemd
systemctl --user daemon-reload

# Restart service to apply changes
systemctl --user restart pai-pulse.service
```

### 3. Check logs before and after changes

```bash
# Before
journalctl --user -u pai-pulse.service -n 20

# Make change, restart
systemctl --user restart pai-pulse.service

# After (watch for errors)
journalctl --user -u pai-pulse.service -f
```

### 4. Use `enable --now` for new services

```bash
# ✅ Single command: enable + start
systemctl --user enable --now pai-new-service.service

# ❌ Two commands (works but unnecessary)
systemctl --user enable pai-new-service.service
systemctl --user start pai-new-service.service
```

### 5. Set resource limits

Always include in `[Service]` section:

```ini
MemoryMax=1G        # Prevent OOM
CPUQuota=50%        # Limit CPU usage
RestartSec=30       # Backoff on failure
```

### 6. Use `EnvironmentFile` for secrets

```ini
# ✅ Correct (load from file)
EnvironmentFile=/home/duane/.config/PAI/secrets.env

# ❌ Wrong (hardcoded in unit file)
Environment=ANTHROPIC_API_KEY=sk-ant-...
```

### 7. Test commands manually first

Before creating a service, test the command works:

```bash
cd /home/duane/.claude/PAI/PULSE
/home/duane/.bun/bin/bun run pulse.ts
```

If it works, then wrap in systemd.

---

## Emergency Recovery

### All Services Down

```bash
# 1. Check if systemd user instance is running
systemctl --user status

# 2. If not, restart it
systemctl --user start

# 3. Check failed services
systemctl --user --failed

# 4. Restart critical services in order
systemctl --user restart pai-litestream.service    # Database replication first
systemctl --user restart pai-pulse.service         # Then orchestrator
systemctl --user restart pai-functions.service     # Then APIs
systemctl --user restart pai-voiceserver.service   # Then optional services
```

### Systemd User Session Issues

If `systemctl --user` commands fail with "Failed to connect to bus":

```bash
# Check if user systemd is running
ps aux | grep "systemd --user"

# If not, log out and back in
# Or manually start (rare)
/usr/lib/systemd/systemd --user &
```

### Port Conflicts

If a service fails because port is in use:

```bash
# Find what's using the port
ss -tlnp | grep :31337

# Kill the conflicting process
kill -9 <PID>

# Start the service
systemctl --user start pai-pulse.service
```

### Database Corruption

If Litestream or SQLite databases are corrupted:

```bash
# 1. Stop services using the database
systemctl --user stop pai-litestream.service
systemctl --user stop pai-memory-sse.service

# 2. Restore from replica
litestream restore -config ~/.claude/config/litestream.yml \
  /home/duane/MEMORY/db/memory.db

# 3. Restart services
systemctl --user start pai-litestream.service
systemctl --user start pai-memory-sse.service
```

---

## Quick Reference Card

| Task | Command |
|------|---------|
| **List all services** | `systemctl --user list-units 'pai-*'` |
| **Start service** | `systemctl --user start pai-pulse.service` |
| **Stop service** | `systemctl --user stop pai-pulse.service` |
| **Restart service** | `systemctl --user restart pai-pulse.service` |
| **Enable auto-start** | `systemctl --user enable pai-pulse.service` |
| **Enable + start now** | `systemctl --user enable --now pai-pulse.service` |
| **Disable auto-start** | `systemctl --user disable pai-pulse.service` |
| **Check status** | `systemctl --user status pai-pulse.service` |
| **View logs (last 100)** | `journalctl --user -u pai-pulse.service -n 100` |
| **Follow logs live** | `journalctl --user -u pai-pulse.service -f` |
| **Reload systemd** | `systemctl --user daemon-reload` |
| **List timers** | `systemctl --user list-timers 'pai-*'` |
| **Failed services** | `systemctl --user --failed` |
| **Resource usage** | `systemd-cgtop --user` |
| **Is service active?** | `systemctl --user is-active pai-pulse.service` |
| **Is service enabled?** | `systemctl --user is-enabled pai-pulse.service` |
| **Health check** | `bash ~/.claude/PAI/Tools/HA/ha-status.sh --summary` |

---

**Related Docs:**
- `PAI/DOCUMENTATION/ENGINE_DAEMON_ARCHITECTURE.md` — Full daemon architecture
- `PAI/INFRA/HA_REDUNDANCY.md` — HA strategy
- `PAI/Tools/HA/topology.json` — Service topology
