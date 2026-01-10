# Security

Security practices, access control, and hardening measures for the homelab infrastructure.

## Security Philosophy

**Defense in Depth**: Multiple layers of security to protect against threats
- Network perimeter (firewall)
- Host-level protection (VM firewalls)
- Application security (authentication, authorization)
- Data protection (encryption, backups)

**Principles**:
- Minimize attack surface (close unnecessary ports)
- Strong authentication (passwords, keys)
- Regular updates (security patches)
- Monitoring and logging (detect anomalies)
- Privacy-first (local services, VPN access)

## Network Security

### Firewall Configuration

**Router/Gateway Firewall**:
- **Default policy**: Deny all incoming from Internet
- **Allow**: Established/related connections only
- **No port forwarding**: Use Tailscale VPN instead
- **Outbound**: Allow all (can be restricted if needed)

**Proxmox Host Firewall**:
```bash
# Enable Proxmox firewall
pve-firewall enable

# Datacenter level (applies to all VMs)
# /etc/pve/firewall/cluster.fw
[OPTIONS]
enable: 1

[RULES]
# Management access from local network only
IN ACCEPT -source 192.168.1.0/24 -p tcp -dport 8006
IN ACCEPT -source 192.168.1.0/24 -p tcp -dport 22
IN DROP  # Drop all other incoming
```

**Per-VM Firewalls**:

Example for web server (Nextcloud):
```
[OPTIONS]
enable: 1

[RULES]
IN ACCEPT -p tcp -dport 22   # SSH
IN ACCEPT -p tcp -dport 80   # HTTP
IN ACCEPT -p tcp -dport 443  # HTTPS
IN DROP                       # Drop everything else
```

Example for internal service (Ollama):
```
[OPTIONS]
enable: 1

[RULES]
IN ACCEPT -source 192.168.1.0/24 -p tcp -dport 11434  # API (local only)
IN ACCEPT -source 192.168.1.0/24 -p tcp -dport 22      # SSH (local only)
IN DROP
```

### Network Segmentation

**Current Setup**:
- Single network (192.168.1.0/24)
- All devices on same subnet
- Security via firewall rules

**Future VLAN Plan** (optional):
- VLAN 1: Critical infrastructure (Proxmox, NAS)
- VLAN 10: General services (Nextcloud, Plex)
- VLAN 20: IoT devices (isolated, restricted)
- VLAN 30: Guest network (no access to internal)

### VPN Security

**Tailscale**:
- WireGuard encryption (industry standard)
- Zero-trust networking
- Device authentication required
- ACLs to control access
- MFA for Tailscale account

**ACL Configuration**:
```json
{
  "acls": [
    {
      "action": "accept",
      "src": ["autogroup:member"],
      "dst": ["192.168.1.0/24:*"]
    }
  ]
}
```

**Best Practices**:
- Never expose services directly to Internet
- Always use Tailscale for remote access
- Regular key rotation
- Remove old/unused devices

## Access Control

### SSH Security

**Configuration** (/etc/ssh/sshd_config):
```bash
# Disable password authentication (keys only)
PasswordAuthentication no
PubkeyAuthentication yes

# Disable root login
PermitRootLogin no

# Use specific user for SSH
AllowUsers [your-username]

# Modern ciphers only
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com

# Disable empty passwords
PermitEmptyPasswords no
```

**SSH Key Management**:
- Generate strong keys: `ssh-keygen -t ed25519 -C "your-email"`
- Store private key securely (encrypted filesystem)
- Use SSH agent for key management
- Different keys for different purposes (work, personal)

**Fail2Ban** (optional):
```bash
# Install fail2ban to block brute force attempts
apt install fail2ban

# Configure SSH jail
# /etc/fail2ban/jail.local
[sshd]
enabled = true
maxretry = 3
bantime = 3600
```

### Web UI Security

**Proxmox Web UI**:
- HTTPS only (self-signed cert acceptable for internal)
- Strong passwords (15+ characters)
- Two-factor authentication (optional)
- Access restricted to local network + Tailscale

**Nextcloud**:
- HTTPS enforced
- Strong passwords
- Two-factor authentication enabled
- Brute force protection enabled
- Access restricted (local + Tailscale)

**Other Services**:
- Disable default credentials
- Use strong, unique passwords
- Enable 2FA where available
- Use password manager (Bitwarden, KeePassXC)

### User Management

**Proxmox Users**:
```bash
# Create admin user (avoid using root)
pveum user add admin@pam
pveum acl modify / -user admin@pam -role Administrator

# Create limited user for specific tasks
pveum user add backup@pam
pveum acl modify / -user backup@pam -role PVEAuditor
```

**VM Users**:
- No default passwords (ubuntu/ubuntu, etc.)
- Create named users during cloud-init
- Sudo access with password (or passwordless for automation)
- Regular user account audits

### API Security

**Ollama API**:
- No authentication by default (bind to localhost or internal network only)
- Optionally add reverse proxy with auth (nginx + basic auth)
- Access control via firewall (192.168.1.0/24 only)

**Other APIs**:
- Use API keys where available
- Rotate keys regularly
- Store keys in environment variables (not code)
- Restrict API access to specific IPs/networks

## Data Security

### Encryption at Rest

**BorgBackup**:
- Repokey encryption mode
- Strong passphrase (20+ characters)
- Passphrase stored securely (password manager + offline backup)

**Disk Encryption** (optional):
- LUKS encryption for sensitive VMs
- Encryption keys in secure location
- Recovery keys backed up offline

**Nextcloud**:
- Server-side encryption available (optional)
- End-to-end encryption for specific folders (optional)
- HTTPS for all transfers

### Encryption in Transit

**All services use encryption**:
- SSH for terminal access
- HTTPS for web interfaces
- Tailscale (WireGuard) for VPN
- BorgBackup encrypted transfers

**Certificate Management**:
- Self-signed certs for internal services (acceptable)
- Let's Encrypt for external services (if any)
- Regular cert renewal
- Monitor expiration dates

### Backup Security

**Backup Protection**:
- Encrypted backups (BorgBackup)
- Immutable backups (optional: append-only mode)
- Off-site backups (different physical location)
- Access control (only authorized users)

**Backup Testing**:
- Regular restore tests
- Verify encryption works
- Document recovery procedures

## System Hardening

### Operating System

**Updates**:
```bash
# Proxmox updates (monthly)
apt update && apt upgrade

# VM updates (automated with unattended-upgrades)
apt install unattended-upgrades
dpkg-reconfigure --priority=low unattended-upgrades
```

**Service Hardening**:
- Disable unnecessary services
- Run services as non-root users
- Use AppArmor/SELinux profiles (optional)
- Minimal software installation

**File Permissions**:
```bash
# Sensitive files
chmod 600 /etc/ssh/ssh_host_*_key
chmod 600 ~/.ssh/id_*
chmod 700 ~/.ssh

# Configuration files
chmod 644 /etc/passwd
chmod 640 /etc/shadow
```

### Application Security

**Docker/Containers** (if used):
- Don't run containers as root
- Use official images from trusted sources
- Regular image updates
- Network isolation between containers

**Web Applications**:
- Keep software updated
- Disable debug modes in production
- Use strong session management
- Input validation and sanitization

## Monitoring & Logging

### Log Collection

**Centralized logs** (optional):
```bash
# Syslog forwarding to central server
# /etc/rsyslog.conf
*.* @@192.168.1.X:514
```

**Important logs to monitor**:
- SSH authentication attempts: /var/log/auth.log
- Proxmox events: /var/log/pve/
- VM logs: /var/log/syslog, /var/log/messages
- Web server logs: /var/log/nginx/, /var/log/apache2/

### Security Monitoring

**Failed login attempts**:
```bash
# Check for failed SSH logins
grep "Failed password" /var/log/auth.log

# Check for sudo attempts
grep "sudo" /var/log/auth.log
```

**Unusual network activity**:
```bash
# Active connections
ss -tunap

# Listening ports
netstat -tulpn

# Firewall logs
tail -f /var/log/pve-firewall.log
```

**Disk usage anomalies**:
```bash
# Check for rapid disk usage growth (ransomware indicator)
df -h
zfs list

# Monitor I/O
iotop
```

### Alerting

**Email alerts for**:
- Failed backups
- Disk space > 80%
- S.M.A.R.T. warnings
- Service failures
- Unusual login activity

**Monitoring tools** (optional):
- Prometheus + Grafana
- Uptime Kuma
- Netdata

## Incident Response

### Security Incident Procedure

1. **Detect**: Monitor logs, alerts, unusual behavior
2. **Isolate**: Disconnect affected systems from network
3. **Investigate**: Analyze logs, determine scope
4. **Remediate**: Remove threat, patch vulnerability
5. **Recover**: Restore from backups if needed
6. **Document**: Record incident, lessons learned

### Backup Restoration (Compromise)

If system is compromised:
```bash
# 1. Isolate affected VM
qm stop [VMID]

# 2. Snapshot current state (for forensics)
qm snapshot [VMID] compromised-$(date +%Y%m%d)

# 3. Restore from clean backup
qmrestore [BACKUP] [NEW-VMID]

# 4. Investigate root cause
# 5. Apply security patches
# 6. Restore data from BorgBackup (verified clean)
```

## Compliance & Best Practices

### Security Checklist

**Monthly**:
- [ ] Apply security updates (Proxmox, VMs)
- [ ] Review firewall logs
- [ ] Check failed login attempts
- [ ] Verify backup completion
- [ ] Test backup restore (random VM)
- [ ] Review user accounts (remove old)

**Quarterly**:
- [ ] Security audit (ports, services, access)
- [ ] Password rotation (critical accounts)
- [ ] Certificate expiration check
- [ ] Disaster recovery test
- [ ] Review and update firewall rules

**Annually**:
- [ ] Full security assessment
- [ ] Documentation review
- [ ] Hardware security check (physical access)
- [ ] Backup strategy review

### Security Resources

**Stay informed**:
- Proxmox security mailing list
- CVE databases for installed software
- Security blogs (Krebs, Schneier)
- r/homelab, r/selfhosted security discussions

## Known Vulnerabilities

Document any known issues and mitigation:

| Issue | Severity | Mitigation | Planned Fix |
|-------|----------|------------|-------------|
| [Example] | [High/Med/Low] | [Temporary mitigation] | [Date] |

## Security Improvements

**Completed**:
- [Date]: Implemented SSH key-only authentication
- [Date]: Enabled Proxmox firewall on all VMs
- [Date]: Migrated from port forwarding to Tailscale
- [Document your improvements]

**Planned**:
- [ ] Implement VLAN segmentation for IoT
- [ ] Deploy intrusion detection (Suricata/Snort)
- [ ] Set up centralized logging (ELK stack)
- [ ] [Your planned improvements]
