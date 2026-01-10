# Disaster Scenarios & Response Plans

Documented procedures for handling various failure scenarios in the homelab infrastructure.

## Overview

**Recovery Objectives**:
- **RTO (Recovery Time Objective)**: 15 minutes for critical VMs, 4-6 hours for full infrastructure
- **RPO (Recovery Point Objective)**: 24 hours (last daily backup)

**Critical Services** (prioritize in this order):
1. Network connectivity (router, Proxmox host)
2. Nextcloud (user data and collaboration)
3. Development environment (emacsOS)
4. Ollama (AI services)
5. Plex and other media services

## Scenario 1: Single VM Failure

### Symptoms
- VM won't boot
- VM filesystem corruption
- VM performing poorly or unstable

### Impact
- **Severity**: Low to Medium
- **Affected users**: Users of that specific service
- **RTO**: 15 minutes

### Response Procedure

**Step 1: Assess the situation**
```bash
# Check VM status
qm status [VMID]

# Try to start if stopped
qm start [VMID]

# Check console for boot errors
qm terminal [VMID]

# Check logs
tail -f /var/log/syslog | grep qm
```

**Step 2: Quick fixes (if applicable)**
```bash
# Unlock if locked
qm unlock [VMID]

# Reset if hung
qm reset [VMID]

# Try safe mode boot (inside VM)
# Edit grub to add 'single' or 'recovery'
```

**Step 3: Restore from backup (if quick fixes fail)**
```bash
# List available backups
pvesm list local-zfs | grep "vm-[VMID]"

# Identify most recent backup
BACKUP="local-zfs:backup-[VMID]-YYYY_MM_DD-HH_MM_SS.vma.zst"

# Restore to new VMID (keep old VM for forensics)
qmrestore $BACKUP [NEW-VMID]

# Start restored VM
qm start [NEW-VMID]

# Verify service is working
curl http://[service]
ssh [VM] 'systemctl status [service]'

# Update DNS/IP if needed
# Document what happened
```

**Step 4: Post-recovery**
```bash
# Investigate root cause (old VM)
qm start [OLD-VMID]
# Review logs, check for corruption

# Clean up when satisfied with new VM
qm stop [OLD-VMID]
qm destroy [OLD-VMID]

# Document incident in maintenance log
```

**Estimated Recovery Time**: 15-20 minutes

---

## Scenario 2: Proxmox Host Failure (Hardware Issue)

### Symptoms
- Server won't boot
- Kernel panic
- Hardware failure (motherboard, CPU, RAM)

### Impact
- **Severity**: **Critical**
- **Affected users**: Everyone (all VMs down)
- **RTO**: 4-8 hours (includes hardware procurement if needed)

### Response Procedure

**Step 1: Diagnose hardware issue**
```bash
# If server accessible via SSH:
dmesg | grep -i error
journalctl -p err -b

# If server not booting:
# - Check POST beeps/LED codes
# - Test RAM (memtest86+)
# - Check drive connections
# - Try minimal boot (1 RAM stick, no extras)
```

**Step 2: Decision tree**

**If fixable quickly (loose cable, RAM reseat)**:
- Fix issue
- Boot server
- Verify VMs start automatically
- Monitor for stability

**If hardware replacement needed**:
- Proceed to Step 3 (complete rebuild)

**Step 3: Complete infrastructure rebuild**

**3a. Acquire replacement hardware**
- Use spare server (if available)
- Purchase new/used server
- Borrow from friend/work (temporary)

**3b. Install Proxmox**
```bash
# Download latest Proxmox ISO
# Boot from USB
# Install Proxmox VE
# Configure network (same IP: 192.168.1.29)
# Apply updates
apt update && apt upgrade
```

**3c. Import ZFS pool**
```bash
# If ZFS drives are intact:
# Connect drives to new server
zpool import
zpool import [POOL-NAME]

# Verify pool health
zpool status

# Add ZFS storage to Proxmox
pvesm add zfspool local-zfs --pool [POOL-NAME]
```

**3d. Restore VMs from backup**
```bash
# List backups
pvesm list local-zfs | grep vzdump

# Restore critical VMs first
# Nextcloud
qmrestore local-zfs:backup-100-*.vma.zst 100

# emacsOS
qmrestore local-zfs:backup-101-*.vma.zst 101

# Ollama
qmrestore local-zfs:backup-102-*.vma.zst 102

# Others...
```

**3e. Start VMs and verify**
```bash
# Start VMs one by one
qm start 100  # Nextcloud
qm start 101  # emacsOS
qm start 102  # Ollama

# Verify network connectivity
ping cloud.bear
ssh emacsOS

# Verify services
curl http://cloud.bear
curl http://ollama:11434/api/version
```

**3f. Reconfigure if needed**
- Update DHCP reservations (if MAC changed)
- Regenerate SSL certificates (if tied to hardware)
- Update Tailscale (may need re-authentication)

**Estimated Recovery Time**: 4-8 hours (depends on hardware availability)

---

## Scenario 3: ZFS Pool Failure (Both Drives Lost)

### Symptoms
- Pool import fails
- Catastrophic corruption
- Both drives in mirror failed simultaneously

### Impact
- **Severity**: **Critical**
- **Affected users**: Everyone (all VM data lost)
- **RPO**: Last BorgBackup (typically <24 hours for critical data)
- **RTO**: 8-12 hours (rebuild infrastructure)

### Response Procedure

**Step 1: Attempt pool recovery**
```bash
# Try to import pool
zpool import

# If degraded but importable
zpool import -f [POOL-NAME]

# Check status
zpool status

# If pool is dead, proceed to Step 2
```

**Step 2: Accept data loss, rebuild from backups**

**2a. Create new ZFS pool**
```bash
# Install new drives
# Create new pool
zpool create [NEW-POOL] mirror /dev/sdX /dev/sdY

# Configure compression
zfs set compression=lz4 [NEW-POOL]

# Add to Proxmox
pvesm add zfspool local-zfs --pool [NEW-POOL]
```

**2b. Recreate VMs manually**

Since Proxmox VM backups are on failed pool, rebuild from BorgBackup:

```bash
# Create new VMs with appropriate specs
qm create 100 --name nextcloud --memory 16384 --cores 6
qm set 100 --scsi0 local-zfs:32
qm set 100 --net0 virtio,bridge=vmbr0

# Install OS from ISO
qm set 100 --cdrom local:iso/ubuntu-22.04.iso
qm start 100

# Repeat for all VMs
```

**2c. Restore data from BorgBackup**

On each VM after OS installation:
```bash
# Install BorgBackup
apt install borgbackup

# Mount backup repository
borg mount /path/to/repo::latest /mnt/backup

# Restore data
cp -a /mnt/backup/home/user/ /home/user/
cp -a /mnt/backup/var/www/ /var/www/

# Unmount
borg umount /mnt/backup

# Verify services work
systemctl restart [services]
```

**2d. Reconfigure services**

- Nextcloud: Restore config from backup
- Ollama: Re-download models
- Stable Diffusion: Re-download models
- Plex: Rescan media libraries

**Estimated Recovery Time**: 8-12 hours (very manual process)

**Lesson**: This scenario reinforces the importance of off-host backups (BorgBackup). Proxmox backups on same ZFS pool are not sufficient alone.

---

## Scenario 4: Complete Infrastructure Loss (Fire, Flood, Theft)

### Symptoms
- Total equipment loss
- Building damage
- All hardware destroyed or inaccessible

### Impact
- **Severity**: **Catastrophic**
- **Affected users**: Everyone
- **RPO**: Last off-site BorgBackup
- **RTO**: 2-5 days (includes hardware procurement)

### Response Procedure

**Step 1: Assess situation and prioritize**
- Ensure personal safety first
- Assess what data is critical to recover
- Determine budget for replacement hardware

**Step 2: Acquire replacement hardware**
- Purchase new server (or use temporary laptop)
- Minimum: CPU, 32GB RAM, storage for VMs
- Network connectivity (home internet or temporary hotspot)

**Step 3: Access off-site backups**
```bash
# Install BorgBackup on temporary system
apt install borgbackup

# Connect to off-site backup repository
# (Ensure passphrase is safely stored externally)
borg list /path/to/offsite/repo

# Restore critical data first
borg extract /path/to/offsite/repo::latest

# Recover:
# - Work files (highest priority)
# - Configuration files
# - Personal data
# - Media (lowest priority)
```

**Step 4: Rebuild critical services only**

**Don't try to rebuild everything immediately**. Prioritize:

1. **Basic workstation** (laptop/desktop with recovered data)
2. **Cloud sync** (set up Nextcloud on VPS if needed)
3. **Development environment** (on workstation)

**Step 5: Long-term recovery**

Once stable:
- Rebuild Proxmox infrastructure
- Restore VMs from backups
- Reconnect services
- Resume normal operations

**Estimated Recovery Time**:
- Critical data access: 4-8 hours
- Basic operations: 1-2 days
- Full infrastructure: 1-2 weeks

**Prevention**:
- Maintain off-site BorgBackup repository
- Store passphrase securely (password manager + paper backup)
- Document infrastructure in Git (GitHub)
- Consider cloud backup for irreplaceable data

---

## Scenario 5: Ransomware / Security Compromise

### Symptoms
- Files encrypted
- Unknown processes running
- Unusual network activity
- Data exfiltration detected

### Impact
- **Severity**: **High**
- **Affected users**: Potentially everyone
- **RTO**: 2-8 hours (isolate, rebuild, restore)

### Response Procedure

**Step 1: IMMEDIATE - Isolate affected systems**
```bash
# Shut down affected VMs
qm stop [VMID]

# Disconnect network if needed
qm set [VMID] --net0 virtio,link_down=1

# Snapshot current state (for forensics)
qm snapshot [VMID] compromised-$(date +%Y%m%d-%H%M)

# DO NOT attempt to clean - assume full compromise
```

**Step 2: Assess scope**
```bash
# Check all VMs for compromise indicators
# Look for:
# - Encrypted files
# - Unknown processes
# - Modified system files
# - Unusual network connections

# Check Proxmox host
# - Review auth logs
# - Check for unauthorized access
# - Verify backup integrity
```

**Step 3: Verify backup integrity**
```bash
# Check BorgBackup repository
borg info /path/to/repo

# Verify backups are not encrypted/corrupted
borg list /path/to/repo
borg mount /path/to/repo::latest /mnt/test
ls -la /mnt/test  # Should see normal files

# If backups are compromised, go to older backups
```

**Step 4: Rebuild from clean backups**
```bash
# Delete compromised VMs
qm destroy [VMID]

# Restore from backup BEFORE compromise
# Identify last known good backup
borg list /path/to/repo

# Restore to new VM
qm restore [BACKUP-FROM-BEFORE-COMPROMISE] [NEW-VMID]

# DO NOT restore from recent backups (may contain malware)
```

**Step 5: Security hardening**
```bash
# Change all passwords
# Rotate SSH keys
# Review firewall rules
# Enable 2FA everywhere possible
# Apply all security updates
# Review access logs
# Document incident
```

**Step 6: Post-incident analysis**
- How did compromise occur?
- What data was accessed?
- What security measures failed?
- What needs to be improved?

**Estimated Recovery Time**: 2-8 hours

**Prevention**:
- Immutable backups (append-only mode for Borg)
- Regular security audits
- No port forwarding (VPN only)
- Strong authentication
- Minimal attack surface

---

## Emergency Contact Information

### Service Providers

| Service | Provider | Contact | Account # |
|---------|----------|---------|-----------|
| Internet | [ISP] | [Phone] | [Account #] |
| Hardware | [Vendor] | [Support] | [Account #] |
| Backup Storage | [If cloud] | [Support] | [Account #] |

### Recovery Resources

**Documentation Locations**:
- GitHub: [Your GitHub profile]
- Local backup: [Encrypted USB drive location]
- Cloud backup: [Location]

**Critical Passwords/Keys**:
- Password manager: [Which service]
- BorgBackup passphrase: [Secure location]
- SSH keys: [Backup location]

**Hardware Vendors** (for quick replacement):
- [Local computer store]
- [Online vendor with fast shipping]
- [Used server marketplace]

## Testing Schedule

**Disaster Recovery Drills**:
- **Monthly**: Test VM restore (random VM)
- **Quarterly**: Simulate complete VM loss
- **Annually**: Simulate host failure

**Last Tests**:
| Scenario | Date | Result | Notes |
|----------|------|--------|-------|
| VM restore | [Date] | ✓ Pass | Took 12 minutes |
| Host failure | [Date] | ✓ Pass | Restored in 6 hours |
| Pool corruption | [Date] | ⚠ Partial | Need better docs |

## Continuous Improvement

After each incident or test:
- [ ] Update this document
- [ ] Document lessons learned
- [ ] Improve automation
- [ ] Add monitoring/alerting
- [ ] Review backup strategy

---

**Remember**: The best disaster recovery plan is one that's tested regularly. Don't wait for a real disaster to find out if your backups work.
