# Homelab Infrastructure

Complete documentation of self-managed infrastructure supporting creative workloads (audio/video production, writing projects, development, AI/ML workloads). 64GB virtualization server, 8TB+ storage, automated backups, private cloud storage, home automation, and local AI models.

## Quick Stats

- **Hypervisor**: Proxmox (deadmall at 192.168.1.29)
- **RAM**: 64GB (virtualized across multiple VMs)
- **Storage**: 8TB+ mirrored ZFS pool
- **Services**: 15+ running production workloads
- **Backup**: BorgBackup with 6-month retention
- **Network**: 192.168.1.0/24 with local .bear domain
- **AI/ML**: Ollama for local LLM inference, Stable Diffusion for image generation
- ### Server Performance Baseline
* **Uptime:** 60 days, 20 minutes
* **Active Users:** 3
* **Load Average:** 4.62 (1 min) / 4.74 (5 min) / 3.99 (15 min)

## Architecture at a Glance

All services run on Proxmox with automated backups to ZFS storage. Remote access via Tailscale VPN.

```
                    Internet
                        ↓
                    Firewall/Router
                        ↓
            Local Network (192.168.1.0/24)
                        ↓
    ┌───────────────────┼───────────────────┐
    ↓                   ↓                   ↓
  Proxmox      Raspberry Pi Hub     Home Automation
(Hypervisor)  (Portal, DNS, Audio)   (Light control)
    ├─ emacsOS     └─ Pi-hole DNS       ↓
    ├─ Nextcloud   └─ Home Assistant   Climate control
    ├─ Plex        └─ Tailscale        Motion detection
    ├─ Ollama                           CCTV integration
    ├─ Stable Diffusion
    └─ ...
         ↓
    ┌────────────────────┐
    │  ZFS Storage Pool  │
    │  8TB mirrored      │
    │  (deadmall)        │
    └────────────────────┘
         ↑
    BorgBackup (encrypted, deduplicated)
```

## Core Services

### Virtualization & Storage

**Proxmox VE**
- Enterprise-grade hypervisor
- KVM/QEMU virtualization
- Web-based management
- VM backup and snapshot capabilities
- See: [proxmox-virtualization](../proxmox-virtualization)

**ZFS Storage**
- 8TB+ mirrored pool (RAID-1)
- Data integrity with checksums
- Snapshot capabilities
- Deduplication and compression
- Backend for Proxmox VMs and services

**BorgBackup**
- Automated encrypted backups
- 6-month retention policy
- Deduplication at file level
- Tested disaster recovery procedures

### Cloud & File Services

**Nextcloud** (192.168.1.19)
- Private cloud storage and collaboration
- 10TB+ capacity
- File sync across devices
- Calendar, contacts, notes
- Mobile app support

**Plex Media Server**
- Personal media streaming
- 4K video transcoding
- Remote access via Plex.tv
- Library management and metadata

**Transmission**
- BitTorrent client
- Web UI for remote management
- Automatic organization

### AI & Machine Learning

**Ollama**
- Local LLM inference server
- Running models: wen2.5-coder:32b, qwen2.5:14b, llama3.2:3b
- Privacy-focused (all processing local)
- API compatible with OpenAI format
- Integration with Home Assistant via Claude MCP
- Use cases: Local AI assistance, automation, creative writing

**Stable Diffusion**
- Local image generation
- GPU-accelerated inference
- Privacy-focused (no cloud dependencies)
- Custom model support
- Use cases: Creative projects, concept art, asset generation

### Development & Automation

**emacsOS VM**
- Primary development workstation
- Emacs-based workflow
- Development tools and environments
- Git repositories

**Home Assistant**
- Home automation platform
- Claude MCP integration for AI-native control
- Device control and monitoring
- Automation workflows
- Voice assistant integration

### Network Services

**Pi-hole**
- Network-wide ad blocking
- DNS sinkhole
- DHCP server
- Local DNS (.bear domain)

**Tailscale VPN**
- WireGuard-based mesh VPN
- Secure remote access
- Cross-platform support
- Zero-trust networking
- Replaced ZeroTier for better reliability

### Audio Production

**Audio Production VM**
- Dedicated VM for DAW workloads
- GPU passthrough for DSP acceleration
- 24GB+ RAM allocation
- CPU pinning for low-latency performance
- Professional audio interface passthrough

## Network Topology

See [NETWORK_TOPOLOGY.md](NETWORK_TOPOLOGY.md) for detailed network diagram and IP assignments.

**IP Assignments**:
  - Router/Gateway: 192.168.1.1
  - Proxmox Host (deadmall): 192.168.1.29
  - Transmission: 192.168.1.29:9091
  - Plex Media: 192.168.1.40:32400
  - Printer: 192.168.1.41:631
  - Raspberry Pi Hub: 10.147.17.15
  - Calendar Submit: 10.147.17.15
  - Destiny's Chores: 10.147.17.15
  - Herb Garden Tracker: 10.147.17.15:3000
  - Nextcloud: cloud.bear
  - Bear Wiki: wiki.bear
  - Ollama AI: ollama.bear:11434
  - Family Chat App: 10.147.17.139:5000/

**DNS (.bear domain)**:
  - cloud.bear → Nextcloud
  - deadmall.bear → Proxmox
  - home.bear → Home Dashboard
  - mail.bear → Mail Server
  - office.bear → Collabora Online
  - ollama.bear → Ollama AI

wiki.bear → MediaWiki
## Hardware Inventory

### Primary Server (deadmall)

- **CPU**: 15
- **RAM**: 64GB DDR4
- **Storage**: 8TB+ ZFS mirrored pool
- **Network**: Gigabit Ethernet
- **GPU**: RTX 3090
- **Role**: Proxmox hypervisor, all VMs

### Raspberry Pi Hub

- **Model**: 
- **Purpose**: Pi-hole DNS, Home Assistant, lightweight services, gallery video looper 
- **Network**: Ethernet connected

### Client Devices

- **Workstations**: EmacsOS
- **Mobile**: Tailscale for remote access
- **Smart home**: Various IoT devices

## What This Infrastructure Proves

✓ **Production-grade thinking**: Proxmox, ZFS, automated backups show enterprise-level infrastructure
✓ **Reliability focus**: Mirrored storage, tested disaster recovery, monitoring
✓ **Documentation discipline**: Comprehensive docs for maintenance and troubleshooting
✓ **Problem-solving**: Documented technical decisions and tradeoffs
✓ **Network security**: VPN access, firewall configuration, segmentation
✓ **Systems thinking**: Multiple integrated components working together
✓ **Self-directed learning**: Built and maintained without external pressure
✓ **Technical depth**: Specific, practical knowledge across multiple domains
✓ **Modern tech stack**: AI/ML integration, automation, creative workflows

## Key Technical Decisions

### Why Proxmox over ESXi/Hyper-V?
- Cost-effective (free, open source)
- Linux-first design
- Excellent community support
- Perfect for homelab scale

### Why ZFS for storage?
- Data integrity guarantees (critical for creative work)
- Snapshot capabilities
- Deduplication reduces storage overhead
- Mature and reliable

### Why Tailscale over ZeroTier?
- Better mobile reliability (especially Android)
- WireGuard-based (industry standard)
- Simpler setup and management
- Superior performance

### Why local AI (Ollama/Stable Diffusion)?
- Privacy: All data stays local
- Cost: No API fees or usage limits
- Control: Custom models and fine-tuning
- Performance: GPU acceleration for faster inference
- Reliability: No dependency on external services

## Documentation Structure

This repository provides the overview. For detailed documentation:

- **[HARDWARE_INVENTORY.md](HARDWARE_INVENTORY.md)** - Complete hardware specs and roles
- **[NETWORK_TOPOLOGY.md](NETWORK_TOPOLOGY.md)** - Network diagram and IP assignments
- **[SERVICES_DIRECTORY.md](SERVICES_DIRECTORY.md)** - All running services and their purposes
- **[SECURITY.md](SECURITY.md)** - Security practices and access control
- **[MONITORING.md](MONITORING.md)** - Health checks and observability
- **[MAINTENANCE_SCHEDULE.md](MAINTENANCE_SCHEDULE.md)** - Regular maintenance tasks
- **[DISASTER_SCENARIOS.md](DISASTER_SCENARIOS.md)** - Response plans for failures
- **[LESSONS_LEARNED.md](LESSONS_LEARNED.md)** - What worked and what didn't

## Related Repositories

- **[proxmox-virtualization](../proxmox-virtualization)** - Detailed Proxmox setup and VM management

## Recent Accomplishments

- Akamai Network Engineering Certification (Dec 2024)
- GPU server infrastructure deployment for audio/video production
- Automated backup system design with comprehensive testing
- Network troubleshooting and stability improvements
- Creative workload optimization (audio DAW infrastructure)
- Local AI/ML deployment (Ollama + Stable Diffusion)
- Home Assistant + Claude MCP integration
