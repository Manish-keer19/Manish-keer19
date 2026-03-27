# 🐧 Linux Mastery Guide: Beginner → Advanced → Production

> **The Complete Linux Handbook for Full-Stack Developers**
> Written by a Senior Linux Architect with 15+ years of production experience.
> This guide takes you from zero to production-grade Linux mastery.

---

## 📋 Table of Contents

| Module | Topic | Level |
|--------|-------|-------|
| 1 | Linux Fundamentals | 🟢 Beginner |
| 2 | Linux File System Hierarchy | 🟢 Beginner |
| 3 | Users and Groups | 🟢 Beginner |
| 4 | Linux Permissions | 🟡 Intermediate |
| 5 | Process Management | 🟡 Intermediate |
| 6 | Services & systemd | 🟡 Intermediate |
| 7 | Linux Networking | 🟡 Intermediate |
| 8 | Package Management | 🟢 Beginner |
| 9 | Disk & Storage | 🟡 Intermediate |
| 10 | Logs & Monitoring | 🟡 Intermediate |
| 11 | SSH | 🟡 Intermediate |
| 12 | Linux Security | 🔴 Advanced |
| 13 | Environment Variables | 🟢 Beginner |
| 14 | Performance Tuning | 🔴 Advanced |
| 15 | Cron Jobs | 🟡 Intermediate |
| 16 | Docker + Linux | 🔴 Advanced |
| 17 | Nginx + Linux | 🔴 Advanced |
| 18 | Production Server Setup | 🔴 Advanced |
| 19 | Linux Debugging | 🔴 Advanced |
| 20 | Advanced Linux Internals | 🔴 Advanced |
| — | Linux Mastery Checklist | ✅ |

---

# MODULE 1 — Linux Fundamentals

## 1.1 What You Will Learn

- What Linux actually is (and why it powers the internet)
- How Linux is architecturally structured
- The difference between Kernel space and User space
- Why there are so many Linux distributions
- How Linux boots from power-on to your terminal

---

## 1.2 What Is Linux?

Linux is an **open-source operating system kernel** created by Linus Torvalds in 1991. But when most people say "Linux," they mean a full **operating system** built around that kernel.

Think of it this way:

```
KERNEL = The engine of the car
LINUX OS = The entire car (engine + body + wheels + dashboard)
```

Linux is the OS that powers:
- ~96% of the world's top 1 million web servers
- All Android smartphones
- AWS, Google Cloud, Azure (their hypervisors and server fleets)
- Your Docker containers
- Supercomputers, satellites, stock exchanges

**Why should a Full-Stack Developer care?**

Because every time you:
- Deploy to AWS EC2 → You're using Linux
- Run a Docker container → You're using Linux
- Configure Nginx → You're using Linux
- SSH into a server → You're on Linux

You ARE working on Linux. Understanding it deeply makes you dramatically more effective.

---

## 1.3 Linux Architecture

Here is how Linux is structured from hardware to your application:

```
┌─────────────────────────────────────┐
│          USER APPLICATIONS          │  ← Your Node.js app, Nginx, etc.
├─────────────────────────────────────┤
│         SYSTEM LIBRARIES            │  ← glibc, libssl, etc.
├─────────────────────────────────────┤
│         SYSTEM CALLS (API)          │  ← Interface between user & kernel
├═════════════════════════════════════╡
│              KERNEL                 │  ← Core of Linux
│  ┌──────────┐ ┌──────────────────┐  │
│  │ Process  │ │   Memory Mgmt    │  │
│  │ Scheduler│ │   (Virtual Mem)  │  │
│  ├──────────┤ ├──────────────────┤  │
│  │ File Sys │ │  Network Stack   │  │
│  │  (VFS)   │ │  (TCP/IP, etc.)  │  │
│  ├──────────┤ ├──────────────────┤  │
│  │  Device  │ │    Security      │  │
│  │  Drivers │ │  (SELinux, etc.) │  │
│  └──────────┘ └──────────────────┘  │
├═════════════════════════════════════╡
│             HARDWARE                │  ← CPU, RAM, Disk, Network Card
│   CPU | RAM | Disk | NIC | GPU     │
└─────────────────────────────────────┘
```

Each layer has a specific job. Let's understand the critical ones:

### The Kernel

The kernel is the **core** of Linux. It:
- Manages CPU scheduling (which process runs when)
- Manages RAM (which process gets what memory)
- Controls I/O (reading/writing to disk and network)
- Manages device drivers
- Enforces security and permissions

The kernel runs in **privileged mode** — it has direct access to hardware.

### System Calls

Your applications cannot directly touch hardware. Instead, they ask the kernel for help through **system calls** (syscalls).

Examples:
- `open()` — ask kernel to open a file
- `read()` — ask kernel to read data
- `write()` — ask kernel to write data
- `fork()` — ask kernel to create a new process
- `socket()` — ask kernel to create a network socket

When your Node.js app reads a file, it calls `fs.readFile()` → Node.js calls C library `fread()` → C library calls Linux syscall `read()` → Kernel reads from disk → Returns data back up the chain.

---

## 1.4 Kernel Space vs. User Space

This is one of the most important concepts in Linux.

```
┌────────────────────────────────────────┐
│              USER SPACE                │
│                                        │
│  Nginx   Node.js   MySQL   Your App   │
│   bash    python   docker    redis     │
│                                        │
│  These processes run with LIMITED      │
│  privileges. They CANNOT directly      │
│  access hardware.                      │
│                                        │
│  If one crashes → only that process    │
│  dies. Others keep running.            │
└──────────────┬─────────────────────────┘
               │  System Calls (glibc)
               │  read(), write(), fork()
               │  socket(), mmap(), etc.
┌──────────────▼─────────────────────────┐
│             KERNEL SPACE               │
│                                        │
│  Runs with FULL hardware access        │
│  Controls EVERYTHING                   │
│                                        │
│  If kernel crashes → ENTIRE system     │
│  crashes (kernel panic)                │
│                                        │
│  Only trusted, signed code runs here   │
└────────────────────────────────────────┘
```

**Why this matters for you:**
- When Nginx crashes, only Nginx stops. Your database keeps running.
- When the kernel panics (very rare), the whole machine freezes.
- Containers (Docker) share the kernel but have isolated user spaces.

---

## 1.5 Linux Distributions

A **distribution (distro)** is a complete operating system built on top of the Linux kernel, packaged with:
- Package manager (how you install software)
- Init system (how services start)
- Default applications
- Configuration conventions

```
┌─────────────────────────────────────────────────────┐
│                   LINUX KERNEL                      │
└───────────────────────┬─────────────────────────────┘
                        │
          ┌─────────────┼─────────────┐
          │             │             │
    ┌─────▼────┐  ┌─────▼────┐  ┌────▼─────┐
    │  Debian  │  │ Red Hat  │  │  Arch    │
    │  Family  │  │  Family  │  │  Family  │
    └─────┬────┘  └─────┬────┘  └────┬─────┘
          │             │             │
    ┌─────▼────┐  ┌─────▼────┐  ┌────▼─────┐
    │  Ubuntu  │  │  CentOS  │  │  Arch    │
    │  Mint    │  │  RHEL    │  │  Manjaro │
    │  Kali    │  │  Fedora  │  │          │
    └──────────┘  └──────────┘  └──────────┘
```

| Distro | Package Manager | Best For |
|--------|----------------|----------|
| Ubuntu | apt | Servers, beginners, AWS EC2 |
| Debian | apt | Stability, production |
| CentOS/RHEL | yum/dnf | Enterprise, stability |
| Alpine | apk | Docker containers (tiny size) |
| Arch | pacman | Advanced users |

**As a Full-Stack Developer, you'll mostly use Ubuntu.** AWS EC2, DigitalOcean, and most tutorials default to Ubuntu LTS.

---

## 1.6 The Linux Boot Process

Understanding how Linux boots helps you debug server startup issues.

```
POWER ON
    │
    ▼
┌─────────────┐
│    BIOS /   │  ← Firmware checks hardware (POST)
│    UEFI     │    Finds boot device (disk)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   GRUB2     │  ← Boot Loader
│ (Bootloader)│    Loads kernel from /boot
│             │    Shows OS selection menu
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   KERNEL    │  ← Linux kernel loads into RAM
│   LOADING   │    Mounts initramfs (temporary root fs)
│             │    Detects hardware
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  initramfs  │  ← Temporary file system in RAM
│             │    Loads drivers needed to mount real /
│             │    Mounts actual root filesystem
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   systemd   │  ← PID 1: First real process
│  (PID 1)    │    Reads /etc/systemd/
│             │    Starts ALL services
│             │    (Nginx, SSH, MySQL, etc.)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Login     │  ← Terminal or GUI login prompt
│   Prompt    │
└─────────────┘
```

**Key files involved in boot:**
- `/boot/grub/grub.cfg` — GRUB configuration
- `/boot/vmlinuz-*` — The actual kernel binary
- `/etc/systemd/system/` — Service definitions
- `/etc/fstab` — Filesystems to mount at boot

---

## 1.7 Hands-On Exercises — Module 1

```bash
# Exercise 1: Check your kernel version
uname -r
# Example output: 5.15.0-89-generic

# Exercise 2: See full system info
uname -a

# Exercise 3: Check what distro you're on
cat /etc/os-release

# Exercise 4: See how long the system has been running
uptime

# Exercise 5: See kernel boot messages
dmesg | head -50

# Exercise 6: See what systemd targets are available
systemctl list-units --type=target

# Exercise 7: See system architecture
arch
# or
lscpu | grep Architecture
```

**Production Task:**
- SSH into any Ubuntu server (or use a local VM)
- Run `uname -r` and note the kernel version
- Run `cat /etc/os-release` to identify the exact distro
- Run `uptime` to see system load
- Run `dmesg | grep -i error` to check for hardware errors

---

# MODULE 2 — Linux File System Hierarchy

## 2.1 What You Will Learn

- Why Linux has a specific folder structure (it's not random!)
- What every directory is for
- Where to put YOUR application files
- Real-world examples with Docker, Nginx, Node.js

---

## 2.2 Why a Standard File System?

Linux follows the **Filesystem Hierarchy Standard (FHS)**. This is like a building blueprint — every Linux system uses the same folder structure so:
- Sysadmins know where to find things
- Software knows where to install itself
- Scripts are portable across machines

```
/                       ← Root of everything (like C:\ on Windows)
├── bin/                ← Essential user binaries
├── sbin/               ← System binaries (admin tools)
├── boot/               ← Boot files (kernel, GRUB)
├── dev/                ← Device files
├── etc/                ← Configuration files
├── home/               ← User home directories
├── lib/                ← Shared libraries
├── media/              ← Mount points for removable media
├── mnt/                ← Temporary mount points
├── opt/                ← Optional/third-party software
├── proc/               ← Virtual FS (process info from kernel)
├── root/               ← Root user's home
├── run/                ← Runtime data (PIDs, sockets)
├── srv/                ← Service data (web server files)
├── sys/                ← Virtual FS (hardware/kernel info)
├── tmp/                ← Temporary files (cleared on reboot)
├── usr/                ← User programs and libraries
└── var/                ← Variable data (logs, databases, mail)
```

---

## 2.3 Deep Dive — Each Directory Explained

### `/` — The Root

Everything in Linux lives under `/`. There is no `C:\`, `D:\` like Windows. All disks, partitions, and network shares are **mounted** somewhere under this single root.

```bash
ls /
# bin  boot  dev  etc  home  lib  lib64  media  mnt  opt
# proc  root  run  sbin  srv  sys  tmp  usr  var
```

---

### `/home` — User Home Directories

This is where **regular users** store their personal files.

```
/home/
├── alice/          ← alice's personal space
│   ├── .bashrc     ← alice's bash config
│   ├── .ssh/       ← alice's SSH keys
│   └── projects/
├── bob/
└── deploy/         ← often a deploy user for CI/CD
```

**Real-world example:** When you SSH as `ubuntu` into an EC2 instance, you land in `/home/ubuntu`.

```bash
# See your home directory
echo $HOME
# /home/youruser

# Go home quickly
cd ~
cd $HOME

# See all home directories
ls /home/
```

**Production note:** Never run your application as `root`. Create a dedicated user (`deploy`, `www-data`, etc.) with home in `/home/appuser`.

---

### `/etc` — Configuration Files

`/etc` stands for "**et cetera**" (historically) but today means **"Editable Text Configurations."** Every system-level configuration file lives here.

```
/etc/
├── nginx/              ← Nginx config files
│   ├── nginx.conf
│   └── sites-available/
├── ssh/                ← SSH server config
│   └── sshd_config
├── systemd/            ← systemd service files
│   └── system/
├── apt/                ← Package manager config
├── hosts               ← Local DNS entries
├── hostname            ← This machine's name
├── fstab               ← Filesystem mount table
├── passwd              ← User database
├── group               ← Group database
├── shadow              ← Encrypted passwords
├── crontab             ← System cron jobs
└── environment         ← System-wide env variables
```

**Real-world example:**
```bash
# View Nginx config
cat /etc/nginx/nginx.conf

# Edit SSH config (to disable password auth)
sudo nano /etc/ssh/sshd_config

# Add a local hostname alias
sudo echo "192.168.1.10 myapp.local" >> /etc/hosts
```

---

### `/var` — Variable Data

`/var` stores data that **changes constantly** — logs, databases, mail queues, package cache.

```
/var/
├── log/                ← System and application logs
│   ├── nginx/
│   │   ├── access.log
│   │   └── error.log
│   ├── syslog          ← General system log
│   ├── auth.log        ← Authentication log
│   └── kern.log        ← Kernel log
├── lib/                ← Application state data
│   ├── mysql/          ← MySQL data files
│   ├── postgresql/     ← PostgreSQL data
│   └── docker/         ← Docker images/containers
├── cache/              ← Application caches
│   └── apt/            ← Downloaded packages
├── spool/              ← Mail, print queues
└── tmp/                ← Temp files (survive reboot unlike /tmp)
```

**Real-world example:**
```bash
# Tail Nginx access log in real time
tail -f /var/log/nginx/access.log

# Check disk usage of logs (logs can fill your disk!)
du -sh /var/log/*

# Where Docker stores everything
ls /var/lib/docker/
```

**Production warning:** `/var/log` can fill up your disk and crash your server. Always set up log rotation.

---

### `/opt` — Optional/Third-Party Software

This is where you install **large third-party applications** that aren't part of the OS package manager.

```
/opt/
├── myapp/              ← Your custom application
│   ├── bin/
│   ├── config/
│   └── data/
├── minio/              ← MinIO object storage
├── node/               ← Custom Node.js installation
└── redis/              ← Custom Redis installation
```

**Real-world example:**
```bash
# Install your app to /opt
sudo mkdir -p /opt/myapp
sudo cp -r ./dist /opt/myapp/
sudo chown -R appuser:appuser /opt/myapp

# Many companies structure like this:
/opt/company/
/opt/company/api/
/opt/company/worker/
/opt/company/frontend/
```

**When to use `/opt` vs `/usr/local`:**
- `/opt` → Self-contained third-party apps with their own directory tree
- `/usr/local` → Software you compiled yourself that follows standard Unix conventions

---

### `/usr` — User Programs and Libraries

Despite the name, `/usr` is NOT for user home files. It contains **system-wide programs and libraries** available to all users.

```
/usr/
├── bin/            ← Most user commands (ls, grep, curl, python3)
├── sbin/           ← System commands (iptables, nginx, sshd)
├── lib/            ← Libraries for /usr/bin and /usr/sbin
├── local/          ← Locally compiled software
│   ├── bin/
│   └── lib/
├── share/          ← Architecture-independent data
│   ├── man/        ← Manual pages
│   └── doc/        ← Documentation
└── include/        ← C/C++ header files
```

```bash
# See where a command lives
which nginx
# /usr/sbin/nginx

which node
# /usr/bin/node

which python3
# /usr/bin/python3
```

---

### `/bin` and `/sbin` — Essential Binaries

These contain commands needed **even when `/usr` is not mounted** (early boot, recovery mode).

- `/bin` → Essential user commands: `ls`, `cat`, `cp`, `mv`, `rm`, `bash`
- `/sbin` → Essential system commands: `mount`, `fsck`, `ifconfig`, `reboot`

**Modern Linux:** On Ubuntu 20+, `/bin` is actually a symlink to `/usr/bin`. The distinction is becoming historical.

```bash
ls -la /bin
# lrwxrwxrwx 1 root root 7 /bin -> usr/bin
```

---

### `/tmp` — Temporary Files

`/tmp` is for **temporary files** that applications create and don't need to keep.

- **Cleared on every reboot** (usually mounted as tmpfs in RAM)
- Any user can write here
- Files can be deleted by other processes or the OS

```bash
# Create a temp file
echo "test data" > /tmp/mytest.txt

# Check if /tmp is in RAM
df -h /tmp
# tmpfs  7.8G  1.2M  7.8G  1%  /tmp
```

**Real-world example:** When you upload a file to a web server, it often lands in `/tmp` first, then gets moved to permanent storage.

---

### `/proc` and `/sys` — Virtual Filesystems

These are **not real files on disk**. They are virtual filesystems that expose kernel information.

```
/proc/
├── cpuinfo         ← CPU information
├── meminfo         ← Memory information
├── net/            ← Network statistics
├── 1234/           ← Information about PID 1234
│   ├── cmdline     ← Command that started it
│   ├── status      ← Process status
│   └── fd/         ← Open file descriptors
└── sys/            ← Kernel parameters (tunables)
```

```bash
# See all CPU info
cat /proc/cpuinfo

# See memory info
cat /proc/meminfo

# See running processes
ls /proc/ | grep '^[0-9]'

# See what a specific process (PID 1) is
cat /proc/1/cmdline

# Tune kernel networking parameter
echo 1 > /proc/sys/net/ipv4/ip_forward
```

---

## 2.4 Hands-On Exercises — Module 2

```bash
# Exercise 1: Explore the root filesystem
ls -la /

# Exercise 2: Find where Nginx stores its config
find /etc -name "nginx.conf" 2>/dev/null

# Exercise 3: See how much space each /var subdirectory uses
sudo du -sh /var/* 2>/dev/null | sort -hr | head -20

# Exercise 4: See all mounted filesystems
df -hT

# Exercise 5: Explore /proc
cat /proc/version      # Kernel info
cat /proc/uptime       # Uptime in seconds
cat /proc/loadavg      # Load average

# Exercise 6: Find all config files for a service
ls /etc/nginx/
ls /etc/ssh/
ls /etc/systemd/system/

# Exercise 7: Practice navigating
cd /var/log && ls
cd /etc/nginx && cat nginx.conf
cd /opt && ls
```

**Production Task:**
Create this directory structure for a production app:
```bash
sudo mkdir -p /opt/myapp/{bin,config,logs,data}
sudo mkdir -p /var/log/myapp
sudo mkdir -p /etc/myapp
echo "app_version=1.0.0" | sudo tee /etc/myapp/config.env
```

---

# MODULE 3 — Linux Users and Groups

## 3.1 What You Will Learn

- The three types of Linux users and why each exists
- How UIDs and GIDs work
- How to read `/etc/passwd` and `/etc/group`
- Real-world examples: Docker user, MinIO user, Nginx user
- Why proper user management is critical for security

---

## 3.2 Why Users and Groups Exist

Linux is a **multi-user operating system**. From day one, it was designed for many people to use the same machine simultaneously. Users and groups provide:

1. **Isolation** — Alice's files are separate from Bob's
2. **Security** — Nginx can only read web files, not your SSH keys
3. **Accountability** — Know who did what
4. **Least Privilege** — Give a process only the permissions it needs

```
WITHOUT users/groups:           WITH users/groups:
Every process can read          nginx can ONLY read /var/www
every file.                     mysql can ONLY read /var/lib/mysql
nginx could read your SSH       postgres has NO access to nginx config
private key!                    Your app has NO access to system files
```

---

## 3.3 Types of Users

### Type 1: Root User (UID 0)

```
Username: root
UID: 0
Home: /root
Shell: /bin/bash
```

Root is the **superuser**. It can:
- Read/write/execute ANY file
- Kill ANY process
- Bind to ANY port (including ports below 1024)
- Install software
- Modify system files
- Override all permissions

**Production rule:** NEVER run your application as root. If a hacker exploits your app running as root, they own the entire server.

```bash
# Check if you're root
whoami
# root  ← danger

id
# uid=0(root) gid=0(root) groups=0(root)

# Check current user
whoami
echo $USER
```

### Type 2: System Users (UID 1–999)

These are accounts created for **services** — not real humans. They cannot log in interactively (no shell, no password).

```bash
# Common system users on Ubuntu
cat /etc/passwd | grep -v /home | grep -v /root | head -20
# daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
# www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
# mysql:x:999:999:MySQL Server:/var/lib/mysql:/bin/false
# nginx:x:101:101:nginx user:/var/cache/nginx:/sbin/nologin
```

`/usr/sbin/nologin` means "this account cannot be used to log in."

### Type 3: Regular Users (UID 1000+)

Real human users.

```bash
# See all regular users
awk -F: '$3 >= 1000 {print $1, $3}' /etc/passwd

# Typical output:
# ubuntu 1000
# alice 1001
# bob 1002
# deploy 1003
```

---

## 3.4 Understanding `/etc/passwd`

Every user on the system is listed here.

```bash
cat /etc/passwd
```

Format: `username:x:UID:GID:comment:home_dir:shell`

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
deploy:x:1001:1001:Deploy User:/home/deploy:/bin/bash
```

| Field | Example | Meaning |
|-------|---------|---------|
| username | `ubuntu` | Login name |
| password | `x` | Password stored in /etc/shadow |
| UID | `1000` | User ID number |
| GID | `1000` | Primary Group ID |
| comment | `Ubuntu` | Full name/description |
| home | `/home/ubuntu` | Home directory |
| shell | `/bin/bash` | Default shell |

**The `x` in password field** means the actual password hash is stored in `/etc/shadow` (which only root can read).

---

## 3.5 Understanding `/etc/shadow`

This file stores **encrypted password hashes** — only root can read it.

```bash
sudo cat /etc/shadow | head -5
# root:*:19000:0:99999:7:::
# ubuntu:$6$xyz...:19000:0:99999:7:::
```

Format: `user:encrypted_password:last_change:min_age:max_age:warn:inactive:expire:`

- `*` or `!` = account locked (no login)
- `$6$...` = SHA-512 hashed password
- `!` before password = account disabled

---

## 3.6 Understanding `/etc/group`

Groups allow multiple users to share access to resources.

```bash
cat /etc/group | head -20
# root:x:0:
# docker:x:999:ubuntu,alice
# www-data:x:33:
# sudo:x:27:ubuntu
```

Format: `groupname:x:GID:member1,member2,...`

```
docker:x:999:ubuntu,alice,deploy
```

This means users `ubuntu`, `alice`, and `deploy` are all members of the `docker` group.

---

## 3.7 Primary Group vs Secondary Groups

Every user has:
- **One primary group** — assigned at user creation, stored in `/etc/passwd`
- **Zero or more secondary groups** — stored in `/etc/group`

```bash
# See your groups
id
# uid=1000(ubuntu) gid=1000(ubuntu) groups=1000(ubuntu),4(adm),24(cdrom),27(sudo),999(docker)

# The primary group is listed first after gid=
# Secondary groups are everything in "groups="
```

**Why it matters:**
- When you create a file, it gets your primary group as its group owner
- Secondary groups give you ADDITIONAL access

```bash
# Example: ubuntu user creates a file
touch /tmp/test.txt
ls -l /tmp/test.txt
# -rw-r--r-- 1 ubuntu ubuntu 0 Jan 1 10:00 /tmp/test.txt
#              ↑user   ↑group (primary group)
```

---

## 3.8 Managing Users and Groups

### Creating Users

```bash
# Create a user with home directory
sudo useradd -m -s /bin/bash alice
# -m = create home directory
# -s = set shell

# Create user with specific UID/GID
sudo useradd -m -u 1050 -g 1050 -s /bin/bash bob

# Create a system user (for services) - no login, no home
sudo useradd --system --no-create-home --shell /usr/sbin/nologin myservice

# Set password
sudo passwd alice
```

### Modifying Users

```bash
# Add user to a group
sudo usermod -aG docker alice
# -a = append (don't remove from other groups)
# -G = secondary groups

# Change user's shell
sudo usermod -s /bin/zsh alice

# Lock an account
sudo usermod -L alice

# Unlock an account
sudo usermod -U alice
```

### Creating Groups

```bash
# Create a group
sudo groupadd developers

# Create a group with specific GID
sudo groupadd -g 2000 appteam

# Add multiple users to a group
sudo usermod -aG developers alice
sudo usermod -aG developers bob
sudo usermod -aG developers deploy
```

### Deleting Users

```bash
# Delete user (keep home directory)
sudo userdel alice

# Delete user AND home directory
sudo userdel -r alice
```

---

## 3.9 Real-World Examples

### Docker User

When you install Docker, it creates a `docker` group:

```bash
# Why Docker uses a group:
# Docker daemon (dockerd) runs as ROOT
# But regular users shouldn't need sudo to use docker
# Solution: add users to "docker" group
# The docker socket is owned by root:docker

ls -la /var/run/docker.sock
# srw-rw---- 1 root docker 0 Jan 1 10:00 /var/run/docker.sock
#              ↑root  ↑docker group

# Users in "docker" group can read/write this socket
sudo usermod -aG docker $USER
newgrp docker   # Apply group change without logout
docker ps       # Now works without sudo!
```

### MinIO User

```bash
# Create a dedicated MinIO user
sudo useradd --system \
  --no-create-home \
  --shell /usr/sbin/nologin \
  --comment "MinIO Object Storage" \
  minio-user

# Create MinIO directories
sudo mkdir -p /opt/minio/data
sudo chown -R minio-user:minio-user /opt/minio
sudo chmod 750 /opt/minio

# MinIO runs as minio-user — if exploited, attacker only
# has minio-user permissions, not root!
```

### Nginx User

```bash
# Nginx creates www-data user during install
id www-data
# uid=33(www-data) gid=33(www-data) groups=33(www-data)

# Web files should be readable by www-data
sudo chown -R www-data:www-data /var/www/html
sudo chmod 755 /var/www/html

# Check Nginx worker processes
ps aux | grep nginx
# root     1234  nginx: master process
# www-data 1235  nginx: worker process
# www-data 1236  nginx: worker process
# Master runs as root (to bind port 80/443)
# Workers run as www-data (to serve files)
```

---

## 3.10 Hands-On Exercises — Module 3

```bash
# Exercise 1: Explore current users
cat /etc/passwd
awk -F: '$3 >= 1000' /etc/passwd   # Show only real users

# Exercise 2: Check your identity
id
whoami
groups

# Exercise 3: Create an application user
sudo useradd --system \
  --no-create-home \
  --shell /usr/sbin/nologin \
  --comment "My App Service" \
  myapp

# Verify it was created
id myapp
grep myapp /etc/passwd

# Exercise 4: Create a development group
sudo groupadd developers
sudo usermod -aG developers $USER

# Verify group membership
groups $USER
id $USER

# Exercise 5: Examine the docker group (if Docker installed)
cat /etc/group | grep docker
ls -la /var/run/docker.sock

# Exercise 6: See what groups exist on your system
cat /etc/group | awk -F: '{print $1, $3}' | sort -t' ' -k2 -n

# Exercise 7: See who is currently logged in
who
w
last | head -20
```

**Production Task:**
Set up a proper deploy user for a web application:

```bash
# 1. Create deploy user
sudo useradd -m -s /bin/bash deploy

# 2. Create app group
sudo groupadd webapp

# 3. Add deploy to webapp group
sudo usermod -aG webapp deploy

# 4. Add www-data to webapp group (so nginx can read files)
sudo usermod -aG webapp www-data

# 5. Create app directory with proper ownership
sudo mkdir -p /opt/webapp
sudo chown deploy:webapp /opt/webapp
sudo chmod 2750 /opt/webapp  # setgid so new files inherit group
```

---

# MODULE 4 — Linux Permissions

## 4.1 What You Will Learn

- The rwx permission system in complete depth
- Numeric (octal) permissions: 755, 644, 700
- Difference between file and directory permissions
- Special permissions: sticky bit, setuid, setgid
- Real production permission scenarios

---

## 4.2 Why Permissions Exist

Permissions are Linux's **access control system**. Every file and directory has:

1. An **owner** (a user)
2. A **group owner** (a group)
3. **Permission bits** (who can do what)

```
Without permissions:              With permissions:
Anyone can read /etc/shadow       Only root can read /etc/shadow
Anyone can delete /etc/nginx/     Only root/nginx can modify it
Anyone can read ~/.ssh/id_rsa     Only YOU can read your private key
```

---

## 4.3 The Permission Model

Every file has 9 permission bits organized into 3 groups of 3:

```
-rwxrwxrwx
│└─┬─┘└─┬─┘└─┬─┘
│  │    │    └── Other (everyone else)
│  │    └─────── Group
│  └──────────── Owner/User
└────────────────  File type (- = file, d = directory, l = symlink)
```

Each group of 3 bits: `rwx`
- `r` = read (4)
- `w` = write (2)
- `x` = execute (1)
- `-` = permission not granted (0)

```bash
# See permissions
ls -l /etc/passwd
# -rw-r--r-- 1 root root 2847 Jan 1 10:00 /etc/passwd
#  ↑↑↑ ↑↑↑ ↑↑↑
#  owner group other
#  rw-   r--   r--
# root:rw, group:r, others:r

ls -l /etc/shadow
# -rw-r----- 1 root shadow 1563 Jan 1 10:00 /etc/shadow
#  rw-  r-- ---
# root:rw, shadow-group:r, others: NOTHING
```

---

## 4.4 Numeric (Octal) Permissions

Each permission has a numeric value:

```
r = 4
w = 2
x = 1
- = 0
```

You add them up:

```
rwx = 4+2+1 = 7
rw- = 4+2+0 = 6
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
--- = 0+0+0 = 0
```

Three digits for owner/group/other:

```
755 = rwxr-xr-x  (owner:rwx, group:r-x, others:r-x)
644 = rw-r--r--  (owner:rw-, group:r--, others:r--)
700 = rwx------  (owner:rwx, group:---, others:---)
600 = rw-------  (owner:rw-, group:---, others:---)
777 = rwxrwxrwx  (everyone has everything - DANGEROUS)
```

### Common Permission Values

| Permission | Numeric | Use Case |
|------------|---------|----------|
| `rwxr-xr-x` | 755 | Directories, executables |
| `rw-r--r--` | 644 | Regular files (config, html) |
| `rwx------` | 700 | Private directories |
| `rw-------` | 600 | Private files (SSH keys, .env) |
| `rwxrwxr-x` | 775 | Shared group directories |
| `rw-rw-r--` | 664 | Group-editable files |
| `rwxrwxrwx` | 777 | ⚠️ NEVER use in production |

---

## 4.5 Permissions on Files vs Directories

**This is where most developers get confused.** `r`, `w`, `x` mean DIFFERENT things for files vs directories:

### File Permissions

| Permission | Meaning for Files |
|------------|-------------------|
| `r` | Can read the file contents (`cat file`, `less file`) |
| `w` | Can modify the file contents (edit, append) |
| `x` | Can execute the file as a program |

### Directory Permissions

| Permission | Meaning for Directories |
|------------|-------------------------|
| `r` | Can list contents (`ls dir/`) |
| `w` | Can create/delete files inside |
| `x` | Can enter the directory (`cd dir/`) AND access files inside |

**Critical insight:** `x` on a directory is needed to ACCESS anything inside it, even if you know the exact filename.

```bash
# Example: directory has r but not x
chmod 644 /tmp/testdir
ls /tmp/testdir    # CAN list (r works)
cat /tmp/testdir/file.txt   # CANNOT read (need x to traverse)

# Example: directory has x but not r
chmod 311 /tmp/secretdir
ls /tmp/secretdir   # CANNOT list (no r)
cat /tmp/secretdir/file.txt  # CAN read if you know the filename (x allows traversal)
```

---

## 4.6 `chmod` — Changing Permissions

### Numeric Mode

```bash
# Set permissions with numbers
chmod 755 script.sh    # Owner: rwx, Group: r-x, Others: r-x
chmod 644 config.txt   # Owner: rw-, Group: r--, Others: r--
chmod 600 ~/.ssh/id_rsa  # Owner: rw-, Group: none, Others: none

# Apply recursively
chmod -R 755 /var/www/html
```

### Symbolic Mode

```bash
# Symbolic: [who][operator][permissions]
# who: u(ser/owner), g(roup), o(ther), a(ll)
# operator: +(add), -(remove), =(set)

chmod u+x script.sh      # Add execute for owner
chmod g-w config.txt     # Remove write from group
chmod o-r secret.txt     # Remove read from others
chmod a+r public.txt     # Add read for everyone
chmod u+x,g-w file.txt   # Multiple changes at once
chmod go= private.txt    # Remove ALL permissions from group and others
```

---

## 4.7 `chown` — Changing Ownership

```bash
# Change owner
sudo chown alice file.txt

# Change owner AND group
sudo chown alice:developers file.txt

# Change only group (using chgrp)
sudo chgrp developers file.txt
# or
sudo chown :developers file.txt

# Recursive (entire directory tree)
sudo chown -R www-data:www-data /var/www/html
sudo chown -R deploy:deploy /opt/myapp
```

---

## 4.8 Real Permission Scenarios

### Scenario 1: Nginx Web Server

```bash
# Web root permissions
# Owner: root (or deploy user), Group: www-data, Mode: 755 for dirs, 644 for files
sudo chown -R root:www-data /var/www/html
sudo find /var/www/html -type d -exec chmod 755 {} \;
sudo find /var/www/html -type f -exec chmod 644 {} \;

# Result:
# drwxr-xr-x root www-data /var/www/html/
# -rw-r--r-- root www-data /var/www/html/index.html
# Nginx (www-data) can READ files but not WRITE them
# This prevents a hacked nginx from modifying your files
```

### Scenario 2: SSH Private Key

```bash
# SSH REQUIRES private keys to be 600 or 400
# If permissions are too open, SSH REFUSES to use the key!
chmod 600 ~/.ssh/id_rsa      # Owner read/write only
chmod 644 ~/.ssh/id_rsa.pub  # Public key can be world-readable
chmod 700 ~/.ssh/             # SSH dir: owner only
chmod 644 ~/.ssh/authorized_keys

# Correct setup:
# drwx------ 2 alice alice  40 ~/.ssh/
# -rw------- 1 alice alice 411 ~/.ssh/id_rsa       (600)
# -rw-r--r-- 1 alice alice 100 ~/.ssh/id_rsa.pub   (644)
# -rw-r--r-- 1 alice alice  50 ~/.ssh/authorized_keys (644)
```

### Scenario 3: Application .env File

```bash
# .env contains DB passwords, API keys
# NEVER make it world-readable!
chmod 600 /opt/myapp/.env
chown deploy:deploy /opt/myapp/.env

# Result: -rw------- deploy deploy .env
# Only the deploy user can read it
```

---

## 4.9 Special Permissions: setuid, setgid, Sticky Bit

These are advanced permission features used in specific scenarios.

### setuid (SUID) — bit 4000

When set on an **executable file**, it runs with the **file owner's permissions** instead of the caller's.

```bash
# Example: /usr/bin/passwd lets any user change their password
# But password file /etc/shadow is only writable by root!
# How? passwd has setuid bit!

ls -l /usr/bin/passwd
# -rwsr-xr-x 1 root root 59976 /usr/bin/passwd
#     ↑ 's' = setuid is set AND execute is set

# When alice runs passwd, it temporarily runs as root
# The 's' replaces 'x' in owner's execute position
# Numeric: 4755 (4000 + 755)

# Set setuid:
chmod u+s /usr/bin/myapp
chmod 4755 /usr/bin/myapp
```

**Security warning:** setuid programs running as root are high-value attack targets. Use carefully.

### setgid (SGID) — bit 2000

Two uses:

**On files:** Runs with file's group permissions.

**On directories (the common production use):** New files created inside inherit the directory's group (not the creator's primary group).

```bash
# Production use: shared project directory
sudo mkdir /opt/webapp
sudo chown alice:developers /opt/webapp
sudo chmod 2775 /opt/webapp   # setgid + rwxrwxr-x
#                ↑ 2 = setgid

# Now when bob (in developers group) creates a file:
su bob
touch /opt/webapp/newfile.txt
ls -l /opt/webapp/
# -rw-r--r-- 1 bob developers newfile.txt
#                  ↑ automatically 'developers' (inherited from dir)

# WITHOUT setgid:
# -rw-r--r-- 1 bob bob newfile.txt  ← bob's primary group
# Other developers can't write to it!
```

### Sticky Bit — bit 1000

When set on a **directory**, only the **file owner** can delete their own files, even if others have write permission.

```bash
# Classic example: /tmp
ls -ld /tmp
# drwxrwxrwt 8 root root 4096 /tmp
#          ↑ 't' = sticky bit set
# Numeric: 1777

# Everyone can write to /tmp
# But you can only DELETE your own files
# Alice cannot delete Bob's files in /tmp

# Set sticky bit:
chmod +t /shared/uploads
chmod 1777 /shared/uploads

# Real-world use:
# Shared upload directories where users shouldn't delete others' files
sudo chmod 1777 /var/www/uploads
```

---

## 4.10 `umask` — Default Permission Control

When you create a new file or directory, what permissions does it get?

The `umask` controls this by **subtracting** from the maximum:

```
Maximum for files:      666 (rw-rw-rw-)
Maximum for dirs:       777 (rwxrwxrwx)
Default umask:          022

File permissions:       666 - 022 = 644 (rw-r--r--)
Directory permissions:  777 - 022 = 755 (rwxr-xr-x)
```

```bash
# Check current umask
umask
# 0022

# Change umask for current session
umask 027
# Files created: 640 (rw-r-----)
# Dirs created:  750 (rwxr-x---)

# Set permanent umask in ~/.bashrc
echo "umask 027" >> ~/.bashrc
```

---

## 4.11 Viewing and Understanding Permission Output

```bash
ls -la /var/www/html
# total 24
# drwxr-xr-x 3 root     www-data 4096 Jan 1 10:00 .
# drwxr-xr-x 3 root     root     4096 Jan 1 09:00 ..
# -rw-r--r-- 1 www-data www-data  612 Jan 1 10:00 index.html
# -rwxr-xr-x 1 root     www-data  450 Jan 1 10:00 deploy.sh
# drwxr-s--- 2 www-data www-data 4096 Jan 1 10:00 uploads
# ↑↑↑↑↑↑↑↑↑↑
# │└────┬───┘
# │     └── rwxr-xr-x (permissions)
# └── d=dir, -=file, l=symlink, s=socket
```

---

## 4.12 Hands-On Exercises — Module 4

```bash
# Exercise 1: Create files and observe default permissions
mkdir /tmp/permtest
touch /tmp/permtest/file.txt
ls -l /tmp/permtest/

# Exercise 2: Practice chmod
chmod 755 /tmp/permtest/file.txt
ls -l /tmp/permtest/file.txt

chmod 600 /tmp/permtest/file.txt
ls -l /tmp/permtest/file.txt

chmod u+x /tmp/permtest/file.txt
ls -l /tmp/permtest/file.txt

# Exercise 3: Practice chown (as root)
sudo chown root:root /tmp/permtest/file.txt
ls -l /tmp/permtest/file.txt
sudo chown $USER:$USER /tmp/permtest/file.txt

# Exercise 4: Test directory permissions
mkdir /tmp/dirtest
chmod 000 /tmp/dirtest
ls /tmp/dirtest        # What happens?
cd /tmp/dirtest        # What happens?
chmod 755 /tmp/dirtest # Fix it

# Exercise 5: Test setgid on directory
sudo mkdir /tmp/shareddir
sudo chown $USER:$(id -gn) /tmp/shareddir
sudo chmod 2775 /tmp/shareddir
ls -ld /tmp/shareddir

# Exercise 6: Find all setuid files on the system
find / -perm -4000 -type f 2>/dev/null

# Exercise 7: Fix "wrong permissions" scenarios
# SSH key setup
mkdir -p ~/.ssh/test
touch ~/.ssh/test/id_rsa
chmod 700 ~/.ssh/test
chmod 600 ~/.ssh/test/id_rsa
ls -la ~/.ssh/test/
```

**Production Task: Secure a web application directory**

```bash
# Full production permission setup:
APP_DIR=/opt/myapp
APP_USER=deploy
WEB_GROUP=www-data

sudo mkdir -p $APP_DIR/{public,config,logs,tmp}

# Set ownership
sudo chown -R $APP_USER:$WEB_GROUP $APP_DIR

# Set directory permissions
sudo find $APP_DIR -type d -exec chmod 755 {} \;

# Set file permissions
sudo find $APP_DIR -type f -exec chmod 644 {} \;

# Private config files (secrets)
sudo chmod 640 $APP_DIR/config/*.env

# Executable scripts
sudo chmod 755 $APP_DIR/bin/*.sh

# Uploads directory (web-writable)
sudo chmod 2775 $APP_DIR/public/uploads
sudo chown $APP_USER:$WEB_GROUP $APP_DIR/public/uploads

# Logs directory (app-writable)
sudo chmod 750 $APP_DIR/logs
sudo chown $APP_USER:$APP_USER $APP_DIR/logs
```

---

# MODULE 5 — Linux Process Management

## 5.1 What You Will Learn

- How Linux manages running programs (processes)
- How to view, monitor, and control processes
- The difference between foreground and background jobs
- What daemon processes are
- Signals and process communication

---

## 5.2 What Is a Process?

A **process** is a running instance of a program. Every time you run `nginx`, `node`, `python`, or any command — it becomes a process.

```
Program (on disk) → Run it → Process (in memory)
/usr/bin/nginx   →  start  → nginx process (PID: 1234)
/usr/bin/node    →  start  → node process  (PID: 5678)
```

Every process has:
- **PID** (Process ID) — unique number
- **PPID** (Parent Process ID) — who spawned it
- **UID** — which user it runs as
- **State** — running, sleeping, stopped, zombie
- **CPU and memory usage**

### Process Hierarchy

Linux processes form a **tree**. Every process has a parent:

```
PID 1: systemd (the mother of all processes)
├── PID 234: sshd (SSH daemon)
│   └── PID 456: sshd (your SSH session)
│       └── PID 457: bash (your shell)
│           └── PID 890: nginx (you started nginx)
├── PID 300: nginx master
│   ├── PID 301: nginx worker
│   └── PID 302: nginx worker
└── PID 400: mysql
```

---

## 5.3 Viewing Processes

### `ps` — Process Snapshot

```bash
# Show processes for current user
ps

# Show ALL processes with full details (most common)
ps aux
# a = all users
# u = user-oriented format
# x = include processes without terminal

# Example output:
# USER       PID %CPU %MEM    VSZ   RSS TTY   STAT  COMMAND
# root         1  0.0  0.1 225408  9012 ?     Ss    /sbin/init
# www-data   301  0.0  0.2  56832 16244 ?     S     nginx: worker
# ubuntu    1234  0.0  0.1  14524  5124 pts/0 Ss    bash
# ubuntu    5678  2.3  3.1 900124 254128 ?    Sl    node server.js

# Filter by process name
ps aux | grep nginx
ps aux | grep node

# Show process tree
ps axjf      # ASCII tree
pstree       # Better tree view
pstree -p    # With PIDs
```

### Process States (STAT column)

| State | Meaning |
|-------|---------|
| `R` | Running or runnable |
| `S` | Sleeping (waiting for event) |
| `D` | Uninterruptible sleep (usually I/O) |
| `Z` | Zombie (finished but parent hasn't cleaned up) |
| `T` | Stopped |
| `s` | Session leader |
| `<` | High priority |
| `N` | Low priority |

### `top` — Real-Time Process Monitor

```bash
top
```

```
top - 10:30:00 up 5 days, 3:22,  2 users,  load average: 0.52, 0.58, 0.59
Tasks: 124 total,   1 running, 123 sleeping,   0 stopped,   0 zombie
%Cpu(s):  2.3 us,  0.7 sy,  0.0 ni, 96.7 id,  0.3 wa,  0.0 hi,  0.0 si
MiB Mem :   7865.4 total,   2341.2 free,   3892.6 used,   1631.6 buff/cache
MiB Swap:   2048.0 total,   2045.3 free,      2.7 used.   3631.8 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
 1234 node      20   0  900124 254128  45632 S   2.3   3.1   5:23.45 node
  300 root      20   0   56832   5124   3456 S   0.3   0.1   0:01.23 nginx
  400 mysql     20   0 1234567 345678  23456 S   0.1   4.4  10:45.67 mysqld
```

**Key top metrics explained:**
- **load average** — 0.52, 0.58, 0.59 = average load over 1m, 5m, 15m
  - On a 4-core machine: load of 4.0 = 100% busy
  - Load > number of cores = overloaded
- **%CPU us** — user space CPU usage
- **%CPU sy** — kernel (system) CPU usage
- **%CPU wa** — waiting for I/O (high = disk bottleneck)
- **RES** — actual RAM being used
- **VIRT** — virtual memory (includes shared libs, etc.)

**Interactive top commands:**
```
k     → kill a process (enter PID)
r     → renice (change priority)
M     → sort by memory
P     → sort by CPU
q     → quit
1     → show per-CPU stats
```

### `htop` — Better Interactive Monitor

```bash
# Install if not present
sudo apt install htop

htop
```

`htop` is `top` with colors, mouse support, and easier killing.

```bash
# Useful htop shortcuts:
F5    → Tree view
F6    → Sort
F9    → Kill
F10   → Quit
/     → Search for process
```

---

## 5.4 Signals — Communicating With Processes

Signals are **messages** you send to processes to trigger behavior:

```bash
# List all signals
kill -l
# 1) SIGHUP  2) SIGINT  3) SIGQUIT  4) SIGILL
# 9) SIGKILL 15) SIGTERM 18) SIGCONT 19) SIGSTOP
```

| Signal | Number | Meaning | Catchable? |
|--------|--------|---------|------------|
| SIGHUP | 1 | Hangup / reload config | Yes |
| SIGINT | 2 | Interrupt (Ctrl+C) | Yes |
| SIGTERM | 15 | Terminate gracefully | Yes |
| SIGKILL | 9 | Kill immediately | **NO** |
| SIGSTOP | 19 | Pause process | **NO** |
| SIGCONT | 18 | Resume paused process | Yes |

### Sending Signals

```bash
# The kill command sends signals (despite its name)
# kill -SIGNAL PID

# Graceful stop (SIGTERM - process cleans up)
kill 1234
kill -15 1234
kill -SIGTERM 1234

# Force kill (SIGKILL - process cannot refuse)
kill -9 1234
kill -SIGKILL 1234

# Reload config without restart (SIGHUP)
kill -1 $(cat /var/run/nginx.pid)
# or
sudo nginx -s reload

# Kill by name
pkill nginx          # Kill all nginx processes
pkill -9 nginx       # Force kill all nginx
killall node         # Kill all 'node' processes

# Kill by name with confirmation
pkill -i -u alice   # Kill all alice's processes
```

### When to Use SIGTERM vs SIGKILL

```
SIGTERM (15) — "Please stop when you're ready"
- Process can catch this
- Can save state, close connections, flush buffers
- Graceful shutdown
- Always try this FIRST

SIGKILL (9) — "Stop RIGHT NOW, no questions asked"
- Cannot be caught or ignored
- Process is killed immediately
- May leave open connections, incomplete writes
- Use when SIGTERM doesn't work
- Wait at least 5-10 seconds after SIGTERM before trying SIGKILL
```

---

## 5.5 Background Jobs

### Running Jobs in Background

```bash
# Start a job in background with &
node server.js &
# [1] 12345  ← job number and PID

# List background jobs
jobs
# [1]+  Running    node server.js

# Bring job to foreground
fg %1
fg     # most recent job

# Send foreground job to background:
# 1. Press Ctrl+Z (suspends/stops it)
# 2. Run: bg %1  (resumes it in background)

# Disconnect from terminal but keep running
nohup node server.js &
# nohup = "no hangup" - keeps running after you log out

# Better: use a process manager or systemd instead
```

### `screen` and `tmux` for Persistent Sessions

```bash
# tmux (recommended)
tmux new -s mysession    # Create named session
# ... do work ...
Ctrl+B, D               # Detach (session keeps running)
tmux attach -t mysession # Reconnect later

# screen
screen -S mysession
# ... do work ...
Ctrl+A, D              # Detach
screen -r mysession    # Reconnect
```

---

## 5.6 System Processes and Daemons

A **daemon** is a background process that provides a service:
- Runs continuously
- No controlling terminal
- Started at boot by systemd
- Name often ends in `d`: `sshd`, `nginx`, `mysqld`, `dockerd`

```bash
# See all daemon processes
ps aux | grep -v grep | awk '$7 == "?" {print}'
# ? in the TTY column means no controlling terminal = daemon
```

### `nice` and Priority

Processes have a **priority** (nice value) from -20 (highest priority) to 19 (lowest):

```bash
# Start a process with low priority (nice 10)
nice -n 10 ./backup.sh

# Change priority of running process
sudo renice 5 -p 1234  # Set PID 1234 to nice 5

# High priority (needs root)
sudo nice -n -10 ./critical-task.sh
```

---

## 5.7 `proc` Filesystem — Deep Process Info

```bash
# Every process has a directory in /proc/PID/
cat /proc/1234/cmdline     # Command that started it (null-separated)
cat /proc/1234/status      # Detailed status
cat /proc/1234/environ     # Environment variables
ls /proc/1234/fd/          # Open file descriptors
cat /proc/1234/maps        # Memory maps
cat /proc/1234/net/tcp     # Network connections

# Practical: find all open files of a process
sudo ls -la /proc/$(pgrep nginx | head -1)/fd
```

---

## 5.8 Hands-On Exercises — Module 5

```bash
# Exercise 1: Explore running processes
ps aux
ps aux | wc -l   # How many processes?

# Exercise 2: Find specific processes
ps aux | grep nginx
ps aux | grep -v grep | grep node

# Exercise 3: Start a background job
sleep 300 &
sleep 400 &
jobs
fg %1          # Bring first job foreground
# Ctrl+Z       # Suspend it
bg %1          # Put it back in background
kill %1 %2     # Kill both jobs

# Exercise 4: Practice signals
# Open another terminal, start a long process
sleep 9999 &
PID=$!
echo "PID is $PID"

# From this terminal:
kill -SIGTERM $PID   # Try graceful
ps aux | grep $PID   # Check if dead
kill -9 $PID         # Force if needed

# Exercise 5: Monitor resources
top          # Watch for 30 seconds
htop         # Compare

# Exercise 6: Explore /proc
ls /proc/ | head -20
cat /proc/cpuinfo | grep "model name" | head -1
cat /proc/meminfo | head -5
cat /proc/$(pgrep bash | head -1)/status

# Exercise 7: Nice values
nice -n 15 dd if=/dev/zero of=/dev/null &
ps aux | grep dd
sudo renice 0 -p $!
kill $!
```

**Production Task: Diagnose a "high load" server**

```bash
# Step 1: Check overall load
uptime
# load average: 8.5, 7.2, 5.1  (on a 4-core machine = overloaded!)

# Step 2: Find CPU-hungry processes
ps aux --sort=-%cpu | head -10

# Step 3: Find memory-hungry processes
ps aux --sort=-%mem | head -10

# Step 4: Check for zombie processes
ps aux | awk '$8 == "Z" {print}'

# Step 5: See what's using I/O
sudo iotop -o   # Show only processes doing I/O

# Step 6: Check per-CPU usage in top
top
# Press '1' to see each CPU core

# Step 7: If a process is stuck
strace -p PID   # Trace system calls it's making
# This shows EXACTLY what the process is doing
```

---

---

# MODULE 6 — Linux Services & systemd

## 6.1 What You Will Learn

- How systemd manages every service on your server
- How to start, stop, enable, disable, and monitor services
- How to create a custom systemd service
- Real-world example: Running a Node.js app as a service

---

## 6.2 What Is systemd?

**systemd** is the init system — PID 1, the first process that starts after the kernel loads, and the parent of everything else.

It replaced the older `SysV init` system and is now standard on Ubuntu, Debian, CentOS, RHEL, and most modern Linux distros.

```
Before systemd (SysV init):
- Services started with shell scripts in /etc/init.d/
- Sequential: one service started after another
- Slow boot, no dependency management

With systemd:
- Services are declarative unit files
- Parallel startup (much faster boot)
- Automatic restart on failure
- Dependency management
- Centralized logging (journald)
- Socket activation, cgroups integration
```

### The systemd Unit File

systemd uses **unit files** (`.service`, `.timer`, `.socket`, etc.) to define services.

```
/etc/systemd/system/         ← Your custom services go here
/usr/lib/systemd/system/     ← System-installed services
/run/systemd/system/         ← Runtime generated units
```

---

## 6.3 `systemctl` — The Main Tool

```bash
# Service lifecycle
sudo systemctl start nginx      # Start now
sudo systemctl stop nginx       # Stop now
sudo systemctl restart nginx    # Stop + Start
sudo systemctl reload nginx     # Reload config without restart
sudo systemctl status nginx     # Check status

# Enable/disable (controls boot behavior)
sudo systemctl enable nginx     # Start automatically at boot
sudo systemctl disable nginx    # Don't start at boot
sudo systemctl is-enabled nginx # Check if enabled

# Combine: enable AND start right now
sudo systemctl enable --now nginx

# System-wide operations
sudo systemctl reboot           # Reboot system
sudo systemctl poweroff         # Shutdown
sudo systemctl suspend          # Suspend
```

### Reading `systemctl status` Output

```bash
sudo systemctl status nginx
```

```
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2024-01-01 10:00:00 UTC; 5 days ago
       Docs: man:nginx(8)
    Process: 300 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
   Main PID: 301 (nginx)
      Tasks: 3 (limit: 4915)
     Memory: 12.3M
        CPU: 1min 23.456s
     CGroup: /system.slice/nginx.service
             ├─301 nginx: master process /usr/sbin/nginx -g daemon off;
             ├─302 nginx: worker process
             └─303 nginx: worker process

Jan 01 10:00:00 server nginx[300]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Jan 01 10:00:00 server systemd[1]: Started A high performance web server...
```

Key fields:
- **Loaded** — Is the unit file found? Is it enabled (starts at boot)?
- **Active** — `active (running)` = good; `failed` = crashed
- **Main PID** — The primary process ID
- **CGroup** — All processes belonging to this service
- **Recent logs** — Last few log lines

---

## 6.4 Anatomy of a systemd Service File

```ini
[Unit]
Description=My Application
Documentation=https://myapp.com/docs
After=network.target postgresql.service
Requires=postgresql.service

[Service]
Type=simple
User=deploy
Group=deploy
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/node /opt/myapp/server.js
ExecReload=/bin/kill -HUP $MAINPID
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal
SyslogIdentifier=myapp
Environment=NODE_ENV=production
EnvironmentFile=/opt/myapp/.env

[Install]
WantedBy=multi-user.target
```

### [Unit] Section

```ini
[Unit]
Description=Human-readable name shown in logs and status
Documentation=URL or man page
After=network.target           # Start AFTER network is up
After=mysql.service            # Start AFTER MySQL
Wants=redis.service            # Start redis if possible (soft dep)
Requires=postgresql.service    # MUST have postgresql (hard dep)
```

### [Service] Section

```ini
[Service]
# Type= defines how systemd knows service is ready
Type=simple      # ExecStart IS the main process (most common)
Type=forking     # Process forks and parent exits (old-style daemons)
Type=oneshot     # Run once and exit (scripts)
Type=notify      # Process sends sd_notify() when ready

User=deploy      # Run as this user
Group=deploy     # Run with this group

WorkingDirectory=/opt/myapp  # Change to this dir before starting

ExecStart=/usr/bin/node server.js   # Command to start service
ExecStop=/usr/bin/node graceful-stop.js  # Optional custom stop
ExecReload=/bin/kill -HUP $MAINPID  # Command for reload

# Restart behavior
Restart=always          # Always restart on any exit
Restart=on-failure      # Restart only on non-zero exit
Restart=on-abnormal     # Restart on signal/timeout
RestartSec=5            # Wait 5 seconds before restarting

# Logging
StandardOutput=journal  # Send stdout to systemd journal
StandardError=journal   # Send stderr to systemd journal
SyslogIdentifier=myapp  # Tag for log filtering

# Environment
Environment=NODE_ENV=production PORT=3000
EnvironmentFile=/opt/myapp/.env   # Load from file
```

### [Install] Section

```ini
[Install]
WantedBy=multi-user.target   # Normal server mode (most services)
# This creates a symlink in multi-user.target.wants/
# That's how "enable" works!
```

---

## 6.5 Real-World Example: Node.js Server as a Service

### Step 1: Create Your App

```bash
# Your app
cat /opt/myapp/server.js
```

```javascript
const http = require('http');
const PORT = process.env.PORT || 3000;

const server = http.createServer((req, res) => {
  res.writeHead(200);
  res.end('Hello from Linux service!\n');
});

server.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

// Graceful shutdown
process.on('SIGTERM', () => {
  console.log('Received SIGTERM, shutting down gracefully...');
  server.close(() => {
    console.log('Server closed');
    process.exit(0);
  });
});
```

### Step 2: Create the Service User

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin nodeapp
sudo mkdir -p /opt/myapp
sudo chown -R nodeapp:nodeapp /opt/myapp
```

### Step 3: Create the Service File

```bash
sudo nano /etc/systemd/system/myapp.service
```

```ini
[Unit]
Description=My Node.js Application
Documentation=https://github.com/mycompany/myapp
After=network.target

[Service]
Type=simple
User=nodeapp
Group=nodeapp
WorkingDirectory=/opt/myapp

# The actual start command
ExecStart=/usr/bin/node /opt/myapp/server.js

# Restart automatically on crash
Restart=on-failure
RestartSec=10
StartLimitInterval=60
StartLimitBurst=3

# Environment
Environment=NODE_ENV=production
Environment=PORT=3000
EnvironmentFile=-/opt/myapp/.env
# The - means: don't fail if file doesn't exist

# Resource limits
LimitNOFILE=65536       # Max open files
LimitNPROC=4096         # Max processes

# Security hardening
NoNewPrivileges=yes
PrivateTmp=yes

# Logging
StandardOutput=journal
StandardError=journal
SyslogIdentifier=myapp

[Install]
WantedBy=multi-user.target
```

### Step 4: Enable and Start

```bash
# Reload systemd to pick up new file
sudo systemctl daemon-reload

# Enable at boot AND start now
sudo systemctl enable --now myapp

# Check status
sudo systemctl status myapp

# Watch logs live
sudo journalctl -u myapp -f
```

### Step 5: Verify It Works

```bash
# Test the service
curl http://localhost:3000

# Simulate a crash (kill the process)
sudo kill -9 $(pgrep -f "node.*server.js")
# Wait 10 seconds...
sudo systemctl status myapp   # Should show it restarted!

# See restart history
sudo journalctl -u myapp | grep -E "start|stop|exit"
```

---

## 6.6 Other systemd Commands

```bash
# List all services
systemctl list-units --type=service

# List failed services (critical for debugging!)
systemctl list-units --failed
# or
systemctl --failed

# See service dependencies
systemctl list-dependencies nginx

# Show unit file content
systemctl cat nginx

# Edit unit file (creates override)
sudo systemctl edit nginx

# Reload all unit files (after any change)
sudo systemctl daemon-reload

# Mask a service (prevent ANY startup)
sudo systemctl mask bluetooth
# Unmask
sudo systemctl unmask bluetooth
```

---

## 6.7 Hands-On Exercises — Module 6

```bash
# Exercise 1: Inspect common services
sudo systemctl status nginx
sudo systemctl status ssh
sudo systemctl status cron

# Exercise 2: Check what's running
systemctl list-units --type=service --state=running

# Exercise 3: Check what's failed
systemctl --failed

# Exercise 4: Read a service file
systemctl cat nginx
systemctl cat ssh

# Exercise 5: Create a simple service
cat > /tmp/hello.service << 'EOF'
[Unit]
Description=Hello World Service
After=network.target

[Service]
Type=oneshot
ExecStart=/bin/echo "Hello from systemd!"
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
EOF

sudo cp /tmp/hello.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl start hello
sudo systemctl status hello

# Exercise 6: View service logs
sudo journalctl -u nginx --since "1 hour ago"
sudo journalctl -u myapp -f   # Follow live

# Exercise 7: Create the full Node.js service from this module
# Follow the 5 steps above
```

---

# MODULE 7 — Linux Networking

## 7.1 What You Will Learn

- How to inspect and configure network interfaces
- How to check open ports and connections
- DNS resolution on Linux
- Essential networking tools: ip, ss, netstat, curl, wget
- Real-world: Nginx reverse proxy setup

---

## 7.2 Network Interfaces

```bash
# Show all network interfaces
ip addr
ip a   # shorthand

# Example output:
# 1: lo: <LOOPBACK,UP> mtu 65536
#     inet 127.0.0.1/8 scope host lo
# 2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500
#     inet 192.168.1.10/24 brd 192.168.1.255 scope global eth0
#     inet6 fe80::1/64 scope link

# lo = loopback (127.0.0.1 = "this machine")
# eth0 = ethernet (or ens3, enp0s3 on modern systems)

# Show routing table
ip route
ip r

# Example output:
# default via 192.168.1.1 dev eth0
# 192.168.1.0/24 dev eth0 proto kernel
# The "default" route is your gateway (router)

# Show network interface stats
ip -s link
```

### Understanding IP Addresses

```
192.168.1.10/24
           ↑  ↑
           │  └── Subnet mask (24 = 255.255.255.0)
           └───── IP address

127.0.0.1   = loopback (yourself)
0.0.0.0     = all interfaces
::1         = IPv6 loopback
192.168.x.x = private network (your LAN)
10.x.x.x    = private network (common in cloud)
```

---

## 7.3 Checking Ports and Connections

### `ss` — Socket Statistics (Modern Tool)

```bash
# Show all listening ports
ss -tlnp
# -t = TCP
# -l = listening
# -n = numeric (show port numbers not service names)
# -p = show process using the socket

# Example output:
# Netid  State  Recv-Q  Send-Q  Local Address:Port  Peer Address:Port
# tcp    LISTEN  0      128     0.0.0.0:22          0.0.0.0:*     users:(("sshd",pid=1234))
# tcp    LISTEN  0      128     0.0.0.0:80          0.0.0.0:*     users:(("nginx",pid=300))
# tcp    LISTEN  0      128     127.0.0.1:3306      0.0.0.0:*     users:(("mysqld",pid=400))

# Show all connections (including established)
ss -tnp

# Show UDP ports
ss -ulnp

# Show everything
ss -s   # summary statistics

# Find what's using port 80
ss -tlnp | grep :80
sudo lsof -i :80
```

### `netstat` — Older Tool (still common)

```bash
# Install if missing
sudo apt install net-tools

# Show listening ports
netstat -tlnp

# Show all connections
netstat -tnp

# Show connection statistics
netstat -s
```

---

## 7.4 DNS — How Domain Names Resolve

```bash
# Test DNS resolution
dig google.com
nslookup google.com

# Simple DNS lookup
host google.com

# Test specific DNS server
dig @8.8.8.8 google.com

# Trace the full DNS resolution path
dig +trace google.com

# Check your DNS servers
cat /etc/resolv.conf
# nameserver 8.8.8.8
# nameserver 8.8.4.4

# Local host overrides
cat /etc/hosts
# 127.0.0.1  localhost
# 127.0.1.1  my-server
# 192.168.1.5 database.local
# Add entries here to override DNS for specific names
```

### DNS Resolution Order

```
Application asks for "database.local"
    │
    ▼
1. /etc/hosts        ← Check local file FIRST
    │ not found
    ▼
2. /etc/resolv.conf  ← Ask configured DNS server(s)
    │
    ▼
3. DNS Server        ← Recursive resolution
    │
    ▼
Returns IP address
```

---

## 7.5 Testing Connectivity

```bash
# Ping — basic reachability test
ping google.com
ping -c 4 192.168.1.1   # Send only 4 packets

# Traceroute — show network path
traceroute google.com
# or on some systems:
mtr google.com   # Better interactive traceroute

# Test TCP connectivity to specific port
telnet google.com 80
# or
nc -zv google.com 80    # netcat: test without telnet
nc -zv 192.168.1.10 3306  # Test MySQL port

# Test if a port is open
curl -v telnet://192.168.1.10:3306
```

---

## 7.6 `curl` and `wget` — HTTP from Command Line

### `curl` — Transfer URLs

```bash
# GET request
curl https://api.example.com/users

# With headers
curl -H "Authorization: Bearer TOKEN" https://api.example.com/users

# POST request with JSON
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice", "email": "alice@example.com"}' \
  https://api.example.com/users

# See full request/response headers
curl -v https://example.com

# Save to file
curl -o output.html https://example.com

# Follow redirects
curl -L https://example.com

# Show response headers only
curl -I https://example.com

# Test a local Nginx reverse proxy
curl -H "Host: myapp.example.com" http://localhost

# Time a request (performance testing)
curl -o /dev/null -s -w "Time: %{time_total}s\nCode: %{http_code}\n" https://example.com
```

### `wget` — Download Files

```bash
# Download a file
wget https://example.com/file.tar.gz

# Download quietly (for scripts)
wget -q https://example.com/file.tar.gz

# Continue interrupted download
wget -c https://example.com/bigfile.tar.gz

# Mirror a website
wget --mirror https://example.com
```

---

## 7.7 `iptables` and `ufw` — Firewall

```bash
# UFW (Uncomplicated Firewall) - Ubuntu's easy interface
sudo ufw status
sudo ufw enable
sudo ufw disable

# Allow/deny ports
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
sudo ufw deny 3306       # Block MySQL from outside

# Allow from specific IP
sudo ufw allow from 192.168.1.0/24 to any port 22

# Delete a rule
sudo ufw delete allow 8080

# View numbered rules
sudo ufw status numbered
sudo ufw delete 3    # Delete rule #3

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

---

## 7.8 Real-World: Nginx Reverse Proxy

The most common Linux networking task for full-stack developers:

```
Internet → Port 443 → Nginx → Port 3000 → Node.js App
                   ↘ Port 5000 → Python API
                   ↘ Port 8080 → Java Service
```

### Setting Up Nginx Reverse Proxy

```nginx
# /etc/nginx/sites-available/myapp

server {
    listen 80;
    server_name myapp.example.com;
    return 301 https://$server_name$request_uri;  # Redirect to HTTPS
}

server {
    listen 443 ssl;
    server_name myapp.example.com;

    ssl_certificate /etc/letsencrypt/live/myapp.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/myapp.example.com/privkey.pem;

    # Proxy to Node.js
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    # Serve static files directly (faster!)
    location /static/ {
        root /opt/myapp/public;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
}
```

```bash
# Enable the site
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t          # Test configuration
sudo systemctl reload nginx
```

---

## 7.9 Hands-On Exercises — Module 7

```bash
# Exercise 1: Inspect your network
ip addr
ip route

# Exercise 2: Check what's listening
ss -tlnp
sudo lsof -i -P -n | grep LISTEN

# Exercise 3: DNS testing
dig google.com
dig +short google.com     # Just the IP
cat /etc/resolv.conf
cat /etc/hosts

# Exercise 4: Test connectivity
ping -c 4 8.8.8.8
curl -I https://google.com
curl -s https://api.ipify.org   # Your public IP

# Exercise 5: Port scanning (your own machine)
nc -zv localhost 22
nc -zv localhost 80
nc -zv localhost 443

# Exercise 6: HTTP testing
curl -v http://localhost
curl -H "Host: example.com" http://localhost

# Exercise 7: Check firewall
sudo ufw status verbose
```

---

# MODULE 8 — Package Management

## 8.1 What You Will Learn

- How the Ubuntu/Debian package system works
- `apt` commands you'll use daily
- How package repositories work
- Installing software the RIGHT way

---

## 8.2 How Package Management Works

```
Package Manager (apt)
        │
        ├── Package Repositories (servers)
        │   ├── main     ← Official Ubuntu packages
        │   ├── universe ← Community packages
        │   ├── PPA      ← Personal Package Archive (third-party)
        │   └── Third-party (nginx.org, docker.com, etc.)
        │
        ├── Package Database (local cache)
        │   └── /var/lib/apt/lists/
        │
        └── Installed packages
            └── /var/lib/dpkg/
```

Packages are `.deb` files containing:
- Binary executables
- Configuration files
- Dependencies list
- Pre/post install scripts
- Man pages and documentation

---

## 8.3 Essential `apt` Commands

```bash
# ALWAYS start with update before installing
sudo apt update           # Download package list from repos
# (Does NOT upgrade software — just refreshes the LIST)

# Upgrade installed packages
sudo apt upgrade          # Upgrade packages (asks before)
sudo apt upgrade -y       # Upgrade without asking

# Full upgrade (handles dependency changes)
sudo apt full-upgrade

# Install packages
sudo apt install nginx
sudo apt install nginx mysql-server nodejs npm    # Multiple at once
sudo apt install -y nginx    # Without confirmation prompt

# Remove packages
sudo apt remove nginx             # Remove but keep config files
sudo apt purge nginx              # Remove AND delete config files
sudo apt autoremove               # Remove unused dependencies
sudo apt autopurge                # Purge unused dependencies

# Search for packages
apt search nginx
apt search "web server"

# Show package info
apt show nginx
apt-cache show nginx

# List installed packages
apt list --installed
apt list --installed | grep nginx

# Check which package owns a file
dpkg -S /usr/sbin/nginx
# nginx: /usr/sbin/nginx

# List files installed by a package
dpkg -L nginx
```

---

## 8.4 Repositories and PPAs

```bash
# View configured repositories
cat /etc/apt/sources.list
ls /etc/apt/sources.list.d/

# Add a PPA (Personal Package Archive)
sudo add-apt-repository ppa:deadsnakes/python
sudo apt update
sudo apt install python3.12

# Add a third-party repository (example: Docker)
# 1. Add the GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker.gpg

# 2. Add the repository
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list

# 3. Update and install
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
```

---

## 8.5 Hands-On Exercises — Module 8

```bash
# Exercise 1: Update system
sudo apt update
sudo apt upgrade -y

# Exercise 2: Install and verify
sudo apt install -y htop tree ncdu curl wget git
which htop tree
htop --version

# Exercise 3: Find packages
apt search "terminal multiplexer"
apt show tmux

# Exercise 4: Package information
dpkg -L nginx   # Files installed by nginx (if installed)
dpkg -S /usr/bin/curl   # Which package owns curl?

# Exercise 5: Clean up
sudo apt autoremove
sudo apt autoclean    # Remove old downloaded packages
df -h /var/cache/apt  # Space used by package cache
```

---

# MODULE 9 — Disk & Storage

## 9.1 What You Will Learn

- How Linux manages disks and partitions
- How to check disk space usage
- Mounting and unmounting filesystems
- Understanding filesystem types

---

## 9.2 Disk Concepts

```
Physical Disk (e.g., /dev/sda or /dev/nvme0n1)
    │
    ├── Partition 1 (/dev/sda1) → Filesystem (ext4) → Mount at /boot
    ├── Partition 2 (/dev/sda2) → Filesystem (swap) → swap space
    └── Partition 3 (/dev/sda3) → Filesystem (ext4) → Mount at /

Device naming:
/dev/sda    = First SATA/SCSI disk
/dev/sdb    = Second SATA/SCSI disk
/dev/sda1   = First partition of first disk
/dev/nvme0n1 = NVMe (SSD) disk
/dev/vda    = Virtual disk (in VMs/cloud)
/dev/xvda   = Xen virtual disk (AWS older instances)
```

---

## 9.3 Checking Disk Space

### `df` — Disk Free (Filesystem Usage)

```bash
# Human-readable disk usage
df -h

# Example output:
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/sda1        20G   14G  5.1G  74% /
# tmpfs           3.9G     0  3.9G   0% /dev/shm
# /dev/sda2       100G   45G   50G  47% /data

# Show filesystem types
df -hT

# Watch: if Use% reaches 100%, your server WILL crash
# Set up monitoring alerts at 80%
```

### `du` — Disk Usage (Directory/File Usage)

```bash
# See how much space a directory uses
du -sh /var/log
# 2.3G  /var/log

# See all subdirectories
du -sh /var/log/*
# 1.2G  /var/log/nginx
# 800M  /var/log/mysql
# 100M  /var/log/syslog

# Sort by size (find biggest directories)
du -sh /var/* | sort -hr | head -20

# Find largest files in current directory
du -ah . | sort -hr | head -20

# Find files larger than 100MB
find / -size +100M -type f 2>/dev/null
```

---

## 9.4 Block Devices and Partitions

```bash
# List all block devices
lsblk

# Example output:
# NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
# sda      8:0    0    20G  0 disk
# ├─sda1   8:1    0   512M  0 part /boot/efi
# ├─sda2   8:2    0     1G  0 part /boot
# └─sda3   8:3    0  18.5G  0 part /
# sdb      8:16   0   100G  0 disk
# └─sdb1   8:17   0   100G  0 part /data

# Show disk info with filesystem type
lsblk -f

# Show partition table
sudo fdisk -l /dev/sda
sudo parted /dev/sda print
```

---

## 9.5 Mounting Filesystems

```bash
# Mount a disk
sudo mount /dev/sdb1 /mnt/data

# Mount with specific filesystem type
sudo mount -t ext4 /dev/sdb1 /mnt/data

# Unmount
sudo umount /mnt/data

# See what's mounted
mount | grep "^/dev"
# or
cat /proc/mounts
```

### `/etc/fstab` — Automatic Mounts at Boot

```bash
cat /etc/fstab
```

```
# device            mount point  fs-type  options        dump fsck
UUID=abc123...      /            ext4     defaults        0    1
UUID=def456...      /boot        ext4     defaults        0    2
/dev/sdb1           /data        ext4     defaults,nofail 0    0
tmpfs               /tmp         tmpfs    defaults        0    0
```

```bash
# Get UUID of a device
sudo blkid /dev/sdb1
# /dev/sdb1: UUID="abc123..." TYPE="ext4"

# Add to fstab (use UUID, not /dev/sdb1 which can change)
echo "UUID=abc123... /data ext4 defaults,nofail 0 0" | sudo tee -a /etc/fstab

# Test fstab without rebooting
sudo mount -a   # Mounts everything in fstab not already mounted
```

---

## 9.6 Hands-On Exercises — Module 9

```bash
# Exercise 1: Check disk usage
df -hT
lsblk

# Exercise 2: Find space hogs
du -sh /var/* 2>/dev/null | sort -hr | head -10
du -sh /home/* 2>/dev/null | sort -hr

# Exercise 3: Find large files
find /var/log -size +10M -type f 2>/dev/null
find / -size +500M -type f 2>/dev/null

# Exercise 4: Monitor disk usage over time
watch -n 5 df -h   # Update every 5 seconds

# Exercise 5: Create and use a filesystem (if you have a spare disk)
# lsblk  # Find the disk
# sudo mkfs.ext4 /dev/sdb
# sudo mkdir /mnt/test
# sudo mount /dev/sdb /mnt/test
# df -h /mnt/test
```

---

# MODULE 10 — Logs & Monitoring

## 10.1 What You Will Learn

- Where Linux stores logs and why
- How to read and filter logs with journalctl
- Real debugging scenario: why a server crashed

---

## 10.2 Log Architecture

```
Application Logs          System Logs
     │                         │
     ▼                         ▼
/var/log/app/           systemd Journal
   access.log              (journald)
   error.log                  │
                              ▼
                         journalctl command
                              │
                    Also written to /var/log/syslog
```

---

## 10.3 Important Log Files

```bash
/var/log/syslog          # General system log
/var/log/auth.log        # Authentication: logins, sudo, SSH
/var/log/kern.log        # Kernel messages
/var/log/nginx/          # Nginx logs
/var/log/mysql/          # MySQL logs
/var/log/dpkg.log        # Package install/remove history
/var/log/apt/            # apt operations

# Always check these when diagnosing
sudo tail -f /var/log/syslog
sudo tail -f /var/log/auth.log
```

---

## 10.4 `journalctl` — systemd Journal

```bash
# Show all logs (newest at end)
sudo journalctl

# Follow live (like tail -f)
sudo journalctl -f

# Show logs for a specific service
sudo journalctl -u nginx
sudo journalctl -u myapp -f   # Follow myapp service logs

# Filter by time
sudo journalctl --since "2024-01-01 00:00:00"
sudo journalctl --since "1 hour ago"
sudo journalctl --since "10 minutes ago"
sudo journalctl --since yesterday

# Filter by priority
sudo journalctl -p err          # Only errors
sudo journalctl -p warning      # Warnings and above
sudo journalctl -p debug        # Everything (very verbose)

# Filter by process
sudo journalctl _PID=1234
sudo journalctl _COMM=nginx

# Show kernel messages
sudo journalctl -k

# Show last N lines
sudo journalctl -u nginx -n 100

# Show boot logs
sudo journalctl -b          # Current boot
sudo journalctl -b -1       # Previous boot

# Disk space used by journal
sudo journalctl --disk-usage

# Clean up old logs
sudo journalctl --vacuum-size=500M    # Keep only 500MB
sudo journalctl --vacuum-time=30days  # Keep only last 30 days
```

---

## 10.5 Log Rotation

```bash
# logrotate prevents logs from filling your disk
cat /etc/logrotate.conf
ls /etc/logrotate.d/   # Per-application configs

# Example: /etc/logrotate.d/nginx
# /var/log/nginx/*.log {
#     daily           # Rotate daily
#     missingok       # Don't error if log missing
#     rotate 14       # Keep 14 old log files
#     compress        # Compress old logs with gzip
#     delaycompress   # Compress day after rotation
#     notifempty      # Don't rotate empty logs
#     create 0640 www-data adm  # New file permissions
#     sharedscripts
#     postrotate
#         nginx -s reopen  # Tell nginx to use new log file
#     endscript
# }

# Test logrotate (dry run)
sudo logrotate -d /etc/logrotate.d/nginx
```

---

## 10.6 Real-World Debugging: Server Crashed — Why?

```bash
# STEP 1: Check if the system rebooted unexpectedly
last reboot
# Shows last reboot times

journalctl --list-boots
# Lists all boots stored in journal

# STEP 2: Look at logs from the previous boot (before crash)
sudo journalctl -b -1 -p err
# -b -1 = previous boot, -p err = errors only

# STEP 3: Check kernel errors (hardware failures, OOM)
sudo journalctl -b -1 -k | tail -50
# Look for: OOM killer, hardware errors, filesystem errors

# STEP 4: Check for OOM (Out of Memory) killer
sudo journalctl -b -1 | grep -i "oom\|killed process\|out of memory"
# OOM killer kills processes when RAM is full
# This WILL crash your app!

# STEP 5: Check specific service that failed
sudo journalctl -b -1 -u nginx | tail -50
sudo journalctl -b -1 -u myapp | tail -50

# STEP 6: Check disk full (very common crash cause!)
df -h   # Is any partition at 100%?
du -sh /var/log/* | sort -hr | head -10

# STEP 7: Check authentication failures (may indicate attack)
sudo grep "Failed password" /var/log/auth.log | tail -20
sudo grep "Invalid user" /var/log/auth.log | tail -20

# EXAMPLE SCENARIO: App keeps crashing
sudo journalctl -u myapp -n 50
# Logs show: "FATAL: Cannot allocate memory"
# → OOM issue: your Node.js app is using too much RAM
# Fix: set --max-old-space-size in Node.js, or add more RAM
```

---

## 10.7 Hands-On Exercises — Module 10

```bash
# Exercise 1: Explore logs
sudo tail -50 /var/log/syslog
sudo tail -50 /var/log/auth.log

# Exercise 2: journalctl basics
sudo journalctl -n 20
sudo journalctl -f   # Follow (Ctrl+C to stop)

# Exercise 3: Filter service logs
sudo journalctl -u ssh --since "1 hour ago"
sudo journalctl -u cron -n 20

# Exercise 4: Find errors
sudo journalctl -p err --since "1 day ago"

# Exercise 5: Check disk usage of journal
sudo journalctl --disk-usage

# Exercise 6: Look for security events
sudo grep "Failed password" /var/log/auth.log | wc -l
sudo grep "Accepted publickey" /var/log/auth.log | tail -10

# Exercise 7: Create a log entry (from your app)
logger "Test message from my app"
sudo journalctl | tail -5   # See it!
```

---

# MODULE 11 — SSH

## 11.1 What You Will Learn

- How SSH works under the hood
- Key-based authentication (the right way)
- SSH config for multiple servers
- Security hardening

---

## 11.2 How SSH Works

```
YOUR MACHINE                    REMOTE SERVER
    │                                │
    │  1. TCP connection to :22      │
    ├────────────────────────────────►│
    │                                │
    │  2. Exchange crypto algorithms │
    │◄───────────────────────────────┤
    │                                │
    │  3. Server sends public key    │
    │◄───────────────────────────────┤
    │                                │
    │  4. Verify server identity     │
    │   (check known_hosts)          │
    │                                │
    │  5. Key exchange (DH/ECDH)     │
    │   Establish encrypted session  │
    │◄──────────────────────────────►│
    │                                │
    │  6. Authentication             │
    │   - Your private key proves    │
    │     you own the public key     │
    │     stored in authorized_keys  │
    │◄──────────────────────────────►│
    │                                │
    │  7. Encrypted shell session    │
    │◄──────────────────────────────►│
```

---

## 11.3 Key-Based Authentication

This is the ONLY secure way to SSH in production.

### Generate SSH Key Pair

```bash
# Generate modern Ed25519 key (recommended)
ssh-keygen -t ed25519 -C "your@email.com"

# Or RSA 4096 (widely compatible)
ssh-keygen -t rsa -b 4096 -C "your@email.com"

# Files created:
# ~/.ssh/id_ed25519      ← PRIVATE KEY (never share this!)
# ~/.ssh/id_ed25519.pub  ← Public key (share this with servers)
```

### Copy Public Key to Server

```bash
# Automatic (recommended)
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server-ip

# Manual
cat ~/.ssh/id_ed25519.pub  # Copy the output
# On server:
mkdir -p ~/.ssh
echo "PASTE_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### SSH to Server

```bash
# Basic
ssh ubuntu@192.168.1.10

# With specific key
ssh -i ~/.ssh/id_ed25519 ubuntu@192.168.1.10

# With port
ssh -p 2222 ubuntu@192.168.1.10

# Run a command without interactive shell
ssh ubuntu@server "df -h"
ssh ubuntu@server "sudo systemctl restart nginx"
```

---

## 11.4 SSH Config File — Manage Multiple Servers

```bash
cat ~/.ssh/config
```

```
# ~/.ssh/config

# Production server
Host prod
    HostName 192.168.1.10
    User ubuntu
    IdentityFile ~/.ssh/id_ed25519
    Port 22

# Staging server
Host staging
    HostName 192.168.1.20
    User deploy
    IdentityFile ~/.ssh/staging_key
    Port 22

# Jump through bastion
Host app-server
    HostName 10.0.1.50
    User ubuntu
    IdentityFile ~/.ssh/id_ed25519
    ProxyJump bastion

Host bastion
    HostName 203.0.113.10
    User ubuntu
    IdentityFile ~/.ssh/id_ed25519
```

```bash
# Now you can simply:
ssh prod
ssh staging
ssh app-server   # Automatically goes through bastion!
```

---

## 11.5 Securing SSH

```bash
sudo nano /etc/ssh/sshd_config
```

```
# Disable password authentication (use keys only!)
PasswordAuthentication no
ChallengeResponseAuthentication no

# Disable root login
PermitRootLogin no

# Only allow specific users
AllowUsers ubuntu deploy alice

# Change default port (reduces automated attacks)
Port 2222

# Limit authentication attempts
MaxAuthTries 3

# Timeout idle sessions
ClientAliveInterval 300
ClientAliveCountMax 2

# Disable unused features
X11Forwarding no
AllowTcpForwarding no  # Unless you need SSH tunneling
```

```bash
# Test config before applying
sudo sshd -t

# Reload SSH
sudo systemctl reload ssh
```

---

## 11.6 SSH Tunneling

```bash
# Local port forwarding: access remote service locally
# Access remote MySQL (3306) via localhost:3307
ssh -L 3307:localhost:3306 ubuntu@server

# Now connect to MySQL locally:
mysql -h 127.0.0.1 -P 3307 -u user -p

# Remote port forwarding: expose local port to remote server
ssh -R 8080:localhost:3000 ubuntu@server
# Now server:8080 forwards to your local:3000

# SOCKS proxy (route all traffic through server)
ssh -D 1080 ubuntu@server
# Configure browser to use SOCKS5 proxy localhost:1080
```

---

## 11.7 Hands-On Exercises — Module 11

```bash
# Exercise 1: Generate an SSH key
ssh-keygen -t ed25519 -C "test@example.com" -f /tmp/test_key
ls -la /tmp/test_key*

# Exercise 2: Set up SSH config
mkdir -p ~/.ssh
cat >> ~/.ssh/config << 'EOF'

Host localhost-test
    HostName localhost
    User $USER
    IdentityFile ~/.ssh/id_ed25519
    Port 22
EOF

# Exercise 3: Check SSH server config
sudo sshd -T | grep -E "passwordauthentication|permitrootlogin|port"

# Exercise 4: View authorized keys
cat ~/.ssh/authorized_keys

# Exercise 5: Check SSH connection details
ssh -v ubuntu@your-server 2>&1 | head -30

# Exercise 6: Local port forwarding (if you have a remote server)
# ssh -L 8080:localhost:80 ubuntu@remote-server
# curl http://localhost:8080

# Exercise 7: Analyze SSH logs
sudo grep "sshd" /var/log/auth.log | tail -20
sudo grep "Failed password" /var/log/auth.log | wc -l
```

---

# MODULE 12 — Linux Security

## 12.1 What You Will Learn

- The principle of least privilege
- sudo configuration
- UFW firewall rules
- Fail2ban for blocking brute-force attacks
- Security hardening best practices

---

## 12.2 sudo — Controlled Privilege Escalation

`sudo` allows specific users to run commands as root (or other users) in a controlled, logged way.

```bash
# Run a command as root
sudo apt install nginx

# Run as a specific user
sudo -u www-data cat /var/www/html/index.html

# Get a root shell (use carefully!)
sudo -i
sudo su -

# See what sudo commands you're allowed
sudo -l

# sudo logs everything!
sudo grep "sudo" /var/log/auth.log | tail -10
# ubuntu : TTY=pts/0 ; PWD=/home/ubuntu ; USER=root ; COMMAND=/usr/bin/apt install nginx
```

### Configuring `sudoers`

```bash
# ALWAYS use visudo to edit (prevents syntax errors that could lock you out)
sudo visudo
# or edit a drop-in file:
sudo visudo -f /etc/sudoers.d/myapp
```

```bash
# /etc/sudoers format:

# Allow ubuntu full sudo access
ubuntu ALL=(ALL:ALL) ALL

# Allow deploy to restart specific services (no password!)
deploy ALL=(ALL) NOPASSWD: /bin/systemctl restart myapp, /bin/systemctl status myapp

# Allow group
%developers ALL=(ALL) ALL

# Allow sudo without password for specific command
%deployers ALL=(ALL) NOPASSWD: /usr/bin/docker
```

---

## 12.3 UFW Firewall — Production Setup

```bash
# Starting fresh firewall setup for a web server:

# Set defaults
sudo ufw default deny incoming    # Block ALL incoming
sudo ufw default allow outgoing   # Allow ALL outgoing

# Allow SSH (do this FIRST before enabling UFW!)
sudo ufw allow 22/tcp
# or if you changed SSH port:
sudo ufw allow 2222/tcp

# Allow web traffic
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS

# Allow from specific IPs only (e.g., your office)
sudo ufw allow from 203.0.113.0/24 to any port 22

# Allow internal communication
sudo ufw allow from 10.0.0.0/8    # Private network

# ENABLE the firewall
sudo ufw enable

# Verify
sudo ufw status verbose
```

---

## 12.4 Fail2ban — Block Brute-Force Attacks

fail2ban watches logs and bans IPs that make too many failed attempts.

```bash
sudo apt install fail2ban

# Config file
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

```ini
[DEFAULT]
bantime  = 3600     # Ban for 1 hour
findtime = 600      # Count failures in 10 minutes
maxretry = 5        # Ban after 5 failures

[sshd]
enabled  = true
port     = ssh
logpath  = /var/log/auth.log
maxretry = 3        # Only 3 attempts for SSH
bantime  = 86400    # Ban SSH violators for 24 hours

[nginx-http-auth]
enabled  = true
port     = http,https
logpath  = /var/log/nginx/error.log
```

```bash
sudo systemctl enable --now fail2ban

# Check status
sudo fail2ban-client status
sudo fail2ban-client status sshd

# See banned IPs
sudo fail2ban-client status sshd | grep "Banned IP"

# Unban an IP (if you accidentally banned yourself!)
sudo fail2ban-client set sshd unbanip 192.168.1.5
```

---

## 12.5 Security Best Practices Checklist

```bash
# 1. System is up to date
sudo apt update && sudo apt upgrade -y

# 2. Firewall is enabled
sudo ufw status

# 3. Unnecessary services are disabled
systemctl list-units --type=service --state=running
# Disable anything you don't need:
sudo systemctl disable --now bluetooth
sudo systemctl disable --now cups  # Printing

# 4. Check for world-writable files
find / -type f -perm -o+w -not -path "/proc/*" -not -path "/sys/*" 2>/dev/null

# 5. Check for setuid/setgid files
find / -type f \( -perm -4000 -o -perm -2000 \) 2>/dev/null

# 6. Check listening services
ss -tlnp
# Only ports you INTENTIONALLY opened should be here

# 7. Check user accounts
awk -F: '$3 >= 1000 {print $1}' /etc/passwd   # Regular users

# 8. Verify no empty passwords
sudo awk -F: '$2 == "" {print $1}' /etc/shadow

# 9. Check SSH config
sudo grep -E "^PasswordAuthentication|^PermitRootLogin" /etc/ssh/sshd_config

# 10. Check sudo config
sudo cat /etc/sudoers | grep -v "^#" | grep -v "^$"
```

---

## 12.6 Hands-On Exercises — Module 12

```bash
# Exercise 1: Check sudo config
sudo -l   # What can YOU sudo?
sudo cat /etc/sudoers

# Exercise 2: Set up firewall
sudo ufw status
sudo ufw enable
sudo ufw allow 22/tcp
sudo ufw status numbered

# Exercise 3: Create a sudoers rule
sudo visudo -f /etc/sudoers.d/test
# Add: yourusername ALL=(ALL) NOPASSWD: /usr/bin/systemctl status nginx

# Exercise 4: Security audit
find / -type f -perm -4000 2>/dev/null | head -20
ss -tlnp
sudo grep "Failed password" /var/log/auth.log | wc -l

# Exercise 5: Check system users
awk -F: '$3 < 1000 && $3 > 0 {print $1, $3, $7}' /etc/passwd
```

---

# MODULE 13 — Environment Variables

## 13.1 What You Will Learn

- What environment variables are and why they exist
- How to set, export, and persist variables
- Using `.env` files with Node.js
- Production environment configuration

---

## 13.2 What Are Environment Variables?

Environment variables are **key-value pairs** passed to processes. Every process inherits its parent's environment.

```bash
# See ALL current environment variables
env
printenv

# See a specific variable
echo $HOME
echo $PATH
echo $USER
echo $SHELL
```

### Important Default Variables

```bash
HOME=/home/ubuntu       # Your home directory
USER=ubuntu             # Current username
SHELL=/bin/bash         # Your shell
PATH=/usr/local/bin:/usr/bin:/bin  # Where to find commands
PWD=/home/ubuntu        # Current directory
LANG=en_US.UTF-8        # System language
TERM=xterm-256color     # Terminal type
```

### The PATH Variable

```bash
# PATH tells bash where to look for commands
echo $PATH
# /usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin

# When you type "nginx", bash looks for it in each directory in PATH
which nginx
# /usr/sbin/nginx   ← found in /usr/sbin

# Add a custom directory to PATH
export PATH="$PATH:/opt/myapp/bin"

# Now commands in /opt/myapp/bin work without full path
myapp-command   # works!
```

---

## 13.3 Setting Variables

```bash
# Set a local variable (only in current shell, not exported)
MY_VAR="hello"
echo $MY_VAR

# Export (makes available to child processes)
export MY_VAR="hello"
export PORT=3000
export DB_HOST=localhost

# Set and export in one line
export NODE_ENV=production

# Unset a variable
unset MY_VAR
```

---

## 13.4 Making Variables Permanent

### For One User

```bash
# ~/.bashrc — runs for interactive non-login shells
echo 'export NODE_ENV=production' >> ~/.bashrc
echo 'export PATH="$PATH:/opt/myapp/bin"' >> ~/.bashrc

# ~/.bash_profile or ~/.profile — runs for login shells
echo 'export MY_API_KEY=abc123' >> ~/.profile

# Apply changes immediately
source ~/.bashrc
# or
. ~/.bashrc
```

### System-Wide

```bash
# /etc/environment — simple KEY=VALUE, no export, no bash syntax
sudo nano /etc/environment
# NODE_ENV=production
# APP_PORT=3000

# /etc/profile.d/ — scripts sourced for all users
sudo nano /etc/profile.d/myapp.sh
# export MYAPP_HOME=/opt/myapp
```

---

## 13.5 Real-World: Node.js Environment Variables

### The `.env` File Pattern

```bash
# /opt/myapp/.env
NODE_ENV=production
PORT=3000
DATABASE_URL=postgresql://user:password@localhost:5432/mydb
REDIS_URL=redis://localhost:6379
JWT_SECRET=super-secret-key-here
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
SENDGRID_API_KEY=SG.xxxxx
```

```bash
# NEVER commit .env to git!
echo ".env" >> .gitignore
echo ".env.*" >> .gitignore
echo "!.env.example" >> .gitignore

# Always have a .env.example (no real values)
cat .env.example
# NODE_ENV=development
# PORT=3000
# DATABASE_URL=postgresql://user:password@localhost:5432/dbname
```

### In systemd Service File

```ini
[Service]
# Direct variables
Environment=NODE_ENV=production
Environment=PORT=3000

# From file (the - means don't fail if file missing)
EnvironmentFile=-/opt/myapp/.env
```

### In Node.js

```javascript
// Use dotenv package in development
require('dotenv').config();

// Access variables
const port = process.env.PORT || 3000;
const dbUrl = process.env.DATABASE_URL;
const nodeEnv = process.env.NODE_ENV || 'development';

if (!dbUrl) {
  console.error('DATABASE_URL is required!');
  process.exit(1);
}
```

---

## 13.6 Hands-On Exercises — Module 13

```bash
# Exercise 1: Explore environment
env | sort
echo "PATH: $PATH"
echo "HOME: $HOME"

# Exercise 2: Create and export variables
export MY_APP="Linux Mastery"
export APP_PORT=8080
echo "Running $MY_APP on port $APP_PORT"

# Check child process inherits them
bash -c 'echo "In child: $MY_APP on port $APP_PORT"'

# Exercise 3: Make permanent
echo 'export LEARNING="Linux"' >> ~/.bashrc
source ~/.bashrc
echo $LEARNING

# Exercise 4: Create a .env file
cat > /tmp/test.env << EOF
DB_HOST=localhost
DB_PORT=5432
DB_NAME=myapp
APP_SECRET=changeme
EOF

# Read and export (simulate dotenv)
export $(cat /tmp/test.env | xargs)
echo "DB: $DB_HOST:$DB_PORT/$DB_NAME"
```

---

# MODULE 14 — Linux Performance

## 14.1 What You Will Learn

- Understanding CPU, memory, disk, and network performance metrics
- Tools to diagnose performance problems
- Common performance bottlenecks and solutions

---

## 14.2 Understanding Load Average

```bash
uptime
# 15:30:00 up 10 days, 3:22, 2 users, load average: 0.52, 0.73, 0.90
#                                                     1min  5min  15min
```

Load average = average number of processes waiting for CPU + processes using CPU

```
On a 4-core machine:
Load 0.5  = 12.5% busy — relaxed
Load 2.0  = 50% busy   — moderate
Load 4.0  = 100% busy  — fully utilized
Load 8.0  = 200% busy  — overloaded, processes waiting!
```

---

## 14.3 CPU Performance

```bash
# Real-time CPU usage
top
htop

# One-time CPU snapshot
mpstat            # Per-CPU stats
mpstat -P ALL 1   # All CPUs, refresh every second

# What's using the most CPU?
ps aux --sort=-%cpu | head -10

# CPU info
lscpu
cat /proc/cpuinfo | grep "model name" | head -1
nproc    # Number of CPU cores

# Detailed CPU usage breakdown
sar -u 1 5   # CPU stats, every 1 second, 5 times
```

---

## 14.4 Memory Performance

```bash
# Memory overview
free -h
# Breakdown:
#               total    used    free  shared  buff/cache  available
# Mem:           7.6G    3.5G    2.1G   126M       2.0G       3.8G
# Swap:          2.0G    100M    1.9G

# "available" is what you can allocate without swapping
# "buff/cache" = disk cache (Linux uses spare RAM for caching — this is GOOD!)

# Detailed memory
cat /proc/meminfo

# What's using the most memory?
ps aux --sort=-%mem | head -10

# Check swap usage (swap use = RAM pressure)
swapon --show
vmstat -s

# Real-time memory + swap
vmstat 1 5   # Stats every 1 second, 5 times
```

---

## 14.5 Disk I/O Performance

```bash
# Install iotop
sudo apt install iotop

# See disk I/O by process
sudo iotop
sudo iotop -o   # Only show processes doing I/O

# Disk stats
iostat
iostat -x 1 5   # Extended stats, every second

# Check disk throughput
dd if=/dev/zero of=/tmp/test bs=1G count=1 oflag=dsync
# (Tests write speed)

# I/O wait in vmstat (shows disk bottleneck)
vmstat 1
# us  sy  id  wa  st
#  2   1  95   2   0
# wa = time waiting for I/O (>10% = disk bottleneck)
```

---

## 14.6 Network Performance

```bash
# Network stats
ifstat   # Install: sudo apt install ifstat
nethogs  # Install: sudo apt install nethogs; sudo nethogs

# Bandwidth by process
sudo nethogs eth0

# Network connection stats
ss -s

# Network interface stats
ip -s link

# Test network speed
curl -s https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py | python3
```

---

## 14.7 Hands-On Exercises — Module 14

```bash
# Exercise 1: Check system load
uptime
cat /proc/loadavg

# Exercise 2: Memory analysis
free -h
cat /proc/meminfo | grep -E "MemTotal|MemFree|MemAvailable|SwapTotal|SwapFree"

# Exercise 3: CPU stress test (install stress first)
sudo apt install stress -y
# Run for 30 seconds
stress --cpu 2 --timeout 30 &
top   # Watch CPU spike
wait

# Exercise 4: Find memory-hungry processes
ps aux --sort=-%mem | head -5
ps aux --sort=-%cpu | head -5

# Exercise 5: Monitor vmstat
vmstat 2 5   # Every 2 seconds, 5 readings

# Exercise 6: Disk performance
# Check I/O wait
iostat 1 5
```

---

# MODULE 15 — Cron Jobs

## 15.1 What You Will Learn

- How cron works for scheduled tasks
- Crontab syntax (the "5 stars" explanation)
- Real-world backup and maintenance jobs

---

## 15.2 What Is Cron?

Cron is the Linux **task scheduler**. It runs commands at specified times automatically.

```
cron daemon (crond/cron)
    │
    ├── /etc/crontab        ← System crontab
    ├── /etc/cron.d/        ← Drop-in cron files
    ├── /etc/cron.daily/    ← Scripts run daily
    ├── /etc/cron.weekly/   ← Scripts run weekly
    ├── /etc/cron.monthly/  ← Scripts run monthly
    └── /var/spool/cron/    ← Per-user crontabs
```

---

## 15.3 Crontab Syntax

```
┌──────────── minute (0-59)
│  ┌─────────── hour (0-23)
│  │  ┌──────────── day of month (1-31)
│  │  │  ┌─────────── month (1-12)
│  │  │  │  ┌──────────── day of week (0-6, Sun=0)
│  │  │  │  │
*  *  *  *  *  command to execute
```

### Examples

```cron
# Every minute
* * * * * /opt/myapp/check.sh

# Every 5 minutes
*/5 * * * * /opt/myapp/check.sh

# Every hour, at minute 0
0 * * * * /opt/myapp/hourly.sh

# Every day at midnight
0 0 * * * /opt/backup/daily-backup.sh

# Every day at 2:30 AM
30 2 * * * /opt/backup/daily-backup.sh

# Every Monday at 3 AM
0 3 * * 1 /opt/maintenance/weekly.sh

# Every 1st of the month at 1 AM
0 1 1 * * /opt/reports/monthly.sh

# Every weekday (Mon-Fri) at 8 AM
0 8 * * 1-5 /opt/reports/daily.sh

# Multiple times: 6 AM and 6 PM
0 6,18 * * * /opt/deploy/health-check.sh
```

---

## 15.4 Managing Crontabs

```bash
# Edit your crontab
crontab -e

# List your crontab
crontab -l

# Remove your crontab
crontab -r

# Edit another user's crontab (as root)
sudo crontab -u www-data -e

# System crontab (has an extra "user" field)
sudo nano /etc/crontab
# 30 2 * * * root /opt/backup/daily.sh
```

---

## 15.5 Real-World: Backup Job

```bash
#!/bin/bash
# /opt/backup/daily-backup.sh

DATE=$(date +%Y-%m-%d)
BACKUP_DIR=/opt/backups
LOG_FILE=/var/log/backup.log
DB_NAME=myapp
BACKUP_FILE="$BACKUP_DIR/db-$DATE.sql.gz"

# Ensure backup dir exists
mkdir -p $BACKUP_DIR

echo "[$(date)] Starting backup..." >> $LOG_FILE

# Database backup
pg_dump $DB_NAME | gzip > $BACKUP_FILE

if [ $? -eq 0 ]; then
    echo "[$(date)] Database backup successful: $BACKUP_FILE" >> $LOG_FILE
else
    echo "[$(date)] ERROR: Database backup FAILED!" >> $LOG_FILE
    exit 1
fi

# Upload to S3
aws s3 cp $BACKUP_FILE s3://my-backups/db/$DATE/ >> $LOG_FILE 2>&1

# Keep only last 7 days of local backups
find $BACKUP_DIR -name "db-*.sql.gz" -mtime +7 -delete

echo "[$(date)] Backup complete." >> $LOG_FILE
```

```bash
# Make executable
chmod +x /opt/backup/daily-backup.sh

# Schedule in crontab
crontab -e
# Run at 2 AM daily
0 2 * * * /opt/backup/daily-backup.sh >> /var/log/backup.log 2>&1
```

---

## 15.6 Hands-On Exercises — Module 15

```bash
# Exercise 1: View existing cron jobs
crontab -l
ls /etc/cron.daily/
ls /etc/cron.weekly/

# Exercise 2: Create a simple cron job
crontab -e
# Add:
# * * * * * echo "Cron test at $(date)" >> /tmp/cron-test.log

# Wait 2 minutes, then check:
cat /tmp/cron-test.log

# Remove the test job when done
crontab -e   # Delete the line

# Exercise 3: Create a maintenance script
cat > /tmp/cleanup.sh << 'EOF'
#!/bin/bash
# Clean temp files older than 7 days
find /tmp -type f -mtime +7 -delete 2>/dev/null
echo "[$(date)] Cleanup done" >> /var/log/cleanup.log
EOF
chmod +x /tmp/cleanup.sh

# Schedule weekly:
crontab -e
# 0 3 * * 0 /tmp/cleanup.sh
```

---

# MODULE 16 — Docker + Linux

## 16.1 What You Will Learn

- How Docker uses Linux kernel features (namespaces, cgroups)
- Proper Docker permissions setup
- Docker socket security
- Managing Docker as a systemd service

---

## 16.2 How Docker Uses Linux

Docker is NOT a virtual machine. It uses Linux kernel features:

```
Docker "Container" is really:

┌─────────────────────────────────────┐
│          Docker Container           │
│                                     │
│  Isolated using:                    │
│  ┌─────────────┐ ┌───────────────┐  │
│  │ Namespaces  │ │   cgroups     │  │
│  │             │ │               │  │
│  │ pid:  own   │ │ CPU: limited  │  │
│  │ net:  own   │ │ RAM: limited  │  │
│  │ mnt:  own   │ │ I/O: limited  │  │
│  │ user: own   │ │               │  │
│  └─────────────┘ └───────────────┘  │
│                                     │
│  SHARED: Host Linux Kernel          │
└─────────────────────────────────────┘
```

---

## 16.3 Docker User and Permissions

```bash
# Docker daemon runs as root
ps aux | grep dockerd
# root ... /usr/bin/dockerd

# Docker socket - only root and docker group can access
ls -la /var/run/docker.sock
# srw-rw---- 1 root docker /var/run/docker.sock

# Add your user to docker group (avoid sudo for docker)
sudo usermod -aG docker $USER
newgrp docker    # Apply without logout

# Verify
groups | grep docker
docker ps        # Should work without sudo
```

### Docker as systemd Service

```bash
# Docker is already a systemd service
sudo systemctl status docker
sudo systemctl enable docker    # Start at boot
sudo systemctl restart docker

# Docker daemon config
cat /etc/docker/daemon.json
```

### Securing Docker

```bash
# /etc/docker/daemon.json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "3"
  },
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 64000,
      "Soft": 64000
    }
  },
  "live-restore": true,
  "icc": false,
  "no-new-privileges": true
}
```

```bash
sudo systemctl restart docker
```

---

## 16.4 Hands-On Exercises — Module 16

```bash
# Exercise 1: Check Docker's Linux integration
docker info | grep -E "Server|Storage|Cgroup"

# Exercise 2: See container as a Linux process
docker run -d --name test nginx
ps aux | grep nginx      # See nginx running as Linux process!
docker exec test ps aux  # See from inside container

# Exercise 3: Namespaces
# Container PID 1 from outside:
docker inspect test | grep Pid

# Exercise 4: cgroups
ls /sys/fs/cgroup/system.slice/   # See Docker cgroups

# Exercise 5: Resource limits
docker run -d --name limited --memory=256m --cpus=0.5 nginx
cat /sys/fs/cgroup/system.slice/docker-*/memory.max

# Cleanup
docker stop test limited
docker rm test limited
```

---

# MODULE 17 — Nginx + Linux

## 17.1 What You Will Learn

- Nginx as a systemd service
- Configuration file locations
- Virtual hosts (server blocks)
- SSL with Let's Encrypt
- Performance tuning

---

## 17.2 Nginx on Linux

```bash
# Install
sudo apt install nginx

# Status
sudo systemctl status nginx

# Control
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl reload nginx   # Reload config without downtime
sudo systemctl restart nginx  # Full restart (brief downtime)

# Test config before applying
sudo nginx -t
```

### File Locations

```
/etc/nginx/
├── nginx.conf              ← Main config
├── sites-available/        ← All site configs
│   ├── default             ← Default site
│   └── myapp               ← Your site config
├── sites-enabled/          ← Symlinks to active sites
│   └── myapp -> ../sites-available/myapp
├── conf.d/                 ← Additional configs
├── snippets/               ← Reusable config pieces
│   ├── fastcgi-php.conf
│   └── snakeoil.conf
└── mime.types              ← MIME type mappings

/var/log/nginx/
├── access.log              ← All requests
└── error.log               ← Errors only

/var/www/html/              ← Default web root
```

### Enabling Sites

```bash
# Enable a site (create symlink)
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/

# Disable a site (remove symlink)
sudo rm /etc/nginx/sites-enabled/myapp

# ALWAYS test before reload!
sudo nginx -t
sudo systemctl reload nginx
```

---

## 17.3 Nginx Configuration Deep Dive

```nginx
# /etc/nginx/nginx.conf

user www-data;                  # Worker processes run as www-data
worker_processes auto;          # One worker per CPU core
pid /run/nginx.pid;             # Store PID here

events {
    worker_connections 1024;    # Max connections per worker
    use epoll;                  # Use epoll for Linux (most efficient)
    multi_accept on;            # Accept multiple connections at once
}

http {
    # Basic settings
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    server_tokens off;          # Hide nginx version (security)

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Logging
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;
    error_log /var/log/nginx/error.log;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_types text/plain text/css application/json application/javascript;

    # Include virtual hosts
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

---

## 17.4 Hands-On Exercises — Module 17

```bash
# Exercise 1: Nginx service management
sudo systemctl status nginx
sudo nginx -t
sudo nginx -V   # Version and compile options

# Exercise 2: Explore config
cat /etc/nginx/nginx.conf
ls /etc/nginx/sites-available/
ls /etc/nginx/sites-enabled/

# Exercise 3: View logs
sudo tail -f /var/log/nginx/access.log &
curl http://localhost
# You should see the request in the log

# Exercise 4: Create a simple site
sudo mkdir -p /var/www/testsite
echo "<h1>Test Site</h1>" | sudo tee /var/www/testsite/index.html

sudo cat > /tmp/testsite.conf << 'EOF'
server {
    listen 8080;
    server_name localhost;
    root /var/www/testsite;
    index index.html;
}
EOF
sudo cp /tmp/testsite.conf /etc/nginx/sites-available/testsite
sudo ln -s /etc/nginx/sites-available/testsite /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
curl http://localhost:8080

# Cleanup
sudo rm /etc/nginx/sites-enabled/testsite
sudo systemctl reload nginx
```

---

# MODULE 18 — Production Server Setup

## 18.1 What You Will Learn

- Standard production directory layout
- Setting up a new server from scratch
- Security hardening checklist
- Monitoring setup

---

## 18.2 Production Directory Structure

```
Production Server Layout:

/
├── etc/
│   ├── nginx/sites-available/myapp   ← Nginx config
│   ├── systemd/system/myapp.service  ← Service definition
│   ├── myapp/                        ← App config files
│   └── cron.d/myapp                  ← Cron jobs
│
├── opt/
│   └── myapp/                        ← Application root
│       ├── current/                  ← Current release (symlink)
│       ├── releases/                 ← Versioned releases
│       │   ├── 20240101-001/
│       │   ├── 20240115-001/
│       │   └── 20240201-001/   ← current → points here
│       ├── shared/                   ← Shared across releases
│       │   ├── .env                  ← Environment variables
│       │   ├── logs/                 ← Application logs
│       │   └── uploads/              ← User uploads
│       └── bin/                      ← Scripts
│           ├── deploy.sh
│           └── rollback.sh
│
├── var/
│   ├── log/
│   │   ├── nginx/
│   │   │   ├── myapp-access.log
│   │   │   └── myapp-error.log
│   │   └── myapp/                    ← App logs
│   └── lib/
│       └── myapp/                    ← App state
│
└── home/
    └── deploy/                       ← Deploy user home
        └── .ssh/authorized_keys      ← CI/CD keys
```

---

## 18.3 New Server Setup Checklist

```bash
# ===================================
# STEP 1: Initial System Update
# ===================================
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget git vim htop tmux \
  net-tools jq fail2ban ufw unzip

# ===================================
# STEP 2: Create Application User
# ===================================
sudo useradd -m -s /bin/bash deploy
sudo mkdir -p /home/deploy/.ssh
sudo touch /home/deploy/.ssh/authorized_keys
sudo chmod 700 /home/deploy/.ssh
sudo chmod 600 /home/deploy/.ssh/authorized_keys
sudo chown -R deploy:deploy /home/deploy/.ssh

# Add your SSH key
echo "YOUR_PUBLIC_KEY" | sudo tee /home/deploy/.ssh/authorized_keys

# Give deploy limited sudo access
echo "deploy ALL=(ALL) NOPASSWD: /bin/systemctl restart myapp, /bin/systemctl status myapp" \
  | sudo tee /etc/sudoers.d/deploy

# ===================================
# STEP 3: SSH Hardening
# ===================================
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sudo systemctl reload ssh

# ===================================
# STEP 4: Firewall
# ===================================
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
sudo ufw enable

# ===================================
# STEP 5: Fail2ban
# ===================================
sudo systemctl enable --now fail2ban

# ===================================
# STEP 6: Directory Structure
# ===================================
sudo mkdir -p /opt/myapp/{releases,shared/{logs,uploads}}
sudo chown -R deploy:deploy /opt/myapp
sudo chmod 750 /opt/myapp

# ===================================
# STEP 7: Install Application Runtime
# ===================================
# Install Node.js (using NodeSource)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Verify
node --version
npm --version

# ===================================
# STEP 8: Nginx
# ===================================
sudo apt install -y nginx
sudo systemctl enable nginx

# ===================================
# STEP 9: SSL Certificate
# ===================================
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d myapp.example.com --email admin@example.com --agree-tos -n

# ===================================
# STEP 10: Log Rotation
# ===================================
cat > /etc/logrotate.d/myapp << 'EOF'
/opt/myapp/shared/logs/*.log {
    daily
    rotate 30
    compress
    delaycompress
    missingok
    notifempty
    copytruncate
}
EOF

# ===================================
# STEP 11: Monitoring (basic)
# ===================================
# Install and configure monitoring
sudo apt install -y prometheus-node-exporter

# Simple disk space alert via cron
cat > /opt/myapp/bin/disk-alert.sh << 'SCRIPT'
#!/bin/bash
USAGE=$(df / | tail -1 | awk '{print $5}' | tr -d '%')
if [ $USAGE -gt 80 ]; then
    echo "ALERT: Disk usage at ${USAGE}% on $(hostname)" | \
    mail -s "Disk Alert" admin@example.com
fi
SCRIPT
chmod +x /opt/myapp/bin/disk-alert.sh
echo "*/30 * * * * deploy /opt/myapp/bin/disk-alert.sh" | sudo tee /etc/cron.d/disk-alert
```

---

## 18.4 Hands-On Exercises — Module 18

```bash
# Exercise 1: Create the full production structure
mkdir -p /tmp/prod-demo/{opt/myapp/{releases,shared/{logs,uploads}},var/log/myapp,etc/myapp}
find /tmp/prod-demo -type d | sort

# Exercise 2: Simulate a deployment
VERSION="20240101-001"
mkdir -p /tmp/prod-demo/opt/myapp/releases/$VERSION
echo "console.log('v1.0.0');" > /tmp/prod-demo/opt/myapp/releases/$VERSION/app.js
ln -sfn /tmp/prod-demo/opt/myapp/releases/$VERSION /tmp/prod-demo/opt/myapp/current
ls -la /tmp/prod-demo/opt/myapp/current

# Exercise 3: Run the full new server checklist
# (Use on a real server or VM)
```

---

# MODULE 19 — Linux Debugging

## 19.1 What You Will Learn

- Systematic debugging methodology
- Tools for debugging processes, ports, logs
- Common production problems and solutions

---

## 19.2 Debugging Methodology

```
Problem Reported
      │
      ▼
1. REPRODUCE: Can you reproduce it?
      │
      ▼
2. ISOLATE: Is it app? OS? Network? DB?
      │
      ▼
3. CHECK LOGS: What do logs say?
│     ├── journalctl -u service
│     ├── /var/log/nginx/error.log
│     └── /var/log/syslog
      │
      ▼
4. CHECK RESOURCES: CPU? RAM? Disk? Network?
│     ├── top / htop
│     ├── free -h
│     ├── df -h
│     └── ss -tlnp
      │
      ▼
5. CHECK PROCESSES: Is service running?
│     ├── systemctl status service
│     └── ps aux | grep service
      │
      ▼
6. FIX and VERIFY
      │
      ▼
7. DOCUMENT: What was the cause? What was the fix?
```

---

## 19.3 Common Production Problems

### Problem 1: Service Won't Start

```bash
# Check status (most important first)
sudo systemctl status myapp

# See full logs
sudo journalctl -u myapp -n 50 --no-pager

# Try starting manually to see errors
sudo -u deploy /usr/bin/node /opt/myapp/current/server.js

# Check for port conflicts
ss -tlnp | grep :3000
# Is something else using the port?

# Check file permissions
ls -la /opt/myapp/current/
sudo -u deploy ls /opt/myapp/current/   # Can the deploy user read it?

# Check the .env file
sudo -u deploy cat /opt/myapp/shared/.env
```

### Problem 2: Site Is Down (502 Bad Gateway)

```bash
# 502 = Nginx can't reach the backend

# Is the backend running?
sudo systemctl status myapp
curl http://localhost:3000   # Test directly

# Is it on the right port?
ss -tlnp | grep :3000

# Check nginx error log
sudo tail -50 /var/log/nginx/error.log

# Check nginx config
sudo nginx -t

# Check backend logs
sudo journalctl -u myapp -n 50
```

### Problem 3: Server Is Slow

```bash
# Quick overview
uptime    # Load average
free -h   # Memory

# CPU hog?
ps aux --sort=-%cpu | head -10

# Memory hog?
ps aux --sort=-%mem | head -10

# Disk full?
df -h     # Check all filesystems

# Disk I/O?
sudo iotop -o

# Network bottleneck?
sudo nethogs eth0

# What's the process doing?
sudo strace -p PID   # System calls
sudo lsof -p PID     # Open files
```

### Problem 4: Disk Full — Emergency

```bash
# EMERGENCY: Disk is 100% full, server is broken

# Step 1: Find the biggest files
sudo du -sh /* 2>/dev/null | sort -hr | head -10

# Step 2: Usually it's logs
sudo du -sh /var/log/* | sort -hr | head -10

# Step 3: Clear logs (carefully!)
sudo journalctl --vacuum-size=100M    # Shrink journal
sudo truncate -s 0 /var/log/nginx/access.log  # Clear nginx log
sudo rm /var/log/nginx/access.log.* 2>/dev/null  # Remove old rotated

# Step 4: Find and remove large old files
find /tmp -type f -mtime +1 -delete
find /var/log -name "*.gz" -mtime +30 -delete

# Step 5: Docker cleanup (if Docker is installed)
docker system prune -af    # Remove all unused images/containers

# Step 6: APT cache
sudo apt clean
```

---

## 19.4 Essential Debugging Commands

```bash
# PROCESS DEBUGGING
strace -p PID           # Trace system calls
strace -p PID -e trace=network  # Only network calls
ltrace -p PID           # Library calls
lsof -p PID             # Open files and connections
/proc/PID/cmdline       # How process was started
/proc/PID/environ       # Environment variables

# NETWORK DEBUGGING
ss -tlnp                # What's listening
tcpdump -i eth0 port 80 # Capture traffic on port 80
tcpdump -i eth0 -w /tmp/capture.pcap   # Save capture
curl -v URL              # Verbose HTTP
curl --trace /tmp/trace URL  # Full trace

# FILE DEBUGGING
inotifywait -m /path     # Watch file changes in real time
lsof /path/to/file       # Who has this file open?
fuser /path/to/file      # Which processes use this file?
fuser -k /path/to/file   # Kill processes using this file

# BOOT/STARTUP DEBUGGING
systemd-analyze          # Boot time analysis
systemd-analyze blame    # Which services are slow to start
systemd-analyze critical-chain nginx  # What delayed nginx?

# ONE-LINER DEBUGGING TOOLKIT
alias debug-app='sudo systemctl status $1; sudo journalctl -u $1 -n 50; ss -tlnp'
```

---

## 19.5 Hands-On Exercises — Module 19

```bash
# Exercise 1: Check system health
uptime
free -h
df -h
ss -tlnp

# Exercise 2: Find the biggest space consumers
sudo du -sh /var/log/* 2>/dev/null | sort -hr | head -10
find / -size +50M -type f 2>/dev/null | head -10

# Exercise 3: Trace a process
# Start a process
sleep 100 &
PID=$!
sudo strace -p $PID 2>&1 | head -20
kill $PID

# Exercise 4: Watch file access
sudo inotifywait -m /etc/hosts &
cat /etc/hosts   # Trigger an event
jobs && kill %1   # Stop watching

# Exercise 5: Full service debug
sudo systemctl status nginx
sudo journalctl -u nginx -n 20
sudo nginx -t
curl -v http://localhost
```

---

# MODULE 20 — Advanced Linux Internals

## 20.1 What You Will Learn

- Linux namespaces (the foundation of containers)
- Control groups (cgroups) — resource limits
- How Docker actually works at the kernel level
- Virtual memory and swap
- Kernel parameters tuning

---

## 20.2 Linux Namespaces

Namespaces provide **isolation** — they make a process think it's the only one on the system.

| Namespace | Isolates | Kernel Flag |
|-----------|----------|-------------|
| PID | Process IDs | CLONE_NEWPID |
| NET | Network interfaces | CLONE_NEWNET |
| MNT | Mount points | CLONE_NEWNS |
| UTS | Hostname | CLONE_NEWUTS |
| IPC | IPC objects | CLONE_NEWIPC |
| USER | User/group IDs | CLONE_NEWUSER |
| CGROUP | cgroup root | CLONE_NEWCGROUP |

```bash
# See namespaces for a process
ls -la /proc/self/ns/
# lrwxrwxrwx cgroup -> cgroup:[4026531835]
# lrwxrwxrwx ipc    -> ipc:[4026531839]
# lrwxrwxrwx mnt    -> mnt:[4026531840]
# lrwxrwxrwx net    -> net:[4026531992]
# lrwxrwxrwx pid    -> pid:[4026531836]

# See Docker container namespaces
docker run -d --name test nginx
docker inspect test | grep Pid
PID=$(docker inspect test --format '{{.State.Pid}}')
ls -la /proc/$PID/ns/
# Different namespace IDs than host!

# Create a namespace manually (what Docker does)
sudo unshare --pid --fork --mount-proc bash
# Now you're in an isolated PID namespace
ps aux   # Only sees processes in this namespace
exit
```

---

## 20.3 Control Groups (cgroups)

cgroups **limit and account for resources** used by process groups.

```
cgroups v2 hierarchy:

/sys/fs/cgroup/
├── system.slice/
│   ├── nginx.service/
│   │   ├── memory.max       ← Memory limit
│   │   ├── cpu.max          ← CPU limit
│   │   └── io.max           ← I/O limit
│   └── myapp.service/
├── docker/
│   └── container-id/
│       ├── memory.max
│       └── cpu.max
└── user.slice/
    └── user-1000.slice/
```

```bash
# See all cgroups
ls /sys/fs/cgroup/

# See memory limit for a service
cat /sys/fs/cgroup/system.slice/nginx.service/memory.max
# max  (no limit)
# or a number in bytes if limited

# See Docker container cgroup
docker run -d --memory=256m --name test nginx
cat /sys/fs/cgroup/system.slice/docker-*/memory.max

# Set cgroup limits via systemd service
# In [Service] section:
# MemoryMax=512M
# CPUQuota=50%
```

---

## 20.4 Virtual Memory and Swap

```bash
# Linux virtual memory manager
# Every process gets its own virtual address space
# Physical RAM is shared underneath

# See virtual vs physical memory
free -h
# Mem: 7.6G total, 3.5G used, 2.1G free, 2.0G buff/cache
# Swap: 2.0G total, 100M used, 1.9G free

# Check swap
swapon --show
cat /proc/swaps

# Swappiness (how aggressively to use swap)
cat /proc/sys/vm/swappiness
# Default: 60  (higher = more swap)
# For database servers: set to 10
sudo sysctl vm.swappiness=10

# OOM (Out of Memory) killer
# When RAM is completely full, kernel kills processes
dmesg | grep -i "oom\|killed"
# This shows which processes got killed and why
```

---

## 20.5 Kernel Parameters (sysctl)

```bash
# View all kernel parameters
sysctl -a | head -30

# View specific parameter
sysctl net.ipv4.ip_forward
sysctl vm.swappiness

# Set temporarily (lost on reboot)
sudo sysctl vm.swappiness=10
sudo sysctl net.ipv4.ip_forward=1   # Enable IP forwarding (needed for Docker)

# Set permanently
sudo nano /etc/sysctl.conf
# or
sudo nano /etc/sysctl.d/99-myapp.conf

# Apply without reboot
sudo sysctl -p

# Production tuning for web servers:
```

```bash
# /etc/sysctl.d/99-production.conf

# Network performance
net.core.somaxconn = 65535          # Max connection queue
net.ipv4.tcp_max_syn_backlog = 65535
net.core.netdev_max_backlog = 5000

# Memory
vm.swappiness = 10                  # Less swap for databases
vm.dirty_ratio = 15                 # Write dirty pages earlier
vm.dirty_background_ratio = 5

# File descriptors
fs.file-max = 2097152               # Max open files system-wide

# TCP performance
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 300
net.ipv4.tcp_max_tw_buckets = 1440000
net.ipv4.tcp_tw_reuse = 1

# IP forwarding (for Docker)
net.ipv4.ip_forward = 1
```

---

## 20.6 Understanding the OOM Killer

```bash
# OOM (Out of Memory) Killer kills processes when RAM is exhausted
# It uses an "oom_score" to decide who to kill first
# Higher score = killed first

# See OOM scores
cat /proc/self/oom_score
cat /proc/$(pgrep nginx | head -1)/oom_score

# Adjust OOM score (-1000 = never kill, 1000 = kill first)
sudo echo -500 > /proc/$(pgrep mysqld)/oom_score_adj
# Protect MySQL from being killed by OOM

# In systemd:
# [Service]
# OOMScoreAdjust=-500   ← Protect this service

# Check OOM events
dmesg | grep -i "out of memory"
journalctl | grep -i "killed process"
```

---

## 20.7 Hands-On Exercises — Module 20

```bash
# Exercise 1: Explore namespaces
ls -la /proc/self/ns/

# Exercise 2: See Docker namespaces
docker run -d --name ns-test alpine sleep 999
PID=$(docker inspect ns-test --format '{{.State.Pid}}')
echo "Container PID: $PID"
ls -la /proc/$PID/ns/
echo "---Host namespaces---"
ls -la /proc/self/ns/
docker stop ns-test && docker rm ns-test

# Exercise 3: Create an isolated namespace
sudo unshare --pid --fork --mount-proc bash
# Inside: ps aux  (very few processes!)
# exit

# Exercise 4: Explore cgroups
ls /sys/fs/cgroup/
cat /sys/fs/cgroup/memory.stat | head -10
systemctl show nginx | grep "MemoryCurrent"

# Exercise 5: Check kernel parameters
sysctl vm.swappiness
sysctl net.ipv4.ip_forward
sysctl fs.file-max

# Exercise 6: Check OOM scores
cat /proc/self/oom_score
cat /proc/1/oom_score   # systemd score

# Exercise 7: Memory analysis
free -h
vmstat 1 5
cat /proc/meminfo | grep -E "MemTotal|MemFree|MemAvailable|Cached|SwapTotal|SwapFree"
```

---

# 🎯 Linux Mastery Checklist

Track your progress. Check off each skill as you master it.

## Module 1: Fundamentals
- [ ] Explain the Linux kernel vs user space distinction
- [ ] Read and interpret `uname -a` output
- [ ] Identify your Linux distribution and version
- [ ] Explain the Linux boot process (BIOS → GRUB → Kernel → systemd)
- [ ] Explain what system calls are and why they exist

## Module 2: File System
- [ ] Navigate the FHS (Filesystem Hierarchy Standard) without a guide
- [ ] Explain the purpose of `/etc`, `/var`, `/opt`, `/home`, `/tmp`, `/proc`
- [ ] Know where to find config files for Nginx, SSH, MySQL
- [ ] Use `find` to locate files by name, size, permissions
- [ ] Set up a proper production directory structure

## Module 3: Users and Groups
- [ ] Explain the difference between root, system, and regular users
- [ ] Read and interpret `/etc/passwd` and `/etc/group`
- [ ] Create system users for services (no-login, no-home)
- [ ] Set primary and secondary groups
- [ ] Create and manage users for Docker, Nginx, custom apps

## Module 4: Permissions
- [ ] Read `ls -la` output instantly (rwxr-xr-x, file type, owner, group)
- [ ] Convert between symbolic (rwx) and octal (755) permissions
- [ ] Know the correct permissions for SSH keys (600), web files (644), scripts (755)
- [ ] Use `chmod`, `chown`, `chgrp` with `-R` (recursive)
- [ ] Explain and configure setuid, setgid, sticky bit
- [ ] Set up a production Nginx web directory with correct permissions

## Module 5: Process Management
- [ ] Use `ps aux` and filter output with `grep`
- [ ] Interpret `top` and `htop` output
- [ ] Send signals with `kill` — SIGTERM vs SIGKILL
- [ ] Manage background jobs with `&`, `fg`, `bg`, `jobs`
- [ ] Use `pgrep`, `pkill`, `killall`
- [ ] Diagnose a high-load server

## Module 6: systemd Services
- [ ] Start, stop, restart, reload services with `systemctl`
- [ ] Enable/disable services for boot
- [ ] Read and interpret `systemctl status` output
- [ ] Write a custom `.service` file from scratch
- [ ] Create a Node.js (or any app) as a systemd service
- [ ] View service logs with `journalctl -u servicename`

## Module 7: Networking
- [ ] Read `ip addr` output — find IP addresses and interfaces
- [ ] Use `ss -tlnp` to find what's listening on which port
- [ ] Test connectivity with `ping`, `curl`, `nc`
- [ ] Configure DNS in `/etc/hosts` and `/etc/resolv.conf`
- [ ] Set up an Nginx reverse proxy
- [ ] Configure UFW firewall for a production web server

## Module 8: Package Management
- [ ] Use `apt update`, `apt upgrade`, `apt install`, `apt remove`
- [ ] Add a third-party repository with its GPG key
- [ ] Find which package owns a file (`dpkg -S`)
- [ ] List files installed by a package (`dpkg -L`)
- [ ] Add a NodeSource or Docker repository

## Module 9: Disk & Storage
- [ ] Use `df -h` to check filesystem usage
- [ ] Use `du -sh` to find large directories and files
- [ ] Read `lsblk` output (disks, partitions, mount points)
- [ ] Mount a filesystem and add it to `/etc/fstab`
- [ ] Identify and handle a "disk full" emergency

## Module 10: Logs
- [ ] Use `journalctl -u service` to view service logs
- [ ] Filter logs by time (`--since`), priority (`-p err`), and follow live (`-f`)
- [ ] Find the cause of a server crash using logs
- [ ] Configure log rotation for an application
- [ ] Check `/var/log/auth.log` for security events

## Module 11: SSH
- [ ] Generate an Ed25519 SSH key pair
- [ ] Set up key-based authentication on a server
- [ ] Configure `~/.ssh/config` for multiple servers
- [ ] Harden SSH (`PasswordAuthentication no`, `PermitRootLogin no`)
- [ ] Use SSH tunneling for port forwarding

## Module 12: Security
- [ ] Configure `sudo` to allow specific commands without password
- [ ] Set up UFW with proper firewall rules
- [ ] Install and configure fail2ban
- [ ] Audit for world-writable files and setuid programs
- [ ] Perform a basic security checklist on a new server

## Module 13: Environment Variables
- [ ] Set, export, and unset environment variables
- [ ] Add permanent variables to `.bashrc` or `/etc/environment`
- [ ] Create and use `.env` files for applications
- [ ] Configure environment variables in a systemd service
- [ ] Understand and modify the `PATH` variable

## Module 14: Performance
- [ ] Interpret load average correctly for your CPU count
- [ ] Distinguish memory usage from memory pressure
- [ ] Identify CPU, memory, disk I/O, and network bottlenecks
- [ ] Use `vmstat` and `iostat` for performance analysis
- [ ] Diagnose and document a server performance problem

## Module 15: Cron Jobs
- [ ] Write crontab entries for any time schedule
- [ ] Create, list, and remove crontab entries
- [ ] Write a complete backup script with logging
- [ ] Configure system-wide cron jobs in `/etc/cron.d/`
- [ ] Ensure cron jobs run as the correct user

## Module 16: Docker + Linux
- [ ] Explain how Docker uses Linux namespaces and cgroups
- [ ] Set up Docker permissions with the `docker` group
- [ ] Configure Docker as a systemd service
- [ ] Apply resource limits to containers
- [ ] Secure the Docker daemon configuration

## Module 17: Nginx + Linux
- [ ] Manage Nginx as a systemd service
- [ ] Read and modify `nginx.conf` and server blocks
- [ ] Enable and disable virtual hosts using symlinks
- [ ] Configure a reverse proxy for a Node.js application
- [ ] Set up SSL with Let's Encrypt

## Module 18: Production Server Setup
- [ ] Set up a new Ubuntu server from scratch using the checklist
- [ ] Create the standard production directory structure
- [ ] Create and configure application and deploy users
- [ ] Configure and enable all security measures
- [ ] Set up log rotation and basic monitoring

## Module 19: Linux Debugging
- [ ] Follow the systematic debugging methodology
- [ ] Debug a service that won't start
- [ ] Diagnose a 502 Bad Gateway error
- [ ] Handle a disk full emergency
- [ ] Use `strace` to trace system calls of a process

## Module 20: Advanced Linux
- [ ] Explain what Linux namespaces are and what each type isolates
- [ ] Read and interpret cgroup resource information
- [ ] Explain the OOM killer and protect critical services
- [ ] Tune kernel parameters with `sysctl`
- [ ] Understand how Docker maps to Linux primitives

---

## 🏆 You Have Achieved Linux Mastery When:

```
✅ You can set up a production Ubuntu server from scratch in < 30 minutes
✅ You can debug any service outage using only standard Linux tools
✅ You can secure a server against common attack vectors
✅ You understand why things work, not just how to run commands
✅ You can write systemd services, cron jobs, and shell scripts
✅ You can diagnose CPU, memory, disk, and network bottlenecks
✅ You can comfortably navigate the filesystem hierarchy
✅ You can manage users, groups, and permissions for any scenario
✅ You can configure Nginx as a reverse proxy with SSL
✅ You understand how Docker uses Linux kernel features
```

---

## 📚 Recommended Next Steps

| Topic | Resource |
|-------|----------|
| Shell Scripting | Learn Bash scripting in depth |
| Ansible | Automate server setup |
| Terraform | Infrastructure as Code |
| Kubernetes | Container orchestration (Linux ++)|
| Linux Performance | "Systems Performance" by Brendan Gregg |
| Security | "The Linux Command Line" by William Shotts |
| Kernel Internals | "Linux Kernel Development" by Robert Love |

---

## 🛠️ Quick Reference Card

```bash
# ===== USERS =====
id                          # Who am I?
whoami                      # Current user
sudo useradd -m -s /bin/bash username  # Create user
sudo usermod -aG group user # Add to group
sudo passwd username        # Set password

# ===== PERMISSIONS =====
ls -la                      # List with permissions
chmod 755 file              # Set permissions
chown user:group file       # Change owner
chmod -R 644 dir/           # Recursive

# ===== PROCESSES =====
ps aux | grep name          # Find process
kill PID                    # Graceful kill
kill -9 PID                 # Force kill
systemctl status service    # Service status
journalctl -u service -f    # Follow service logs

# ===== NETWORKING =====
ip addr                     # Show IPs
ss -tlnp                    # Open ports
curl -I https://url         # HTTP headers
sudo ufw status             # Firewall status

# ===== DISK =====
df -h                       # Disk space
du -sh dir/                 # Dir size
find / -size +100M 2>/dev/null  # Large files

# ===== PERFORMANCE =====
uptime                      # Load average
free -h                     # Memory
htop                        # Interactive monitor

# ===== LOGS =====
journalctl -u nginx -f      # Follow nginx logs
journalctl -p err --since "1h ago"   # Recent errors
tail -f /var/log/syslog     # System log

# ===== SSH =====
ssh-keygen -t ed25519       # Generate key
ssh-copy-id user@server     # Copy key to server
ssh user@server -p 2222     # Custom port
```

---

*Linux Mastery Guide — Beginner to Advanced to Production*
*Built for Full-Stack Developers who want to truly master their infrastructure*
*Version 1.0 | 20 Modules | 200+ Commands | Production Ready*
