# Lessons Learned

Documenting what worked, what didn't, and what I'd do differently. This is a living document updated as the infrastructure evolves.

## Major Decisions

### Choosing Proxmox over ESXi

**Decision**: Use Proxmox VE as hypervisor
**Outcome**: ✅ **Great choice**

**What worked**:
- Free and open source (no licensing headaches)
- Excellent community support and documentation
- Web UI is intuitive and feature-rich
- Perfect for homelab scale
- Linux-native (comfortable environment)
- KVM/QEMU under the hood (industry standard)

**What I'd do differently**:
- Nothing - Proxmox was the right choice

**Lesson**: For homelab/small infrastructure, Proxmox hits the sweet spot of features, cost, and ease of use.

---

### ZFS RAID-1 (Mirror) vs RAID-Z

**Decision**: Use 2-disk mirror instead of RAID-Z
**Outcome**: ✅ **Correct for my use case**

**What worked**:
- Easy to expand (add mirror pairs)
- Fast resilver times if drive fails
- Good random I/O performance
- Simple to understand and manage

**Tradeoff accepted**:
- 50% storage overhead vs 33% for RAID-Z1
- Only 8TB usable from 16TB raw

**What I'd do differently**:
- Nothing - mirrors are easier to expand, which matters more than 25% more space

**Lesson**: For homelab, operational simplicity > maximum storage efficiency. The peace of mind from easy expansion is worth the storage overhead.

---

### Tailscale vs ZeroTier

**Decision**: Migrated from ZeroTier to Tailscale
**Outcome**: ✅ **Significant improvement**

**ZeroTier problems**:
- Frequent connection drops on Android
- More complex setup
- Occasional routing issues
- Less reliable mobile experience

**Tailscale benefits**:
- Rock-solid reliability on all platforms
- WireGuard-based (proven protocol)
- Dead simple setup (literally 2 commands)
- Excellent mobile apps
- MagicDNS just works
- Exit node feature is fantastic

**Migration effort**: ~1 day to fully transition

**What I'd do differently**:
- Use Tailscale from day one

**Lesson**: When a critical service (VPN) has reliability issues, migrating to proven alternatives is worth the effort. WireGuard-based solutions are the future.

---

### 64GB RAM vs 32GB

**Decision**: Invest in 64GB RAM
**Outcome**: ✅ **Excellent investment**

**What worked**:
- Enables running multiple heavy VMs simultaneously
- Audio production VM needs 24GB+
- Ollama LLMs benefit from more RAM
- Nextcloud + Plex + dev VMs all comfortable
- Room for future expansion

**Cost**: ~$150-200 (used server RAM)

**What I'd do differently**:
- Nothing - 64GB was the right capacity

**Lesson**: RAM is cheap and enables running sophisticated workloads. Skimping on RAM limits what you can do. Buy more than you think you need.

---

### GPU Passthrough for Audio/Stable Diffusion

**Decision**: Pass through GPU to specific VMs
**Outcome**: ✅ **Game-changer for certain workloads**

**What worked**:
- Stable Diffusion is usable (vs impossibly slow on CPU)
- Audio DSP processing offload works well
- Learning experience with IOMMU and PCI passthrough

**Challenges**:
- Requires UEFI firmware (not BIOS)
- Can't easily switch GPU between VMs
- Need to blacklist GPU on host
- IOMMU grouping can be tricky

**What I'd do differently**:
- Plan GPU allocation before building VMs
- Consider dual GPUs for flexibility (one for host, one for VMs)

**Lesson**: GPU passthrough is powerful but requires upfront planning. Understand IOMMU groups before buying hardware.

---

## Infrastructure Evolution

### What I Built First vs What I Should Have Built First

**Actual order**:
1. Proxmox installation
2. emacsOS dev VM
3. Nextcloud
4. Plex
5. Backup system (should have been #2!)
6. Monitoring

**Better order**:
1. Proxmox installation
2. **Backup system** ← Should be #2!
3. emacsOS dev VM
4. Nextcloud
5. Plex
6. Monitoring

**Lesson**: Implement backups BEFORE putting important data on the system. I was lucky nothing failed before backups were in place.

---

## Technical Mistakes & Fixes

### Mistake: Over-provisioning VM resources

**What happened**:
- Initially allocated max resources to each VM
- Total vCPU allocation exceeded physical cores 4:1
- Some VMs had 16GB RAM but only used 4GB

**Fix**:
- Started conservative (2 vCPU, 4GB RAM)
- Monitor actual usage with htop, free
- Increase only when needed
- Most VMs need way less than you think

**Lesson**: Start small, monitor, scale up based on actual usage. VMs almost always use less than allocated.

---

### Mistake: Not testing restores initially

**What happened**:
- Set up BorgBackup, assumed it worked
- Didn't test restore for 2 months
- Could have lost data if backup was misconfigured

**Fix**:
- Immediately tested single file restore
- Tested full directory restore
- Tested mounting backup as filesystem
- Now test quarterly

**Lesson**: **Backups you haven't tested are not backups.** Test restore immediately after setting up backup system.

---

### Mistake: Using e1000 network driver initially

**What happened**:
- Default VM network adapter was e1000
- Network performance was poor (200 Mbps on gigabit)
- High CPU usage for network I/O

**Fix**:
- Switched to VirtIO network driver
- Required OS reinstall on some VMs
- Now get near-gigabit speeds

**Lesson**: Always use VirtIO for Linux VMs. The performance difference is dramatic.

---

### Mistake: No monitoring initially

**What happened**:
- Ran infrastructure for months "blind"
- Didn't notice disk space issues until 95% full
- Missed failed backups for a week

**Fix**:
- Implemented basic monitoring (scripts + cron)
- Email alerts for critical issues
- Regular log review schedule

**Lesson**: Even basic monitoring prevents surprises. You can't manage what you don't measure.

---

## What Worked Really Well

### BorgBackup deduplication

**Result**: 500GB source data → 550GB with 6 months of daily backups

The deduplication is incredible:
- Only changed blocks are stored
- Compression further reduces size
- Combined with ZFS dedup on backend
- Minimal storage overhead for long retention

**Lesson**: Deduplication isn't just marketing - it really works and enables long retention.

---

### Documentation from the start

**Decision**: Document everything as I built it
**Outcome**: ✅ **Saved countless hours**

When things break (and they do):
- I have exact commands used
- I have reasoning for decisions
- I can troubleshoot faster
- I can replicate successful configurations

**Lesson**: Document NOW, not later. Future-you will thank present-you.

---

### Local AI (Ollama + Stable Diffusion)

**Decision**: Run AI models locally instead of cloud APIs
**Outcome**: ✅ **Privacy, cost, and control benefits**

**What worked**:
- Zero recurring costs (no API fees)
- Complete privacy (data never leaves network)
- Fast inference with local GPU
- Experimentation without usage limits
- Integration with Home Assistant

**Challenges**:
- Requires good GPU
- Model downloads are large (10-50GB)
- More complex than API calls
- Need to manage model versions

**Lesson**: If you have GPU and care about privacy/cost, local AI is absolutely worth it. The control and flexibility are unmatched.

---

## What I'd Do Differently Next Time

### Infrastructure as Code from Day 1

**Current**: Mostly manual VM provisioning
**Better**: Use Terraform/Ansible from the start

**Why**:
- Reproducible infrastructure
- Documentation in code
- Easy to rebuild if disaster
- Version control for infrastructure

**Action**: Gradually migrate to IaC

---

### Proper VLAN segmentation

**Current**: Single flat network
**Better**: VLANs from the start

**Why**:
- IoT devices should be isolated
- Better security through segmentation
- Easier to firewall by network segment

**Action**: Plan VLAN migration (requires network downtime)

---

### UPS from the beginning

**Current**: No UPS (risky)
**Better**: UPS for server and critical network equipment

**Why**:
- Protect against power outages
- Graceful shutdown on extended outage
- Prevent ZFS corruption from power loss

**Action**: Budget for UPS (priority purchase)

---

## Biggest Surprises

### Surprise: How well ZFS works

**Expected**: Storage filesystem
**Reality**: Data integrity guarantees, snapshots, compression, dedup all work flawlessly

ZFS is incredible. The checksumming, self-healing, and snapshot features provide peace of mind.

---

### Surprise: How little RAM most VMs actually need

**Expected**: Need 16GB for web server
**Reality**: Nextcloud with 10 users happily runs on 8GB with headroom

Most guides over-specify resources. Start small, monitor, scale up.

---

### Surprise: How important good cooling is

**Problem**: Initial temps were high (80°C+ CPU under load)
**Fix**: Better case fans, airflow management
**Result**: Now runs 50-60°C under load

Good cooling extends hardware life significantly.

---

## Things I'm Still Learning

### ZFS tuning

- ARC cache sizing
- Recordsize optimization per workload
- When to use dedup vs compression
- L2ARC caching strategies

**Action**: Continue experimenting and documenting results

---

### Network optimization

- Jumbo frames (MTU 9000)
- 10Gb Ethernet benefits
- Advanced firewall rules
- QoS for different services

**Action**: Research and test in isolated environment first

---

### Advanced Proxmox features

- HA clustering (requires 3+ nodes)
- Ceph distributed storage
- Container (LXC) vs VMs
- Advanced networking (OVS)

**Action**: Lab environment to test before production

---

## Advice to My Past Self

1. **Backups first, everything else second** - Test them immediately
2. **Start small, scale up** - Don't over-provision resources
3. **Document everything** - You'll forget why you did things
4. **Use VirtIO drivers** - Performance matters
5. **Test disaster recovery quarterly** - Not when disaster strikes
6. **Invest in good cooling** - Hardware lasts longer
7. **Join communities** - r/homelab, Proxmox forums are goldmines
8. **Automate repetitive tasks** - Your time is valuable
9. **Security from the start** - Firewall, VPN, no port forwarding
10. **Enjoy the learning** - Breaking things teaches more than reading docs

---

## What's Next

### Short-term improvements

- [ ] Implement UPS
- [ ] Add monitoring dashboard (Grafana)
- [ ] Migrate to infrastructure as code
- [ ] VLAN segmentation

### Long-term goals

- [ ] 10Gb Ethernet networking
- [ ] Storage expansion (16TB drives)
- [ ] Second server (HA clustering)
- [ ] Centralized logging (ELK stack)

---

## Conclusion

Building this homelab has been incredibly educational. Every mistake was a learning opportunity. Every success built confidence.

**Most important lesson**: Don't be afraid to break things in a homelab. That's what it's for. Having good backups means you can experiment freely.

The infrastructure I have now is reliable, documented, and maintainable. It started rough but improved iteratively.

**Keep learning. Keep documenting. Keep improving.**
