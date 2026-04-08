# 🚀 Ubuntu VM DevOps Optimization Guide
### Tailored for: Intel i5 12th Gen | Host RAM (varies) | VM RAM: 3.5GB | Ubuntu 25.04 | VMware

---

> **How to use this guide:** Read top to bottom on first setup. Each section has commands you can copy-paste directly into your terminal. Sections marked ⚠️ include risk notes — read them before applying.

---

## Table of Contents

1. [VMware Optimization](#1-vmware-optimization)
2. [Ubuntu OS Optimization](#2-ubuntu-os-optimization)
3. [Memory Optimization (ZRAM / Swap / zswap)](#3-memory-optimization)
4. [System & Kernel Tuning](#4-system--kernel-tuning)
5. [Disk & I/O Optimization](#5-disk--io-optimization)
6. [Dev Environment Optimization](#6-dev-environment-optimization)
7. [Network Optimization](#7-network-optimization)
8. [Monitoring Tools](#8-monitoring-tools)
9. [Optional Advanced Optimizations](#9-optional-advanced-optimizations)
10. [Security + Performance Balance](#10-security--performance-balance)
11. [Lightweight Desktop Recommendations](#11-lightweight-desktop-recommendations)
12. [Quick Reference Cheatsheet](#12-quick-reference-cheatsheet)

---

## 1. VMware Optimization

### 1.1 CPU Configuration

**Best Practice for i5 12th Gen:**

The i5 12th Gen has Performance-cores (P-cores) and Efficient-cores (E-cores). In VMware, always configure as:

```
Processors: 1
Cores per processor: 4
```

> ✅ **Why:** Using 1 processor with multiple cores avoids NUMA (Non-Uniform Memory Access) overhead. Multiple virtual processors create inter-processor communication latency inside the VM.

> ⚠️ **Avoid:** Setting `Processors: 2, Cores: 2` — this simulates a dual-socket machine, which is slower for most workloads.

**VMware .vmx setting:**
```
numvcpus = "4"
cpuid.coresPerSocket = "4"
```

---

### 1.2 RAM Allocation

With **3.5GB VM RAM**, this is your tight budget. Allocate carefully:

| Component | Recommended |
|-----------|-------------|
| VM RAM | 3584 MB (3.5 GB) |
| Host reserve | Leave at least 2GB free on host |
| VMware balloon driver | Keep enabled (default) |

**Enable Memory Page Sharing (saves RAM on host):**

In VMware Workstation → Edit → Preferences → Memory:
```
✅ Allow most virtual machine memory to be swapped
✅ Fit all virtual machine memory into reserved host RAM (if you have enough)
```

**In .vmx file:**
```
memsize = "3584"
MemAllowAutoScaleDown = "FALSE"
MemTrimRate = "0"
```

> ✅ **Why:** `MemTrimRate = 0` prevents VMware from constantly trimming guest memory, reducing overhead when RAM is actively used.

---

### 1.3 Disk Configuration

**Always use SSD for VM files.** HDDs cause catastrophic I/O wait during Docker builds, `apt` operations, and database writes.

**Best disk settings in VMware:**
```
Disk type:        SCSI (not IDE, not NVMe for compatibility)
Provisioning:     Pre-allocated (not thin)  ← important!
Split:            Single file (better sequential I/O)
```

> ✅ **Why pre-allocated:** Thin-provisioned disks grow dynamically, causing I/O spikes and fragmentation. Pre-allocated disks give consistent performance.

> ✅ **Why single file:** Avoids overhead of managing multiple 2GB splits during large sequential writes (Docker image pulls, database dumps).

**In .vmx file:**
```
scsi0:0.fileName = "ubuntu-devops.vmdk"
```

---

### 1.4 Enable Hardware Virtualization (VT-x / AMD-V)

Required for Docker (nested virtualization) and Kubernetes:

In VMware: **VM Settings → Processors → Virtualization Engine:**
```
✅ Virtualize Intel VT-x/EPT or AMD-V/RVI
✅ Virtualize CPU performance counters
✅ Virtualize IOMMU (if available)
```

**Verify inside Ubuntu:**
```bash
# Should return a number > 0 if enabled
grep -c vmx /proc/cpuinfo

# Check if KVM is accessible
ls /dev/kvm
```

> ⚠️ **If KVM not available:** Docker Desktop and minikube may fall back to slower emulation modes.

---

### 1.5 VMware Performance Settings

**Disable unnecessary VM features to reduce overhead:**

In .vmx file, add:
```
# Disable unnecessary devices
usb.present = "FALSE"           # Disable USB if not needed
sound.present = "FALSE"         # No audio needed for DevOps
svga.vramSize = "16777216"      # 16MB VRAM is enough (no heavy GUI)

# Performance
mainMem.useNamedFile = "FALSE"  # Don't use .vmem file on SSD
prefvmx.useRecommendedLockedMemSize = "TRUE"
```

**Disable VMware animations on host (Windows host):**
- VMware Workstation → View → Uncheck "Use animated transitions"

---

## 2. Ubuntu OS Optimization

### 2.1 Update System First

Always start fresh:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y && sudo apt autoclean
```

---

### 2.2 Disable GNOME Animations (if using desktop)

Animations consume CPU and feel sluggish on a 3.5GB VM:

```bash
# Disable all animations
gsettings set org.gnome.desktop.interface enable-animations false

# Disable overlay scrollbars (faster rendering)
gsettings set org.gnome.desktop.interface overlay-scrolling false

# Reduce window manager effects
gsettings set org.gnome.mutter experimental-features "[]"
```

**Verify:**
```bash
gsettings get org.gnome.desktop.interface enable-animations
# Should output: false
```

---

### 2.3 Reduce Swappiness

**What is swappiness?** A kernel parameter (0–100) that controls how aggressively Linux moves RAM pages to swap.

| Value | Behavior |
|-------|----------|
| 60 | Default — swaps too eagerly |
| 10 | Recommended for DevOps VMs |
| 1  | Almost never swap (use with ZRAM) |
| 0  | Never swap (risky, OOM killer activates) |

**For your 3.5GB VM:**
```bash
# Check current value
cat /proc/sys/vm/swappiness

# Set temporarily (until reboot)
sudo sysctl vm.swappiness=10

# Set permanently
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.d/99-devops.conf
sudo sysctl -p /etc/sysctl.d/99-devops.conf
```

> ✅ **Why 10:** With ZRAM enabled (see Section 3), the kernel will prefer ZRAM compression over swapping to disk. Value `10` strikes the right balance — avoids OOM kills while not trashing your disk.

---

### 2.4 Enable ZRAM

**What is ZRAM?** A compressed RAM-based swap device. Instead of writing to disk, data is compressed and stored in RAM itself. On a 3.5GB VM, this is essential.

```bash
# Install zram-tools
sudo apt install zram-tools -y

# Configure ZRAM
sudo nano /etc/default/zramswap
```

**Set these values:**
```bash
# /etc/default/zramswap

# Use 50% of RAM for ZRAM (1.75GB compressed → effectively ~3GB usable)
PERCENT=50

# Use lz4 (fastest compression, low CPU overhead)
ALGO=lz4
```

```bash
# Apply and enable
sudo systemctl restart zramswap
sudo systemctl enable zramswap

# Verify ZRAM is active
zramctl
swapon --show
```

**Expected output from `swapon --show`:**
```
NAME       TYPE      SIZE  USED PRIO
/dev/zram0 partition 1.8G    0B  100
```

> ✅ **Why lz4:** lz4 is 3-5x faster than gzip/zstd for compression/decompression with acceptable ratios. On CPU-limited VMs, speed matters more than compression ratio.

---

### 2.5 Disable Unnecessary Services

Check what's running:
```bash
# List all active services with startup time
systemd-analyze blame | head -30

# See total boot time
systemd-analyze
```

**Safe services to disable on a DevOps VM:**

```bash
# Bluetooth (not needed in VM)
sudo systemctl disable --now bluetooth.service

# Avahi mDNS (local network discovery, not needed)
sudo systemctl disable --now avahi-daemon.service

# ModemManager (for mobile broadband, useless in VM)
sudo systemctl disable --now ModemManager.service

# cups (printing service)
sudo systemctl disable --now cups.service cups-browsed.service

# snapd (if you don't use snaps — saves RAM and CPU)
sudo systemctl disable --now snapd.service snapd.socket snapd.seeded.service

# Ubuntu telemetry/crash reporting
sudo systemctl disable --now apport.service
sudo systemctl disable --now whoopsie.service

# Ubuntu advantage / pro tools (if not using Ubuntu Pro)
sudo systemctl disable --now ua-timer.service ubuntu-advantage.service 2>/dev/null || true
```

> ⚠️ **Don't disable:** `NetworkManager`, `ssh`, `systemd-resolved`, `docker`, `cron` — these are needed.

**Verify boot time improvement:**
```bash
systemd-analyze
systemd-analyze critical-chain
```

---

### 2.6 Clean Up Snap Packages

Snaps are slow to start and consume extra RAM. Replace with apt equivalents where possible:

```bash
# List installed snaps
snap list

# Remove common heavy snaps (if installed)
sudo snap remove firefox          # Use apt version
sudo snap remove snap-store       # Use GNOME Software or nothing
sudo snap remove core18 core20    # Remove unused cores

# If removing all snaps:
sudo apt install --install-suggests gnome-software   # apt-based software center
```

---

## 3. Memory Optimization

### 3.1 ZRAM vs Swap vs zswap — Full Comparison

| Feature | ZRAM | Disk Swap | zswap |
|---------|------|-----------|-------|
| Location | RAM (compressed) | HDD/SSD | RAM cache + disk fallback |
| Speed | Very fast (RAM speed) | Slow (disk I/O) | Fast until disk fallback |
| Compression | Yes (lz4/zstd) | No | Yes (lz4/zstd) |
| Best for | Low-RAM VMs | Overflow only | Systems with large RAM |
| Risk | None | Disk wear (SSD) | Complexity |
| Recommended for 3.5GB VM | ✅ Yes | ✅ Small (1–2GB) | ❌ Not needed |

---

### 3.2 How Memory Compression Works

```
RAM Usage: 3.5GB available

Without ZRAM:
[Active RAM: 2.5GB] [Free: 1GB] → Out of RAM → Swap to Disk (slow!)

With ZRAM:
[Active RAM: 2.5GB] [ZRAM compressed: 1GB of data in 400MB] [Free: 600MB]
→ Effectively ~4.5GB usable memory without touching disk
```

**Compression ratios (typical):**
- lz4: 2:1 to 2.5:1 ratio
- zstd: 2.5:1 to 3:1 ratio  
- lzo: 2:1 ratio (default on older systems)

---

### 3.3 Complete Memory Setup for 3.5GB VM

```bash
# Step 1: Install ZRAM
sudo apt install zram-tools -y

# Step 2: Configure ZRAM
sudo tee /etc/default/zramswap << 'EOF'
ALGO=lz4
PERCENT=50
EOF

# Step 3: Restart ZRAM
sudo systemctl restart zramswap

# Step 4: Add a small disk swap as safety net (2GB)
# Create swap file
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Step 5: Set swappiness (prefer ZRAM, use disk swap only as last resort)
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.d/99-devops.conf

# Step 6: Verify
free -h
swapon --show
```

**Expected `free -h` output:**
```
               total        used        free      shared  buff/cache   available
Mem:           3.4Gi       1.2Gi       800Mi       120Mi       1.4Gi       1.9Gi
Swap:          3.8Gi          0B       3.8Gi   ← ZRAM(1.8G) + disk(2G)
```

---

### 3.4 Monitor Memory Compression Live

```bash
# Watch ZRAM stats live
watch -n 1 'zramctl && echo "---" && free -h'

# Check compression ratio
cat /sys/block/zram0/stat

# Detailed ZRAM stats
cat /proc/swaps
```

---

## 4. System & Kernel Tuning

### 4.1 Kernel Parameters (Safe Optimizations)

Create a dedicated sysctl config file:

```bash
sudo tee /etc/sysctl.d/99-devops.conf << 'EOF'
# ── Memory ──────────────────────────────────────────
vm.swappiness=10
vm.dirty_ratio=15
vm.dirty_background_ratio=5
vm.vfs_cache_pressure=50

# ── Network Performance ──────────────────────────────
net.core.somaxconn=65535
net.core.rmem_max=16777216
net.core.wmem_max=16777216
net.ipv4.tcp_rmem=4096 87380 16777216
net.ipv4.tcp_wmem=4096 65536 16777216
net.ipv4.tcp_fastopen=3
net.ipv4.tcp_fin_timeout=15
net.ipv4.tcp_tw_reuse=1
net.ipv4.ip_local_port_range=1024 65535

# ── File System ──────────────────────────────────────
fs.file-max=2097152
fs.inotify.max_user_watches=524288
fs.inotify.max_user_instances=512

# ── Docker / Kubernetes ──────────────────────────────
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1

# ── Kernel ───────────────────────────────────────────
kernel.pid_max=4194304
EOF

# Apply all settings
sudo sysctl -p /etc/sysctl.d/99-devops.conf
```

**What each setting does:**

| Parameter | Purpose |
|-----------|---------|
| `vm.dirty_ratio=15` | Write dirty pages to disk when 15% of RAM is dirty — prevents I/O spikes |
| `vm.vfs_cache_pressure=50` | Keep directory/inode cache longer (speeds up repeated file access) |
| `fs.inotify.max_user_watches=524288` | Required for VS Code, webpack, Docker — default (8192) is too low |
| `net.ipv4.tcp_fastopen=3` | Reduces TCP handshake latency for repeated connections |
| `net.bridge.*` | Required for Docker networking and Kubernetes CNI plugins |

---

### 4.2 File Descriptor Limits

Docker, Node.js, and databases open thousands of file descriptors. Default limits cause crashes.

```bash
# Check current limits
ulimit -n        # Soft limit (usually 1024 — too low!)
ulimit -Hn       # Hard limit

# Set persistent limits for all users
sudo tee /etc/security/limits.d/99-devops.conf << 'EOF'
# DevOps file descriptor limits
*    soft    nofile    65536
*    hard    nofile    1048576
*    soft    nproc     32768
*    hard    nproc     65536
root soft    nofile    65536
root hard    nofile    1048576
EOF

# Apply for systemd services too
sudo mkdir -p /etc/systemd/system.conf.d/
sudo tee /etc/systemd/system.conf.d/limits.conf << 'EOF'
[Manager]
DefaultLimitNOFILE=1048576
DefaultLimitNPROC=65536
EOF

# Reload systemd
sudo systemctl daemon-reexec

# Verify (logout/login or new terminal needed)
ulimit -n
```

---

### 4.3 Startup Optimization

```bash
# Set boot to multi-user (no GUI) for server mode
# Only if using headless/server Ubuntu
sudo systemctl set-default multi-user.target

# To switch back to GUI:
# sudo systemctl set-default graphical.target

# Speed up GRUB (reduce wait time)
sudo nano /etc/default/grub
# Change: GRUB_TIMEOUT=5  →  GRUB_TIMEOUT=1
sudo update-grub
```

---

## 5. Disk & I/O Optimization

### 5.1 Filesystem — ext4 vs XFS

| Feature | ext4 | XFS |
|---------|------|-----|
| Boot speed | Faster | Slower |
| Large files | Good | Excellent |
| Docker layers | Good | Better |
| Recovery | Mature | Good |
| **Recommendation** | ✅ Use for root `/` | Use for `/var/lib/docker` |

**Optimize ext4 mount options:**
```bash
# Check current mount options
cat /proc/mounts | grep ext4

# Edit fstab for better performance
sudo nano /etc/fstab
```

Change your root mount to:
```fstab
UUID=xxxx-xxxx  /  ext4  defaults,noatime,nodiratime,commit=60  0  1
```

> - `noatime`: Don't update file access time on every read (major I/O reduction)
> - `commit=60`: Write journal every 60s instead of 5s (less I/O, slight data loss risk on hard crash — acceptable for dev VM)

```bash
# Remount without reboot to test
sudo mount -o remount,noatime /
```

---

### 5.2 Disk I/O Scheduler

For VM on SSD, use `none` (or `mq-deadline`) scheduler:

```bash
# Check current scheduler
cat /sys/block/sda/queue/scheduler

# Set to none (best for SSD/VM)
echo none | sudo tee /sys/block/sda/queue/scheduler

# Make permanent via udev rule
sudo tee /etc/udev/rules.d/60-scheduler.rules << 'EOF'
ACTION=="add|change", KERNEL=="sd[a-z]|vd[a-z]|nvme*", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="none"
EOF
```

> ✅ **Why:** The `none` scheduler passes I/O directly to the NVMe/SSD hardware queue. VM disk I/O is already serialized by the hypervisor — double-scheduling wastes time.

---

### 5.3 Log Rotation & Cleanup

Logs grow fast during Docker builds and CI/CD runs:

```bash
# Configure journald to limit log size
sudo tee /etc/systemd/journald.conf.d/size.conf << 'EOF'
[Journal]
SystemMaxUse=500M
SystemMaxFileSize=50M
MaxRetentionSec=7day
Compress=yes
EOF

sudo systemctl restart systemd-journald

# Clean old logs immediately
sudo journalctl --vacuum-size=200M
sudo journalctl --vacuum-time=7d

# Set up automatic Docker log rotation
sudo tee /etc/docker/daemon.json << 'EOF'
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "storage-driver": "overlay2"
}
EOF
```

---

### 5.4 Periodic Disk Cleanup Script

```bash
# Create weekly cleanup script
sudo tee /usr/local/bin/devops-cleanup.sh << 'EOF'
#!/bin/bash
echo "=== DevOps VM Cleanup ==="

# Remove unused Docker resources
docker system prune -f --volumes 2>/dev/null
docker image prune -f 2>/dev/null

# Clear apt cache
apt-get clean -y
apt-get autoremove -y

# Clear old journals
journalctl --vacuum-size=200M

# Clear thumbnail cache
rm -rf ~/.cache/thumbnails/*

echo "=== Cleanup done ==="
df -h /
EOF

sudo chmod +x /usr/local/bin/devops-cleanup.sh

# Run weekly via cron
echo "0 3 * * 0 root /usr/local/bin/devops-cleanup.sh >> /var/log/devops-cleanup.log 2>&1" | sudo tee /etc/cron.d/devops-cleanup
```

---

## 6. Dev Environment Optimization

### 6.1 Docker Performance Tuning

```bash
# Install Docker (official method)
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
newgrp docker

# Configure Docker daemon for performance
sudo tee /etc/docker/daemon.json << 'EOF'
{
  "storage-driver": "overlay2",
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 65536,
      "Soft": 65536
    }
  },
  "max-concurrent-downloads": 6,
  "max-concurrent-uploads": 4
}
EOF

sudo systemctl restart docker
sudo systemctl enable docker
```

**Docker RAM optimization for 3.5GB VM:**
```bash
# Limit containers' default memory usage
# Run containers with limits:
docker run --memory="512m" --memory-swap="1g" nginx

# Check Docker resource usage
docker stats --no-stream

# Remove build cache regularly
docker builder prune -f
```

> ✅ **overlay2** is the fastest and most stable Docker storage driver on ext4/XFS.

---

### 6.2 Node.js Optimization

```bash
# Install Node.js via nvm (best practice — avoid snap/apt versions)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc

# Install LTS version
nvm install --lts
nvm use --lts

# Set Node.js memory limit for 3.5GB VM
# Prevent Node from using too much RAM during builds
echo 'export NODE_OPTIONS="--max-old-space-size=1024"' >> ~/.bashrc
source ~/.bashrc

# Install performance tools
npm install -g npm@latest
npm install -g clinic     # Node.js performance profiler
npm install -g autocannon # HTTP benchmarking

# Speed up npm installs
npm config set fund false
npm config set audit false
npm config set prefer-offline true
```

**Use pnpm for faster installs (recommended):**
```bash
npm install -g pnpm
# pnpm is 2-3x faster than npm, uses symlinks to avoid duplication
```

---

### 6.3 Git Performance

```bash
# Configure Git for performance
git config --global core.preloadindex true
git config --global core.fscache true
git config --global gc.auto 256

# Use libsecret for credential caching (avoids repeated auth prompts)
sudo apt install libsecret-1-0 libsecret-1-dev -y
git config --global credential.helper /usr/lib/git-core/git-credential-libsecret

# Speed up large repos
git config --global fetch.parallel 4
git config --global pack.threads 4

# Set default branch name
git config --global init.defaultBranch main
```

---

### 6.4 Terminal Optimization

```bash
# Install Zsh + Oh My Zsh (faster, smarter shell)
sudo apt install zsh -y
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# Install zsh-autosuggestions (saves typing time)
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

# Install fast syntax highlighting
git clone https://github.com/zdharma-continuum/fast-syntax-highlighting ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/fast-syntax-highlighting

# Edit ~/.zshrc plugins line:
# plugins=(git zsh-autosuggestions fast-syntax-highlighting docker kubectl)

# Install tmux for terminal multiplexing
sudo apt install tmux -y
```

**Useful aliases for DevOps (add to `~/.zshrc` or `~/.bashrc`):**
```bash
cat >> ~/.zshrc << 'EOF'

# ── DevOps Aliases ────────────────────────────────────
alias k='kubectl'
alias dk='docker'
alias dkps='docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"'
alias dklogs='docker logs -f'
alias dkstop='docker stop $(docker ps -q) 2>/dev/null || echo "No containers"'
alias dkclean='docker system prune -f'
alias sys='systemctl'
alias jctl='journalctl -f'
alias mem='free -h && echo "---" && vmstat -s | head -5'
alias ports='ss -tulpn'
alias myip='curl -s ifconfig.me'
alias update='sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y'
EOF
source ~/.zshrc
```

---

## 7. Network Optimization

### 7.1 TCP Performance Tuning

Already covered in Section 4.1. Additional network settings:

```bash
# Add to /etc/sysctl.d/99-devops.conf
sudo tee -a /etc/sysctl.d/99-devops.conf << 'EOF'

# ── Additional Network ────────────────────────────────
net.ipv4.tcp_slow_start_after_idle=0
net.ipv4.tcp_no_metrics_save=1
net.core.netdev_max_backlog=5000
net.ipv4.tcp_max_syn_backlog=8192
EOF

sudo sysctl -p /etc/sysctl.d/99-devops.conf
```

---

### 7.2 DNS Optimization

Slow DNS = slow `apt install`, slow `docker pull`, slow `npm install`:

```bash
# Check current DNS resolution time
time nslookup google.com

# Install and configure dnsmasq as local cache
sudo apt install dnsmasq -y

sudo tee /etc/dnsmasq.conf << 'EOF'
# Local DNS cache
cache-size=1000
no-resolv
server=1.1.1.1
server=8.8.8.8
server=9.9.9.9
EOF

sudo systemctl enable --now dnsmasq

# Point systemd-resolved to use dnsmasq
sudo tee /etc/systemd/resolved.conf.d/dns.conf << 'EOF'
[Resolve]
DNS=127.0.0.1
DNSStubListener=no
EOF

sudo systemctl restart systemd-resolved
```

---

## 8. Monitoring Tools

### 8.1 Essential Monitoring Stack

Install all at once:
```bash
sudo apt install -y htop iotop iftop nload sysstat net-tools
```

---

### 8.2 Tool Reference

| Tool | Install | Use Case | Command |
|------|---------|----------|---------|
| **htop** | `apt install htop` | CPU/RAM per process | `htop` |
| **iotop** | `apt install iotop` | Disk I/O per process | `sudo iotop -o` |
| **nload** | `apt install nload` | Network bandwidth live | `nload` |
| **iftop** | `apt install iftop` | Per-connection network | `sudo iftop` |
| **glances** | `pip install glances` | All-in-one dashboard | `glances` |
| **dstat** | `apt install dstat` | Combined stats | `dstat -cdngy` |
| **vmstat** | built-in | Memory/swap stats | `vmstat -s` |
| **ss** | built-in | Socket/port info | `ss -tulpn` |
| **iostat** | `apt install sysstat` | Disk throughput | `iostat -x 1` |

---

### 8.3 Glances (All-in-One Dashboard)

```bash
# Install glances
pip3 install glances

# Run with web interface (access from browser)
glances -w
# Visit http://localhost:61208

# Run in terminal
glances
```

---

### 8.4 Netdata (Real-time Monitoring Dashboard)

```bash
# Install netdata (lightweight, powerful)
bash <(curl -Ss https://my-netdata.io/kickstart.sh) --non-interactive

# Access dashboard
# http://localhost:19999

# Configure to use less RAM
sudo nano /etc/netdata/netdata.conf
# Set: history = 3600   (1 hour instead of 24 hours)
# Set: update every = 2  (every 2s instead of 1s)

sudo systemctl restart netdata
```

---

### 8.5 Quick Performance Check Script

```bash
# Save as ~/check-perf.sh
cat > ~/check-perf.sh << 'EOF'
#!/bin/bash
echo "════════════════════════════════════════"
echo "       VM Performance Snapshot"
echo "════════════════════════════════════════"
echo ""
echo "── CPU Load ─────────────────────────"
uptime
echo ""
echo "── Memory ───────────────────────────"
free -h
echo ""
echo "── ZRAM Status ──────────────────────"
zramctl 2>/dev/null || echo "ZRAM not active"
echo ""
echo "── Disk Usage ───────────────────────"
df -h / /var 2>/dev/null
echo ""
echo "── Top 5 Memory Processes ───────────"
ps aux --sort=-%mem | head -6
echo ""
echo "── Docker Status ────────────────────"
docker info 2>/dev/null | grep -E "Containers|Images|Memory"
echo "════════════════════════════════════════"
EOF
chmod +x ~/check-perf.sh
~/check-perf.sh
```

---

## 9. Optional Advanced Optimizations

### 9.1 earlyoom — Prevent System Freeze

When RAM fills up, Linux freezes before the OOM killer acts. `earlyoom` kills the biggest process early:

```bash
sudo apt install earlyoom -y

# Configure: kill when RAM < 10% or swap < 10%
sudo tee /etc/default/earlyoom << 'EOF'
EARLYOOM_ARGS="-m 10 -s 10 --prefer '(^|/)(node|java|firefox)$' --avoid '(^|/)(docker|sshd|systemd)$'"
EOF

sudo systemctl enable --now earlyoom
```

> ✅ **Perfect for 3.5GB VM.** Prevents the entire VM from becoming unresponsive during heavy Node.js builds or Docker operations.

---

### 9.2 preload — Faster App Launch

Preload learns which apps you use and preloads them into RAM:

```bash
sudo apt install preload -y
sudo systemctl enable --now preload
```

> ⚠️ **Note:** Uses ~30-50MB RAM. On 3.5GB VM, only useful if you repeatedly open the same GUI apps. Skip for headless/server setups.

---

### 9.3 tuned — Profile-Based System Tuning

```bash
sudo apt install tuned -y
sudo systemctl enable --now tuned

# List available profiles
tuned-adm list

# Apply 'virtual-guest' profile (optimized for VMs)
sudo tuned-adm profile virtual-guest

# Verify active profile
tuned-adm active
```

**Profiles explained:**

| Profile | Use Case |
|---------|----------|
| `virtual-guest` | ✅ Best for VMware VM — tuned for guest OS |
| `throughput-performance` | Server workloads with high I/O |
| `latency-performance` | Low-latency applications |
| `balanced` | Default general-purpose |

---

### 9.4 zswap (Alternative to ZRAM)

> **Recommendation: Use ZRAM instead of zswap for 3.5GB VM.**

zswap is a compressed cache *in front of* disk swap. ZRAM replaces disk swap entirely. For a low-RAM VM, ZRAM is better.

If you want to try zswap:
```bash
# Enable zswap (add to GRUB)
sudo nano /etc/default/grub
# Add to GRUB_CMDLINE_LINUX_DEFAULT:
# zswap.enabled=1 zswap.compressor=lz4 zswap.max_pool_percent=20

sudo update-grub
sudo reboot
```

---

## 10. Security + Performance Balance

### 10.1 UFW Firewall (Lightweight, Essential)

```bash
# Install and configure UFW
sudo apt install ufw -y

# Set defaults
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (important — do before enabling!)
sudo ufw allow ssh

# Allow common DevOps ports
sudo ufw allow 3000/tcp   # Node.js apps
sudo ufw allow 8080/tcp   # Web servers
sudo ufw allow 5432/tcp   # PostgreSQL
sudo ufw allow 6379/tcp   # Redis
sudo ufw allow 27017/tcp  # MongoDB

# Enable
sudo ufw enable
sudo ufw status verbose
```

> ✅ **UFW is lightweight** — less than 5MB, no performance impact.

---

### 10.2 SSH Optimization

```bash
sudo nano /etc/ssh/sshd_config
```

Add/modify:
```
# Performance
UseDNS no                     # Skip reverse DNS lookup (faster login)
GSSAPIAuthentication no       # Skip Kerberos (faster login)

# Security
PermitRootLogin no
PasswordAuthentication no     # Use SSH keys only (safer)
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2

# Connection multiplexing (faster repeated connections)
MaxSessions 10
```

```bash
sudo systemctl restart ssh
```

---

### 10.3 Fail2ban (Lightweight Brute-force Protection)

```bash
sudo apt install fail2ban -y

sudo tee /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5

[sshd]
enabled = true
EOF

sudo systemctl enable --now fail2ban
```

---

## 11. Lightweight Desktop Recommendations

### For a 3.5GB VM, ranked best to worst:

| Desktop | RAM Usage | Best For |
|---------|-----------|---------|
| **No desktop (server/CLI)** | ~200MB | Pure DevOps, headless |
| **XFCE** | ~350MB | Light GUI, good performance |
| **LXDE/LXQt** | ~280MB | Extremely light GUI |
| **GNOME (minimal)** | ~700MB | Familiar but heavy |
| **KDE Plasma** | ~600MB | Feature-rich but heavy |
| **GNOME (full)** | ~900MB | ❌ Too heavy for 3.5GB |

---

### Install XFCE (Recommended for 3.5GB VM):

```bash
# Minimal XFCE install (no bloat)
sudo apt install xfce4 xfce4-terminal lightdm -y

# Set XFCE as default desktop
sudo systemctl set-default graphical.target

# Disable compositor (saves GPU/CPU in VM)
# XFCE Settings → Window Manager Tweaks → Compositor → Uncheck "Enable display compositing"

# Or via command:
xfconf-query -c xfwm4 -p /general/use_compositing -s false
```

---

### Or use minimal Ubuntu Server + GUI when needed:

```bash
# Install Ubuntu Server (no desktop)
# Then add GUI on demand:
sudo apt install --no-install-recommends xorg xfce4 xfce4-terminal -y

# Start GUI manually when needed
startx
```

---

## 12. Quick Reference Cheatsheet

### All Commands Summary

```bash
# ── INITIAL SETUP ─────────────────────────────────────

# 1. Update system
sudo apt update && sudo apt upgrade -y

# 2. Install ZRAM
sudo apt install zram-tools -y
echo -e "ALGO=lz4\nPERCENT=50" | sudo tee /etc/default/zramswap
sudo systemctl restart zramswap && sudo systemctl enable zramswap

# 3. Set swappiness
echo 'vm.swappiness=10' | sudo tee /etc/sysctl.d/99-devops.conf
sudo sysctl -p /etc/sysctl.d/99-devops.conf

# 4. File descriptor limits
echo -e "*    soft    nofile    65536\n*    hard    nofile    1048576" | sudo tee /etc/security/limits.d/99-devops.conf

# 5. Disable heavy services
sudo systemctl disable --now bluetooth avahi-daemon ModemManager cups snapd

# 6. Install earlyoom
sudo apt install earlyoom -y && sudo systemctl enable --now earlyoom

# 7. Install tuned + set VM profile
sudo apt install tuned -y
sudo systemctl enable --now tuned
sudo tuned-adm profile virtual-guest

# 8. Docker setup
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER

# 9. Monitoring tools
sudo apt install -y htop iotop nload glances sysstat

# 10. inotify limit (for VS Code, Docker)
echo 'fs.inotify.max_user_watches=524288' | sudo tee -a /etc/sysctl.d/99-devops.conf
sudo sysctl -p /etc/sysctl.d/99-devops.conf
```

---

### Memory Budget for 3.5GB VM

```
Total VM RAM:          3584 MB
────────────────────────────────
Ubuntu OS/kernel:      ~400 MB
ZRAM overhead:         ~100 MB
Docker daemon:         ~200 MB
2x Node.js processes:  ~500 MB
PostgreSQL:            ~200 MB
Available for work:    ~2184 MB  ✅
ZRAM compressed pool:  +1800 MB  ✅
────────────────────────────────
Effective total:       ~4000 MB  🚀
```

---

### Performance Verification Commands

```bash
# Check all optimizations at once
echo "=== Swappiness ===" && cat /proc/sys/vm/swappiness
echo "=== ZRAM ===" && zramctl
echo "=== Swap ===" && swapon --show
echo "=== File descriptors ===" && ulimit -n
echo "=== inotify watches ===" && cat /proc/sys/fs/inotify/max_user_watches
echo "=== tuned profile ===" && tuned-adm active
echo "=== Boot time ===" && systemd-analyze
echo "=== Memory ===" && free -h
```

---

## Troubleshooting

### Docker permission denied
```bash
sudo usermod -aG docker $USER
newgrp docker
# or logout and login again
```

### ZRAM not showing
```bash
sudo modprobe zram
sudo systemctl restart zramswap
zramctl
```

### VS Code too many files error
```bash
echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### System slow after Docker builds
```bash
docker builder prune -f
docker system prune -f
sync && echo 3 | sudo tee /proc/sys/vm/drop_caches
```

### Port already in use
```bash
sudo ss -tulpn | grep :PORT_NUMBER
sudo kill -9 $(sudo lsof -t -i:PORT_NUMBER)
```

---

*Guide version: 2025 | Ubuntu 25.04 | VMware Workstation | i5 12th Gen*
*Last reviewed: April 2026*
