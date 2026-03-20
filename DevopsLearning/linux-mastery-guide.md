# 🐧 Linux Logging, Monitoring & File Management Mastery Guide

> **The complete production-ready handbook for Linux system administration and DevOps engineers.**
> *From zero to job-ready — in one document.*

---

## 📋 Table of Contents

1. [Introduction](#section-1-introduction)
2. [Linux Logging (Deep Dive)](#section-2-linux-logging)
3. [System Monitoring](#section-3-system-monitoring)
4. [File Management Commands](#section-4-file-management-commands)
5. [Real-World DevOps Scenarios](#section-5-real-world-devops-scenarios)
6. [Security & Best Practices](#section-6-security--best-practices)
7. [Cheat Sheet](#section-7-cheat-sheet)
8. [Practice Tasks](#section-8-practice-tasks)

---

# SECTION 1: INTRODUCTION

## What is Linux System Administration?

Linux system administration is the art and science of managing, maintaining, and securing Linux-based servers and systems. As a sysadmin or DevOps engineer, your job is to keep systems running, healthy, and secure — at any scale, from a single VPS to thousands of cloud servers.

Core responsibilities include:

- **Monitoring** system health (CPU, RAM, disk, network)
- **Managing logs** to trace bugs, intrusions, and failures
- **Handling files and directories** across the system
- **Troubleshooting** crashes, slowdowns, and security events
- **Automating** routine tasks with shell scripts and cron jobs

## Why Logging and Monitoring Matter

Imagine running a production e-commerce server that goes down at 2 AM on Black Friday. Without logs, you're blind. Without monitoring, you don't even know it's down until customers start complaining.

**Real-world analogy:**

> Think of a server like an airplane. Logging is the black box — it records everything that happens. Monitoring is the cockpit dashboard — it shows real-time readings of altitude, speed, fuel. A pilot (sysadmin) needs both to fly safely. Without either, you're flying blind in a storm.

Logs allow you to:

- Trace the exact moment and cause of a failure
- Detect unauthorized login attempts
- Debug application errors
- Meet compliance requirements (HIPAA, PCI-DSS, ISO 27001)
- Perform forensic analysis after a security incident

Monitoring allows you to:

- Catch problems before they become outages
- Understand system trends and capacity
- Alert on anomalies automatically
- Optimize performance proactively

---

# SECTION 2: LINUX LOGGING

## What is Logging?

Logging is the process of recording system events, errors, warnings, and informational messages to persistent files or a journal. Every meaningful action on a Linux system — from a user login to a kernel panic — can be logged.

### Types of Logs

| Log Type | Description | Example |
|---|---|---|
| **System logs** | General OS activity | Kernel messages, service start/stop |
| **Authentication logs** | Login attempts, sudo usage | SSH logins, su commands |
| **Application logs** | App-specific messages | Nginx errors, MySQL queries |
| **Kernel logs** | Hardware and driver events | USB device attached, kernel errors |
| **Audit logs** | Security and compliance | File access by specific users |
| **Access logs** | Web server request records | HTTP GET/POST requests |

---

## Important Log Locations

### `/var/log/syslog` (Debian/Ubuntu) or `/var/log/messages` (RHEL/CentOS)

The **master system log**. Nearly all system-level events are written here by the `rsyslog` or `syslog-ng` daemon.

```bash
# View the last 50 lines
tail -n 50 /var/log/syslog

# Watch in real time
tail -f /var/log/syslog

# Filter for errors only
grep -i "error" /var/log/syslog

# Filter by date (e.g., today)
grep "$(date '+%b %e')" /var/log/syslog
```

**Sample output:**
```
Mar 15 10:22:01 webserver01 systemd[1]: Started Session 42 of user deploy.
Mar 15 10:22:03 webserver01 sshd[3421]: Accepted publickey for deploy from 192.168.1.10
```

---

### `/var/log/auth.log` (Debian/Ubuntu) or `/var/log/secure` (RHEL)

Records all **authentication events**: SSH logins, sudo usage, failed passwords, PAM events.

```bash
# Watch for failed SSH login attempts (brute force detection)
grep "Failed password" /var/log/auth.log

# See who used sudo
grep "sudo" /var/log/auth.log

# Count failed logins per IP (potential attack)
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn | head -20

# Successful logins
grep "Accepted" /var/log/auth.log
```

**Sample output:**
```
Mar 15 10:30:01 webserver01 sshd[4412]: Failed password for root from 45.33.32.156 port 55234 ssh2
Mar 15 10:30:03 webserver01 sshd[4412]: Failed password for root from 45.33.32.156 port 55238 ssh2
```

> ⚠️ **Warning:** Hundreds of "Failed password for root" entries from the same IP = active brute force attack. Block it immediately with `ufw deny from <IP>` or `fail2ban`.

---

### `/var/log/kern.log`

**Kernel-level messages**: hardware errors, driver loading, OOM (Out of Memory) killer events.

```bash
# View kernel log
cat /var/log/kern.log

# Find OOM killer events (memory exhaustion)
grep -i "oom" /var/log/kern.log

# Find hardware errors
grep -i "error\|fail\|critical" /var/log/kern.log
```

> 💡 **Tip:** OOM Killer messages mean your server ran out of RAM and the kernel started killing processes. This is a critical event requiring immediate RAM analysis.

---

### `/var/log/dmesg`

The **kernel ring buffer** — messages generated during the boot process and by hardware drivers. Extremely useful for diagnosing hardware issues.

```bash
# View full dmesg output
dmesg

# Filter for errors
dmesg | grep -i "error\|fail\|warn"

# Show with human-readable timestamps
dmesg -T

# Watch new dmesg entries in real time
dmesg -w

# Check disk-related messages
dmesg | grep -i "sda\|nvme\|disk"
```

**Sample output:**
```
[    2.341234] nvme0n1: p1 p2 p3
[   14.512345] EXT4-fs (nvme0n1p2): mounted filesystem
[  142.231456] ata1.00: exception Emask 0x0 SAct 0x0 SErr 0x0 action 0x6
```

---

### `/var/log/nginx/`

Nginx keeps two important logs:

- **`access.log`** — every HTTP request (IP, method, URL, status code, size)
- **`error.log`** — errors, failed upstreams, permission issues

```bash
# Watch Nginx access log in real time
tail -f /var/log/nginx/access.log

# Find all 500 errors
grep " 500 " /var/log/nginx/access.log

# Find 404s (missing pages)
grep " 404 " /var/log/nginx/access.log | wc -l

# Top 10 requesting IPs (traffic analysis)
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# View error log
tail -100 /var/log/nginx/error.log
```

**Sample access.log entry:**
```
192.168.1.55 - - [15/Mar/2025:10:44:01 +0530] "GET /api/users HTTP/1.1" 200 3421 "-" "Mozilla/5.0"
```

---

### `/var/log/apache2/`

Same pattern as Nginx:

```bash
# Apache access log
tail -f /var/log/apache2/access.log

# Apache error log
tail -f /var/log/apache2/error.log

# Find PHP errors
grep "PHP" /var/log/apache2/error.log
```

---

## journalctl — Systemd Journal Logs

Modern Linux systems (using systemd) store logs in a **binary journal** managed by `journald`. `journalctl` is the command to query this journal.

### Basic Usage

```bash
# View all journal entries (oldest to newest)
journalctl

# View with newest entries first
journalctl -r

# View last N lines
journalctl -n 100

# Follow in real time (like tail -f)
journalctl -f
```

---

### `journalctl -xe`

Shows the most recent journal entries with **extra context and explanations**.

```bash
journalctl -xe
```

- `-x` — adds explanatory text for some messages (catalog entries)
- `-e` — jumps to the **end** of the journal

> 💡 **Tip:** Run `journalctl -xe` immediately after a service fails to start. It shows the exact failure reason with context.

---

### `journalctl -u <service>`

Filter logs for a **specific systemd service**:

```bash
# Logs for nginx
journalctl -u nginx

# Logs for SSH daemon
journalctl -u ssh

# Logs for your Node.js app (if it's a systemd service)
journalctl -u myapp.service

# Follow logs for a specific service
journalctl -u nginx -f

# Last 50 lines for a service
journalctl -u nginx -n 50

# With extra context and jump to end
journalctl -u nginx -xe
```

---

### `journalctl --since` and `--until`

Filter by **time range**:

```bash
# Logs since today
journalctl --since today

# Logs since yesterday
journalctl --since yesterday

# Specific time range
journalctl --since "2025-03-15 10:00:00" --until "2025-03-15 12:00:00"

# Last 1 hour
journalctl --since "1 hour ago"

# Last 30 minutes
journalctl --since "30 minutes ago"

# Combine with service filter
journalctl -u nginx --since "2025-03-15 09:00:00" --until "2025-03-15 11:00:00"
```

---

### `journalctl -f` — Real-Time Following

```bash
# Follow all logs
journalctl -f

# Follow specific service
journalctl -u myapp -f

# Follow with priority filter (only errors)
journalctl -f -p err
```

---

### Priority Filtering

```bash
# Only emergency messages
journalctl -p emerg

# Errors and above (err, crit, alert, emerg)
journalctl -p err

# Debug level (very verbose)
journalctl -p debug

# Priority levels: 0=emerg, 1=alert, 2=crit, 3=err, 4=warning, 5=notice, 6=info, 7=debug
```

---

### Other Useful journalctl Options

```bash
# Show logs from previous boot
journalctl -b -1

# Show logs from current boot
journalctl -b

# Disk usage of the journal
journalctl --disk-usage

# Clean old journal entries (keep only last 100MB)
journalctl --vacuum-size=100M

# Keep only last 2 weeks of logs
journalctl --vacuum-time=2weeks

# Output in JSON format (useful for log pipelines)
journalctl -u nginx -o json | head -5
```

---

## Log Monitoring Commands

### `tail` — View End of File

```bash
# Last 10 lines (default)
tail /var/log/syslog

# Last N lines
tail -n 50 /var/log/syslog

# Follow file in real time (most important!)
tail -f /var/log/nginx/access.log

# Follow multiple files at once
tail -f /var/log/nginx/access.log /var/log/nginx/error.log

# Follow and show filename headers
tail -f -v /var/log/nginx/*.log
```

> 💡 **Tip:** `tail -f` is your best friend for live debugging. Keep it open in a terminal while reproducing a bug.

---

### `head` — View Beginning of File

```bash
# First 10 lines
head /var/log/syslog

# First N lines
head -n 20 /var/log/syslog

# Useful for checking log file format or headers
head -5 /var/log/nginx/access.log
```

---

### `less` — Page Through Large Files

```bash
# Open large log file safely (doesn't load entire file into memory)
less /var/log/syslog

# Navigate: 
#   Space or PgDn — next page
#   b or PgUp     — previous page
#   G             — jump to end
#   g             — jump to beginning
#   /pattern      — search forward
#   ?pattern      — search backward
#   n             — next match
#   N             — previous match
#   q             — quit

# Open and jump to end
less +G /var/log/syslog

# Follow mode (like tail -f but with navigation)
less +F /var/log/syslog
# Press Ctrl+C to stop following, navigate, then F again to resume
```

> ⚠️ **Warning:** Never use `cat` on a huge log file (e.g., a 2GB access.log) — it will flood your terminal. Use `less` instead.

---

### `cat` — Print Entire File

```bash
# View small log files
cat /var/log/auth.log

# Concatenate and view multiple files
cat /var/log/syslog /var/log/kern.log

# With line numbers
cat -n /var/log/auth.log

# Combine with grep for quick search
cat /var/log/syslog | grep "error"
```

---

### `watch` — Run Command Repeatedly

```bash
# Watch a command every 2 seconds (default)
watch df -h

# Custom interval (every 1 second)
watch -n 1 free -h

# Highlight differences between updates
watch -d df -h

# Watch log line count
watch -n 1 "wc -l /var/log/nginx/access.log"

# Watch for new errors in a log
watch -n 2 "tail -5 /var/log/nginx/error.log"
```

---

### `grep` — Search Text Patterns (CRITICAL SKILL)

`grep` is one of the most important tools in a sysadmin's arsenal.

```bash
# Basic search
grep "error" /var/log/syslog

# Case-insensitive search
grep -i "error" /var/log/syslog

# Show line numbers
grep -n "error" /var/log/syslog

# Show N lines of context around match
grep -C 3 "segfault" /var/log/syslog    # 3 lines before and after
grep -B 5 "error" /var/log/syslog       # 5 lines before
grep -A 5 "error" /var/log/syslog       # 5 lines after

# Count matches
grep -c "Failed password" /var/log/auth.log

# Invert match (lines NOT containing pattern)
grep -v "127.0.0.1" /var/log/nginx/access.log

# Search recursively in directory
grep -r "OutOfMemoryError" /var/log/

# Extended regex (multiple patterns)
grep -E "error|warning|critical" /var/log/syslog

# Search multiple files
grep "error" /var/log/syslog /var/log/kern.log

# Show filename in output
grep -H "error" /var/log/*.log

# Quiet mode (just return exit code: 0=found, 1=not found)
grep -q "error" /var/log/syslog && echo "Errors found!"

# Real-world: find failed SSH logins and count per IP
grep "Failed password" /var/log/auth.log | grep -oP '(?<=from )\S+' | sort | uniq -c | sort -rn
```

---

### `awk` — Text Processing and Filtering

`awk` is a powerful text processing tool. For log analysis, it excels at extracting specific fields.

```bash
# Print specific column (field 1 = IP address in Nginx access log)
awk '{print $1}' /var/log/nginx/access.log

# Print multiple fields
awk '{print $1, $7}' /var/log/nginx/access.log   # IP and URL

# Filter lines where HTTP status (field 9) is 500
awk '$9 == 500' /var/log/nginx/access.log

# Filter lines where field 9 starts with 5 (all 5xx errors)
awk '$9 ~ /^5/' /var/log/nginx/access.log

# Count requests per status code
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn

# Sum total bytes transferred (field 10)
awk '{sum += $10} END {print "Total bytes:", sum}' /var/log/nginx/access.log

# Print lines with more than 3 fields
awk 'NF > 3' /var/log/syslog

# Custom delimiter (for CSV logs)
awk -F',' '{print $2}' /var/log/app.csv

# Real-world: Top 10 URLs by request count
awk '{print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10
```

---

## Log Rotation

### What is logrotate?

Log files grow continuously. Without rotation, a single log file can fill your entire disk. `logrotate` is a utility that:

- Automatically rotates log files on a schedule
- Compresses old log files
- Deletes logs older than a defined period
- Can run pre/post-rotation scripts (e.g., reload Nginx after rotation)

### Why is it needed?

Imagine your Nginx server gets 1 million requests per day. The access log grows by ~100MB daily. In 30 days, that's 3GB — just for one log file. Without rotation, your disk fills up, the web server crashes, and your site goes down.

### Configuration File Location

```
/etc/logrotate.conf         ← Main configuration
/etc/logrotate.d/           ← Per-application configs
/etc/logrotate.d/nginx      ← Nginx-specific config
/etc/logrotate.d/mysql      ← MySQL-specific config
```

### Example: `/etc/logrotate.d/nginx`

```
/var/log/nginx/*.log {
    daily               # Rotate every day
    missingok           # Don't error if log file is missing
    rotate 14           # Keep 14 rotated copies
    compress            # Compress old log files with gzip
    delaycompress       # Don't compress the most recent rotated file
    notifempty          # Don't rotate if file is empty
    create 0640 www-data adm   # New file permissions and ownership
    sharedscripts       # Run scripts only once (not per file)
    postrotate
        # Tell Nginx to reopen log files
        [ -f /var/run/nginx.pid ] && kill -USR1 `cat /var/run/nginx.pid`
    endscript
}
```

### Example: Custom Application Log

```
/var/log/myapp/*.log {
    weekly              # Rotate weekly
    rotate 4            # Keep 4 weeks
    compress
    delaycompress
    missingok
    notifempty
    dateext             # Add date to rotated file name
    dateformat -%Y%m%d  # Format: myapp.log-20250315
    size 50M            # Also rotate if file exceeds 50MB
    postrotate
        systemctl restart myapp > /dev/null 2>&1 || true
    endscript
}
```

### Testing logrotate

```bash
# Test configuration without actually rotating
logrotate -d /etc/logrotate.d/nginx

# Force rotation (even if not due yet)
logrotate -f /etc/logrotate.d/nginx

# Check last rotation status
cat /var/lib/logrotate/status

# Run all rotations manually
logrotate /etc/logrotate.conf
```

> 💡 **Tip:** Always test with `-d` (dry run) first before forcing rotation in production.

---

# SECTION 3: SYSTEM MONITORING

## Basic Commands

### `top` — Interactive Process Viewer

```bash
top
```

**Understanding top output:**

```
top - 10:44:05 up 12 days,  3:22,  2 users,  load average: 0.45, 0.38, 0.29
Tasks: 195 total,   1 running, 194 sleeping,   0 stopped,   0 zombie
%Cpu(s):  4.2 us,  1.1 sy,  0.0 ni, 94.3 id,  0.2 wa,  0.0 hi,  0.2 si
MiB Mem :   7848.3 total,    423.1 free,   4215.4 used,   3209.8 buff/cache
MiB Swap:   2048.0 total,   1832.6 free,    215.4 used.   3292.1 avail Mem

  PID  USER      PR  NI    VIRT    RES    SHR  %CPU  %MEM  TIME+    COMMAND
 1234  www-data  20   0  123456  45678   8765   8.5   0.6   1:23.45  node
```

**Key fields explained:**

| Field | Meaning |
|---|---|
| `load average: 0.45, 0.38, 0.29` | CPU load over 1, 5, 15 minutes |
| `us` | User-space CPU % |
| `sy` | System (kernel) CPU % |
| `id` | Idle CPU % |
| `wa` | I/O Wait % (high = disk bottleneck) |
| `RES` | Actual RAM used by process |
| `VIRT` | Virtual memory (includes swapped) |
| `%MEM` | Percentage of total RAM |

**Interactive keys:**

| Key | Action |
|---|---|
| `P` | Sort by CPU usage |
| `M` | Sort by memory usage |
| `T` | Sort by time |
| `k` | Kill a process (enter PID) |
| `r` | Renice a process |
| `1` | Toggle per-CPU view |
| `q` | Quit |
| `f` | Add/remove columns |

**Load average rules of thumb:**

- On a **4-core** system: load of 4.0 = fully loaded, >4.0 = overloaded
- `load / CPU_cores > 1.0` = system is under stress

---

### `htop` — Enhanced Interactive Process Viewer

Install if not present: `sudo apt install htop`

```bash
htop
```

Advantages over `top`:
- Color-coded display
- Mouse support
- Horizontal scrolling for long command names
- F-key shortcuts displayed at bottom
- Tree view of processes (`F5`)
- Easy kill, nice, search

**Useful htop shortcuts:**

| Key | Action |
|---|---|
| `F2` | Setup / customize display |
| `F3` | Search for a process |
| `F4` | Filter processes |
| `F5` | Tree view |
| `F6` | Sort by column |
| `F9` | Kill process |
| `F10` | Quit |
| `Space` | Tag/untag process |

---

### `uptime` — System Load Overview

```bash
uptime
# Output: 10:44:05 up 12 days,  3:22,  2 users,  load average: 0.45, 0.38, 0.29
```

```bash
# Just load averages
uptime -p   # Output: up 12 days, 3 hours, 22 minutes
uptime -s   # Output: 2025-03-03 07:22:01 (when it booted)
```

---

### `free -h` — Memory Usage

```bash
free -h
```

**Sample output:**
```
              total        used        free      shared  buff/cache   available
Mem:          7.7Gi       4.1Gi       425Mi       256Mi       3.2Gi       3.2Gi
Swap:         2.0Gi       211Mi       1.8Gi
```

**Key columns:**

| Column | Meaning |
|---|---|
| `total` | Total installed RAM |
| `used` | Currently used RAM |
| `free` | Truly unused RAM |
| `buff/cache` | Used by OS for disk caching (releasable) |
| `available` | RAM that *can* be freed for apps |

> 💡 **Tip:** `available` is the real number to check, not `free`. Linux uses spare RAM for caching — that's normal and healthy.

```bash
# Show in megabytes
free -m

# Refresh every 2 seconds
free -h -s 2

# Check if swap is being used heavily (indicates RAM pressure)
free -h | grep Swap
```

---

### `df -h` — Disk Usage by Filesystem

```bash
df -h
```

**Sample output:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/nvme0n1p2   50G   23G   25G  48% /
/dev/nvme0n1p1  512M  6.1M  506M   2% /boot/efi
tmpfs           3.9G   89M  3.8G   3% /dev/shm
```

```bash
# Show inode usage (important! can fill up even if disk space is free)
df -i

# Show specific filesystem
df -h /var/log

# Show all filesystems including virtual
df -ha

# Find filesystems over 80% used
df -h | awk 'NR>1 && $5+0 > 80 {print}'
```

> ⚠️ **Warning:** A server can fail even with disk space remaining if **inodes** are exhausted (too many small files). Always check `df -i` when debugging "no space left on device" errors.

---

### `du -sh` — Disk Usage by Directory

```bash
# Summary of current directory
du -sh .

# Summary of specific directory
du -sh /var/log

# List all subdirectories with sizes
du -sh /var/log/*

# Sort by size (largest first)
du -sh /var/log/* | sort -rh

# Top 10 largest directories under /var
du -sh /var/* | sort -rh | head -10

# Find largest directories recursively (very useful!)
du -h --max-depth=2 /var | sort -rh | head -20

# Find all files over 100MB
find / -type f -size +100M -exec du -sh {} \; 2>/dev/null
```

---

### `ps aux` — List Running Processes

```bash
ps aux
```

**Column meanings:**

| Column | Meaning |
|---|---|
| `USER` | Process owner |
| `PID` | Process ID |
| `%CPU` | CPU usage |
| `%MEM` | Memory usage |
| `VSZ` | Virtual memory size |
| `RSS` | Resident (actual) memory |
| `STAT` | Process state (R=running, S=sleeping, Z=zombie) |
| `COMMAND` | Full command with arguments |

```bash
# Find specific process
ps aux | grep nginx

# Sort by CPU usage (top 10)
ps aux --sort=-%cpu | head -10

# Sort by memory usage (top 10)
ps aux --sort=-%mem | head -10

# Show process tree
ps axjf

# Show threads
ps -eLf | grep nginx

# Processes by specific user
ps -u www-data
```

---

### `kill` and `killall`

```bash
# Send SIGTERM (graceful shutdown) to PID
kill 1234

# Force kill (SIGKILL — can't be ignored)
kill -9 1234

# Send specific signal
kill -HUP 1234     # HUP = reload config (for many daemons)
kill -TERM 1234    # Graceful termination

# Kill by process name (kills ALL matching processes)
killall nginx

# Force kill by name
killall -9 node

# Kill by name with confirmation
killall -i nginx

# Kill processes matching pattern
pkill -f "python3 myapp.py"

# Kill all processes of a user
pkill -u baduser
```

**Signal reference:**

| Signal | Number | Meaning |
|---|---|---|
| `SIGTERM` | 15 | Graceful termination (default) |
| `SIGKILL` | 9 | Forced kill (cannot be caught) |
| `SIGHUP` | 1 | Reload configuration |
| `SIGINT` | 2 | Interrupt (like Ctrl+C) |
| `SIGSTOP` | 19 | Pause process |
| `SIGCONT` | 18 | Resume paused process |

> ⚠️ **Warning:** Always try `kill PID` (SIGTERM) first. Only use `kill -9` if the process doesn't respond — it prevents graceful cleanup, which can corrupt databases or leave temp files.

---

## Advanced Monitoring

### `vmstat` — Virtual Memory Statistics

```bash
# One-shot snapshot
vmstat

# Update every 1 second, 10 times
vmstat 1 10
```

**Sample output:**
```
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0 215424 434556  68120 3284556    0    0     5    12  245  512  4  1 94  1  0
```

**Key columns:**

| Column | Meaning | Warning threshold |
|---|---|---|
| `r` | Processes waiting for CPU | > number of CPUs |
| `b` | Processes in uninterruptible sleep | > 2-3 |
| `si/so` | Swap in/out (MB/s) | Any sustained value |
| `bi/bo` | Disk read/write (blocks/s) | Context-dependent |
| `wa` | CPU waiting for I/O | > 10-20% |
| `us` | User CPU % | Context-dependent |

---

### `iostat` — I/O Statistics

Install: `sudo apt install sysstat`

```bash
# Single snapshot
iostat

# Every 2 seconds
iostat 2

# Extended stats with device details
iostat -x 2

# Specific device
iostat -x nvme0n1 2
```

**Key `iostat -x` fields:**

| Field | Meaning |
|---|---|
| `r/s`, `w/s` | Reads/writes per second |
| `rkB/s`, `wkB/s` | KB read/written per second |
| `await` | Average I/O wait time (ms) |
| `%util` | Disk utilization % (100% = saturated) |

> 💡 **Tip:** `%util` near 100% means your disk is the bottleneck. Consider SSDs, RAID, or moving high-I/O apps to faster storage.

---

### `netstat` — Network Connections (Legacy)

```bash
# Show all connections
netstat -a

# Show listening ports
netstat -l

# Show TCP connections
netstat -t

# Show UDP
netstat -u

# Show with PID and program name
netstat -tlnp

# Show statistics
netstat -s

# Show routing table
netstat -r
```

> 💡 **Tip:** `netstat` is deprecated on modern systems. Use `ss` instead (faster, more features).

---

### `ss` — Socket Statistics (Modern netstat)

```bash
# Show all sockets
ss -a

# Listening TCP sockets with PIDs
ss -tlnp

# All TCP connections
ss -t

# UDP sockets
ss -u

# Show socket details including memory
ss -tme

# Filter by port
ss -tnp | grep :80

# Count connections per state
ss -ant | awk '{print $1}' | sort | uniq -c

# Show connections to specific host
ss dst 192.168.1.1
```

**Common states:**

| State | Meaning |
|---|---|
| `LISTEN` | Waiting for connections |
| `ESTABLISHED` | Active connection |
| `TIME_WAIT` | Closing, waiting for cleanup |
| `CLOSE_WAIT` | Remote end closed, local waiting |

---

### `lsof` — List Open Files

Everything in Linux is a file. `lsof` shows all open files — including network sockets, pipes, and devices.

```bash
# All open files
lsof

# Files opened by specific process
lsof -p 1234

# Files opened by specific user
lsof -u www-data

# What process is using a port
lsof -i :80
lsof -i :443

# What's using a specific file
lsof /var/log/nginx/access.log

# All network connections
lsof -i

# TCP connections only
lsof -i TCP

# Find deleted files still held open (prevents disk space from being freed)
lsof +L1

# Kill all processes opened by a user
lsof -t -u baduser | xargs kill
```

> 💡 **Tip:** `lsof +L1` is invaluable when `df -h` shows disk is full but you can't find the files. Deleted files held open by processes still consume space — `lsof +L1` finds them.

---

### `dmesg` — Kernel Messages

```bash
# View all kernel messages
dmesg

# Human-readable timestamps
dmesg -T

# Follow new messages
dmesg -w

# Filter errors
dmesg -T | grep -i "error\|fail\|warn\|oom"

# Last 20 lines
dmesg | tail -20

# Hardware/disk errors
dmesg | grep -E "ata|nvme|scsi|sd[a-z]"

# USB events
dmesg | grep -i usb
```

---

## Real-Time Monitoring

### `watch` for Live Dashboards

```bash
# Monitor disk usage every 2 seconds
watch -n 2 df -h

# Monitor top 5 memory processes
watch -n 1 "ps aux --sort=-%mem | head -6"

# Monitor network connections count
watch -n 1 "ss -ant | wc -l"

# Monitor log errors in real time
watch -n 3 "grep -c 'error' /var/log/nginx/error.log"

# Highlight changes
watch -d -n 2 "cat /proc/meminfo | grep -E 'MemFree|Cached|Buffers'"
```

### Live Debugging Session (Combined)

```bash
# Terminal 1: Watch processes
htop

# Terminal 2: Watch logs
tail -f /var/log/nginx/error.log /var/log/nginx/access.log

# Terminal 3: Watch network
watch -n 1 "ss -ant | awk '{print \$1}' | sort | uniq -c"

# Terminal 4: Watch disk I/O
iostat -x 1
```

---

## Network Monitoring

### `ping` — Connectivity Test

```bash
# Basic ping
ping google.com

# Limit to 4 packets
ping -c 4 google.com

# Set packet interval (0.2 seconds between packets)
ping -i 0.2 google.com

# Set packet size
ping -s 1400 google.com

# Flood ping (stress test, requires root)
sudo ping -f -c 1000 192.168.1.1

# Check if host is reachable (exit code 0 = success)
ping -c 1 -q google.com > /dev/null 2>&1 && echo "up" || echo "down"
```

---

### `traceroute` — Network Path Tracing

```bash
# Trace route to host
traceroute google.com

# Use ICMP instead of UDP
traceroute -I google.com

# Set max hops
traceroute -m 20 google.com

# Install: sudo apt install traceroute
# Modern alternative:
mtr google.com    # Combines ping + traceroute, real-time
```

---

### `curl` — HTTP Testing

```bash
# Basic GET request
curl https://api.example.com/health

# Show HTTP headers
curl -I https://example.com

# Follow redirects
curl -L https://example.com

# POST with JSON
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice", "email": "alice@example.com"}'

# With authentication
curl -u username:password https://api.example.com/secure

# Bearer token
curl -H "Authorization: Bearer YOUR_TOKEN" https://api.example.com/data

# Measure response time
curl -w "\nTime: %{time_total}s\nHTTP: %{http_code}\n" -o /dev/null -s https://example.com

# Download file
curl -O https://example.com/file.tar.gz

# Test with specific DNS resolution
curl --resolve api.example.com:443:192.168.1.100 https://api.example.com/health
```

---

### `wget` — File Download

```bash
# Download file
wget https://example.com/file.tar.gz

# Download quietly (no progress)
wget -q https://example.com/file.tar.gz

# Continue interrupted download
wget -c https://example.com/largefile.iso

# Download in background
wget -b https://example.com/largefile.iso

# Mirror a website
wget --mirror --page-requisites --no-parent https://example.com

# Check if URL is accessible (spider mode)
wget --spider https://example.com/api/health
```

---

## Process Management

### `jobs`, `bg`, `fg`

```bash
# Start long process
sleep 100

# Pause it (Ctrl+Z) — now it's stopped
^Z
# [1]+  Stopped   sleep 100

# List background/stopped jobs
jobs
# [1]+  Stopped   sleep 100

# Resume in background
bg 1
# [1]+  sleep 100 &

# Resume in foreground
fg 1

# Start process directly in background
sleep 200 &

# Multiple jobs
jobs
# [1]   Running    sleep 200 &
# [2]+  Stopped    sleep 100
```

---

### `nice` and `renice` — CPU Priority

Linux processes have priorities from **-20** (highest priority) to **19** (lowest priority). Default is 0.

```bash
# Start process with lower priority (be nice to other processes)
nice -n 10 ./backup-script.sh

# Start with higher priority (needs root for negative nice)
sudo nice -n -5 ./critical-job.sh

# Change priority of running process
renice -n 15 -p 1234         # Lower priority of PID 1234
sudo renice -n -5 -p 1234    # Raise priority (requires root)

# Lower priority of all processes by user
renice -n 10 -u www-data

# View niceness in top (NI column) or:
ps -o pid,ni,comm -p 1234
```

> 💡 **Tip:** Run batch jobs (backups, log processing, archiving) with `nice -n 15` so they don't steal CPU from your production app.

---

## Disk & Memory Debugging

### Finding Large Files

```bash
# Find files over 100MB system-wide
find / -type f -size +100M 2>/dev/null

# Show with sizes, sorted
find / -type f -size +100M -exec du -sh {} \; 2>/dev/null | sort -rh

# Find largest files in /var
find /var -type f -printf '%s %p\n' | sort -rn | head -20

# Find old log files (not accessed in 30 days)
find /var/log -type f -atime +30

# Find large files owned by specific user
find /home -user john -type f -size +50M
```

### Disk Full Issues — Debugging Workflow

```bash
# Step 1: Check which filesystem is full
df -h

# Step 2: Find largest directories
du -sh /* 2>/dev/null | sort -rh | head -10

# Step 3: Drill down
du -sh /var/* | sort -rh | head -10
du -sh /var/log/* | sort -rh | head -10

# Step 4: Find deleted files still held open (common issue!)
lsof +L1

# Step 5: If a log file is huge, truncate safely (don't delete!)
> /var/log/nginx/access.log    # Truncate to 0 bytes (safe)
# OR
truncate -s 0 /var/log/nginx/access.log

# Step 6: Clean package caches
sudo apt clean     # Debian/Ubuntu
sudo yum clean all # RHEL

# Step 7: Remove old kernels (Ubuntu)
sudo apt autoremove --purge

# Step 8: Find and remove old core dumps
find / -name "core" -type f 2>/dev/null

# Step 9: Docker cleanup (if Docker is installed)
docker system prune -a
```

> ⚠️ **Warning:** Never `rm` a log file that's actively being written to by a running process! The file will appear deleted but the process still holds the file descriptor open, and disk space won't be freed. Use `truncate -s 0` or `> filename` instead.

### Memory Usage Analysis

```bash
# Overall memory
free -h

# Detailed memory info
cat /proc/meminfo

# Per-process memory (sorted by RAM)
ps aux --sort=-%mem | head -15

# Virtual memory stats
vmstat -s

# Top memory consumers with details
smem -rs rss | head -20    # Install: sudo apt install smem

# Check if OOM killer fired recently
dmesg | grep -i "oom\|out of memory\|killed process"
journalctl -k | grep -i "oom"

# Memory of specific process
cat /proc/1234/status | grep -i vm
```

---

# SECTION 4: FILE MANAGEMENT COMMANDS

## Basic Commands

### `ls` — List Directory Contents

```bash
# Basic listing
ls

# Long format (permissions, owner, size, date)
ls -l

# Include hidden files (starting with .)
ls -a

# Long format + hidden files
ls -la

# Human-readable sizes
ls -lh

# Sort by modification time (newest first)
ls -lt

# Sort by modification time (oldest first)
ls -ltr

# Sort by size (largest first)
ls -lS

# Recursive listing
ls -lR /var/log

# One file per line
ls -1

# List only directories
ls -d */

# Colorized output (usually default)
ls --color=auto

# Show inode numbers
ls -i

# Show full timestamps
ls -l --full-time

# Combined: all files, human sizes, sort by time, newest first
ls -lahtr
```

---

### `pwd` — Print Working Directory

```bash
pwd
# Output: /home/deploy/myapp

# Logical path (follows symlinks as-is)
pwd -L

# Physical path (resolves all symlinks)
pwd -P
```

---

### `cd` — Change Directory

```bash
# Go to specific directory
cd /var/log/nginx

# Go to home directory
cd
cd ~
cd $HOME

# Go up one level
cd ..

# Go up two levels
cd ../..

# Go to previous directory (toggle)
cd -

# Navigate using variables
cd $HOME/projects/myapp

# Absolute vs relative paths
cd /etc/nginx          # Absolute (starts with /)
cd ../nginx            # Relative (relative to current location)
```

---

## File Operations

### `touch` — Create Files / Update Timestamps

```bash
# Create empty file
touch newfile.txt

# Create multiple files
touch file1.txt file2.txt file3.txt

# Update timestamp of existing file
touch existingfile.txt

# Set specific timestamp
touch -t 202503151030.00 file.txt   # 2025-03-15 10:30:00

# Create file only if it doesn't exist
touch -a newfile.txt   # Update only access time
touch -m newfile.txt   # Update only modification time
```

---

### `mkdir` — Create Directories

```bash
# Create directory
mkdir myproject

# Create nested directories (recursive)
mkdir -p myproject/src/components/ui

# Create with specific permissions
mkdir -m 755 public_dir
mkdir -m 700 private_dir

# Create multiple directories
mkdir -p app/{logs,config,data,tmp}

# Verbose output
mkdir -v mydir
```

---

### `cp` — Copy Files

```bash
# Copy file
cp source.txt destination.txt

# Copy to directory
cp file.txt /var/www/html/

# Copy multiple files to directory
cp *.conf /etc/nginx/conf.d/

# Recursive copy (copy directory and contents)
cp -r /etc/nginx /backup/nginx-backup

# Preserve timestamps, permissions, ownership
cp -p file.txt destination/

# Recursive + preserve attributes
cp -rp /etc/nginx /backup/nginx-backup

# Interactive (prompt before overwrite)
cp -i file.txt /backup/

# Verbose (show what's being copied)
cp -v file.txt /backup/

# Update (only copy if source is newer)
cp -u source.txt destination.txt

# Create hard link instead of copy
cp -l file.txt hardlink.txt

# Force overwrite without prompt
cp -f source.txt destination.txt

# Production backup example
cp -rpv /var/www/html /backup/html-$(date +%Y%m%d)
```

---

### `mv` — Move or Rename Files

```bash
# Rename file
mv oldname.txt newname.txt

# Move file to directory
mv file.txt /var/www/html/

# Move multiple files
mv *.log /var/log/archive/

# Interactive (prompt before overwrite)
mv -i source.txt destination.txt

# Verbose
mv -v file.txt /backup/

# Don't overwrite existing files
mv -n source.txt destination.txt

# Rename directory
mv old_dirname new_dirname
```

---

### `rm` — Remove Files and Directories

```bash
# Remove file
rm file.txt

# Remove multiple files
rm file1.txt file2.txt file3.txt

# Remove with glob
rm *.log

# Recursive remove (delete directory and all contents)
rm -r mydir/

# Force remove (no error if file doesn't exist)
rm -f nonexistent.txt

# Recursive + force (DANGEROUS!)
rm -rf mydir/

# Interactive (prompt for each file)
rm -i *.txt

# Verbose
rm -v *.log
```

> ⚠️ **Warning:** `rm -rf` is IRREVERSIBLE. There is no trash. No undo. Before running `rm -rf`, first run `ls -la <path>` to confirm what you're about to delete. A common disaster: `rm -rf / var/log` (note the space) deletes your entire root filesystem!

> 💡 **Tip:** Alias `rm` to `rm -i` in your `.bashrc` for interactive mode by default. For production scripts, consider using `trash` (`sudo apt install trash-cli`) instead: `trash myfile.txt`

---

## File Viewing

### `cat`

```bash
cat file.txt                    # Print file
cat -n file.txt                 # With line numbers
cat file1.txt file2.txt         # Concatenate multiple files
cat >> file.txt                 # Append stdin to file (Ctrl+D to end)
cat > newfile.txt               # Write stdin to file (overwrite)
```

### `less`

```bash
less largefile.log              # Page through file
less +G largefile.log           # Open at end
less +F largefile.log           # Follow mode (like tail -f)
less +/pattern file.txt         # Open and search for pattern
```

### `more`

```bash
more largefile.txt              # Page through (forward only, simpler than less)
```

### `head`

```bash
head file.txt                   # First 10 lines
head -n 20 file.txt             # First 20 lines
head -c 100 file.txt            # First 100 bytes
```

### `tail`

```bash
tail file.txt                   # Last 10 lines
tail -n 50 file.txt             # Last 50 lines
tail -f /var/log/syslog         # Follow in real time
tail -c 200 file.txt            # Last 200 bytes
tail -f -n 0 /var/log/nginx/access.log   # Follow, show only new entries
```

---

## Searching Files (VERY IMPORTANT)

### `find` — Comprehensive File Search

`find` is one of the most powerful commands in Linux. It searches in real time (doesn't use a database).

```bash
# Basic syntax: find [path] [options] [actions]

# Find file by name (exact)
find / -name "nginx.conf"

# Case-insensitive name search
find / -iname "nginx.conf"

# Find in current directory only
find . -name "*.log"

# Find only files (-type f)
find /var -type f -name "*.log"

# Find only directories (-type d)
find /etc -type d -name "nginx"

# Find symbolic links
find /etc -type l

# Find by size
find / -size +100M              # Larger than 100MB
find / -size -1k                # Smaller than 1KB
find / -size +50M -size -200M   # Between 50MB and 200MB

# Find by age
find /tmp -mtime +7             # Modified more than 7 days ago
find /var/log -mtime -1         # Modified in last 24 hours
find / -newer /etc/passwd       # Newer than /etc/passwd
find / -mmin -30                # Modified in last 30 minutes

# Find by owner
find /home -user alice
find /var/www -user www-data

# Find by permissions
find / -perm 777                # Exact permission 777 (dangerous!)
find / -perm /u+s               # Files with SUID bit set
find / -perm -004               # World-readable files

# Combining conditions (AND is default)
find /var/log -type f -name "*.log" -size +10M

# OR condition
find . -name "*.jpg" -o -name "*.png"

# NOT condition
find /etc -not -name "*.conf"

# Actions on found files
find /tmp -mtime +7 -delete                     # Delete old tmp files
find . -type f -name "*.log" -exec ls -lh {} \; # Run ls on each result
find . -type f -name "*.txt" -exec grep "error" {} \;  # Grep in each file

# More efficient with xargs (faster than -exec for many files)
find . -type f -name "*.log" | xargs grep "error"

# Find and compress old logs
find /var/log -name "*.log" -mtime +30 -exec gzip {} \;

# Find world-writable files (security check)
find / -perm -o+w -type f 2>/dev/null

# Find SUID binaries (security check)
find / -perm /4000 -type f 2>/dev/null

# Find files changed in last 10 minutes (post-attack check)
find / -mmin -10 -type f 2>/dev/null | head -30

# Print with details (no need to pipe to ls)
find /var/log -name "*.log" -printf "%s\t%p\n" | sort -rn | head -10
```

---

### `locate` — Fast File Search (Database-based)

```bash
# Install
sudo apt install mlocate

# Build/update database
sudo updatedb

# Find file (very fast!)
locate nginx.conf

# Case-insensitive
locate -i NGINX.CONF

# Count matches
locate -c "*.log"

# Use regex
locate --regex "nginx\.(conf|cfg)"

# Limit results
locate -l 10 "*.conf"

# Show files that still exist (skip deleted)
locate -e "*.conf"
```

> 💡 **Tip:** `locate` is fast because it searches a pre-built database. But it won't find files created after the last `updatedb` run. Use `find` for real-time accuracy, `locate` for quick searches.

---

### `which` — Find Executable Location

```bash
# Find where a command is located
which python3
# Output: /usr/bin/python3

which nginx
# Output: /usr/sbin/nginx

# Find all instances (if multiple versions)
which -a python
# Output:
# /usr/bin/python
# /usr/local/bin/python

# Other location finders
whereis nginx    # Find binary, source, and man page
type nginx       # Show how shell resolves the command
```

---

## File Permissions (OVERVIEW)

### Understanding Permissions

```
-rwxr-xr--  1  alice  webdev  4096  Mar 15 10:44  deploy.sh
^ ^^^ ^^^ ^^^  ^^^^^  ^^^^^^
| |   |   |    |      group
| |   |   |    owner
| |   |   other permissions (r--)
| |   group permissions (r-x)
| owner permissions (rwx)
file type (- = regular, d = directory, l = symlink)
```

Permission bits: `r`=4, `w`=2, `x`=1

Common numeric permissions:

| Numeric | Symbolic | Meaning |
|---|---|---|
| 777 | rwxrwxrwx | Everyone can do everything (DANGEROUS!) |
| 755 | rwxr-xr-x | Owner: full, others: read+execute |
| 644 | rw-r--r-- | Owner: read+write, others: read only |
| 600 | rw------- | Owner only: read+write |
| 700 | rwx------ | Owner only: full access |
| 440 | r--r----- | Owner + group: read only |

---

### `chmod` — Change Permissions

```bash
# Numeric mode
chmod 755 script.sh
chmod 644 config.json
chmod 600 ~/.ssh/id_rsa        # SSH private key must be 600!

# Symbolic mode
chmod u+x script.sh            # Add execute for owner
chmod g-w file.txt             # Remove write for group
chmod o-r sensitive.conf       # Remove read for others
chmod a+r public.txt           # Add read for all
chmod ug+rw shared.txt         # Add read+write for owner and group

# Recursive
chmod -R 755 /var/www/html

# Set exact permissions recursively
find /var/www/html -type f -exec chmod 644 {} \;
find /var/www/html -type d -exec chmod 755 {} \;
```

---

### `chown` — Change Ownership

```bash
# Change owner
chown alice file.txt

# Change owner and group
chown alice:webdev file.txt

# Change group only
chown :webdev file.txt

# Recursive
chown -R www-data:www-data /var/www/html

# Change to numeric UID/GID
chown 1001:1001 file.txt

# Verbose
chown -v alice:alice /home/alice -R
```

---

### `chgrp` — Change Group

```bash
chgrp webdev file.txt
chgrp -R webdev /var/www/html
```

---

## Compression & Archiving

### `tar` — Tape Archive

```bash
# CREATE archive
tar -cvf archive.tar /path/to/dir          # Create (verbose)
tar -czvf archive.tar.gz /path/to/dir      # Create + gzip compress
tar -cjvf archive.tar.bz2 /path/to/dir     # Create + bzip2 compress
tar -cJvf archive.tar.xz /path/to/dir      # Create + xz compress (best ratio)

# EXTRACT archive
tar -xvf archive.tar                        # Extract
tar -xzvf archive.tar.gz                    # Extract gzipped
tar -xvf archive.tar -C /target/dir        # Extract to specific dir

# LIST contents (without extracting)
tar -tvf archive.tar
tar -tzvf archive.tar.gz

# Extract single file
tar -xzvf archive.tar.gz path/to/file.txt

# Create archive of /etc, excluding certain dirs
tar -czvf etc-backup.tar.gz /etc --exclude=/etc/ssl --exclude=/etc/shadow

# Common flags:
# -c  create
# -x  extract
# -v  verbose
# -f  filename (must be last flag before filename)
# -z  gzip
# -j  bzip2
# -J  xz
# -C  change to directory before extracting
# -t  list contents
# -p  preserve permissions
```

---

### `gzip` / `gunzip`

```bash
# Compress (replaces original file)
gzip file.txt           # Creates file.txt.gz, removes file.txt

# Compress keeping original
gzip -k file.txt

# Best compression
gzip -9 file.txt

# Decompress
gunzip file.txt.gz
gzip -d file.txt.gz

# View compressed file without decompressing
zcat file.txt.gz
zless file.txt.gz
zgrep "error" file.txt.gz
```

---

### `zip` / `unzip`

```bash
# Create zip archive
zip archive.zip file1.txt file2.txt

# Zip directory recursively
zip -r archive.zip mydir/

# Password protected zip
zip -e secret.zip file.txt

# View contents
unzip -l archive.zip

# Extract
unzip archive.zip

# Extract to specific directory
unzip archive.zip -d /target/dir

# Extract single file
unzip archive.zip file.txt
```

---

## File Comparison

### `diff` — Compare Files Line by Line

```bash
# Compare two files
diff file1.txt file2.txt

# Unified format (most readable, used in patches/git)
diff -u file1.txt file2.txt

# Side by side
diff -y file1.txt file2.txt

# Ignore whitespace
diff -w file1.txt file2.txt

# Ignore blank lines
diff -B file1.txt file2.txt

# Recursive directory comparison
diff -r dir1/ dir2/

# Create patch file
diff -u original.txt modified.txt > changes.patch

# Apply patch
patch original.txt < changes.patch
```

---

### `cmp` — Byte-by-Byte Comparison

```bash
# Compare two binary files (or text files)
cmp file1.bin file2.bin
# Output if different: file1.bin file2.bin differ: byte 1024, line 1

# Silent mode (just return exit code)
cmp -s file1.bin file2.bin && echo "identical" || echo "different"

# Show all differing bytes
cmp -l file1.bin file2.bin
```

---

## Advanced Tools

### `xargs` — Build Commands from Input

`xargs` takes stdin and passes it as arguments to another command. Essential for processing large numbers of files.

```bash
# Delete all .log files found by find
find /tmp -name "*.log" -mtime +7 | xargs rm -f

# Grep inside multiple files
find . -name "*.js" | xargs grep "console.log"

# Run command on each item (with -I for placeholder)
cat servers.txt | xargs -I{} ssh deploy@{} "uptime"

# Limit processes (parallel execution)
find . -name "*.jpg" | xargs -P 4 -I{} convert {} -resize 800x600 {}

# Handle filenames with spaces
find . -name "*.txt" -print0 | xargs -0 grep "pattern"

# Limit args per command invocation
echo "file1 file2 file3 file4 file5" | xargs -n 2 echo
# Output:
# file1 file2
# file3 file4
# file5

# Dry run / preview
echo "file1 file2" | xargs -t echo
```

---

### `tee` — Read from Stdin, Write to Both Stdout and File

```bash
# Run command, display output AND save to file
ls -la | tee output.txt

# Append to file (instead of overwrite)
ls -la | tee -a output.txt

# Save to multiple files
ls -la | tee file1.txt file2.txt

# Useful in pipelines for intermediate debugging
cat access.log | grep "error" | tee errors.txt | wc -l
# Shows count AND saves matching lines to errors.txt

# Save command output while piping to next command
some-command | tee /tmp/debug.log | another-command
```

---

### `stat` — File Metadata

```bash
# Full file stats
stat file.txt
```

**Sample output:**
```
  File: file.txt
  Size: 2048            Blocks: 8          IO Block: 4096   regular file
Device: 802h/2050d      Inode: 1234567     Links: 1
Access: (0644/-rw-r--r--)  Uid: ( 1000/   alice)   Gid: ( 1000/   alice)
Access: 2025-03-15 10:44:01.234567890 +0530
Modify: 2025-03-14 09:30:00.123456789 +0530
Change: 2025-03-14 09:30:00.123456789 +0530
 Birth: 2025-03-01 08:00:00.000000000 +0530
```

```bash
# Get specific field
stat -c "%n: %s bytes, modified %y" file.txt

# Check filesystem stat
stat -f /dev/nvme0n1p2

# Useful stat format strings:
# %n = filename
# %s = size in bytes
# %a = access permissions (octal)
# %U = owner username
# %G = group name
# %y = last modification time
# %z = last status change time
```

---

# SECTION 5: REAL-WORLD DEVOPS SCENARIOS

## Scenario 1: Debugging a Server Crash

**Situation:** Your production Node.js app crashed at 3 AM. Users can't connect.

```bash
# Step 1: Check if service is running
systemctl status myapp.service

# Step 2: Check for recent errors in service logs
journalctl -u myapp.service -n 100 --since "3 hours ago"

# Step 3: Check for OOM (out of memory) kills
dmesg -T | grep -i "oom\|killed process" | tail -20
journalctl -k | grep -i "oom\|out of memory"

# Step 4: Check system logs around crash time
journalctl --since "2025-03-15 02:50:00" --until "2025-03-15 03:10:00"

# Step 5: Check resource state at current moment
free -h
df -h
uptime

# Step 6: Check if the port is still in use
ss -tlnp | grep :3000

# Step 7: Restart the service
sudo systemctl restart myapp.service

# Step 8: Monitor after restart
journalctl -u myapp.service -f
```

---

## Scenario 2: Finding Error Logs

**Situation:** Users report a "500 Internal Server Error" on `/api/orders`.

```bash
# Step 1: Check Nginx error log
tail -100 /var/log/nginx/error.log | grep -i "error\|upstream\|fail"

# Step 2: Find 500 errors in access log with timestamp
grep " 500 " /var/log/nginx/access.log | tail -20

# Step 3: Filter for the specific endpoint
grep "POST /api/orders" /var/log/nginx/access.log | grep " 500 " | tail -20

# Step 4: Cross-reference with app logs (same timestamp)
# Find the timestamp from Nginx log, then check app logs
journalctl -u myapp --since "2025-03-15 10:43:00" --until "2025-03-15 10:45:00"

# Step 5: Check database logs (if MySQL/PostgreSQL)
tail -50 /var/log/mysql/error.log
# OR PostgreSQL
tail -50 /var/log/postgresql/postgresql-15-main.log
```

---

## Scenario 3: CPU Spike Analysis

**Situation:** Server load spiked to 32 (on an 8-core server). Users experiencing slowdowns.

```bash
# Step 1: Immediate — find what's consuming CPU
top  # Press P to sort by CPU

# Or one-liner
ps aux --sort=-%cpu | head -15

# Step 2: Check for runaway processes
ps aux | awk '$3 > 50 {print}'   # Processes using >50% CPU

# Step 3: Check load history (if sar is installed)
sar -u 1 5    # CPU usage every 1s, 5 times

# Step 4: Check if it's I/O wait (not CPU)
iostat -x 1 5
vmstat 1 5

# Step 5: Find what the process is doing
strace -p <PID>   # Trace system calls (advanced debugging)
lsof -p <PID>     # What files it has open

# Step 6: If a runaway cron job
crontab -l
cat /etc/cron.d/*

# Step 7: Kill if confirmed rogue
kill -15 <PID>   # Graceful
kill -9 <PID>    # Force if it doesn't respond
```

---

## Scenario 4: Disk Cleanup

**Situation:** `/var` partition is at 98% usage. Production is at risk.

```bash
# Step 1: Confirm the issue
df -h /var

# Step 2: Find the biggest consumers
du -sh /var/* | sort -rh | head -15

# Step 3: Typically it's logs
du -sh /var/log/* | sort -rh | head -15

# Step 4: Find the largest individual files
find /var -type f -printf '%s %p\n' 2>/dev/null | sort -rn | head -20 | numfmt --to=iec --field=1

# Step 5: Find and compress old logs
find /var/log -name "*.log" -mtime +7 -not -name "*.gz" -exec gzip {} \;

# Step 6: Remove logs older than 60 days
find /var/log -name "*.gz" -mtime +60 -delete

# Step 7: Clear journal (be conservative!)
journalctl --disk-usage
journalctl --vacuum-time=30days

# Step 8: Docker cleanup (if applicable)
docker system prune -f
docker image prune -a -f
docker volume prune -f

# Step 9: Package manager cache
sudo apt clean
sudo apt autoremove --purge

# Step 10: Find zombie core dumps
find / -name "core.*" -type f 2>/dev/null
```

---

## Scenario 5: Permission Issues

**Situation:** Nginx returns 403 Forbidden or Node app can't write to `/uploads/`.

```bash
# Step 1: Check error log for permission errors
grep -i "permission denied\|forbidden\|13" /var/log/nginx/error.log | tail -20

# Step 2: Check the file/directory permissions
ls -la /var/www/html/uploads/
stat /var/www/html/uploads/

# Step 3: Check who the web server runs as
ps aux | grep nginx | grep -v grep
# Usually: www-data

# Step 4: Check parent directory permissions (all must be traversable!)
namei -l /var/www/html/uploads/

# Step 5: Fix permissions
# Make www-data the owner
sudo chown -R www-data:www-data /var/www/html/uploads/

# Set proper permissions
sudo chmod -R 755 /var/www/html/uploads/

# Step 6: If uploads directory needs to be writable by app
sudo chmod 775 /var/www/html/uploads/
sudo usermod -aG www-data deploy   # Add deploy user to www-data group

# Step 7: Check SELinux context (RHEL/CentOS)
ls -Z /var/www/html/uploads/
restorecon -Rv /var/www/html/uploads/   # Reset SELinux context
```

---

# SECTION 6: SECURITY & BEST PRACTICES

## Avoid 777 Permissions

```bash
# NEVER do this
chmod 777 /var/www/html       # Any user, any process can write!
chmod 777 /etc/nginx/         # Configuration files exposed!
chmod 777 ~/.ssh/             # SSH directory too permissive!

# CORRECT practices
chmod 644 /var/www/html/*.html     # Files: owner write, others read
chmod 755 /var/www/html/           # Dirs: owner write, others traverse
chmod 600 ~/.ssh/id_rsa            # Private key: owner only
chmod 700 ~/.ssh/                  # SSH dir: owner only

# Find and audit dangerous permissions
find / -perm -o+w -type f 2>/dev/null   # World-writable files
find / -perm 777 2>/dev/null            # Full permission files
find / -perm /4000 -type f 2>/dev/null  # SUID binaries
```

## Safe File Handling

```bash
# Use trash instead of rm in interactive sessions
sudo apt install trash-cli
alias rm='trash'

# Backup before editing critical files
cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak.$(date +%Y%m%d)
# Then edit
nano /etc/nginx/nginx.conf

# Use atomic writes for important files (write to temp, then move)
# This prevents partial writes from corrupting the file
write_config_safely() {
    local target="$1"
    local tmpfile=$(mktemp)
    cat > "$tmpfile"
    mv "$tmpfile" "$target"
}

# Validate before replacing config
nginx -t && systemctl reload nginx   # Only reload if config is valid

# Set umask for new files (secure default)
umask 022    # New files: 644, new dirs: 755
umask 027    # Stricter: files 640, dirs 750

# Lock sensitive files from accidental modification
chattr +i /etc/passwd   # Make immutable (even root can't modify!)
chattr -i /etc/passwd   # Remove immutable flag
```

## Log Monitoring for Security

```bash
# Set up a simple security monitoring cron job
# /etc/cron.d/security-check
*/15 * * * * root /usr/local/bin/security-check.sh

# /usr/local/bin/security-check.sh:
#!/bin/bash
THRESHOLD=10

# Check for brute force
FAILED=$(grep "Failed password" /var/log/auth.log | \
    grep "$(date '+%b %e')" | wc -l)

if [ "$FAILED" -gt "$THRESHOLD" ]; then
    echo "ALERT: $FAILED failed SSH attempts today!" | \
        mail -s "Security Alert: $(hostname)" admin@example.com
fi

# Block IPs with too many failures (using fail2ban is better)
grep "Failed password" /var/log/auth.log | \
    awk '{print $11}' | sort | uniq -c | \
    awk -v t=$THRESHOLD '$1 > t {print $2}' | \
    while read ip; do
        ufw deny from "$ip"
    done
```

## Backup Strategies

```bash
# Daily backup script
#!/bin/bash
BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d_%H%M%S)
HOSTNAME=$(hostname)

# Backup /etc
tar -czvf "$BACKUP_DIR/etc-${HOSTNAME}-${DATE}.tar.gz" /etc

# Backup web content
tar -czvf "$BACKUP_DIR/www-${HOSTNAME}-${DATE}.tar.gz" /var/www

# Backup MySQL
mysqldump -u root -p"$DB_PASS" --all-databases | \
    gzip > "$BACKUP_DIR/mysql-${HOSTNAME}-${DATE}.sql.gz"

# Remove backups older than 30 days
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +30 -delete

# Sync to remote (rsync over SSH)
rsync -avz --delete "$BACKUP_DIR/" backup-server:/backups/"$HOSTNAME"/

echo "Backup completed: $DATE"
```

> 💡 **Tip:** Follow the **3-2-1 rule**: 3 copies of data, on 2 different media, with 1 offsite copy.

---

# SECTION 7: CHEAT SHEET

## Logging Commands

| Command | Example | Description |
|---|---|---|
| `tail -f` | `tail -f /var/log/syslog` | Follow log in real time |
| `journalctl -f` | `journalctl -f` | Follow systemd journal |
| `journalctl -u` | `journalctl -u nginx -xe` | Service-specific logs |
| `journalctl --since` | `journalctl --since today` | Filter by time |
| `grep` | `grep -i "error" /var/log/syslog` | Search log for pattern |
| `awk` | `awk '{print $1}' access.log` | Extract log fields |
| `logrotate -f` | `logrotate -f /etc/logrotate.d/nginx` | Force log rotation |
| `less +G` | `less +G /var/log/syslog` | Open at end of file |
| `zgrep` | `zgrep "error" app.log.gz` | Search compressed log |

## System Monitoring Commands

| Command | Example | Description |
|---|---|---|
| `top` | `top` | Interactive process viewer |
| `htop` | `htop` | Enhanced process viewer |
| `ps aux` | `ps aux --sort=-%cpu \| head -10` | List processes |
| `free -h` | `free -h` | RAM usage |
| `df -h` | `df -h` | Disk usage by filesystem |
| `du -sh` | `du -sh /var/log/*` | Directory sizes |
| `vmstat` | `vmstat 1 10` | Virtual memory stats |
| `iostat -x` | `iostat -x 2` | Disk I/O stats |
| `ss -tlnp` | `ss -tlnp` | Listening ports + PIDs |
| `lsof -i` | `lsof -i :80` | Who's using port 80 |
| `dmesg -T` | `dmesg -T \| grep error` | Kernel messages |
| `uptime` | `uptime` | Load average |
| `watch` | `watch -n 1 df -h` | Refresh command |
| `kill -9` | `kill -9 1234` | Force kill process |
| `nice` | `nice -n 15 ./backup.sh` | Set process priority |

## File Management Commands

| Command | Example | Description |
|---|---|---|
| `ls -lahtr` | `ls -lahtr` | List with sizes, hidden, newest last |
| `find` | `find / -name "*.conf" -type f` | Search files |
| `find -size` | `find / -size +100M` | Find large files |
| `locate` | `locate nginx.conf` | Fast indexed search |
| `which` | `which python3` | Locate executable |
| `cp -rp` | `cp -rp /etc /backup/` | Recursive copy with attrs |
| `mv` | `mv oldname.txt newname.txt` | Move or rename |
| `rm -ri` | `rm -ri mydir/` | Interactive recursive delete |
| `chmod` | `chmod 755 script.sh` | Change permissions |
| `chown -R` | `chown -R www-data /var/www/` | Change ownership recursively |
| `tar -czvf` | `tar -czvf backup.tar.gz /etc` | Create gzipped archive |
| `tar -xzvf` | `tar -xzvf backup.tar.gz` | Extract gzipped archive |
| `diff -u` | `diff -u file1.txt file2.txt` | Compare files (unified) |
| `stat` | `stat file.txt` | Full file metadata |
| `xargs` | `find . -name "*.log" \| xargs grep "err"` | Pipe to commands |
| `tee` | `command \| tee output.txt` | Save + display output |

## Permission Quick Reference

| Octal | Symbolic | Typical Use |
|---|---|---|
| `600` | `rw-------` | SSH private keys, sensitive configs |
| `644` | `rw-r--r--` | Web files, config files |
| `700` | `rwx------` | SSH directory, secret scripts |
| `755` | `rwxr-xr-x` | Directories, public scripts |
| `775` | `rwxrwxr-x` | Shared team directories |
| `640` | `rw-r-----` | Group-readable configs |

---

# SECTION 8: PRACTICE TASKS

## 🟢 Beginner Tasks

1. **Explore your system logs**
   - Open `/var/log/syslog` with `less`
   - Search for "error" using `/error` inside less
   - Count how many lines contain "error": `grep -c "error" /var/log/syslog`

2. **Monitor system resources**
   - Run `top`, press `P` to sort by CPU, then `M` to sort by memory
   - Run `free -h` and identify total RAM and available RAM
   - Run `df -h` and find the filesystem with most usage

3. **File management basics**
   - Create directory structure: `mkdir -p ~/practice/{logs,scripts,backups}`
   - Create 5 empty files in `~/practice/logs/`
   - Copy them to `~/practice/backups/`
   - List them with `ls -lah`

4. **Find files**
   - Find all `.conf` files in `/etc`: `find /etc -name "*.conf" -type f`
   - Find files modified in last 10 minutes: `find /tmp -mmin -10`

5. **Basic log analysis**
   - Tail the last 20 lines of `/var/log/auth.log`
   - Follow `journalctl -f` for 30 seconds, then quit with Ctrl+C

---

## 🟡 Intermediate Tasks

1. **Log analysis pipeline**
   - If you have Nginx installed, analyze the access log:
     ```bash
     awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10
     ```
   - Find all 404 errors and count them

2. **Process management**
   - Start `sleep 1000` in the background: `sleep 1000 &`
   - Find its PID: `ps aux | grep sleep`
   - Find it with `pgrep sleep`
   - Change its niceness: `renice -n 10 -p <PID>`
   - Kill it gracefully: `kill <PID>`

3. **Disk usage investigation**
   - Find the 10 largest files on your system: `find / -type f -printf '%s %p\n' 2>/dev/null | sort -rn | head -10`
   - Find the largest directories under `/var`: `du -sh /var/* | sort -rh`

4. **Compression practice**
   - Create a test directory with some files
   - Archive it: `tar -czvf mybackup.tar.gz ~/practice/`
   - List its contents: `tar -tzvf mybackup.tar.gz`
   - Extract to `/tmp`: `tar -xzvf mybackup.tar.gz -C /tmp/`

5. **Permission audit**
   - Find world-writable files in `/tmp`: `find /tmp -perm -o+w -type f`
   - Create a file, change its permissions to 600, verify with `stat`

---

## 🔴 Advanced Real-World Challenges

1. **Log analyzer script**
   
   Write a bash script that:
   - Reads `/var/log/auth.log`
   - Counts failed SSH attempts per IP
   - Prints a report of IPs with more than 5 failures
   - Outputs to both terminal and `/var/log/ssh-report.txt` using `tee`
   - Automatically runs via cron every hour

2. **Disk space alert**
   
   Write a script that:
   - Checks `df -h` on all mounted filesystems
   - Sends an alert message to stdout if any is above 85% usage
   - Finds the top 5 largest files on the near-full filesystem
   - Runs every 15 minutes via cron

3. **Memory pressure simulation**
   
   - Use `stress-ng` to simulate high memory usage
   - In another terminal, watch `free -h` and `vmstat 1`
   - Observe when swap starts being used
   - Kill the stress process and watch recovery
   - Check dmesg for any OOM killer messages

4. **Live log parser**
   
   Write a command pipeline that:
   - Follows a growing log file (`tail -f`)
   - Filters for only ERROR lines
   - Extracts timestamp and error message using `awk`
   - Writes to a separate error-only log file using `tee`
   - Counts occurrences in real time using `watch`

5. **Full production debug simulation**
   
   Simulate this scenario:
   - Start a service that fails intentionally (bad config)
   - Use `journalctl -xe` to diagnose the failure
   - Fix the config
   - Reload the service
   - Verify it's running with `systemctl status`
   - Verify it's listening on the right port with `ss -tlnp`
   - Confirm it responds with `curl localhost:<port>/health`
   - Set up log rotation for its log file in `/etc/logrotate.d/`

---

> 💡 **Final Tip:** The best way to master Linux is to build a homelab. Spin up a free Ubuntu VPS (Oracle Cloud free tier, AWS EC2 free tier, or a local VM), deploy a real app (Nginx + Node.js), and practice all these commands on it. Break things on purpose — that's how you truly learn.

---

*Guide version 1.0 — Linux Logging, Monitoring & File Management Mastery*  
*Covers: Ubuntu 22.04/24.04 LTS · Debian 12 · RHEL/CentOS equivalents noted where different*
