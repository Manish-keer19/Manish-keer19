# 🛡️ Linux VPS Backup & Disaster Recovery Master Guide

> **Industry-standard, production-focused, beginner-to-advanced reference**  
> Covers full VPS backup, disaster recovery, compression, migration, and restoration.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Linux Filesystem Basics](#2-linux-filesystem-basics)
3. [Types of VPS Backups](#3-types-of-vps-backups)
4. [Linux Compression Systems Deep Dive](#4-linux-compression-systems-deep-dive)
5. [tar Command Mastery](#5-tar-command-mastery)
6. [Full VPS Backup Strategies](#6-full-vps-backup-strategies)
7. [Full Server Backup Commands](#7-full-server-backup-commands)
8. [rsync Deep Dive](#8-rsync-deep-dive)
9. [Database Backup Inside VPS](#9-database-backup-inside-vps)
10. [Docker Backup Strategies](#10-docker-backup-strategies)
11. [Cloud VPS Snapshot Systems](#11-cloud-vps-snapshot-systems)
12. [Automated Backup Systems](#12-automated-backup-systems)
13. [Cloud Storage Backup](#13-cloud-storage-backup)
14. [Encryption & Security](#14-encryption--security)
15. [Disaster Recovery](#15-disaster-recovery)
16. [VPS Migration Strategies](#16-vps-migration-strategies)
17. [Monitoring & Verification](#17-monitoring--verification)
18. [Production Backup Architecture](#18-production-backup-architecture)
19. [Common Errors & Troubleshooting](#19-common-errors--troubleshooting)
20. [Best Practices Checklist](#20-best-practices-checklist)
21. [Cheat Sheet](#21-cheat-sheet)

---

## 1. Introduction

### What Is VPS Backup?

A **VPS (Virtual Private Server) backup** is a copy of your server's data, configuration, and state stored separately from the live environment. Backups allow you to recover from data loss, system failure, ransomware, or human error — restoring your server to a known good state.

Backups are not optional in production. They are the last line of defense between a business-critical failure and total data loss.

---

### Why Backups Matter — Real-World Disaster Scenarios

#### Hardware Failure
Cloud VPS providers run on physical machines. Disk failures, RAID degradation, hypervisor crashes, and datacenter power events happen. In 2021, OVHcloud's Strasbourg datacenter suffered a major fire that destroyed physical servers. Customers **without external backups lost all data permanently**.

#### Ransomware
Ransomware encrypts your files and demands payment. If backups are stored on the same server or a network-accessible path, they get encrypted too. Only **offsite, isolated backups** survive ransomware.

#### Accidental Deletion
The most common disaster: `rm -rf /var/www/` run in the wrong directory, a `DROP TABLE` on production, or a misconfigured deployment script wiping data. Without backups, recovery is impossible.

#### Cloud Provider Failures
- Providers can terminate accounts, suspend services, or experience outages.
- DigitalOcean, Linode, Vultr, and AWS all have SLA limitations.
- Provider-managed snapshots are NOT a substitute for independent backups.

#### Botched Updates
A kernel update, a PHP version upgrade, or a misconfigured Nginx change can break production. Backups let you roll back within minutes.

---

### Recovery Planning Concepts

#### RPO — Recovery Point Objective

> **RPO** defines the maximum acceptable amount of data loss measured in time.

- RPO = 1 hour → You can lose at most 1 hour of data
- RPO = 24 hours → Daily backups are acceptable
- RPO = 0 → You need real-time replication

**Example:** A financial application processes thousands of transactions per hour. RPO = 15 minutes means backups must run at least every 15 minutes.

#### RTO — Recovery Time Objective

> **RTO** defines the maximum acceptable time to restore service after a failure.

- RTO = 4 hours → You have 4 hours to bring systems back online
- RTO = 30 minutes → Requires pre-staged restore automation
- RTO = 0 → Requires fully redundant hot standby

**Example:** An e-commerce store losing $10,000/hour during downtime may need RTO = 30 minutes, requiring automated failover.

#### RPO vs RTO Summary

```
┌──────────────────────────────────────────────────────────────────┐
│                    DISASTER RECOVERY TIMELINE                    │
│                                                                  │
│  Disaster                              Service                   │
│  Occurs      Last Backup               Restored                  │
│     │             │                        │                     │
│─────▼─────────────▼────────────────────────▼─────               │
│                                                                  │
│     ◄── RPO ──────►◄────────── RTO ─────────►                   │
│     (data loss     (recovery time allowed)                       │
│      tolerance)                                                  │
└──────────────────────────────────────────────────────────────────┘
```

| Metric | Measures | Driven By |
|--------|----------|-----------|
| RPO | How much data you can lose | Backup frequency |
| RTO | How fast you must restore | Restore speed & automation |

---

## 2. Linux Filesystem Basics

### Linux Directory Structure

Understanding which directories contain what data is critical for designing effective backups.

```
/
├── etc/          ← System & application configuration files
├── var/          ← Variable data: logs, databases, mail, spool
├── home/         ← User home directories
├── root/         ← Root user home directory
├── srv/          ← Service data (web, FTP, etc.)
├── opt/          ← Optional/third-party software
├── usr/          ← User binaries, libraries, docs
├── boot/         ← Bootloader, kernel images
├── bin/          ← Essential system binaries
├── sbin/         ← System administration binaries
├── lib/          ← Shared libraries
├── tmp/          ← Temporary files (cleared on reboot)
├── proc/         ← Virtual filesystem (kernel/process info)
├── sys/          ← Virtual filesystem (hardware/driver info)
├── dev/          ← Device files
├── run/          ← Runtime data (PIDs, sockets)
├── mnt/          ← Temporary mount points
└── media/        ← Removable media mount points
```

---

### Critical Directories to Back Up

| Directory | Contents | Priority |
|-----------|----------|----------|
| `/etc` | All system/app config files | 🔴 Critical |
| `/var/www` | Web application files | 🔴 Critical |
| `/var/lib` | Database files (MySQL, PostgreSQL) | 🔴 Critical |
| `/home` | User files and project data | 🔴 Critical |
| `/root` | Root user config, scripts | 🔴 Critical |
| `/srv` | Web/FTP service data | 🔴 Critical |
| `/opt` | Third-party application installs | 🟡 Important |
| `/usr/local` | Locally compiled software | 🟡 Important |
| `/boot` | Kernel and bootloader (for full restore) | 🟡 Important |

---

### Directories to EXCLUDE from Backups

| Directory | Reason to Exclude |
|-----------|-------------------|
| `/proc` | Virtual filesystem — kernel data, not real files |
| `/sys` | Virtual filesystem — hardware/driver state |
| `/dev` | Device files — recreated at boot |
| `/run` | Runtime state — PIDs, sockets |
| `/tmp` | Temporary files — no value |
| `/var/tmp` | Temporary files |
| `/var/log` | Logs optional — large, usually not critical |
| `/var/cache` | Cached data — can be regenerated |
| `/var/spool/cups` | Print spooler data |
| `/lost+found` | Filesystem recovery directory |
| `swap` | Swap space — not a directory but never back up |

---

### Permissions, Ownership, and Symlinks

#### File Permissions
Linux permissions use a 3-group model: **owner**, **group**, **others**.

```bash
# View permissions
ls -la /etc/nginx/nginx.conf
# -rw-r--r-- 1 root root 2814 Jan 1 10:00 /etc/nginx/nginx.conf
#  ↑↑↑↑↑↑↑↑↑   ↑↑↑↑ ↑↑↑↑
#  permissions  user  group
```

Permission bits:
- `r` = 4 (read)
- `w` = 2 (write)
- `x` = 1 (execute)

#### Ownership
```bash
# Change ownership
chown user:group file
chown -R www-data:www-data /var/www/html

# Change permissions
chmod 755 /var/www/html
chmod -R 640 /etc/ssl/private/
```

#### Symbolic Links
Symlinks are pointers to other files or directories. They **must be preserved** during backup.

```bash
# View symlinks
ls -la /etc/nginx/sites-enabled/
# lrwxrwxrwx 1 root root 34 Jan 1 -> /etc/nginx/sites-available/default

# tar preserves symlinks by default
# rsync: use -a or --links to preserve
```

#### Mounts
Check active mounts before backup — mounted volumes may need separate handling:

```bash
# List all mounts
mount | grep -v "^cgroup\|^proc\|^sys\|^dev\|^run"
cat /etc/fstab
```

> ⚠️ **Production Note:** Never assume all your data is under `/`. External NFS mounts, EBS volumes, or network filesystems must be explicitly included or excluded.

---

## 3. Types of VPS Backups

### Full Backup

A **complete copy of all selected data** at a specific point in time.

```
Day 1: [FULL: 50GB]
Day 2: [FULL: 51GB]
Day 3: [FULL: 52GB]
```

| Aspect | Detail |
|--------|--------|
| **Storage** | High — every backup is complete |
| **Restore Speed** | Fastest — single file to restore |
| **Backup Speed** | Slowest — copies everything every time |
| **Best For** | Weekly base backups, pre-migration snapshots |

---

### Incremental Backup

Only backs up data **changed since the last backup** (full or incremental).

```
Day 1: [FULL: 50GB]
Day 2: [INCR: 2GB changed]
Day 3: [INCR: 1.5GB changed]
Day 4: [INCR: 3GB changed]
```

**Restore requires:** Full + all incrementals in sequence.

| Aspect | Detail |
|--------|--------|
| **Storage** | Very low |
| **Restore Speed** | Slowest — must chain all incrementals |
| **Backup Speed** | Fastest |
| **Best For** | Daily backups between full backups |

---

### Differential Backup

Backs up data **changed since the last FULL backup** only.

```
Day 1: [FULL: 50GB]
Day 2: [DIFF: 2GB vs Day 1]
Day 3: [DIFF: 3.5GB vs Day 1]
Day 4: [DIFF: 5GB vs Day 1]
```

**Restore requires:** Full + latest differential only.

| Aspect | Detail |
|--------|--------|
| **Storage** | Medium — grows until next full |
| **Restore Speed** | Fast — only 2 files needed |
| **Backup Speed** | Moderate |
| **Best For** | Balance between storage and restore speed |

---

### Backup Type Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│ BACKUP STRATEGY COMPARISON                                      │
│                                                                 │
│ Full:         F────────────F────────────F────────────F          │
│                                                                 │
│ Incremental:  F────I──I──I─F────I──I──I─F────I──I──I           │
│               (chain all to restore)                            │
│                                                                 │
│ Differential: F────D──D──D─F────D──D──D─F────D──D──D           │
│               (F + latest D to restore)                         │
└─────────────────────────────────────────────────────────────────┘
```

---

### Snapshot Backup

A **point-in-time copy** taken at the block or filesystem level, often near-instant using copy-on-write (COW) technology.

```bash
# LVM snapshot example
lvcreate -L10G -s -n mydata_snap /dev/vg0/mydata

# ZFS snapshot
zfs snapshot tank/data@backup-2024-01-01
```

| Aspect | Detail |
|--------|--------|
| **Speed** | Near-instant (COW) |
| **Consistency** | Crash-consistent or application-consistent |
| **Storage** | Low initially, grows with changes |
| **Best For** | Pre-update checkpoints, cloud provider backups |

---

### Block-Level vs Filesystem-Level Backup

| Type | How It Works | Tools | Use Case |
|------|-------------|-------|----------|
| **Block-level** | Copies raw disk blocks | `dd`, `Clonezilla` | Full disk image, bare-metal restore |
| **Filesystem-level** | Copies files through filesystem | `tar`, `rsync` | Selective backup, file-level restore |
| **Image-based** | Full disk image snapshot | Cloud snapshots, `dd` | Rapid full server restore |

---

### Live (Hot) vs Cold Backup

| Type | Server State | Risk | Use Case |
|------|-------------|------|----------|
| **Hot backup** | Running during backup | Possible inconsistency unless app-aware | Production servers that can't go offline |
| **Cold backup** | Stopped during backup | None — fully consistent | Non-critical or maintenance window backups |
| **Warm backup** | Services paused briefly | Minimal | Databases with brief lock |

> ⚠️ **Production Note:** For databases, always use native dump tools (mysqldump, pg_dump) or application-consistent snapshots. A hot filesystem backup of running database files may be inconsistent and unrestorable.

---

## 4. Linux Compression Systems Deep Dive

### Why Compression Matters for Backups

Backup files are large. Compression reduces:
- **Storage cost** (smaller backup files)
- **Transfer time** (faster offsite upload)
- **Bandwidth usage** (cheaper cloud egress)

The trade-off is CPU time and compression/decompression speed.

---

### gzip

The most common Linux compression tool. Balances speed and ratio well.

```bash
# Compress
gzip file.txt              # Creates file.txt.gz, removes original
gzip -k file.txt           # Keep original
gzip -9 file.txt           # Max compression
gzip -1 file.txt           # Fastest compression

# Decompress
gunzip file.txt.gz
gzip -d file.txt.gz

# View compressed without extracting
zcat file.txt.gz

# Check integrity
gzip -t file.txt.gz
```

---

### bzip2

Better compression ratio than gzip, but significantly slower.

```bash
# Compress
bzip2 file.txt             # Creates file.txt.bz2
bzip2 -k file.txt          # Keep original
bzip2 -9 file.txt          # Max compression

# Decompress
bunzip2 file.txt.bz2
bzip2 -d file.txt.bz2
```

---

### xz

Highest compression ratio in the standard Linux toolkit. Very slow compression, fast decompression.

```bash
# Compress
xz file.txt                # Creates file.txt.xz
xz -k file.txt             # Keep original
xz -9 file.txt             # Max compression (very slow)
xz -0 file.txt             # Fastest/lowest compression

# Decompress
unxz file.txt.xz
xz -d file.txt.xz
```

---

### zstd (Zstandard)

Modern compression algorithm by Facebook. **Best balance of speed and ratio** — recommended for production backups.

```bash
# Install
apt install zstd            # Debian/Ubuntu
yum install zstd            # CentOS/RHEL

# Compress
zstd file.txt               # Creates file.txt.zst
zstd -19 file.txt           # Max compression
zstd -1 file.txt            # Fastest
zstd --rm file.txt          # Remove original after compression

# Decompress
zstd -d file.txt.zst
unzstd file.txt.zst

# Multi-threaded compression (use all CPU cores)
zstd -T0 -19 file.txt
```

---

### zip / unzip

Cross-platform archive format. Common for sharing with Windows users.

```bash
# Create zip archive
zip archive.zip file1 file2
zip -r archive.zip /directory/    # Recursive
zip -9 archive.zip file.txt       # Max compression

# Extract
unzip archive.zip
unzip archive.zip -d /target/dir/

# List contents
unzip -l archive.zip

# Test integrity
unzip -t archive.zip
```

---

### 7z (p7zip)

Excellent compression ratio. Supports many formats. Useful for maximum compression.

```bash
# Install
apt install p7zip-full

# Compress
7z a archive.7z file.txt
7z a -mx=9 archive.7z directory/  # Max compression
7z a -p"password" archive.7z file # Password protect

# Extract
7z x archive.7z
7z x archive.7z -o/target/dir/

# List contents
7z l archive.7z

# Test integrity
7z t archive.7z
```

---

### rar / unrar

Proprietary format. Common for multi-volume archives. Rarely used in Linux server environments.

```bash
# Install
apt install rar unrar

# Extract rar files
unrar x archive.rar
unrar e archive.rar /target/

# Test integrity
unrar t archive.rar
```

---

### Compression Algorithm Comparison Table

| Algorithm | Extension | Compression Ratio | Speed (compress) | Speed (decompress) | CPU Usage | Best For |
|-----------|-----------|-------------------|------------------|--------------------|-----------|----------|
| **gzip** | `.gz` | Medium | Fast | Fast | Low | General use, widely compatible |
| **bzip2** | `.bz2` | Medium-High | Slow | Slow | Medium | Better ratio needed, not time-sensitive |
| **xz** | `.xz` | High | Very Slow | Fast | High | Maximum compression, archives |
| **zstd** | `.zst` | High | Very Fast | Very Fast | Low-Medium | **Production backups — recommended** |
| **zip** | `.zip` | Medium | Fast | Fast | Low | Cross-platform sharing |
| **7z** | `.7z` | Highest | Slow | Moderate | High | Maximum compression |
| **rar** | `.rar` | Medium-High | Moderate | Fast | Medium | Multi-volume archives |

---

### Benchmark: Compressing a 1GB Log Directory

```
Algorithm   Time    Output Size   Ratio   CPU
─────────────────────────────────────────────
none        -       1000 MB       1.0x    -
gzip -6     12s     310 MB        3.2x    Low
bzip2 -9    45s     285 MB        3.5x    Medium
xz -6       90s     245 MB        4.1x    High
zstd -3     4s      320 MB        3.1x    Low
zstd -19    25s     255 MB        3.9x    Medium
7z -mx=9    120s    230 MB        4.3x    High
```

> 💡 **Production Recommendation:** Use `zstd -3` for daily automated backups (fast, good ratio). Use `xz -6` for long-term archive storage where speed doesn't matter.

---

## 5. tar Command Mastery

`tar` (Tape ARchive) is the backbone of Linux backup. It bundles files into a single archive while preserving permissions, ownership, and symlinks.

### Core Flags Reference

| Flag | Long Form | Meaning |
|------|-----------|---------|
| `-c` | `--create` | Create a new archive |
| `-x` | `--extract` | Extract from archive |
| `-v` | `--verbose` | Show files being processed |
| `-f` | `--file` | Specify archive filename |
| `-z` | `--gzip` | Compress/decompress with gzip |
| `-j` | `--bzip2` | Compress/decompress with bzip2 |
| `-J` | `--xz` | Compress/decompress with xz |
| `--zstd` | `--zstd` | Compress/decompress with zstd |
| `-p` | `--preserve-permissions` | Preserve file permissions |
| `-P` | `--absolute-names` | Keep absolute paths (leading /) |
| `--exclude` | `--exclude=PATTERN` | Exclude matching files |
| `-t` | `--list` | List archive contents |
| `-C` | `--directory` | Change to directory before operation |
| `--numeric-owner` | | Use numeric UID/GID |
| `-g` | `--listed-incremental` | Incremental backup with snapshot file |

---

### Creating Archives

```bash
# Basic tar archive (no compression)
tar -cf archive.tar /path/to/backup/

# With gzip compression
tar -czf archive.tar.gz /path/to/backup/

# With bzip2 compression
tar -cjf archive.tar.bz2 /path/to/backup/

# With xz compression
tar -cJf archive.tar.xz /path/to/backup/

# With zstd compression
tar -cf - /path/to/backup/ | zstd -3 > archive.tar.zst
# OR (tar 1.31+)
tar --zstd -cf archive.tar.zst /path/to/backup/

# Verbose output (see what's being archived)
tar -czvf archive.tar.gz /var/www/html/

# Preserve permissions and ownership (recommended for system backups)
tar -czpf archive.tar.gz /etc/

# Use numeric UID/GID (portable across systems)
tar --numeric-owner -czpf archive.tar.gz /home/
```

---

### Extracting Archives

```bash
# Extract to current directory
tar -xf archive.tar
tar -xzf archive.tar.gz
tar -xjf archive.tar.bz2
tar -xJf archive.tar.xz

# Extract to specific directory
tar -xzf archive.tar.gz -C /restore/path/

# Extract with verbose output
tar -xzvf archive.tar.gz

# Extract single file from archive
tar -xzf archive.tar.gz etc/nginx/nginx.conf

# Extract preserving permissions
tar -xzpf archive.tar.gz -C /

# Preview archive contents without extracting
tar -tzf archive.tar.gz
tar -tjf archive.tar.bz2 | less
```

---

### Excluding Files and Directories

```bash
# Exclude a specific directory
tar -czf backup.tar.gz /var/www/ --exclude=/var/www/cache

# Exclude multiple patterns
tar -czf backup.tar.gz /home/ \
  --exclude='*.log' \
  --exclude='*.tmp' \
  --exclude='.git' \
  --exclude='node_modules'

# Exclude from a file
cat > /tmp/exclude.txt << 'EOF'
/proc
/sys
/dev
/run
/tmp
/var/cache
/var/tmp
/var/log
*.swp
*.tmp
node_modules
.git
EOF

tar -czpf /backup/full.tar.gz / \
  --exclude-from=/tmp/exclude.txt
```

---

### Full System Backup with tar

```bash
# Production full server backup
tar -czpf /backup/server-$(hostname)-$(date +%Y%m%d).tar.gz \
  --exclude=/proc \
  --exclude=/sys \
  --exclude=/dev \
  --exclude=/run \
  --exclude=/tmp \
  --exclude=/var/tmp \
  --exclude=/var/cache \
  --exclude=/mnt \
  --exclude=/media \
  --exclude=/lost+found \
  --exclude=/backup \
  /

# Verify the archive after creation
tar -tzf /backup/server-$(hostname)-$(date +%Y%m%d).tar.gz | tail -20
echo "Archive exit code: $?"
```

---

### Incremental Backups with tar

tar supports incremental backups using a **snapshot file** (`-g` flag). The snapshot file tracks which files have changed.

```bash
# LEVEL 0: Create full backup (snapshot file created)
tar -czpf /backup/full-$(date +%Y%m%d).tar.gz \
  -g /backup/snapshot.snar \
  /var/www/ /etc/ /home/

# LEVEL 1: Incremental (only changed files since last backup)
tar -czpf /backup/incr-$(date +%Y%m%d).tar.gz \
  -g /backup/snapshot.snar \
  /var/www/ /etc/ /home/

# To start fresh incrementals, delete and recreate snapshot
# Weekly: delete snapshot.snar, run full backup
# Daily: keep snapshot.snar, run incremental
```

---

### Streaming Backup Over SSH

```bash
# Backup local system, send directly to remote server
tar -czp \
  --exclude=/proc --exclude=/sys --exclude=/dev --exclude=/run \
  / | ssh user@remote-server "cat > /backup/server-$(date +%Y%m%d).tar.gz"

# Backup remote server directly to local storage
ssh user@remote-server \
  "tar -czp --exclude=/proc --exclude=/sys --exclude=/dev / " \
  > /local/backup/remote-server-$(date +%Y%m%d).tar.gz

# Backup with progress bar (requires pv)
tar -czp / | pv | ssh user@backup-server "cat > /backup/full.tar.gz"
```

---

### Testing Archive Integrity

```bash
# Test gzip integrity
gzip -t backup.tar.gz && echo "OK" || echo "CORRUPTED"

# Test tar archive
tar -tzf backup.tar.gz > /dev/null && echo "OK" || echo "CORRUPTED"

# Verbose integrity test (list files while testing)
tar -tvzf backup.tar.gz | wc -l

# Compare archive to filesystem (verify contents match)
tar -dzf backup.tar.gz
# d = diff mode, shows files that differ from archive
```

---

## 6. Full VPS Backup Strategies

### Universal Full VPS Backup Command

Works on Ubuntu, Debian, CentOS, AlmaLinux, Rocky Linux:

```bash
#!/bin/bash
# ============================================================
# full-backup.sh - Production Full VPS Backup
# ============================================================

BACKUP_DIR="/backup"
HOSTNAME=$(hostname)
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="${BACKUP_DIR}/fullbackup-${HOSTNAME}-${DATE}.tar.gz"
EXCLUDE_FILE="/etc/backup/exclude.txt"
LOG_FILE="/var/log/backup.log"

# Create backup directory if it doesn't exist
mkdir -p "$BACKUP_DIR"

# Create exclusion list
cat > "$EXCLUDE_FILE" << 'EOF'
/proc
/sys
/dev
/run
/tmp
/var/tmp
/var/cache/apt
/var/cache/yum
/var/cache/dnf
/mnt
/media
/lost+found
/backup
/var/log/journal
EOF

echo "[$(date)] Starting full backup to $BACKUP_FILE" >> "$LOG_FILE"

tar -czpf "$BACKUP_FILE" \
  --exclude-from="$EXCLUDE_FILE" \
  --numeric-owner \
  /

EXIT_CODE=$?

if [ $EXIT_CODE -eq 0 ]; then
  SIZE=$(du -sh "$BACKUP_FILE" | cut -f1)
  echo "[$(date)] Backup completed: $BACKUP_FILE ($SIZE)" >> "$LOG_FILE"
else
  echo "[$(date)] ERROR: Backup failed with exit code $EXIT_CODE" >> "$LOG_FILE"
  exit 1
fi

# Verify backup integrity
tar -tzf "$BACKUP_FILE" > /dev/null 2>&1
if [ $? -eq 0 ]; then
  echo "[$(date)] Integrity check passed" >> "$LOG_FILE"
else
  echo "[$(date)] ERROR: Integrity check FAILED" >> "$LOG_FILE"
  exit 1
fi
```

---

### Ubuntu/Debian-Specific Notes

```bash
# Package list backup (critical for exact restore)
dpkg --get-selections > /backup/package-list.txt
apt-mark showmanual > /backup/manual-packages.txt

# Restore packages on new system
dpkg --set-selections < /backup/package-list.txt
apt-get dselect-upgrade

# Or with manual list
xargs apt-get install -y < /backup/manual-packages.txt

# Backup apt sources
cp -r /etc/apt/ /backup/apt-config/

# Backup snap packages
snap list > /backup/snap-list.txt
```

---

### CentOS/AlmaLinux/Rocky Linux-Specific Notes

```bash
# Package list backup
rpm -qa > /backup/rpm-packages.txt
dnf history > /backup/dnf-history.txt

# Restore packages
dnf install $(cat /backup/rpm-packages.txt)

# Backup repo configuration
cp -r /etc/yum.repos.d/ /backup/yum-repos/

# Backup systemd services state
systemctl list-unit-files --state=enabled > /backup/enabled-services.txt
```

---

### Configuration-Only Backup (Fast Daily Backup)

```bash
#!/bin/bash
# config-backup.sh - Backs up only critical config and data

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backup/configs"
mkdir -p "$BACKUP_DIR"

tar -czpf "${BACKUP_DIR}/configs-${DATE}.tar.gz" \
  /etc/ \
  /root/ \
  /home/ \
  /var/www/ \
  /srv/ \
  /opt/ \
  --exclude='/var/www/*/cache' \
  --exclude='/var/www/*/.git' \
  --exclude='/home/*/.cache' \
  --exclude='/home/*/node_modules'

echo "Config backup: ${BACKUP_DIR}/configs-${DATE}.tar.gz"
```

---

## 7. Full Server Backup Commands

### tar Full Backup

```bash
# Minimal full server backup
tar -czpf /backup/full.tar.gz \
  --exclude=/proc --exclude=/sys --exclude=/dev \
  --exclude=/run --exclude=/tmp --exclude=/backup \
  /

# Check backup size
du -sh /backup/full.tar.gz

# List top-level backup contents
tar -tzf /backup/full.tar.gz | head -50
```

---

### rsync Backup

```bash
# Local directory mirror
rsync -avz --delete /source/ /backup/mirror/

# Remote server backup via SSH
rsync -avz -e ssh /var/www/ user@backup-server:/backup/www/

# Full server backup with exclusions
rsync -avzAX \
  --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found"} \
  / user@backup-server:/backup/$(hostname)/
```

---

### dd Disk Imaging

`dd` creates a **raw block-level image** of an entire disk or partition.

```bash
# Create full disk image
dd if=/dev/vda of=/backup/disk.img bs=4M status=progress

# Compress on the fly
dd if=/dev/vda bs=4M | gzip > /backup/disk.img.gz

# Or with faster zstd
dd if=/dev/vda bs=4M | zstd -3 > /backup/disk.img.zst

# Restore from image
dd if=/backup/disk.img of=/dev/vda bs=4M status=progress

# Restore compressed image
zstd -d /backup/disk.img.zst --stdout | dd of=/dev/vda bs=4M status=progress

# Single partition backup
dd if=/dev/vda1 bs=4M | gzip > /backup/boot-partition.img.gz
```

> ⚠️ **Warning:** `dd` requires the source disk to be unmounted or the server to be offline for consistent images. Running `dd` on a live system may produce inconsistent images.

---

### LVM Snapshots

LVM (Logical Volume Manager) snapshots are the safest way to back up live Linux systems.

```bash
# Check LVM setup
pvs    # Physical volumes
vgs    # Volume groups
lvs    # Logical volumes

# Create snapshot (requires free space in volume group)
lvcreate -L10G -s -n mydata_snap /dev/vg0/mydata

# Mount snapshot read-only
mkdir -p /mnt/snapshot
mount -o ro /dev/vg0/mydata_snap /mnt/snapshot

# Backup from snapshot (consistent, no application impact)
tar -czpf /backup/lvm-backup.tar.gz -C /mnt/snapshot .

# Or rsync from snapshot
rsync -avz /mnt/snapshot/ user@backup:/backup/

# Cleanup
umount /mnt/snapshot
lvremove -y /dev/vg0/mydata_snap
```

---

### Clonezilla (Bare-Metal Imaging)

Clonezilla is a partition/disk cloning tool ideal for full bare-metal restore.

```bash
# Clonezilla is run from a bootable USB, not the live system
# Key commands run inside Clonezilla environment:

# Save disk to image
clonezilla -x # GUI mode on boot

# CLI: disk-to-local image
ocs-sr -q2 -c -j2 -z1p -i 4096 -sfsck -senc savedisk \
  my_disk_image sda

# Restore from image
ocs-sr -g auto -e1 auto -e2 -r -j2 -p true restoredisk \
  my_disk_image sda
```

---

### External Disk Backup

```bash
# Mount external disk
mkdir -p /mnt/external
mount /dev/sdb1 /mnt/external

# Run backup to external disk
tar -czpf /mnt/external/backup-$(date +%Y%m%d).tar.gz \
  --exclude=/proc --exclude=/sys --exclude=/dev \
  --exclude=/run --exclude=/tmp --exclude=/mnt \
  /

# Verify and unmount
tar -tzf /mnt/external/backup-$(date +%Y%m%d).tar.gz > /dev/null
umount /mnt/external
```

---

## 8. rsync Deep Dive

`rsync` is the most powerful and flexible tool for incremental, network-aware file synchronization and backup.

### rsync Flag Reference

| Flag | Meaning |
|------|---------|
| `-a` | Archive mode: `-rlptgoD` (recursive, links, permissions, times, group, owner, devices) |
| `-v` | Verbose |
| `-z` | Compress during transfer |
| `-P` | Show progress + partial transfers |
| `--delete` | Delete files at destination that no longer exist at source |
| `--exclude` | Exclude files/patterns |
| `--exclude-from` | Read exclusion patterns from file |
| `-n` | Dry run (test without making changes) |
| `--checksum` | Use checksum instead of timestamp for change detection |
| `--bwlimit` | Limit bandwidth (KB/s) |
| `-e` | Specify remote shell (e.g., `-e ssh`) |
| `--stats` | Show transfer statistics |
| `--log-file` | Write log to file |
| `-A` | Preserve ACLs |
| `-X` | Preserve extended attributes |
| `--backup` | Keep backups of changed files |
| `--backup-dir` | Where to store backups of changed files |
| `--link-dest` | Hard-link unchanged files to a reference directory |

---

### Basic rsync Examples

```bash
# Sync two local directories
rsync -av /source/ /destination/

# IMPORTANT: trailing slash on source
# /source/  = sync contents of source INTO destination
# /source   = sync source directory itself into destination

# Mirror (delete files at dest not in source)
rsync -av --delete /source/ /destination/

# Dry run first (always recommended before --delete)
rsync -avnP --delete /source/ /destination/
```

---

### Remote Backup via SSH

```bash
# Push local to remote
rsync -avzP -e "ssh -p 22" /var/www/ user@192.168.1.100:/backup/www/

# Pull remote to local
rsync -avzP -e "ssh -p 22" user@192.168.1.100:/var/www/ /backup/www/

# Custom SSH key
rsync -avzP -e "ssh -i /root/.ssh/backup_key -p 2222" \
  /etc/ user@backup-server:/backup/etc/

# SSH with compression disabled (for already-compressed data)
rsync -avP -e "ssh -o Compression=no" \
  /backup/archive.tar.gz user@remote:/backup/
```

---

### Incremental Backup with --link-dest

This technique creates **space-efficient incremental backups** where unchanged files are hard-linked (not copied).

```bash
#!/bin/bash
# incremental-rsync.sh — Space-efficient incremental backups

SOURCE="/"
DEST="/backup/incremental"
DATE=$(date +%Y-%m-%d_%H%M%S)
LATEST="$DEST/latest"
CURRENT="$DEST/$DATE"

EXCLUDES=(
  --exclude=/proc
  --exclude=/sys
  --exclude=/dev
  --exclude=/run
  --exclude=/tmp
  --exclude=/backup
)

mkdir -p "$CURRENT"

rsync -avzAX \
  "${EXCLUDES[@]}" \
  --link-dest="$LATEST" \
  / "$CURRENT/"

# Update latest symlink
ln -snf "$CURRENT" "$LATEST"

echo "Backup complete: $CURRENT"
```

Each backup appears to be a full backup on disk, but only changed files consume new space. Unchanged files are hard links to previous backups.

---

### Bandwidth Optimization

```bash
# Limit bandwidth to 5 MB/s (40000 KB/s)
rsync -avzP --bwlimit=40000 /backup/ user@remote:/backup/

# Use SSH compression for text-heavy data
rsync -avzP -e "ssh -C" /var/log/ user@remote:/backup/logs/

# Use faster compression algorithm
rsync -avP --compress-choice=zstd /large/ user@remote:/backup/

# Checksum-based sync (slower but more accurate than mtime)
rsync -avzc /important/ user@remote:/backup/important/
```

---

### Production rsync Backup Script

```bash
#!/bin/bash
# rsync-backup.sh — Production remote backup script

REMOTE_USER="backup"
REMOTE_HOST="backup.example.com"
REMOTE_DIR="/backups/$(hostname)"
SSH_KEY="/root/.ssh/backup_rsa"
LOG="/var/log/rsync-backup.log"
MAX_ATTEMPTS=3

exec >> "$LOG" 2>&1
echo "=== Backup started: $(date) ==="

for attempt in $(seq 1 $MAX_ATTEMPTS); do
  rsync -avzAXP \
    --delete \
    --exclude={"/proc/*","/sys/*","/dev/*","/run/*","/tmp/*","/var/tmp/*","/var/cache/*"} \
    -e "ssh -i $SSH_KEY -o StrictHostKeyChecking=accept-new" \
    / \
    "${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/"
  
  if [ $? -eq 0 ]; then
    echo "=== Backup successful: $(date) ==="
    exit 0
  else
    echo "Attempt $attempt failed, retrying..."
    sleep 30
  fi
done

echo "=== BACKUP FAILED after $MAX_ATTEMPTS attempts ==="
# Send alert (replace with your alerting method)
mail -s "BACKUP FAILED: $(hostname)" admin@example.com < "$LOG"
exit 1
```

---

## 9. Database Backup Inside VPS

### MySQL / MariaDB Backup

```bash
# Single database dump
mysqldump -u root -p database_name > /backup/db_name-$(date +%Y%m%d).sql

# With compression
mysqldump -u root -p database_name | gzip > /backup/db_name-$(date +%Y%m%d).sql.gz

# All databases
mysqldump -u root -p --all-databases > /backup/all-databases-$(date +%Y%m%d).sql

# All databases + stored procedures + events
mysqldump -u root -p \
  --all-databases \
  --routines \
  --events \
  --triggers \
  | gzip > /backup/mysql-full-$(date +%Y%m%d).sql.gz

# Non-interactive with password in config
cat > /root/.my.cnf << 'EOF'
[client]
user=root
password=YOUR_PASSWORD
EOF
chmod 600 /root/.my.cnf

# Now run without -p prompt
mysqldump --all-databases | gzip > /backup/mysql-$(date +%Y%m%d).sql.gz

# Restore
gunzip < /backup/all-databases-20240101.sql.gz | mysql -u root -p

# Restore single database
mysql -u root -p database_name < /backup/db_name-20240101.sql
```

---

### PostgreSQL Backup

```bash
# Single database dump
pg_dump -U postgres database_name > /backup/pg_db-$(date +%Y%m%d).sql

# Compressed dump (custom format — faster restore)
pg_dump -U postgres -Fc database_name > /backup/pg_db-$(date +%Y%m%d).dump

# All databases
pg_dumpall -U postgres > /backup/pg_all-$(date +%Y%m%d).sql

# Run as postgres user
sudo -u postgres pg_dumpall | gzip > /backup/pg_all-$(date +%Y%m%d).sql.gz

# Restore custom format (fast, parallel)
pg_restore -U postgres -d database_name /backup/pg_db-20240101.dump

# Restore SQL dump
psql -U postgres database_name < /backup/pg_db-20240101.sql

# Continuous archiving (WAL archiving for point-in-time recovery)
# In postgresql.conf:
# archive_mode = on
# archive_command = 'cp %p /backup/wal_archive/%f'
```

---

### MongoDB Backup

```bash
# Dump entire MongoDB instance
mongodump --out /backup/mongo-$(date +%Y%m%d)/

# Specific database
mongodump --db myapp --out /backup/mongo-$(date +%Y%m%d)/

# With authentication
mongodump --uri="mongodb://user:pass@localhost:27017" \
  --out /backup/mongo-$(date +%Y%m%d)/

# Compressed
mongodump --archive=/backup/mongo-$(date +%Y%m%d).gz --gzip

# Restore
mongorestore /backup/mongo-20240101/

# Restore compressed
mongorestore --archive=/backup/mongo-20240101.gz --gzip

# Restore specific database
mongorestore --db myapp /backup/mongo-20240101/myapp/
```

---

### Redis Backup

```bash
# Method 1: Save RDB snapshot via redis-cli
redis-cli BGSAVE
# Wait for completion
redis-cli LASTSAVE

# Copy the RDB file
cp /var/lib/redis/dump.rdb /backup/redis-$(date +%Y%m%d).rdb

# Method 2: Direct file copy (when Redis is stopped)
systemctl stop redis
cp /var/lib/redis/dump.rdb /backup/redis-$(date +%Y%m%d).rdb
systemctl start redis

# Enable AOF (Append Only File) for better durability
# In redis.conf:
# appendonly yes
# appendfilename "appendonly.aof"

# Backup AOF file
cp /var/lib/redis/appendonly.aof /backup/redis-aof-$(date +%Y%m%d).aof

# Restore: copy RDB to Redis data dir and restart
systemctl stop redis
cp /backup/redis-20240101.rdb /var/lib/redis/dump.rdb
chown redis:redis /var/lib/redis/dump.rdb
systemctl start redis
```

---

### Database Backup Integration Script

```bash
#!/bin/bash
# db-backup.sh — Backup all databases

BACKUP_DIR="/backup/databases"
DATE=$(date +%Y%m%d_%H%M%S)
LOG="/var/log/db-backup.log"

mkdir -p "$BACKUP_DIR"

echo "[$(date)] Starting database backups" >> "$LOG"

# MySQL
if command -v mysqldump &>/dev/null; then
  mysqldump --all-databases --routines --events \
    | gzip > "${BACKUP_DIR}/mysql-${DATE}.sql.gz"
  echo "[$(date)] MySQL backup complete" >> "$LOG"
fi

# PostgreSQL
if command -v pg_dumpall &>/dev/null; then
  sudo -u postgres pg_dumpall \
    | gzip > "${BACKUP_DIR}/postgres-${DATE}.sql.gz"
  echo "[$(date)] PostgreSQL backup complete" >> "$LOG"
fi

# MongoDB
if command -v mongodump &>/dev/null; then
  mongodump --archive="${BACKUP_DIR}/mongo-${DATE}.gz" --gzip
  echo "[$(date)] MongoDB backup complete" >> "$LOG"
fi

# Redis
if [ -f /var/lib/redis/dump.rdb ]; then
  redis-cli BGSAVE
  sleep 2
  cp /var/lib/redis/dump.rdb "${BACKUP_DIR}/redis-${DATE}.rdb"
  echo "[$(date)] Redis backup complete" >> "$LOG"
fi

echo "[$(date)] All database backups completed" >> "$LOG"
```

---

## 10. Docker Backup Strategies

### Docker Volume Backup

```bash
# List Docker volumes
docker volume ls

# Backup a named volume
docker run --rm \
  -v myapp_data:/source:ro \
  -v /backup:/backup \
  alpine tar -czf /backup/volume-myapp_data-$(date +%Y%m%d).tar.gz -C /source .

# Restore a volume from backup
docker volume create myapp_data
docker run --rm \
  -v myapp_data:/target \
  -v /backup:/backup \
  alpine sh -c "cd /target && tar -xzf /backup/volume-myapp_data-20240101.tar.gz"
```

---

### docker-compose Stack Backup

```bash
#!/bin/bash
# docker-compose-backup.sh

COMPOSE_DIR="/opt/myapp"
BACKUP_DIR="/backup/docker"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# 1. Backup docker-compose files and config
tar -czf "${BACKUP_DIR}/compose-config-${DATE}.tar.gz" \
  "${COMPOSE_DIR}/"

# 2. Get all volume names from compose
cd "$COMPOSE_DIR"
VOLUMES=$(docker-compose config --volumes 2>/dev/null)

# 3. Backup each volume
for VOL in $VOLUMES; do
  FULL_VOL="${COMPOSE_DIR##*/}_${VOL}"
  echo "Backing up volume: $FULL_VOL"
  docker run --rm \
    -v "${FULL_VOL}:/source:ro" \
    -v "${BACKUP_DIR}:/backup" \
    alpine tar -czf "/backup/vol-${FULL_VOL}-${DATE}.tar.gz" \
    -C /source .
done

echo "Docker backup complete: $BACKUP_DIR"
```

---

### Container Export and Image Backup

```bash
# Export container filesystem (not image layers)
docker export container_name > /backup/container-$(date +%Y%m%d).tar
docker export container_name | gzip > /backup/container-$(date +%Y%m%d).tar.gz

# Save Docker image
docker save image_name:tag | gzip > /backup/image-$(date +%Y%m%d).tar.gz

# Save all images
docker save $(docker images -q) | gzip > /backup/all-images-$(date +%Y%m%d).tar.gz

# Load image from backup
docker load < /backup/image-20240101.tar.gz

# Import exported container as image
docker import /backup/container-20240101.tar my-restored-image:latest
```

---

### Persistent Storage Backup Pattern

```bash
#!/bin/bash
# Full Docker environment backup

BACKUP_DIR="/backup/docker-full"
DATE=$(date +%Y%m%d_%H%M%S)
mkdir -p "$BACKUP_DIR"

# Stop all containers for consistency (optional)
# docker stop $(docker ps -q)

# Backup all named volumes
docker volume ls -q | while read VOL; do
  echo "Backing up volume: $VOL"
  docker run --rm \
    -v "${VOL}:/data:ro" \
    -v "${BACKUP_DIR}:/backup" \
    busybox tar -czf "/backup/vol-${VOL}-${DATE}.tar.gz" -C /data .
done

# Backup Docker configs and compose files
find /opt -name "docker-compose*.yml" -exec tar -rzvf \
  "${BACKUP_DIR}/compose-files-${DATE}.tar" {} \;

# Backup /etc/docker
tar -czf "${BACKUP_DIR}/docker-config-${DATE}.tar.gz" /etc/docker/

echo "Docker backup complete"
```

---

## 11. Cloud VPS Snapshot Systems

### DigitalOcean Snapshots

```
Pros:
  ✓ One-click, no agent required
  ✓ Stored outside the Droplet
  ✓ Can create new Droplet from snapshot
  ✓ Supports cross-region transfer

Cons:
  ✗ $0.06/GB/month storage cost
  ✗ Droplet must be powered off for consistent snapshot (or use live)
  ✗ No incremental snapshots (each is full)
  ✗ Not a substitute for application-consistent database backups

Restore workflow:
  Snapshots → Select snapshot → Create Droplet
  OR: Droplet → Restore from snapshot
```

```bash
# DigitalOcean CLI (doctl)
doctl compute droplet-action snapshot DROPLET_ID --snapshot-name "backup-$(date +%Y%m%d)"
doctl compute snapshot list
doctl compute droplet create --snapshot-id SNAPSHOT_ID my-restored-server
```

---

### AWS EC2 Snapshots (EBS)

```bash
# AWS CLI — Create snapshot
aws ec2 create-snapshot \
  --volume-id vol-xxxxxxxxxx \
  --description "Backup $(date +%Y-%m-%d)"

# Create AMI (full server image)
aws ec2 create-image \
  --instance-id i-xxxxxxxxxx \
  --name "server-backup-$(date +%Y%m%d)" \
  --no-reboot

# List snapshots
aws ec2 describe-snapshots --owner-ids self

# Restore: launch instance from AMI
aws ec2 run-instances \
  --image-id ami-xxxxxxxxxx \
  --instance-type t3.medium \
  --key-name mykey
```

---

### Google Cloud Snapshots

```bash
# gcloud CLI — Create disk snapshot
gcloud compute disks snapshot DISK_NAME \
  --snapshot-names "backup-$(date +%Y%m%d)" \
  --zone us-central1-a

# List snapshots
gcloud compute snapshots list

# Create disk from snapshot
gcloud compute disks create restored-disk \
  --source-snapshot backup-20240101 \
  --zone us-central1-a

# Create instance from disk
gcloud compute instances create restored-server \
  --disk name=restored-disk,boot=yes \
  --zone us-central1-a
```

---

### Cloud Snapshot Comparison Table

| Provider | Storage Cost | Incremental | Cross-Region | Consistent | Max Retention |
|----------|-------------|-------------|--------------|------------|---------------|
| DigitalOcean | $0.06/GB/mo | No | Yes | Crash | Unlimited |
| AWS EBS | $0.05/GB/mo | Yes | Yes | Crash/App | Configurable |
| Google Cloud | $0.026/GB/mo | Yes | Yes | Crash | Configurable |
| Azure | $0.05/GB/mo | Yes | Yes | App | Configurable |
| Hetzner | €0.011/GB/mo | No | No | Crash | Unlimited |
| Vultr | $0.05/GB/mo | No | No | Crash | Unlimited |

> ⚠️ **Critical Note:** Cloud snapshots are NOT a backup strategy by themselves. They are subject to the same cloud provider failure that might take your VPS offline. Always maintain **independent offsite backups**.

---

## 12. Automated Backup Systems

### cron Job Basics

```bash
# Edit root crontab
crontab -e

# Crontab format:
# MIN HOUR DOM MON DOW COMMAND
# ─┬─  ─┬─  ─┬─ ─┬─ ─┬─
#  │    │    │   │   └─ Day of week (0=Sun, 7=Sun)
#  │    │    │   └───── Month (1-12)
#  │    │    └───────── Day of month (1-31)
#  │    └────────────── Hour (0-23)
#  └─────────────────── Minute (0-59)

# Examples:
0 2 * * *     /usr/local/bin/daily-backup.sh   # Daily at 2:00 AM
0 3 * * 0     /usr/local/bin/weekly-backup.sh  # Weekly Sunday 3:00 AM
0 1 1 * *     /usr/local/bin/monthly-backup.sh # Monthly 1st at 1:00 AM
*/30 * * * *  /usr/local/bin/db-backup.sh      # Every 30 minutes
```

---

### systemd Timer (Modern Alternative to cron)

```bash
# Create backup service
cat > /etc/systemd/system/vps-backup.service << 'EOF'
[Unit]
Description=VPS Backup Service
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/full-backup.sh
User=root
StandardOutput=journal
StandardError=journal
EOF

# Create backup timer
cat > /etc/systemd/system/vps-backup.timer << 'EOF'
[Unit]
Description=Run VPS Backup Daily at 2AM
Requires=vps-backup.service

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true
RandomizedDelaySec=300

[Install]
WantedBy=timers.target
EOF

# Enable and start timer
systemctl daemon-reload
systemctl enable vps-backup.timer
systemctl start vps-backup.timer

# Check timer status
systemctl list-timers
systemctl status vps-backup.timer

# View backup logs
journalctl -u vps-backup.service -f
```

---

### Backup Rotation & Retention Policy

```bash
#!/bin/bash
# rotate-backups.sh — Keep N backups, delete older ones

BACKUP_DIR="/backup"
KEEP_DAILY=7      # Keep 7 daily backups
KEEP_WEEKLY=4     # Keep 4 weekly backups
KEEP_MONTHLY=6    # Keep 6 monthly backups

# Delete daily backups older than 7 days
find "$BACKUP_DIR/daily/" -name "*.tar.gz" -mtime +$KEEP_DAILY -delete

# Delete weekly backups older than 4 weeks
find "$BACKUP_DIR/weekly/" -name "*.tar.gz" -mtime +$((KEEP_WEEKLY * 7)) -delete

# Delete monthly backups older than 6 months
find "$BACKUP_DIR/monthly/" -name "*.tar.gz" -mtime +$((KEEP_MONTHLY * 30)) -delete

echo "Backup rotation completed: $(date)"
```

---

### Complete Automated Backup System

```bash
#!/bin/bash
# automated-backup-system.sh
# Features: full/incremental logic, rotation, S3 upload, alerting

set -euo pipefail

# === Configuration ===
HOSTNAME=$(hostname)
BACKUP_BASE="/backup"
DATE=$(date +%Y%m%d)
DAY_OF_WEEK=$(date +%u)  # 1=Mon, 7=Sun
LOG="/var/log/backup-system.log"
S3_BUCKET="s3://my-backup-bucket/${HOSTNAME}"
ALERT_EMAIL="admin@example.com"
RETENTION_DAILY=7
RETENTION_WEEKLY=4

# === Setup ===
mkdir -p "$BACKUP_BASE"/{daily,weekly,monthly}
exec >> "$LOG" 2>&1
echo "============================================"
echo "Backup started: $(date)"

backup_failed() {
  echo "BACKUP FAILED: $1"
  echo "Backup failed on $(hostname) at $(date): $1" | \
    mail -s "BACKUP FAILURE: $HOSTNAME" "$ALERT_EMAIL" 2>/dev/null || true
  exit 1
}

# === Database Backups ===
echo "--- Database backups ---"
DB_BACKUP="${BACKUP_BASE}/databases-${DATE}.sql.gz"
mysqldump --all-databases --routines --events 2>/dev/null \
  | gzip > "$DB_BACKUP" || backup_failed "MySQL backup failed"
echo "MySQL backup: OK ($(du -sh "$DB_BACKUP" | cut -f1))"

# === Filesystem Backup ===
echo "--- Filesystem backup ---"

if [ "$DAY_OF_WEEK" -eq 7 ]; then
  # Sunday: Full backup
  BACKUP_TYPE="weekly"
  DEST="${BACKUP_BASE}/weekly/full-${DATE}.tar.gz"
  SNAPSHOT=""
else
  # Other days: Incremental
  BACKUP_TYPE="daily"
  DEST="${BACKUP_BASE}/daily/incr-${DATE}.tar.gz"
  SNAPSHOT="-g ${BACKUP_BASE}/.snapshot.snar"
fi

echo "Running $BACKUP_TYPE backup..."

tar -czpf "$DEST" $SNAPSHOT \
  --exclude=/proc --exclude=/sys --exclude=/dev \
  --exclude=/run --exclude=/tmp --exclude=/backup \
  --exclude=/var/cache --exclude=/var/tmp \
  / || backup_failed "tar backup failed"

echo "$BACKUP_TYPE backup: OK ($(du -sh "$DEST" | cut -f1))"

# === Upload to S3 ===
if command -v aws &>/dev/null; then
  echo "--- Uploading to S3 ---"
  aws s3 cp "$DEST" "${S3_BUCKET}/${BACKUP_TYPE}/" \
    --storage-class STANDARD_IA || echo "WARNING: S3 upload failed"
  aws s3 cp "$DB_BACKUP" "${S3_BUCKET}/databases/" \
    --storage-class STANDARD_IA || echo "WARNING: DB S3 upload failed"
fi

# === Rotation ===
echo "--- Rotating old backups ---"
find "${BACKUP_BASE}/daily/" -name "*.tar.gz" -mtime +$RETENTION_DAILY -delete
find "${BACKUP_BASE}/weekly/" -name "*.tar.gz" -mtime +$((RETENTION_WEEKLY * 7)) -delete

echo "Backup completed successfully: $(date)"
echo "============================================"
```

---

## 13. Cloud Storage Backup

### rclone — Universal Cloud Storage Tool

`rclone` syncs files to 40+ cloud storage providers using a unified interface.

```bash
# Install rclone
curl https://rclone.org/install.sh | bash

# Configure (interactive setup)
rclone config

# Sync to AWS S3
rclone sync /backup/ s3:my-bucket/backups/ --progress

# Sync to Backblaze B2
rclone sync /backup/ b2:my-bucket/backups/ --progress

# Sync to Google Drive
rclone sync /backup/ gdrive:backups/ --progress

# Sync to Wasabi
rclone sync /backup/ wasabi:my-bucket/backups/ --progress

# Copy (does not delete destination files)
rclone copy /backup/ s3:my-bucket/ --progress

# List remote files
rclone ls s3:my-bucket/backups/

# Dry run
rclone sync --dry-run /backup/ s3:my-bucket/

# With bandwidth limit (10 MB/s)
rclone sync /backup/ s3:my-bucket/ --bwlimit 10M

# Encrypted remote
rclone config  # Create crypt remote wrapping your storage remote
rclone sync /backup/ encrypted-remote:/ --progress
```

---

### AWS S3 Backup

```bash
# Install AWS CLI
apt install awscli

# Configure credentials
aws configure
# Or use IAM role on EC2 (no credentials needed)

# Upload single file
aws s3 cp /backup/full.tar.gz s3://my-bucket/backups/

# Sync directory to S3
aws s3 sync /backup/ s3://my-bucket/backups/ \
  --storage-class STANDARD_IA \
  --sse AES256

# Upload with lifecycle (auto-delete after 90 days)
# Set lifecycle policy in S3 console or via CLI

# List backups
aws s3 ls s3://my-bucket/backups/

# Download from S3
aws s3 cp s3://my-bucket/backups/full.tar.gz /restore/

# Enable versioning (protection against accidental deletion)
aws s3api put-bucket-versioning \
  --bucket my-bucket \
  --versioning-configuration Status=Enabled
```

---

### Backblaze B2 (Cost-Effective Alternative)

```bash
# B2 pricing: $0.006/GB/month (83% cheaper than S3)
# Free egress to Cloudflare-partnered services

# Install B2 CLI
pip install b2

# Authorize
b2 authorize-account APPLICATION_KEY_ID APPLICATION_KEY

# Create bucket
b2 create-bucket my-backup-bucket allPrivate

# Upload
b2 upload-file my-backup-bucket /backup/full.tar.gz backups/full.tar.gz

# Sync
b2 sync /backup/ b2://my-backup-bucket/backups/

# Or via rclone (recommended)
rclone sync /backup/ b2:my-backup-bucket/backups/ --progress
```

---

### Encrypted Offsite Backup Script

```bash
#!/bin/bash
# encrypted-cloud-backup.sh
# Encrypts with GPG before uploading to cloud

BACKUP_FILE="/backup/full-$(date +%Y%m%d).tar.gz"
GPG_RECIPIENT="backup@example.com"  # Your GPG key email
S3_BUCKET="s3://my-secure-backup-bucket"

# Create backup
tar -czpf "$BACKUP_FILE" \
  --exclude=/proc --exclude=/sys --exclude=/dev --exclude=/run --exclude=/tmp \
  /etc/ /var/www/ /home/ /root/

# Encrypt with GPG
gpg --recipient "$GPG_RECIPIENT" \
  --encrypt \
  --compress-algo none \
  "$BACKUP_FILE"

# Upload encrypted file
aws s3 cp "${BACKUP_FILE}.gpg" "${S3_BUCKET}/$(date +%Y/%m/%d)/"

# Remove local unencrypted file
rm -f "$BACKUP_FILE"
echo "Encrypted backup uploaded: $(date)"
```

---

## 14. Encryption & Security

### GPG Encryption for Backups

```bash
# Generate GPG key for backups
gpg --gen-key
# Or non-interactively:
gpg --batch --gen-key << 'EOF'
Key-Type: RSA
Key-Length: 4096
Subkey-Type: RSA
Subkey-Length: 4096
Name-Real: Backup Key
Name-Email: backup@example.com
Expire-Date: 0
%no-passphrase
%commit
EOF

# List keys
gpg --list-keys

# Export public key (store safely — needed for encryption)
gpg --export --armor backup@example.com > /secure/backup-public.asc

# Export private key (KEEP SECURE — needed for decryption)
gpg --export-secret-keys --armor backup@example.com > /secure/backup-private.asc

# Encrypt a file
gpg --recipient backup@example.com --encrypt archive.tar.gz
# Output: archive.tar.gz.gpg

# Decrypt
gpg --decrypt archive.tar.gz.gpg > archive.tar.gz
# Or:
gpg --output archive.tar.gz --decrypt archive.tar.gz.gpg

# Encrypt with symmetric password (no key needed, just passphrase)
gpg --symmetric --cipher-algo AES256 archive.tar.gz

# Decrypt symmetric
gpg --decrypt archive.tar.gz.gpg > archive.tar.gz
```

---

### OpenSSL Encryption

```bash
# Encrypt with AES-256
openssl enc -aes-256-cbc -salt \
  -in archive.tar.gz \
  -out archive.tar.gz.enc \
  -k "YourStrongPassword"

# Decrypt
openssl enc -aes-256-cbc -d \
  -in archive.tar.gz.enc \
  -out archive.tar.gz \
  -k "YourStrongPassword"

# Better: use key file + PBKDF2
openssl enc -aes-256-cbc -pbkdf2 -iter 100000 \
  -in archive.tar.gz \
  -out archive.tar.gz.enc \
  -pass file:/secure/backup.key

# Generate a random key file
openssl rand -base64 32 > /secure/backup.key
chmod 600 /secure/backup.key

# Pipe directly: compress → encrypt
tar -czp /backup/source/ | \
  openssl enc -aes-256-cbc -pbkdf2 -pass file:/secure/backup.key \
  > /backup/encrypted-$(date +%Y%m%d).tar.gz.enc
```

---

### Immutable Backups (Ransomware Protection)

```bash
# S3 Object Lock (immutable for set period)
aws s3api put-object-lock-configuration \
  --bucket my-backup-bucket \
  --object-lock-configuration \
    '{"ObjectLockEnabled":"Enabled","Rule":{"DefaultRetention":{"Mode":"COMPLIANCE","Days":30}}}'

# Linux immutable attribute (prevents deletion even by root)
# After backup is written:
chattr +i /backup/full-20240101.tar.gz

# Remove immutable attribute (needed before deletion)
chattr -i /backup/full-20240101.tar.gz

# List immutable files
lsattr /backup/
# ----i-----------e- full-20240101.tar.gz

# Backblaze B2 Object Lock
# Enable via B2 console or API for WORM compliance
```

---

### Access Control Best Practices

```bash
# Create dedicated backup user (no login shell)
useradd -r -s /usr/sbin/nologin backup-agent

# Create SSH key for backup agent
ssh-keygen -t ed25519 -f /home/backup-agent/.ssh/backup_key -N ""

# Restrict SSH key to rsync-only (in authorized_keys on backup server)
# command="rsync --server --daemon .",no-port-forwarding,no-x11-forwarding \
#   ssh-ed25519 AAAAC3... backup-agent@source

# Restrict backup directory permissions
mkdir -p /backup
chmod 700 /backup
chown root:root /backup

# Backup encryption keys storage
mkdir -p /etc/backup/keys
chmod 700 /etc/backup/keys
chown root:root /etc/backup/keys
```

---

## 15. Disaster Recovery

### Full VPS Restore Workflow

```
╔════════════════════════════════════════════════════════════════╗
║              DISASTER RECOVERY DECISION TREE                   ║
╠════════════════════════════════════════════════════════════════╣
║  Server offline/unrecoverable?                                 ║
║     ├── YES → Provision new VPS → Restore from backup          ║
║     └── NO  → Diagnose issue → Partial restore                 ║
║                                                                ║
║  What type of failure?                                         ║
║     ├── Config corruption → Restore /etc from backup           ║
║     ├── Database corruption → Restore DB from dump             ║
║     ├── App data loss → Restore /var/www from backup           ║
║     ├── Full disk loss → Restore full backup to new disk       ║
║     └── Ransomware → Wipe + full restore from clean backup     ║
╚════════════════════════════════════════════════════════════════╝
```

### Step-by-Step Full Server Restore

```bash
# ============================================================
# FULL SERVER RESTORE PROCEDURE
# Assumes: new VPS is provisioned, backup accessible
# ============================================================

# STEP 1: Boot into rescue mode or new VPS
# Provision new VPS with same OS as backup was taken from

# STEP 2: Access your backup
# Option A: Download from S3
aws s3 cp s3://my-bucket/backups/full-20240101.tar.gz /tmp/

# Option B: Pull from backup server
rsync -avz user@backup-server:/backup/full-20240101.tar.gz /tmp/

# Option C: Decrypt first if encrypted
gpg --output /tmp/full-20240101.tar.gz --decrypt /tmp/full-20240101.tar.gz.gpg

# STEP 3: Extract full backup to /
tar -xzpf /tmp/full-20240101.tar.gz -C / \
  --numeric-owner \
  --preserve-permissions

# STEP 4: Recreate virtual filesystems
mount -t proc proc /proc
mount -t sysfs sys /sys
mount -t devtmpfs dev /dev

# STEP 5: Fix bootloader (if needed)
grub-install /dev/vda
update-grub

# STEP 6: Reinstall packages (if needed, faster than backup)
# Ubuntu/Debian:
apt-get update
xargs apt-get install -y < /backup/manual-packages.txt

# STEP 7: Restore databases
gunzip < /backup/mysql-20240101.sql.gz | mysql -u root -p
sudo -u postgres psql < /backup/postgres-20240101.sql

# STEP 8: Restart services
systemctl daemon-reload
systemctl start nginx mysql php-fpm redis

# STEP 9: Verify
systemctl status nginx
curl -I http://localhost
```

---

### Restoring Specific Components

```bash
# === Restore only /etc ===
tar -xzpf full-backup.tar.gz -C / etc/
systemctl daemon-reload
nginx -t && systemctl reload nginx

# === Restore only web files ===
tar -xzpf full-backup.tar.gz -C / var/www/
chown -R www-data:www-data /var/www/
systemctl reload nginx

# === Restore MySQL only ===
systemctl stop mysql
gunzip < /backup/mysql-20240101.sql.gz | mysql -u root -p
systemctl start mysql

# === Restore Docker stack ===
# 1. Restore compose files
tar -xzf compose-config-20240101.tar.gz -C /opt/

# 2. Restore volumes
for VOL_BACKUP in /backup/docker/vol-*.tar.gz; do
  VOL_NAME=$(basename "$VOL_BACKUP" | sed 's/vol-//' | sed 's/-[0-9_]*.tar.gz//')
  docker volume create "$VOL_NAME"
  docker run --rm \
    -v "${VOL_NAME}:/target" \
    -v "/backup/docker:/backup" \
    alpine sh -c "cd /target && tar -xzf /backup/$(basename $VOL_BACKUP)"
done

# 3. Start stack
cd /opt/myapp && docker-compose up -d
```

---

## 16. VPS Migration Strategies

### Full Server Migration Workflow

```
Phase 1: PREPARE
  ├── Provision new VPS (same or better specs)
  ├── Configure DNS TTL to 60s (prepare for fast cutover)
  └── Set up SSH access to new server

Phase 2: SYNC
  ├── Initial rsync (may take hours for large servers)
  └── Verify data integrity

Phase 3: FINAL SYNC + CUTOVER
  ├── Enable maintenance mode on old server
  ├── Final rsync (only changes since initial sync)
  ├── Restore databases
  ├── Update DNS to new server IP
  └── Test, then decommission old server
```

---

### rsync-Based Migration

```bash
#!/bin/bash
# vps-migrate.sh — Migrate VPS to new server

OLD_SERVER="user@old-server-ip"
NEW_SERVER="user@new-server-ip"
SSH_KEY="/root/.ssh/migrate_key"

echo "=== Phase 1: Initial sync ==="
rsync -avzAX \
  --exclude={"/proc/*","/sys/*","/dev/*","/run/*","/tmp/*"} \
  -e "ssh -i $SSH_KEY" \
  "${OLD_SERVER}:/" \
  "${NEW_SERVER}:/received/"

echo "=== Phase 2: Sync databases ==="
ssh -i "$SSH_KEY" "$OLD_SERVER" \
  "mysqldump --all-databases | gzip" \
  | ssh -i "$SSH_KEY" "$NEW_SERVER" \
  "gzip -d | mysql"

echo "=== Phase 3: Final sync (enable maintenance mode first) ==="
rsync -avzAX \
  --exclude={"/proc/*","/sys/*","/dev/*","/run/*","/tmp/*"} \
  -e "ssh -i $SSH_KEY" \
  "${OLD_SERVER}:/" \
  "${NEW_SERVER}:/received/"

echo "Migration complete — update DNS to new server"
```

---

### Zero-Downtime Migration

```bash
# 1. Set DNS TTL to 60 seconds (do this 24h in advance)
# 2. Set up new server and sync data
# 3. Enable maintenance page on old server
# 4. Final rsync + database sync (takes seconds)
# 5. Update DNS A record to new IP
# 6. Verify site is live on new server
# 7. Keep old server running for 1 hour (DNS propagation)
# 8. Decommission old server

# Enable maintenance mode (Nginx)
cat > /etc/nginx/maintenance.html << 'EOF'
<html><body><h1>Maintenance in progress — back shortly</h1></body></html>
EOF

# Nginx maintenance snippet
# location / {
#   return 503;
# }
# error_page 503 /maintenance.html;

# Verify new server with /etc/hosts before DNS change
echo "NEW_IP  yourdomain.com" >> /etc/hosts
curl -I https://yourdomain.com
# Remove after verification
```

---

### SSL Certificate Migration

```bash
# Copy Let's Encrypt certificates to new server
rsync -avz /etc/letsencrypt/ user@new-server:/etc/letsencrypt/

# Copy certificate permissions
ssh user@new-server "chown -R root:root /etc/letsencrypt && chmod -R 755 /etc/letsencrypt/live"

# On new server: verify certbot is installed and renewal works
certbot certificates
certbot renew --dry-run
```

---

## 17. Monitoring & Verification

### Checksum Verification

```bash
# Generate checksums when creating backup
sha256sum /backup/full-20240101.tar.gz > /backup/full-20240101.tar.gz.sha256

# Verify integrity
sha256sum -c /backup/full-20240101.tar.gz.sha256
# full-20240101.tar.gz: OK

# For multiple files
sha256sum /backup/*.tar.gz > /backup/checksums-$(date +%Y%m%d).sha256

# Verify all
sha256sum -c /backup/checksums-20240101.sha256

# MD5 (faster, less secure, still useful for integrity)
md5sum /backup/full.tar.gz > /backup/full.tar.gz.md5
md5sum -c /backup/full.tar.gz.md5
```

---

### Automated Backup Verification Script

```bash
#!/bin/bash
# verify-backup.sh — Automated backup integrity testing

BACKUP_DIR="/backup"
LOG="/var/log/backup-verify.log"
ALERT_EMAIL="admin@example.com"
ERRORS=0

exec >> "$LOG" 2>&1
echo "=== Backup Verification: $(date) ==="

for BACKUP in $(find "$BACKUP_DIR" -name "*.tar.gz" -newer "$BACKUP_DIR/.last_verified" 2>/dev/null); do
  echo -n "Testing: $BACKUP ... "
  
  # Test archive integrity
  if tar -tzf "$BACKUP" > /dev/null 2>&1; then
    echo "OK ($(du -sh "$BACKUP" | cut -f1))"
  else
    echo "FAILED!"
    echo "CORRUPTED: $BACKUP" | mail -s "Backup Corruption Detected" "$ALERT_EMAIL"
    ERRORS=$((ERRORS + 1))
  fi
done

# Update verification timestamp
touch "$BACKUP_DIR/.last_verified"

if [ $ERRORS -gt 0 ]; then
  echo "=== $ERRORS CORRUPTED backups detected ==="
  exit 1
else
  echo "=== All backups verified OK ==="
fi
```

---

### Restore Testing (Critical)

> ⚠️ **A backup that has never been tested is not a backup — it's a hope.**

```bash
#!/bin/bash
# test-restore.sh — Automated restore test

BACKUP_FILE="/backup/latest.tar.gz"
TEST_DIR="/tmp/restore-test-$(date +%Y%m%d)"

mkdir -p "$TEST_DIR"

echo "Testing restore from: $BACKUP_FILE"

# Extract to test directory
tar -xzpf "$BACKUP_FILE" -C "$TEST_DIR" --strip-components=0

# Check critical files are present
for FILE in etc/nginx/nginx.conf etc/mysql/my.cnf var/www/html/index.html; do
  if [ -f "${TEST_DIR}/${FILE}" ]; then
    echo "✓ $FILE present"
  else
    echo "✗ MISSING: $FILE"
  fi
done

# Cleanup
rm -rf "$TEST_DIR"
echo "Restore test complete"
```

---

### Monitoring Script with Alerting

```bash
#!/bin/bash
# backup-monitor.sh — Check backup freshness and alert

BACKUP_DIR="/backup"
MAX_AGE_HOURS=26  # Alert if no backup newer than 26 hours
ALERT_EMAIL="admin@example.com"

LATEST=$(find "$BACKUP_DIR" -name "*.tar.gz" -printf "%T@ %p\n" | sort -n | tail -1 | cut -d' ' -f2-)
AGE_HOURS=$(( ( $(date +%s) - $(stat -c %Y "$LATEST") ) / 3600 ))

if [ $AGE_HOURS -gt $MAX_AGE_HOURS ]; then
  echo "ALERT: Latest backup is ${AGE_HOURS}h old: $LATEST" \
    | mail -s "BACKUP STALE: $(hostname)" "$ALERT_EMAIL"
  exit 1
fi

# Check backup size (alert if abnormally small)
SIZE_BYTES=$(stat -c %s "$LATEST")
MIN_SIZE=100000000  # 100MB minimum

if [ $SIZE_BYTES -lt $MIN_SIZE ]; then
  echo "ALERT: Backup suspiciously small: $(du -sh "$LATEST")" \
    | mail -s "BACKUP SIZE ALERT: $(hostname)" "$ALERT_EMAIL"
  exit 1
fi

echo "Backup OK: $LATEST (${AGE_HOURS}h old, $(du -sh "$LATEST" | cut -f1))"
```

---

## 18. Production Backup Architecture

### The 3-2-1 Backup Rule

```
┌─────────────────────────────────────────────────────────────────┐
│                    THE 3-2-1 RULE                               │
│                                                                 │
│  3 → Maintain 3 COPIES of your data                            │
│      (1 production + 2 backups)                                │
│                                                                 │
│  2 → Store backups on 2 DIFFERENT MEDIA types                  │
│      (local disk + cloud storage)                               │
│                                                                 │
│  1 → Keep 1 copy OFFSITE                                       │
│      (different datacenter or cloud region)                     │
└─────────────────────────────────────────────────────────────────┘
```

---

### Production Backup Architecture

```
┌───────────────────────────────────────────────────────────────────────────┐
│                     PRODUCTION BACKUP ARCHITECTURE                        │
│                                                                           │
│  PRODUCTION VPS (Primary)                                                 │
│  ┌─────────────────────┐                                                  │
│  │  /var/www           │                                                  │
│  │  /etc               │──── rsync ──────────────────────────────────┐   │
│  │  Databases          │                                              │   │
│  └─────────────────────┘                                              ↓   │
│          │                                                  BACKUP SERVER │
│          │ cron job                                      ┌──────────────┐ │
│          ↓                                               │ /backup/     │ │
│  ┌─────────────────────┐                                │  daily/      │ │
│  │  Local Backup       │                                │  weekly/     │ │
│  │  /backup/ (same VPS)│                                │  monthly/    │ │
│  │  (short retention)  │                                └──────────────┘ │
│  └─────────────────────┘                                      │           │
│          │                                                     │           │
│          │ rclone / aws s3                          rclone / aws s3       │
│          ↓                                                     ↓           │
│  ┌─────────────────────────────────────────────────────────────────────┐  │
│  │                     CLOUD STORAGE (Offsite)                         │  │
│  │                                                                     │  │
│  │   AWS S3 / Backblaze B2 / Google Cloud Storage                      │  │
│  │   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │  │
│  │   │  Region A    │  │  Region B    │  │  Archive     │             │  │
│  │   │  (hot copy)  │  │  (replicated)│  │  (Glacier)   │             │  │
│  │   └──────────────┘  └──────────────┘  └──────────────┘             │  │
│  └─────────────────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────────────────┘
```

---

### Retention Schedule

```
┌──────────────────────────────────────────────────────────────┐
│                  BACKUP RETENTION SCHEDULE                    │
│                                                              │
│  Hourly  ──► Keep 24 hours                                   │
│  Daily   ──► Keep 7 days                                     │
│  Weekly  ──► Keep 4 weeks                                    │
│  Monthly ──► Keep 12 months                                  │
│  Yearly  ──► Keep 7 years (compliance/legal)                 │
│                                                              │
│  Storage Tiers:                                              │
│  Hot   (S3 Standard)     ← Last 7 days                       │
│  Warm  (S3 Standard-IA)  ← 7 days – 3 months                │
│  Cold  (S3 Glacier)      ← 3 months – 7 years               │
└──────────────────────────────────────────────────────────────┘
```

---

### Immutable Backup Storage

```bash
# S3 Bucket with Object Lock — prevents deletion/overwrite
aws s3api create-bucket \
  --bucket my-immutable-backups \
  --object-lock-enabled-for-bucket

# Apply WORM (Write Once Read Many) policy
aws s3api put-object-lock-configuration \
  --bucket my-immutable-backups \
  --object-lock-configuration \
    '{"ObjectLockEnabled":"Enabled","Rule":{"DefaultRetention":{"Mode":"GOVERNANCE","Days":90}}}'

# Upload to immutable storage
aws s3 cp /backup/full.tar.gz \
  s3://my-immutable-backups/backups/ \
  --sse AES256
```

---

## 19. Common Errors & Troubleshooting

### Permission Denied

```bash
# Error: tar: /etc/shadow: Cannot open: Permission denied

# Solution 1: Run as root
sudo tar -czpf backup.tar.gz /etc/

# Solution 2: Check specific file permissions
ls -la /etc/shadow
# Adjust if needed (do NOT chmod system files loosely)

# For rsync permission errors
rsync -avz --rsync-path="sudo rsync" user@remote:/etc/ /backup/etc/
```

---

### Disk Full

```bash
# Check disk usage
df -h
du -sh /backup/* | sort -hr | head -20

# Find largest files
find /backup -size +1G -ls

# Clean old backups immediately
find /backup -name "*.tar.gz" -mtime +30 -delete

# Clean apt cache
apt-get clean
du -sh /var/cache/apt/

# Check inode usage (can be "full" even with free space)
df -i

# Compress existing uncompressed backups
for f in /backup/*.tar; do gzip "$f" && rm "$f"; done
```

---

### Corrupted Archive

```bash
# Test archive
tar -tzf backup.tar.gz > /dev/null
gzip -t backup.tar.gz

# Try to salvage partial archive
# Extract what's recoverable, ignoring errors
tar -xzf corrupted.tar.gz --ignore-failed-read 2>/dev/null

# Check if it's actually not gzip-compressed
file backup.tar.gz
# If it shows "POSIX tar archive" not "gzip compressed":
tar -xf backup.tar.gz  # (without -z)

# Recover from partial dd image using testdisk
testdisk /dev/sda
```

---

### Failed rsync

```bash
# Check SSH connectivity
ssh -v user@backup-server

# Test rsync manually
rsync -avz --dry-run /test/ user@backup-server:/backup/test/

# Fix "rsync: [sender] read error" — usually network interruption
# Use --partial --append-verify for resumable transfers
rsync -avzP --partial user@remote:/large-file.tar.gz /backup/

# Fix "ssh_exchange_identification: read: Connection reset"
# Check sshd is running and not blocking the IP
ssh -o StrictHostKeyChecking=no -v user@server
```

---

### Broken Symlinks

```bash
# Find broken symlinks
find /var/www -xtype l 2>/dev/null

# List all symlinks in directory
find /etc/nginx/sites-enabled -type l -ls

# Fix broken symlink
ln -sfn /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/mysite

# Verify tar preserves symlinks
tar -tzf backup.tar.gz | grep "^l" | head -10
```

---

### SSH Issues During Backup

```bash
# Test SSH key
ssh -i /root/.ssh/backup_key -T user@backup-server

# Ensure backup SSH key is in authorized_keys
cat /root/.ssh/backup_key.pub >> /home/backup-user/.ssh/authorized_keys

# Debug SSH connection issues
ssh -vvv -i /root/.ssh/backup_key user@backup-server 2>&1 | head -50

# Disable host key checking for automated scripts (use carefully)
rsync -avz -e "ssh -o StrictHostKeyChecking=no" /backup/ user@server:/backup/

# Better: pre-add host key
ssh-keyscan backup-server >> /root/.ssh/known_hosts
```

---

## 20. Best Practices Checklist

### ✅ VPS Backup Checklist

```
BACKUP SETUP
□ Full backup configured and tested
□ Incremental/differential backup configured
□ Database backups configured (MySQL, Postgres, etc.)
□ Docker volumes backed up (if applicable)
□ Backup schedule set in cron or systemd
□ Backup rotation/retention policy set
□ Backup directory has adequate free space
□ Backup user has minimum required permissions

STORAGE & OFFSITE
□ Local backup exists (on server or attached volume)
□ Remote backup server configured
□ Cloud/offsite backup configured (S3, B2, etc.)
□ 3-2-1 rule satisfied (3 copies, 2 media, 1 offsite)
□ Cloud bucket versioning enabled
□ Cloud lifecycle rules set (auto-archive/delete)

SECURITY
□ Backups are encrypted (GPG or OpenSSL)
□ Encryption keys stored securely and separately
□ Backup user uses key-based SSH (no password)
□ Backup permissions are restricted (chmod 700)
□ Immutable/WORM storage configured for critical backups
□ Ransomware: backup target not accessible from source as writable
```

### ✅ Disaster Recovery Checklist

```
PREPARATION
□ Documented DR procedure (written runbook)
□ Backup access credentials documented and secured
□ Recovery tested on staging environment
□ RTO and RPO defined and communicated
□ New VPS provisioning documented

RECOVERY PROCEDURE
□ Provision new VPS
□ Configure network/firewall
□ Restore filesystem from backup
□ Restore databases from dumps
□ Restore Docker stacks/volumes
□ Verify all services start correctly
□ Update DNS / load balancer
□ Notify stakeholders
□ Post-incident review scheduled
```

### ✅ Security Checklist

```
□ Backup files owned by root, not readable by other users
□ Backup scripts not world-readable
□ GPG keys backed up to separate secure location
□ Offsite backups encrypted at rest
□ Cloud bucket is private (no public access)
□ Backup access logs reviewed monthly
□ Immutable backups enabled
□ Backup credentials rotated quarterly
□ SSH access to backup server limited to backup user
```

### ✅ Automation Checklist

```
□ Backup runs unattended (no manual intervention)
□ Failure alerts configured (email/Slack/PagerDuty)
□ Backup log reviewed weekly
□ Checksum verification automated
□ Restore test scheduled (monthly minimum)
□ Backup age monitoring configured
□ Disk space monitoring configured
□ Backup size anomaly detection configured
```

---

## 21. Cheat Sheet

### tar Commands

```bash
# CREATE
tar -czf archive.tar.gz /path/            # gzip
tar -cjf archive.tar.bz2 /path/           # bzip2
tar -cJf archive.tar.xz /path/            # xz
tar -czpf archive.tar.gz /path/           # preserve permissions
tar --numeric-owner -czpf archive.tar.gz /path/  # numeric UID/GID
tar -czf archive.tar.gz --exclude=/path/cache /path/  # exclude

# EXTRACT
tar -xzf archive.tar.gz                   # extract here
tar -xzf archive.tar.gz -C /target/       # extract to dir
tar -xzpf archive.tar.gz -C /             # restore with permissions
tar -xzf archive.tar.gz file.txt          # extract single file

# INSPECT
tar -tzf archive.tar.gz                   # list contents
tar -tzf archive.tar.gz | wc -l           # count files
gzip -t archive.tar.gz && echo OK         # test integrity

# INCREMENTAL
tar -czpf full.tar.gz -g snapshot.snar /data/    # full
tar -czpf incr.tar.gz -g snapshot.snar /data/    # incremental
```

---

### rsync Commands

```bash
# LOCAL
rsync -av /source/ /dest/                 # basic sync
rsync -av --delete /source/ /dest/        # mirror (delete extras)
rsync -avnP /source/ /dest/               # dry run with progress

# REMOTE
rsync -avzP /local/ user@host:/remote/    # push
rsync -avzP user@host:/remote/ /local/    # pull
rsync -avzAX -e "ssh -i key" / user@host:/backup/  # full server backup

# OPTIONS
rsync -avz --bwlimit=10000 /src/ /dst/    # limit bandwidth
rsync -avzc /src/ /dst/                   # checksum comparison
rsync -avz --exclude='*.log' /src/ /dst/  # exclude pattern
rsync -avz --link-dest=/prev/ /src/ /dst/ # incremental with hard links
```

---

### Compression Commands

```bash
# GZIP
gzip file                   # compress (replaces original)
gzip -k file                # compress, keep original
gunzip file.gz              # decompress
gzip -t file.gz             # test

# ZSTD (recommended for production)
zstd -3 file                # fast compression
zstd -19 -T0 file           # max compression, multi-threaded
zstd -d file.zst            # decompress
zstd -T0 -3 -r dir/         # compress directory recursively

# XZ (maximum compression)
xz -6 file                  # balanced compression
xz -0 file                  # fastest
xz -d file.xz               # decompress

# 7Z
7z a archive.7z file        # create
7z a -mx=9 archive.7z dir/  # max compression
7z x archive.7z             # extract
7z t archive.7z             # test integrity
```

---

### Database Backup Commands

```bash
# MYSQL
mysqldump -u root -p dbname > dump.sql
mysqldump -u root -p --all-databases | gzip > all.sql.gz
mysql -u root -p dbname < dump.sql

# POSTGRESQL
pg_dump -U postgres dbname > dump.sql
pg_dump -U postgres -Fc dbname > dump.dump
sudo -u postgres pg_dumpall | gzip > all.sql.gz
pg_restore -U postgres -d dbname dump.dump

# MONGODB
mongodump --archive=dump.gz --gzip
mongorestore --archive=dump.gz --gzip

# REDIS
redis-cli BGSAVE
cp /var/lib/redis/dump.rdb /backup/
```

---

### Restore Commands

```bash
# FULL RESTORE
tar -xzpf full-backup.tar.gz -C /
tar -xzpf full-backup.tar.gz -C /target/ --strip-components=1

# SINGLE FILE RESTORE
tar -xzf backup.tar.gz -C / etc/nginx/nginx.conf

# DATABASE RESTORE
mysql -u root -p < backup.sql
gunzip < backup.sql.gz | mysql -u root -p
pg_restore -U postgres -d mydb backup.dump

# RSYNC RESTORE
rsync -avz /backup/mirror/ /

# FROM S3
aws s3 cp s3://bucket/backup.tar.gz /tmp/
aws s3 sync s3://bucket/backups/ /backup/
```

---

### Monitoring Commands

```bash
# DISK SPACE
df -h                        # disk usage
du -sh /backup/*             # backup sizes
du -sh /backup/ --max-depth=1

# BACKUP FRESHNESS
find /backup -name "*.tar.gz" -mtime -1   # modified in last 24h
stat /backup/latest.tar.gz | grep Modify

# INTEGRITY CHECK
sha256sum -c checksums.sha256
tar -tzf backup.tar.gz > /dev/null && echo OK || echo FAILED
gzip -t backup.tar.gz

# PROCESS MONITORING
ps aux | grep rsync
ps aux | grep tar
jobs -l

# LOG MONITORING
tail -f /var/log/backup.log
journalctl -u vps-backup.service -f

# CRON VERIFICATION
crontab -l
systemctl list-timers
```

---

> 📌 **Final Notes**
>
> - **Test your restores.** A backup is only valuable if you can restore from it. Run monthly restore tests.
> - **Monitor backup freshness.** Set up alerts when backups are older than expected.
> - **Encrypt before offsite.** Never store unencrypted backups in third-party cloud storage.
> - **Follow the 3-2-1 rule.** Local + remote + offsite. No exceptions in production.
> - **Document your DR procedure.** The middle of an incident is not the time to figure out how to restore.
>
> *Built for production. Tested in disaster.*
