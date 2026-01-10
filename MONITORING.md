# Monitoring

Health checks, observability, and monitoring strategies for the homelab infrastructure.

## Monitoring Philosophy

**Objectives**:
- Early detection of issues before they cause outages
- Track resource usage trends for capacity planning
- Maintain visibility into infrastructure health
- Automate alerts for critical events

**Approach**:
- Simple monitoring first (scripts + cron)
- Progressive enhancement (add dashboards later)
- Focus on actionable metrics
- Avoid alert fatigue (only alert on critical issues)

## Current Monitoring Stack

### Built-in Proxmox Monitoring

**Proxmox Web UI** (https://deadmall.bear:8006):
- Real-time resource graphs (CPU, RAM, network, disk I/O)
- VM status dashboard
- Storage usage
- Task history

**Metrics available**:
```bash
# Via Proxmox API
pvesh get /nodes/deadmall/status

# Detailed per-VM
pvesh get /nodes/deadmall/qemu/[VMID]/status/current
```

### ZFS Monitoring

**Pool Health**:
```bash
# Check pool status (run daily)
zpool status

# Should show: "state: ONLINE" and "scan: scrub repaired 0B"
# Alerts on: DEGRADED, FAULTED, errors > 0
```

**Space Monitoring**:
```bash
# Check space usage
zfs list -o space

# Alert when:
# - Any dataset > 80% full
# - Pool > 85% full
```

**I/O Performance**:
```bash
# Real-time I/O stats
zpool iostat -v [POOL] 5

# Watch for:
# - Sustained high I/O wait
# - Increasing latency
# - Checksum errors
```

### Service Health Checks

**HTTP endpoints**:
```bash
#!/bin/bash
# /usr/local/bin/check-services.sh

# Nextcloud
curl -f -s http://cloud.bear > /dev/null || echo "Nextcloud DOWN"

# Plex
curl -f -s http://plex:32400/web > /dev/null || echo "Plex DOWN"

# Ollama
curl -f -s http://ollama:11434/api/version > /dev/null || echo "Ollama DOWN"

# Stable Diffusion
curl -f -s http://sd:7860 > /dev/null || echo "Stable Diffusion DOWN"

# Home Assistant
curl -f -s http://homeassistant:8123 > /dev/null || echo "Home Assistant DOWN"
```

**Run via cron**:
```cron
*/5 * * * * /usr/local/bin/check-services.sh | grep DOWN && mail -s "Service Down" your-email@example.com
```

### Disk Health (S.M.A.R.T.)

**Check all drives**:
```bash
#!/bin/bash
# /usr/local/bin/check-smart.sh

for drive in /dev/sda /dev/sdb; do
  STATUS=$(smartctl -H $drive | grep "SMART overall-health" | awk '{print $6}')
  if [ "$STATUS" != "PASSED" ]; then
    echo "WARNING: $drive health check: $STATUS"
  fi

  # Check for reallocated sectors (bad sign)
  REALLOC=$(smartctl -a $drive | grep "Reallocated_Sector_Ct" | awk '{print $10}')
  if [ "$REALLOC" -gt 0 ]; then
    echo "WARNING: $drive has $REALLOC reallocated sectors"
  fi
done
```

**Run daily**:
```cron
0 6 * * * /usr/local/bin/check-smart.sh | grep WARNING && mail -s "SMART Alert" your-email@example.com
```

### Backup Monitoring

**Verify backups completed**:
```bash
#!/bin/bash
# /usr/local/bin/check-backups.sh

# Check BorgBackup
LAST_BACKUP=$(borg list /path/to/repo | tail -1 | awk '{print $3,$4}')
AGE=$(( ($(date +%s) - $(date -d "$LAST_BACKUP" +%s)) / 3600 ))

if [ $AGE -gt 30 ]; then
  echo "WARNING: Last BorgBackup is $AGE hours old"
fi

# Check Proxmox VM backups
LAST_VZDUMP=$(grep "INFO: Backup job finished successfully" /var/log/vzdump.log | tail -1 | awk '{print $1,$2}')
AGE=$(( ($(date +%s) - $(date -d "$LAST_VZDUMP" +%s)) / 3600 ))

if [ $AGE -gt 30 ]; then
  echo "WARNING: Last Proxmox backup is $AGE hours old"
fi
```

## Metrics to Track

### Resource Usage

**CPU**:
- Per-VM CPU usage (%)
- Host CPU usage (%)
- CPU steal time (should be near 0)
- Load average

**Memory**:
- Per-VM memory usage (%)
- Host memory usage (%)
- ZFS ARC size
- Swap usage (should be minimal)

**Disk**:
- Pool space used (%)
- Per-dataset space used (%)
- I/O wait time
- Disk throughput (MB/s)

**Network**:
- Bandwidth usage (MB/s)
- Packet loss (should be 0)
- DNS query rate (Pi-hole)
- Failed connection attempts

### Service-Specific Metrics

**Nextcloud**:
- Active users
- Storage used
- Sync errors
- PHP-FPM queue length

**Plex**:
- Active streams
- Transcode load
- Library size

**Ollama**:
- Active model requests
- Average inference time
- Memory usage per model

**Stable Diffusion**:
- Generation queue length
- GPU utilization
- Average generation time

## Alerting Rules

### Critical Alerts (Immediate Action)

- **Pool DEGRADED/FAULTED**: Drive failure imminent
- **Disk space > 90%**: Risk of service outage
- **Backup failed**: Data loss risk
- **S.M.A.R.T. failure**: Drive replacement needed
- **Host unreachable**: Complete outage

**Alert method**: Email + SMS (if configured)

### Warning Alerts (Review Soon)

- **Disk space > 80%**: Plan expansion
- **Backup age > 30 hours**: Investigate backup failure
- **High CPU usage sustained**: Potential resource constraint
- **Service down > 5 minutes**: Non-critical service issue

**Alert method**: Email

### Informational (Log Only)

- **VM started/stopped**: Normal operation
- **Backup completed successfully**: Confirmation
- **Routine updates applied**: Change tracking

**Alert method**: Log file only

## Monitoring Scripts

### Daily Health Check

```bash
#!/bin/bash
# /usr/local/bin/daily-health-check.sh
# Run via cron: 0 7 * * * /usr/local/bin/daily-health-check.sh

echo "=== Daily Health Check $(date) ===" > /tmp/health-check.txt

# ZFS health
echo "ZFS Pool Status:" >> /tmp/health-check.txt
zpool status | grep -E "state:|errors:" >> /tmp/health-check.txt

# Disk space
echo -e "\nDisk Space:" >> /tmp/health-check.txt
df -h | grep -E "Filesystem|zfs|/$" >> /tmp/health-check.txt

# Memory usage
echo -e "\nMemory Usage:" >> /tmp/health-check.txt
free -h | grep -E "Mem:|Swap:" >> /tmp/health-check.txt

# Service status
echo -e "\nService Health:" >> /tmp/health-check.txt
systemctl is-active pveproxy pvedaemon pve-cluster >> /tmp/health-check.txt

# Recent errors
echo -e "\nRecent Errors:" >> /tmp/health-check.txt
journalctl -p err -S -24h --no-pager | tail -10 >> /tmp/health-check.txt

# Email report
mail -s "Daily Health Check - $(hostname)" your-email@example.com < /tmp/health-check.txt
```

### Resource Usage Logging

```bash
#!/bin/bash
# /usr/local/bin/log-resource-usage.sh
# Run via cron: */15 * * * * /usr/local/bin/log-resource-usage.sh

LOGFILE="/var/log/resource-usage.log"
TIMESTAMP=$(date "+%Y-%m-%d %H:%M:%S")

# CPU load
LOAD=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}' | sed 's/,//')

# Memory usage (%)
MEM=$(free | grep Mem | awk '{printf "%.1f", ($3/$2) * 100}')

# Disk usage (%)
DISK=$(df -h / | tail -1 | awk '{print $5}' | sed 's/%//')

# ZFS pool usage (%)
POOL=$(zpool list -H [POOL-NAME] | awk '{print $8}' | sed 's/%//')

echo "$TIMESTAMP,$LOAD,$MEM,$DISK,$POOL" >> $LOGFILE

# Rotate log monthly
if [ $(date +%d) -eq 01 ]; then
  mv $LOGFILE $LOGFILE.$(date +%Y%m -d "last month")
fi
```

## Dashboard Ideas (Future)

### Grafana + Prometheus

**Metrics to visualize**:
- CPU/RAM/Disk over time (trends)
- Service uptime (availability percentage)
- Backup success rate
- Network bandwidth
- Per-VM resource allocation vs usage

**Setup** (when time permits):
```bash
# Install Prometheus on monitoring VM
apt install prometheus prometheus-node-exporter

# Install Grafana
apt install grafana

# Configure Proxmox exporter
# Import Proxmox dashboard
# Add ZFS metrics
```

### Simple Web Dashboard

**Using simple HTML + Python**:
```python
# /usr/local/bin/dashboard.py
# Generates simple status page

import subprocess
import datetime

html = """
<html>
<head><title>Homelab Status</title></head>
<body>
<h1>Homelab Infrastructure Status</h1>
<p>Last updated: {timestamp}</p>

<h2>ZFS Pool</h2>
<pre>{zpool_status}</pre>

<h2>VM Status</h2>
<pre>{vm_status}</pre>

<h2>Disk Space</h2>
<pre>{disk_space}</pre>

<h2>Last Backup</h2>
<pre>{last_backup}</pre>

</body>
</html>
"""

timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
zpool_status = subprocess.check_output(['zpool', 'status']).decode()
vm_status = subprocess.check_output(['qm', 'list']).decode()
disk_space = subprocess.check_output(['df', '-h']).decode()
last_backup = subprocess.check_output(['borg', 'list', '/path/to/repo']).decode().split('\n')[-2]

print(html.format(
    timestamp=timestamp,
    zpool_status=zpool_status,
    vm_status=vm_status,
    disk_space=disk_space,
    last_backup=last_backup
))
```

Serve via nginx or Python http.server.

## Log Management

### Important Logs

| Log File | Purpose | Retention |
|----------|---------|-----------|
| /var/log/syslog | System events | 7 days |
| /var/log/auth.log | Authentication attempts | 30 days |
| /var/log/pve/ | Proxmox events | 30 days |
| /var/log/vzdump.log | Backup logs | 90 days |
| /var/log/resource-usage.log | Custom metrics | 12 months |

### Log Rotation

**Configure logrotate**:
```bash
# /etc/logrotate.d/homelab

/var/log/resource-usage.log {
    monthly
    rotate 12
    compress
    missingok
    notifempty
}
```

### Centralized Logging (Future)

**ELK Stack** (Elasticsearch, Logstash, Kibana):
- Aggregate logs from all VMs
- Search and filter logs easily
- Create dashboards
- Set up alerts

**Setup complexity**: High (requires dedicated VM, 4GB+ RAM)
**Benefit**: Powerful log analysis and correlation

## Performance Monitoring

### Baseline Performance

**Establish baselines** (normal operation):
```bash
# CPU baseline
mpstat 1 10 | grep Average

# Disk I/O baseline
iostat -x 1 10 | grep [DEVICE]

# Network baseline
iftop -i vmbr0 -t -s 60

# Document these baselines for comparison
```

### Benchmarking

**Run quarterly to detect performance degradation**:
```bash
# Disk performance
fio --name=seqread --rw=read --bs=1M --size=1G --numjobs=1

# Network performance (between VMs)
iperf3 -s  # On server
iperf3 -c [server-ip]  # On client

# CPU performance
sysbench cpu --threads=4 run

# Record results over time
# Detect performance degradation trends
```

## Monitoring Best Practices

### Do's

- ✓ Monitor what's actionable (don't collect useless metrics)
- ✓ Set up alerts for critical issues only
- ✓ Review logs regularly (weekly at minimum)
- ✓ Establish baselines for normal operation
- ✓ Track trends over time (capacity planning)
- ✓ Test alerting (ensure emails arrive)
- ✓ Document what triggers alerts and how to respond

### Don'ts

- ✗ Alert on everything (leads to alert fatigue)
- ✗ Monitor without acting on data
- ✗ Ignore trends (until capacity crisis)
- ✗ Set up monitoring and forget about it
- ✗ Collect metrics without understanding them

## Future Monitoring Improvements

- [ ] Deploy Grafana dashboard
- [ ] Add Prometheus exporters for all services
- [ ] Implement Uptime Kuma for uptime monitoring
- [ ] Set up centralized logging (ELK or Loki)
- [ ] Add mobile notifications (Pushover, Gotify)
- [ ] Create custom dashboards per service
- [ ] Implement automated remediation (restart failed services)

## Monitoring Checklist

**Weekly review**:
- [ ] Check service health report
- [ ] Review disk space trends
- [ ] Check backup completion
- [ ] Review error logs
- [ ] Verify all VMs running

**Monthly review**:
- [ ] Analyze resource usage trends
- [ ] Review S.M.A.R.T. data for all drives
- [ ] Check ZFS scrub results
- [ ] Review network bandwidth usage
- [ ] Update capacity planning estimates

**Quarterly review**:
- [ ] Run performance benchmarks
- [ ] Review alerting effectiveness
- [ ] Update monitoring scripts
- [ ] Document any infrastructure changes
- [ ] Test disaster recovery procedures

---

**Remember**: Good monitoring prevents surprises. The goal is to detect issues before users (or you) notice them.
