# Hardware Inventory

Complete inventory of all hardware in the homelab infrastructure.

## Primary Server: deadmall

### Specifications

**System Information**:
- **Hostname**: deadmall
- **IP Address**: 192.168.1.29
- **Role**: Proxmox hypervisor (primary infrastructure host)
- **OS**: Proxmox VE [Your version]
- **Uptime**: [Your typical uptime]

**CPU**:
- **Model**: [Your CPU model, e.g., Intel Xeon E5-2680 v4]
- **Cores**: [Number of physical cores]
- **Threads**: [Number of threads]
- **Base Clock**: [Clock speed]
- **Features**: VT-x/VT-d for virtualization, [other features]

**Memory**:
- **Total RAM**: 64GB
- **Type**: DDR4
- **Speed**: [Speed, e.g., 2400MHz]
- **Configuration**: [e.g., 4x 16GB DIMMs]
- **ECC**: [Yes/No]

**Storage**:
- **Boot Drive**: [Your boot drive, e.g., 120GB SSD]
- **ZFS Pool**:
  - Drive 1: [Model, size, e.g., 8TB WD Red]
  - Drive 2: [Model, size, e.g., 8TB WD Red]
  - RAID Level: Mirror (RAID-1)
  - Total Capacity: 8TB usable
  - Filesystem: ZFS

**GPU**:
- **Model**: [Your GPU, e.g., NVIDIA GTX 1080]
- **VRAM**: [VRAM amount]
- **Purpose**: Passthrough for Stable Diffusion and Audio Production VMs
- **Driver**: [Driver version on VMs]

**Network**:
- **Ethernet**: [Your NIC, e.g., Intel i219-V Gigabit]
- **Speed**: Gigabit (1000 Mbps)
- **Ports**: [Number of ports]
- **MAC Address**: [Your MAC]

**Power Supply**:
- **Wattage**: [Your PSU wattage]
- **Efficiency**: [80+ rating]

**Cooling**:
- **CPU Cooler**: [Your cooler]
- **Case Fans**: [Number and configuration]

### Hardware Health

**Disk Health** (S.M.A.R.T status):
```bash
# Check with: smartctl -a /dev/sdX

Drive 1: PASSED
Drive 2: PASSED
Last check: [Date]
```

**Temperature Monitoring**:
- CPU: [Typical temperature range]
- GPU: [Typical temperature range]
- Drives: [Typical temperature range]

**Power Consumption**:
- Idle: [Watts]
- Under load: [Watts]
- Average: [Watts]

## Secondary Devices

### Raspberry Pi Hub

**Model**: [Your Pi model, e.g., Raspberry Pi 4 Model B]
- **RAM**: [e.g., 4GB/8GB]
- **Storage**: [microSD size or SSD]
- **Network**: Ethernet (Gigabit)
- **IP Address**: [Your Pi IP]
- **Purpose**: Pi-hole DNS, Home Assistant, lightweight services
- **Power**: [Power supply specs]

### Network Equipment

**Router**:
- **Model**: [Your router model]
- **IP Address**: 192.168.1.1
- **Features**: [NAT, DHCP, firewall, etc.]
- **Wireless**: [Wireless specs if applicable]

**Switch** (if applicable):
- **Model**: [Your switch model]
- **Ports**: [Number of ports]
- **Speed**: Gigabit
- **Features**: [Managed/unmanaged, VLAN support, etc.]

### Client Workstations

**Primary Workstation**:
- **OS**: [macOS/Linux/Windows]
- **CPU**: [CPU model]
- **RAM**: [RAM amount]
- **Storage**: [Storage config]
- **Network**: [Connection type]
- **Purpose**: Daily work, access to homelab services

**Other Devices**:
- [List other devices that connect to your homelab]

### Smart Home Devices

**Automation Hub**:
- Home Assistant on [Device]

**Connected Devices**:
- Smart lights: [Number and types]
- Sensors: [Motion, temperature, etc.]
- Cameras: [CCTV cameras]
- Other: [Additional smart home devices]

## Hardware Expansion History

| Date | Change | Reason | Cost |
|------|--------|--------|------|
| [Date] | Added 64GB RAM | Support more VMs | $[Cost] |
| [Date] | Added [GPU] | Stable Diffusion and audio DSP | $[Cost] |
| [Date] | Added 2x 8TB drives | ZFS mirrored storage | $[Cost] |
| [Date] | [Other changes] | [Reason] | $[Cost] |

## Hardware Decisions

### Why 64GB RAM?

**Requirement**: Running multiple VMs simultaneously
- Nextcloud: 16GB
- Audio Production: 24GB
- Ollama: [XGB]
- Stable Diffusion: [XGB]
- Dev environments: 8-16GB
- Host overhead: 8GB

**Total needed**: ~56GB minimum, 64GB provides headroom

### Why ZFS Mirror (RAID-1)?

**vs RAID-Z**:
- Easier expansion (add mirror pairs)
- Faster resilver times
- Better random I/O performance
- Acceptable space overhead (50%)

**vs RAID-0**:
- Data protection (1 drive can fail)
- Critical for creative work

### Why GPU Passthrough?

**Use cases**:
- Stable Diffusion inference (requires GPU)
- Audio production DSP acceleration
- Video transcoding
- Future ML workloads

**Alternative considered**: CPU-only (too slow for image generation)

### Why Raspberry Pi for Pi-hole?

**Advantages**:
- Low power consumption (runs 24/7)
- Dedicated device (no VM overhead)
- Easy to maintain
- Cheap and reliable

## Future Hardware Plans

### Short-term (Next 6 months)

- [ ] [Planned hardware additions]
- [ ] [Upgrades to existing hardware]

### Long-term (1-2 years)

- [ ] Storage expansion: Add 2x 16TB drives (second mirror)
- [ ] RAM upgrade: [If needed, to 128GB]
- [ ] [Network upgrade: 10Gb Ethernet]
- [ ] [UPS for power protection]
- [ ] [Additional GPU for ML workloads]

## Hardware Procurement

### Vendors

**Primary sources**:
- [Where you buy hardware, e.g., Newegg, Amazon, eBay]
- Used/refurb market for server hardware
- Local stores for immediate needs

### Budget

**Annual hardware budget**: $[Amount]
**Priority allocation**:
1. Storage expansion (data growth)
2. RAM (VM capacity)
3. Backup solutions (external drives)
4. Network improvements

## Hardware Maintenance

### Cleaning Schedule

- **Dust cleaning**: Every 3 months
- **Cable management**: As needed
- **Fan inspection**: Every 6 months

### Monitoring

```bash
# CPU temperature
sensors

# Disk health
smartctl -a /dev/sdX

# RAM testing
# Run memtest86+ annually

# GPU temperature
nvidia-smi  # On VMs with passthrough
```

### Replacement Policy

**Drives**:
- Replace when S.M.A.R.T. warnings appear
- Proactive replacement at 5 years
- Keep spare drive on hand

**Fans**:
- Replace when noisy or failing
- Cheap and easy to replace

**RAM**:
- Test if system instability occurs
- Replace faulty modules immediately

**Other components**:
- Monitor and replace as needed
- Keep critical spares (PSU, cables)

## Power & Cooling

### Power Management

**UPS** (if installed):
- Model: [Your UPS model]
- Capacity: [VA rating]
- Runtime: [Estimated runtime]
- Connected devices: [What's on UPS]

**Power consumption**:
- deadmall server: [Watts]
- Raspberry Pi: ~5W
- Network equipment: [Watts]
- Total: [Watts]
- Monthly cost (at $[rate]/kWh): $[amount]

### Cooling

**Ambient temperature**: [Your room temp]
**Case airflow**: [Your airflow configuration]
**Monitoring**: Temperature sensors on critical components

## Warranty & Support

| Component | Purchase Date | Warranty Until | Notes |
|-----------|---------------|----------------|-------|
| [Component] | [Date] | [Date] | [Extended warranty, etc.] |
| [Component] | [Date] | [Date] | [Notes] |

## Decommissioned Hardware

Document hardware you've replaced or retired:

| Hardware | Decommission Date | Reason | Disposition |
|----------|------------------|---------|-------------|
| [Old component] | [Date] | [Reason] | [Sold/recycled/spare] |

## Hardware Documentation

### Manuals & Documentation

- [Links to manufacturer documentation]
- [Driver downloads]
- [BIOS/firmware updates]

### Configuration Backups

- BIOS settings: [Documented or screenshot]
- Network config: [Backed up]
- RAID config: [Documented]
