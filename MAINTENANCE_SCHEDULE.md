# Maintenance Schedule

Regular maintenance tasks to keep the homelab infrastructure running reliably.

## Daily Tasks (Automated)

### Backups

**BorgBackup** (3:00 AM):
```bash
# Automated via systemd timer
# /etc/systemd/system/borgbackup.timer
[Unit]
Description=Daily BorgBackup

[Timer]
OnCalendar=daily
Persistent=true
OnCalendar=03:00

[Install]
WantedBy=timers.target
```

**Proxmox VM Backups** (2:00 AM):
```bash
# Via Proxmox backup schedule or cron
# /usr/local/bin/proxmox-backup.sh
vzdump --all --storage local-zfs --mode snapshot --compress zstd \
  --prune-backups keep-daily=7,keep-weekly=4,keep-monthly=6
```

### Monitoring

**Automated checks**:
- Disk space monitoring
- Service health checks
- Temperature monitoring
- Network connectivity

## Weekly Tasks

### Sunday Morning Routine (~30 minutes)

**Review backup status**:
```bash
# Check BorgBackup logs
journalctl -u borgbackup.service -S -1week

# Check Proxmox backup logs
cat /var/log/vzdump.log | tail -50

# Verify backup sizes (should be reasonable)
ls -lh /path/to/backups/
borg info /path/to/borg/repo
```

**Check resource usage**:
```bash
# On Proxmox host
htop
free -h
df -h
zpool list
zfs list -o space

# Per-VM usage
qm list
for vm in $(qm list | awk 'NR>1 {print $1}'); do
  echo "VM $vm:"
  qm config $vm | grep -E 'cores|memory'
done
```

**Review logs for errors**:
```bash
# System errors
journalctl -p err -S -1week

# Proxmox errors
tail -100 /var/log/syslog | grep -i error

# Service-specific logs
# Nextcloud, Plex, etc.
```

**Update dashboard** (if you track metrics):
- Total storage used
- Backup success rate
- Service uptime
- Notable events

## Monthly Tasks

### First Weekend of Month (~2-3 hours)

**System updates**:
```bash
# Proxmox host
apt update && apt list --upgradable
apt upgrade -y
# Reboot if kernel updated
[ -f /var/run/reboot-required ] && reboot

# VMs (one at a time)
ssh [VM] 'sudo apt update && sudo apt upgrade -y'
# Reboot if needed
ssh [VM] '[ -f /var/run/reboot-required ] && sudo reboot'
```

**ZFS maintenance**:
```bash
# Start scrub (verifies data integrity)
zpool scrub [POOL-NAME]

# Check scrub progress
zpool status

# Wait for completion (can take hours)
# Review results
zpool status -v  # Should show "no errors"
```

**Disk health check**:
```bash
# Check S.M.A.R.T. status for all drives
for drive in /dev/sd*; do
  echo "Checking $drive"
  smartctl -H $drive
  smartctl -a $drive | grep -E 'Power_On_Hours|Reallocated_Sector|Current_Pending_Sector'
done

# Document any warnings
# Plan drive replacement if issues found
```

**Security review**:
```bash
# Failed login attempts
grep "Failed password" /var/log/auth.log | wc -l

# Unusual sudo usage
grep "sudo" /var/log/auth.log | grep -v [your-username]

# Open ports audit
nmap -p- localhost

# Firewall rule review
iptables -L -n
pve-firewall status
```

**Backup verification**:
```bash
# Test restore of random VM
BACKUP=$(pvesm list local-zfs | grep vzdump | shuf -n 1 | awk '{print $1}')
echo "Testing restore of: $BACKUP"

# Restore to test VMID
qmrestore $BACKUP 999

# Boot and verify
qm start 999
# Test SSH, services, data integrity
ssh [test-vm] 'ls -la && df -h'

# Clean up
qm stop 999
qm destroy 999
```

**Service health checks**:
```bash
# Nextcloud
curl -I https://cloud.bear
# Check for occ warnings
ssh nextcloud-vm 'sudo -u www-data php /var/www/nextcloud/occ status'

# Plex
curl -I http://plex:32400

# Ollama
curl http://ollama:11434/api/version

# Stable Diffusion
curl http://sd:7860

# Home Assistant
curl http://homeassistant:8123
```

**Certificate expiration check** (if using Let's Encrypt):
```bash
# Check cert expiration
echo | openssl s_client -connect cloud.bear:443 2>/dev/null | \
  openssl x509 -noout -dates
```

## Quarterly Tasks

### Every 3 Months (~4-6 hours)

**Full disaster recovery test**:
```bash
# Simulate complete VM loss
# 1. Document current state
# 2. Delete test VM
# 3. Restore from backup
# 4. Verify all data and services
# 5. Document recovery time
# 6. Update disaster recovery procedures
```

**Capacity planning**:
```bash
# Storage growth analysis
zfs list -o space
# Calculate growth rate
# Estimate time until 80% full
# Plan expansion if needed

# RAM usage trends
# Review per-VM allocation
# Adjust as needed

# CPU usage trends
# Identify over/under-allocated VMs
```

**Security audit**:
```bash
# Review all user accounts
pveum user list
# Remove unused accounts

# Review firewall rules
iptables -L -n -v
# Remove obsolete rules

# Check for outdated software
apt list --upgradable

# Review SSH authorized_keys
cat ~/.ssh/authorized_keys
# Remove old/unused keys
```

**Hardware maintenance**:
```bash
# Clean dust from server
# Inspect fans (noise, speed)
# Check cable management
# Verify cooling (temperature trends)
# Inspect for physical damage
```

**Documentation review**:
```bash
# Update this maintenance schedule
# Update network topology
# Update service directory
# Review and update disaster recovery procedures
# Document changes made this quarter
```

**Backup retention cleanup**:
```bash
# Review old backups
# Prune if needed (BorgBackup auto-prunes)
borg list /path/to/repo
borg prune /path/to/repo --keep-daily=7 --keep-weekly=4 --keep-monthly=6

# Proxmox backups (should auto-prune)
pvesm list local-zfs
# Manually delete if needed
```

## Annual Tasks

### Once per Year (~8-12 hours)

**Complete infrastructure review**:
- [ ] Review all hardware (plan replacements)
- [ ] Review all services (decommission unused)
- [ ] Review all documentation (update as needed)
- [ ] Review network topology (optimize)
- [ ] Review security posture (penetration test?)

**Major updates**:
- [ ] Proxmox major version upgrade (if available)
- [ ] VM OS upgrades (Ubuntu LTS releases)
- [ ] Firmware updates (BIOS, NIC, etc.)

**Full backup rotation**:
- [ ] Create annual archive backup
- [ ] Store off-site (different location)
- [ ] Verify backup integrity
- [ ] Document storage location

**Hardware deep clean**:
- [ ] Full server teardown and clean
- [ ] Replace thermal paste (if CPU temps rising)
- [ ] Replace fans (if noisy)
- [ ] Cable re-organization

**Disaster recovery drill**:
- [ ] Simulate complete infrastructure loss
- [ ] Restore from off-site backups
- [ ] Document recovery time
- [ ] Update disaster recovery plan
- [ ] Train family members on basic recovery (if applicable)

**Financial review**:
- [ ] Calculate annual operating costs
  - Power consumption
  - Internet service
  - Hardware purchases
  - Subscriptions (if any)
- [ ] Budget for next year
- [ ] Plan major purchases

## As-Needed Tasks

### When Adding New Service

- [ ] Document service in SERVICES_DIRECTORY.md
- [ ] Configure firewall rules
- [ ] Add to backup schedule (if needed)
- [ ] Add monitoring/health checks
- [ ] Update network topology
- [ ] Test disaster recovery

### When Decommissioning Service

- [ ] Create final backup
- [ ] Export/migrate data
- [ ] Stop service
- [ ] Remove from monitoring
- [ ] Update documentation
- [ ] Clean up storage
- [ ] Remove firewall rules

### When Expanding Storage

- [ ] Plan expansion (see zfs-storage-architecture)
- [ ] Order new drives
- [ ] Test drives before installation
- [ ] Add to ZFS pool
- [ ] Verify resilver completion
- [ ] Update documentation
- [ ] Update capacity planning

### When Adding New VM

- [ ] Plan resource allocation
- [ ] Create VM with appropriate specs
- [ ] Configure networking
- [ ] Configure firewall
- [ ] Add to backup schedule
- [ ] Document in SERVICES_DIRECTORY.md
- [ ] Test backup/restore

## Maintenance Calendar Template

### January
- [x] Monthly tasks (updates, scrub, health check)
- [ ] Quarterly tasks (disaster recovery test, audit)

### February
- [ ] Monthly tasks

### March
- [ ] Monthly tasks

### April
- [ ] Monthly tasks
- [ ] Quarterly tasks

### May
- [ ] Monthly tasks

### June
- [ ] Monthly tasks

### July
- [ ] Monthly tasks
- [ ] Quarterly tasks
- [ ] Annual tasks (full infrastructure review)

### August
- [ ] Monthly tasks

### September
- [ ] Monthly tasks

### October
- [ ] Monthly tasks
- [ ] Quarterly tasks

### November
- [ ] Monthly tasks

### December
- [ ] Monthly tasks
- [ ] Year-end review

## Maintenance Log

Document significant maintenance performed:

| Date | Task | Duration | Notes |
|------|------|----------|-------|
| 2025-01-10 | Monthly updates | 1h | Updated Proxmox and all VMs |
| [Date] | [Task] | [Time] | [Notes] |

## Lessons Learned

- [Document what maintenance tasks were most valuable]
- [What tasks could be automated further]
- [What issues were prevented by proactive maintenance]
- [What you wish you had done sooner]

## Automation Ideas

**Future automation opportunities**:
- [ ] Automated email reports (weekly summary)
- [ ] Automated health checks with alerting
- [ ] Automated update deployment (with rollback)
- [ ] Automated capacity planning reports
- [ ] Dashboard for at-a-glance status
