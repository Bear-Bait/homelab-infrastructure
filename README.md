# Homelab Infrastructure

Complete documentation of self-managed infrastructure supporting creative workloads (audio/video production, writing projects, development, AI/ML workloads). Features a 64GB virtualization server, ZFS storage, automated backups, and local AI inference.

## Quick Stats

- **Hypervisor**: Proxmox VE 8.4 (Host: `deadmall` @ 192.168.1.29)
- **Compute**: Intel Core i5-6500 + NVIDIA RTX 3090 (Offsite)
- **RAM**: 64GB DDR4 (Production & Lab Allocation)
- **Storage**: 8TB ZFS Pool (Single vDev) + 8TB Cold Spare
- **Services**: 10+ active production workloads
- **Backup**: BorgBackup (Encrypted, Deduplicated)
- **AI/ML**: Ollama (Local LLM), Stable Diffusion (Image Gen)
- **Baseline Uptime**: > 60 days

## Architecture at a Glance

All core services (DNS, Storage, Automation) are virtualized on Proxmox for reliability and snapshot capability. The Raspberry Pi serves as a dedicated writing workstation.
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
    ├─ Pi-hole DNS └─EmacsOS        ↓
    ├─ Nextcloud   └─ Volumio   Climate control
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
- Enterprise-grade hypervisor (KVM/QEMU)
- Web-based management & automated backups
- GPU Passthrough enabled for AI & Audio workloads
- See: [proxmox-virtualization](../proxmox-virtualization)

**ZFS Storage**
- **Pool**: `bearden` (8TB Single vDev)
- **Features**: Compression (LZ4), Deduplication, Snapshots
- **Strategy**: Capacity priority with external backups (Cold spare on site)

### AI & Machine Learning

**Ollama (Local Inference)**
- **Hardware**: Offloaded to RTX 3090 (24GB VRAM)
- **Models**: qwen2.5-coder:32b, llama3.2:3b, qwen2.5:14b
- **Integration**: API compatible with OpenAI; integrated into Home Assistant
- **Privacy**: 100% local processing (No data leakage)

**Stable Diffusion**
- Local image generation server
- Custom LoRA training and high-res output
- Powered by dedicated GPU passthrough

### Application Services

**Nextcloud** (`192.168.1.19`)
- 1TB Private Cloud Storage
- Calendar, Contacts, and Note synchronization
- Replaces Dropbox/Google Drive

**Plex Media Server** (`192.168.1.40`)
- 4K Video Transcoding
- Remote streaming via Plex.tv
- Hardware acceleration support

**Pi-hole** (`192.168.1.15`)
- Network-wide Ad & Tracker blocking
- Local DNS Authority (`.bear` domain)
- DHCP Server

**Home Assistant**
- Centralized home automation
- "Claude MCP" integration for AI-native control
- Controls lighting, climate, and security sensors

### Specialized Workstations

**emacsOS (Raspberry Pi 5)**
- Dedicated writing and development machine
- Distraction-free environment
- Synchronized via Syncthing/Nextcloud

## Network Topology

See [NETWORK_TOPOLOGY.md](NETWORK_TOPOLOGY.md) for the complete diagram.

**Key IP Assignments**:
- **Gateway**: `192.168.1.1`
- **Proxmox Host**: `192.168.1.29`
- **Ollama AI**: `192.168.1.50`
- **Nextcloud**: `192.168.1.19`
- **Pi-hole**: `192.168.1.15`
- **Plex**: `192.168.1.40`
- **Wiki**: `192.168.1.18`
- **Raspberry Pi (Emacs)**: `192.168.1.26`

**Local DNS (.bear)**:
- `proxmox.bear` → Host Management
- `cloud.bear` → Nextcloud
- `ollama.bear` → AI API
- `wiki.bear` → Documentation

## Hardware Inventory

### Primary Server (deadmall)
- **CPU**: Intel Core i5-6500 (4C/4T @ 3.2GHz)
- **RAM**: 64GB DDR4
- **GPU**: NVIDIA RTX 3090 (24GB)
- **Storage**: 1TB NVMe (System) + 8TB HDD (Data)
- **Role**: Hypervisor for all production services

### Secondary Devices
- **Raspberry Pi 5**: 8GB RAM / 1TB Storage (Writing Station)
- **Workstation**: Custom PC (Audio/Video Production)

## Key Technical Decisions

### Why Proxmox over ESXi?
- **GPU Passthrough**: Superior handling of consumer GPU passthrough (IOMMU) compared to VMware.
- **ZFS Integration**: Native support for ZFS without needing a separate storage controller VM.
- **Open Source**: No licensing fees or artificial limitations on CPU cores/RAM.

### Why Single-Drive ZFS?
- **Decision**: Prioritized capacity (8TB) over redundancy (4TB Mirror) given current budget.
- **Mitigation**: Critical data is backed up nightly to external encryption targets. A cold spare drive is available on-site for immediate restoration if the primary drive fails.

### Why Local AI?
- **Privacy**: Sensitive creative writing and personal data never leave the network.
- **Cost**: Eliminates subscription fees for API access.
- **Speed**: The RTX 3090 offers minimal latency for real-time assistance.

## Documentation Structure

- **[HARDWARE_INVENTORY.md](HARDWARE_INVENTORY.md)** - Detailed specs (CPU, RAM, Drives)
- **[NETWORK_TOPOLOGY.md](NETWORK_TOPOLOGY.md)** - IP Map and VLANs
- **[SERVICES_DIRECTORY.md](SERVICES_DIRECTORY.md)** - Resource allocation per VM
- **[MAINTENANCE_SCHEDULE.md](MAINTENANCE_SCHEDULE.md)** - Patching and Backup logs

## Recent Accomplishments

- **Infrastructure**: Deployed RTX 3090 with stable PCI-Passthrough for AI workloads.
- **Virtualization**: Migrated physical Raspberry Pi services (Pi-hole, HA) to Proxmox VMs for better reliability.
- **Automation**: Implemented "Claude MCP" for natural language control of home automation.

<!-- Local Variables: -->
<!-- gptel-model: claude-haiku-4-5-20251001 -->
<!-- gptel--backend-name: "Claude-Haiku-4.5" -->
<!-- gptel-max-tokens: 6000 -->
<!-- gptel--bounds: nil -->
<!-- End: -->
