# Hardware Inventory & Infrastructure

Living documentation of the homelab hardware, network topology, and resource allocation.

## Primary Server: deadmall

**System Information**
- **Hostname**: `deadmall`
- **Model**: Dell OptiPlex 5050 (Modified)
- **Role**: Proxmox VE Hypervisor
- **OS**: Proxmox VE 8.4.16 (Kernel 6.8.12-10-pve)
- **Uptime**: > 60 days
- **IP Address**: `192.168.1.29`

**Compute & Graphics**
- **CPU**: Intel Core i5-6500 @ 3.20 GHz (4 Cores / 4 Threads)
- **GPU**: NVIDIA GeForce RTX 3090 (24 GB GDDR6X)
  - *Driver*: NVIDIA Linux Driver (Passthrough enabled)
  - *Purpose*: Local LLM Inference (Ollama), Stable Diffusion, Audio DSP
- **Memory**: 64 GB DDR4 2133 MT/s (4x 16GB Config)

**Storage Configuration**
- **Boot Drive**: 60 GB Generic Flash Storage
- **Fast Storage**: 1 TB Crucial P3 NVMe (VM Boot Disks & Cache)
- **Bulk Storage**:
  - **Active**: 8 TB Seagate BarraCuda (`/dev/sda`) - *Single vDev ZFS Pool*
  - **Cold Spare**: 8 TB Seagate IronWolf (`/dev/sdb`) - *Mounted/Standby*
- **Filesystem**: ZFS (Pool: `bearden`)

**Networking**
- **Interface**: Intel I219-V Gigabit Ethernet
- **Bridge**: `vmbr0` (Standard Linux Bridge)

**Power & Cooling**
- **Power Supply**: Custom Power Delivery [High-Wattage Configuration for GPU]
- **Cooling**: Stock CPU cooler + Custom airflow management

---

## Secondary Devices

### Raspberry Pi (Primary)
- **Model**: Raspberry Pi 5
- **Specs**: 8 GB RAM / 1 TB Storage
- **Hostname**: `[emacs-pi]`
- **IP Address**: `192.168.1.26`
- **Role**: emacsOS / Dedicated Writing Machine
- **Network**: Gigabit Ethernet

### Network Infrastructure
- **Router**: Gateway @ `192.168.1.1`
- **Subnet**: `192.168.1.0/24`
- **DNS**: Pi-hole (Virtualized) -> Cloudflare 1.1.1.1
- **Overlay**: ZeroTier (`10.147.17.x`) for remote administration

---

## Service Inventory & Resource Allocation

### Active Virtual Infrastructure (Proxmox)

| VMID | Service | Type | IP Address | Resource Allocation |
|------|---------|------|------------|---------------------|
| **102** | **Nextcloud** | VM | `192.168.1.19` | 12GB RAM / 1TB Disk |
| **100** | **Plex Media Server** | VM | `192.168.1.40` | 4GB RAM / 32GB Disk |
| **101** | **Pi-hole (DNS)** | VM | `192.168.1.15` | 4GB RAM / 20GB Disk |
| **200** | **Ollama (AI)** | LXC | `192.168.1.50` | Shared Kernel / Host GPU |
| **105** | **Wiki** | LXC | `192.168.1.18` | Shared Kernel |
| **106** | **Forest Creatures** | VM | *DHCP* | 4GB RAM / 64GB Disk |
| **107** | **Home Assistant** | VM | *Internal* | 4GB RAM / 32GB Disk |

### Maintenance / Standby Services

| VMID | Service | Status | Notes |
|------|---------|--------|-------|
| **103** | Mail | Stopped | IP Reserved: `.25` |
| **104** | Desktop-Ubuntu | Stopped | Reserved for GUI Tasks |
| **108** | Urbit | Stopped | |

---

## Architectural Decisions

### Why 64GB RAM?
**Requirement**: Mixed-use Production & Lab environment.
- **Allocation**: ~28GB is permanently reserved for active infrastructure (Nextcloud, Plex, Pi-hole).
- **AI/ML**: The remaining ~36GB provides necessary headroom for loading large models into memory before offloading to the GPU, preventing OOM kills during heavy inference tasks.

### Why Single Drive ZFS?
**Current Strategy**: Capacity over Redundancy.
- The system currently utilizes a single 8TB ZFS vDev to maximize usable space.
- **Risk Mitigation**: Essential configuration and documents are backed up externally.
- **Future Plan**: A matching 8TB IronWolf is physically installed and ready to be added to the pool as a Mirror (RAID-1) vDev when the storage policy changes.

### Why NVIDIA RTX 3090?
**Critical Spec**: 24GB VRAM.
- **Local LLMs**: Enables running unquantized 30B+ parameter models locally via Ollama, which is impossible on consumer cards with 8-12GB VRAM.
- **Stable Diffusion**: Provides the necessary buffer for high-resolution generation and LoRA training.
- **Value**: Offers near-datacenter inference performance for a fraction of the cost of A100/H100 hardware.

### Why Virtualized Pi-hole?
**Migration**: Moved from physical Raspberry Pi to Proxmox VM (ID 101).
- **Benefit**: Faster DNS resolution due to x86 processor speed vs ARM.
- **Reliability**: Eliminates SD card corruption risks associated with physical Raspberry Pis.
- **Snapshots**: Allows for instant rollback before applying blocklist updates.
<!-- Local Variables: -->
<!-- gptel-model: claude-haiku-4-5-20251001 -->
<!-- gptel--backend-name: "Claude-Haiku-4.5" -->
<!-- gptel-max-tokens: 6000 -->
<!-- gptel--bounds: nil -->
<!-- End: -->
