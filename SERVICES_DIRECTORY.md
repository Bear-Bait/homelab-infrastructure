# Services Directory

Complete inventory of all services running in the homelab infrastructure.

## Core Infrastructure Services

### Proxmox VE
- **Type**: Hypervisor
- **Location**: deadmall (192.168.1.29)
- **Port**: 8006 (HTTPS web UI)
- **Purpose**: Virtualization platform managing all VMs
- **Access**: Web UI, SSH
- **Documentation**: [proxmox-virtualization](../proxmox-virtualization)

### ZFS Storage
- **Type**: Filesystem/Storage
- **Location**: Proxmox host (deadmall)
- **Capacity**: 8TB+ mirrored
- **Purpose**: Backend storage for VMs, backups, media
- **Features**: Checksums, snapshots, deduplication, compression
- **Monitoring**: zpool status, automated scrubs

### BorgBackup
- **Type**: Backup system
- **Location**: Runs from client → deadmall NAS
- **Capacity**: ~550GB (deduplicated from 500GB source)
- **Purpose**: Encrypted, deduplicated backups
- **Retention**: 7 daily, 4 weekly, 6 monthly
- **Schedule**: Daily at 3 AM

## Cloud & File Services

### Nextcloud
- **Type**: Private cloud storage
- **Location**: VM on Proxmox (192.168.1.19)
- **Port**: 80 (HTTP), 443 (HTTPS)
- **Capacity**: 10TB+
- **Purpose**: File sync, calendar, contacts, notes, collaboration
- **Access**: Web UI, desktop client, mobile apps
- **URL**: cloud.bear (internal), [external domain if applicable]
- **Users**: [Number of users]

### Plex Media Server
- **Type**: Media streaming
- **Location**: VM on Proxmox
- **Port**: 32400 (HTTPS)
- **Purpose**: Personal media library and streaming
- **Content**: Movies, TV shows, music
- **Features**: 4K transcoding, remote access, mobile apps
- **Access**: plex.tv, local network

### Transmission
- **Type**: BitTorrent client
- **Location**: VM on Proxmox
- **Port**: 9091 (web UI)
- **Purpose**: Torrent downloads
- **Features**: Web UI, automatic organization
- **Storage**: Downloads to ZFS pool

## AI & Machine Learning

### Ollama
- **Type**: Local LLM inference server
- **Location**: VM on Proxmox (or bare metal)
- **Port**: 11434 (API)
- **Models Running**:
  - [Your specific models, e.g., llama3:70b]
  - [mistral:7b]
  - [codellama:13b]
  - [Add your models]
- **Purpose**:
  - Local AI assistance for development
  - Creative writing and brainstorming
  - Code generation and review
  - Integration with Home Assistant via Claude MCP
- **API**: OpenAI-compatible REST API
- **Resources**: [vCPU/RAM allocation]
- **Storage**: Models stored on ZFS (~10-50GB per model)
- **Privacy**: All inference happens locally, no data sent to cloud

### Stable Diffusion
- **Type**: AI image generation
- **Location**: VM on Proxmox with GPU passthrough
- **Port**: [Your port, typically 7860 for WebUI]
- **Interface**: AUTOMATIC1111 WebUI or ComfyUI
- **Purpose**:
  - Creative image generation
  - Concept art and design
  - Asset creation for projects
  - Experimentation with AI art
- **Models**: [Your installed models, e.g., SD 1.5, SDXL]
- **GPU**: [Your passthrough GPU model]
- **Resources**:
  - GPU: [Your GPU]
  - RAM: [Your RAM allocation, e.g., 16GB+]
  - Storage: [Model storage, typically 50-200GB]
- **Features**: Txt2img, img2img, inpainting, LoRA support
- **Privacy**: All generation happens locally

## Development & Automation

### emacsOS VM
- **Type**: Development workstation
- **Location**: VM on Proxmox
- **Purpose**: Primary development environment
- **Tools**: Emacs, Git, development toolchains
- **Resources**: [vCPU/RAM allocation]
- **Storage**: Code repositories, projects

### Home Assistant
- **Type**: Home automation platform
- **Location**: Raspberry Pi or VM
- **Port**: 8123 (web UI)
- **Purpose**:
  - Smart home device control
  - Automation workflows
  - Monitoring and alerts
  - Claude MCP integration for AI-native control
- **Integrations**:
  - Lighting control
  - Climate control
  - Motion sensors
  - CCTV integration
  - Ollama integration via Claude MCP
- **Access**: Web UI, mobile app

## Network Services

### Pi-hole
- **Type**: DNS sinkhole / ad blocker
- **Location**: Raspberry Pi
- **IP**: [Your Pi-hole IP]
- **Port**: 80 (web UI), 53 (DNS)
- **Purpose**:
  - Network-wide ad blocking
  - DNS server for .bear domain
  - DHCP server (optional)
  - Network monitoring
- **Blocking**: [Your block percentage, e.g., ~30% of queries]
- **Upstream DNS**: 8.8.8.8, 1.1.1.1

### Tailscale VPN
- **Type**: Mesh VPN
- **Location**: Installed on multiple devices
- **Purpose**:
  - Secure remote access to homelab
  - Connect devices across networks
  - Mobile access to internal services
- **Protocol**: WireGuard
- **Features**:
  - Exit node capability
  - MagicDNS for easy hostname resolution
  - ACLs for access control
  - Cross-platform (Linux, macOS, iOS, Android)

## Audio Production

### Audio Production VM
- **Type**: Digital Audio Workstation environment
- **Location**: VM on Proxmox with GPU passthrough
- **Purpose**:
  - Music production
  - Audio editing and mixing
  - Plugin hosting
  - Sample library management
- **Resources**:
  - vCPU: 8+ (pinned cores)
  - RAM: 24GB+ (ballooning disabled)
  - GPU: Passthrough for DSP acceleration
  - Storage: Large sample libraries
- **Software**: [Your DAW, e.g., Reaper, Bitwig]
- **Audio Interface**: USB passthrough for professional audio I/O

## Monitoring & Management

### Proxmox Monitoring
- **Built-in**: Proxmox web UI statistics
- **Metrics**: CPU, RAM, disk I/O, network
- **Alerts**: Email notifications for critical events

### ZFS Monitoring
- **Health checks**: zpool status
- **Scrubs**: Monthly automated scrubs
- **Space monitoring**: zfs list, automated alerts
- **S.M.A.R.T**: Disk health monitoring

### System Logs
- **Location**: /var/log on each system
- **Centralized**: [If you have centralized logging]
- **Review**: Regular log analysis for issues

## Service Dependencies

### Critical Path
```
Internet → Router → Proxmox → VMs → Services
                  ↓
                 ZFS Storage (all VMs depend on this)
                  ↓
                 Backups
```

### Service Relationships
- **Nextcloud** depends on: ZFS storage, network, Proxmox
- **Plex** depends on: ZFS storage (media files), network, Proxmox
- **Ollama** depends on: Proxmox, sufficient CPU/RAM
- **Stable Diffusion** depends on: Proxmox, GPU passthrough, ZFS storage
- **Home Assistant** depends on: Network, Pi-hole DNS, Ollama (for MCP)
- **Pi-hole** depends on: Raspberry Pi, network, upstream DNS

## Service Access Matrix

| Service | Internal Access | External Access | Authentication |
|---------|----------------|-----------------|----------------|
| Proxmox | 192.168.1.29:8006 | Tailscale only | PAM/LDAP |
| Nextcloud | cloud.bear | [External domain] | Username/password |
| Plex | Local:32400 | plex.tv | Plex account |
| Ollama | 192.168.1.X:11434 | Tailscale only | API key (optional) |
| Stable Diffusion | 192.168.1.X:7860 | Tailscale only | Optional password |
| Home Assistant | 192.168.1.X:8123 | Tailscale only | Username/password |
| Pi-hole | 192.168.1.X/admin | Tailscale only | Admin password |

## Resource Allocation Summary

| Service | vCPU | RAM | Storage | Priority |
|---------|------|-----|---------|----------|
| Proxmox (host) | - | 8GB reserved | - | Critical |
| Nextcloud | 6 | 16GB | 10TB | High |
| Plex | 4 | 8GB | 500GB | Medium |
| Ollama | [X] | [XGB] | [XGB] | High |
| Stable Diffusion | [X] | [XGB] | [XGB] | Medium |
| Audio Production | 8+ | 24GB+ | 1TB | High |
| emacsOS | 4 | 8GB | 64GB | High |
| Home Assistant | 2 | 4GB | 32GB | Medium |

Total allocated: [Calculate your total]
Available: [Calculate remaining]

## Service Maintenance Schedule

See [MAINTENANCE_SCHEDULE.md](MAINTENANCE_SCHEDULE.md) for detailed schedules.

**Daily**:
- Automated backups (BorgBackup, Proxmox VM backups)

**Weekly**:
- Review service logs
- Check resource usage
- Verify backup completion

**Monthly**:
- ZFS scrub
- Update Proxmox and VMs
- Review and prune old backups
- Test disaster recovery procedures

**Quarterly**:
- Full disaster recovery test
- Security audit
- Capacity planning review
- Hardware health check

## Decommissioned Services

Document services you've tried and retired:

| Service | Reason for Decommission | Date | Replacement |
|---------|------------------------|------|-------------|
| ZeroTier | Poor mobile reliability | [Date] | Tailscale |
| [Other] | [Reason] | [Date] | [Replacement] |

## Future Service Plans

- [Services you plan to add]
- [Upgrades to existing services]
- [Experimental services in testing]
