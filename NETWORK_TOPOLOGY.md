# Network Topology

Complete network architecture documentation for the homelab infrastructure.

## Network Overview

- **Subnet**: `192.168.1.0/24`
- **Gateway**: `192.168.1.1`
- **DNS**: Pi-hole (`192.168.1.15`) → Cloudflare (`1.1.1.1`)
- **Domain**: `.bear` (Local), `[External Domain]` (Public)
- **Overlay Network**: ZeroTier (`10.147.17.x`)

## Network Diagram
```
                         Internet
                             ↓
                    [ISP Modem/Router]
                      192.168.1.1
                             ↓
          ┌──────────────────┼──────────────────┐
          ↓                  ↓                  ↓
    [Proxmox Host]    [Raspberry Pi]    [Other Devices]
     192.168.1.29
          ↓
    ┌─────┴─────┬─────────┬──────────┬──────────┐
    ↓           ↓         ↓          ↓          ↓
[emacsOS]  [Nextcloud] [Plex]   [Ollama]  [Stable Diffusion]
              .19


          Tailscale VPN Overlay Network
    ┌──────────────────────────────────────┐
    │  100.x.x.x mesh network              │
    │  - Proxmox host                      │
    │  - Mobile devices                    │
    │  - Remote access                     │
    │  - Exit node capability              │
    └──────────────────────────────────────┘
```

## IP Address Allocation

### Physical Infrastructure

| Device | Hostname | IP Address | Interface | Purpose |
|--------|----------|------------|-----------|---------|
| **Router** | `gateway` | `192.168.1.1` | LAN | Internet Gateway / Firewall |
| **Proxmox Host** | `deadmall` | `192.168.1.29` | vmbr0 | Hypervisor / ZFS Storage |
| **Raspberry Pi 5** | `emacs-pi` | `192.168.1.26` | eth0 | Dedicated Writing Station |
| **Workstation** | `main-pc` | `192.168.1.38` | eth0 | Audio Production / General |

### Virtual Infrastructure (Active)

| VMID | Service | Hostname | IP Address | Resource Allocation |
|------|---------|----------|------------|---------------------|
| **101** | **Pi-hole** | `pihole.bear` | `192.168.1.15` | 4GB RAM / 20GB Disk |
| **102** | **Nextcloud** | `cloud.bear` | `192.168.1.19` | 12GB RAM / 1TB Disk |
| **100** | **Plex** | `plex.bear` | `192.168.1.40` | 4GB RAM / 32GB Disk |
| **200** | **Ollama** | `ollama.bear` | `192.168.1.50` | CT / Host GPU |
| **105** | **Wiki** | `wiki.bear` | `192.168.1.18` | CT / Shared |
| **106** | **Forest Creatures** | `forest.bear` | *DHCP* | 4GB RAM / 64GB Disk |
| **107** | **Home Assistant** | `hass.bear` | *Internal* | 4GB RAM / 32GB Disk |

### Overlay Network (ZeroTier)

| Device | Managed IP | Status |
|--------|------------|--------|
| **Proxmox Host** | `10.147.17.50` | Active |
| **Remote Client** | `10.147.17.15` | Active |
| **Mobile** | `10.147.17.x` | Active |

## DNS Configuration

### Local DNS Strategy
1. **Primary**: Pi-hole (`192.168.1.15`) handles all internal `.bear` requests.
2. **Filtering**: Blocks ads and telemetry at the network level.
3. **Upstream**: Unmatched requests forwarded to Cloudflare (`1.1.1.1`).

### Local Records (.bear)
```text
proxmox.bear  → 192.168.1.29
cloud.bear    → 192.168.1.19
plex.bear     → 192.168.1.40
wiki.bear     → 192.168.1.18
ollama.bear   → 192.168.1.50
### Local DNS (.bear domain)

**Pi-hole local records**:
```
192.168.1.29  deadmall.bear proxmox.bear
192.168.1.19  cloud.bear nextcloud.bear
192.168.1.X   plex.bear
192.168.1.X   ollama.bear ai.bear
192.168.1.X   sd.bear stablediffusion.bear
192.168.1.X   emacs.bear dev.bear
192.168.1.X   audio.bear daw.bear
192.168.1.X   pi.bear pihole.bear
```

### DNS Resolution Flow

```
Client → Pi-hole (192.168.1.X:53)
    ↓
Local .bear domain? → Return local IP
    ↓
Blocked domain? → Return NXDOMAIN
    ↓
Forward to upstream DNS (8.8.8.8, 1.1.1.1)
```

### External DNS (if applicable)

- **Domain**: [Your external domain if you have one]
- **DNS Provider**: [Your DNS provider]
- **Records**:
  - A record: [Your domain] → [Your public IP]
  - CNAME: cloud.[domain] → [Your domain]
  - [Other records]

## Network Services

### DHCP Server

**Primary DHCP**: Router (192.168.1.1) or Pi-hole

**Configuration**:
```
Subnet: 192.168.1.0/24
Range: 192.168.1.51 - 192.168.1.100
Gateway: 192.168.1.1
DNS: 192.168.1.X (Pi-hole)
Lease time: 24 hours
```

**Static DHCP Reservations**:
- All VMs have static IPs (configured in Netplan)
- Raspberry Pi has static IP
- Other critical devices have DHCP reservations

### Firewall Rules

**Router firewall**:
- Default deny all incoming from Internet
- Allow established/related connections
- Allow outgoing connections
- Port forwarding (if any):
  - [Port] → [Internal IP:Port] ([Service])

**Proxmox firewall** (per-VM):
```
# Example for Nextcloud VM
IN ACCEPT -p tcp -dport 22   # SSH
IN ACCEPT -p tcp -dport 80   # HTTP
IN ACCEPT -p tcp -dport 443  # HTTPS
IN DROP                       # Drop all other incoming
```

### NAT Configuration

**Outbound NAT**:
- All internal devices (192.168.1.0/24) NAT to public IP
- Configured on router

**Port Forwarding** (if used):
- Minimize external exposure
- Use Tailscale VPN instead of port forwarding when possible

## VPN Configuration

### Tailscale Mesh Network

**Devices on Tailscale**:
- Proxmox host (deadmall)
- Mobile devices (phones, tablets)
- Laptops when remote
- [Other devices]

**Tailscale IPs** (100.x.x.x range):
| Device | Tailscale IP | Purpose |
|--------|--------------|---------|
| deadmall | 100.x.x.x | Exit node, homelab access |
| Phone | 100.x.x.x | Remote access |
| Laptop | 100.x.x.x | Remote access |

**Exit Node Configuration**:
- Proxmox host advertises as exit node
- Route all traffic through homelab when away
- Access to entire 192.168.1.0/24 network

**MagicDNS**:
- Enabled for easy hostname resolution
- Access devices by name (e.g., deadmall.tail-scale.net)

## Network Performance

### Bandwidth

**Internet Connection**:
- Download: [Your speed, e.g., 500 Mbps]
- Upload: [Your speed, e.g., 50 Mbps]
- Latency: [Your ping, e.g., ~10ms]
- Provider: [Your ISP]

**Internal Network**:
- LAN: Gigabit (1000 Mbps)
- VM to VM: Virtual bridge (no bottleneck)
- Storage: Direct ZFS access (fast)

### Monitoring

**Tools**:
- iftop: Real-time bandwidth monitoring
- vnstat: Historical bandwidth usage
- Pi-hole: DNS query statistics

**Typical bandwidth usage**:
- Nextcloud sync: [Amount]
- Plex streaming: [Amount]
- Backups: [Amount]
- General web: [Amount]

## Network Security

### Security Measures

**Defense in Depth**:
1. **Perimeter**: Router firewall (block incoming)
2. **Network**: Segmentation via VLANs (if configured)
3. **Host**: Proxmox firewall per VM
4. **Application**: Service-level authentication 

**Access Control**:
- No direct Internet exposure (use Tailscale)
- Strong passwords on all services
- SSH key authentication (no passwords)
- Regular security updates

**Monitoring**:
- Pi-hole query logs
- Proxmox firewall logs
- Failed authentication attempts
- Unusual network activity

### VLANs (if configured)

| VLAN ID | Subnet | Purpose |
|---------|--------|---------|
| 1 | 192.168.1.0/24 | Default / main network |
| 10 | 192.168.10.0/24 | IoT devices (isolated) |
| 20 | 192.168.20.0/24 | Guest network |

## Troubleshooting

### Common Network Issues

**VM can't reach Internet**:
```bash
# On VM
ping 192.168.1.1  # Test gateway
ping 8.8.8.8      # Test Internet
ping google.com   # Test DNS

# Check DNS
cat /etc/resolv.conf

# Check routing
ip route
```

**Slow network performance**:
```bash
# Test internal bandwidth
iperf3 -s  # On server VM
iperf3 -c [SERVER-IP]  # On client

# Check for packet loss
mtr google.com

# Check interface errors
ip -s link
```

**DNS not resolving**:
```bash
# Test DNS directly
nslookup cloud.bear 192.168.1.X

# Check Pi-hole status
pihole status

# Verify DNS in resolv.conf
cat /etc/resolv.conf
```

### Diagnostic Commands

```bash
# Network connectivity
ping -c 4 [IP]

# Trace route
traceroute [IP/hostname]

# Port testing
nc -zv [IP] [PORT]
nmap -p [PORT] [IP]

# Active connections
ss -tunap
netstat -an

# Bandwidth monitoring
iftop -i [INTERFACE]
nload

# DNS lookup
dig cloud.bear
nslookup cloud.bear
```


## Future Network Plans

- [ ] New router
- [ ] IPv6 deployment

<!-- Local Variables: -->
<!-- gptel-model: claude-haiku-4-5-20251001 -->
<!-- gptel--backend-name: "Claude-Haiku-4.5" -->
<!-- gptel-max-tokens: 6000 -->
<!-- gptel--bounds: nil -->
<!-- End: -->
