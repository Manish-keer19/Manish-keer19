# 🐧 Linux Permissions: The Complete Mastery Guide
### From Beginner to Advanced — A Production-Quality Reference

---

> **Who is this guide for?**
> Absolute beginners who just installed Linux, developers who want to stop guessing `chmod` values, sysadmins looking for a deep reference, and DevOps engineers who need to harden servers. This guide will take you from zero to job-ready.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Linux User Types](#2-linux-user-types)
3. [File Types in Linux](#3-file-types-in-linux)
4. [Understanding the Permission Model](#4-understanding-the-permission-model)
5. [Numeric (Octal) Permissions](#5-numeric-octal-permissions)
6. [chmod Command — Deep Dive](#6-chmod-command--deep-dive)
7. [chown and chgrp — Ownership Management](#7-chown-and-chgrp--ownership-management)
8. [Special Permissions — SUID, SGID, Sticky Bit](#8-special-permissions--suid-sgid-sticky-bit)
9. [Default Permissions and umask](#9-default-permissions-and-umask)
10. [Access Control Lists (ACL)](#10-access-control-lists-acl)
11. [Real-World Scenarios](#11-real-world-scenarios)
12. [Security Best Practices](#12-security-best-practices)
13. [Troubleshooting Permission Issues](#13-troubleshooting-permission-issues)
14. [Cheat Sheet](#14-cheat-sheet)
15. [Practice Exercises](#15-practice-exercises)

---

## 1. Introduction

### What Are Linux Permissions?

Linux is a **multi-user operating system**. Multiple people (or processes) can be logged in and running programs simultaneously. To keep users from accidentally — or maliciously — reading, modifying, or executing each other's files, Linux uses a **permission system**.

Every single file and directory on a Linux system has a set of permissions attached to it. These permissions answer three fundamental questions:

1. **Who** is trying to access this file?
2. **What** are they trying to do with it (read, write, execute)?
3. **Are they allowed to do it?**

### Why Do Permissions Matter?

| Scenario | Without Permissions | With Permissions |
|---|---|---|
| Web server | Any user can delete your HTML files | Only `www-data` can write to web root |
| Password file | Any program can read `/etc/shadow` | Only `root` can read it |
| Shared project | Teammates overwrite each other's work | Group permissions control write access |
| Script execution | Any script runs as any user | Only the owner can execute a sensitive script |

Permissions are the **first line of defense** in Linux security. They are not optional — they are fundamental.

### Real-World Analogy

Think of a Linux filesystem like an **office building**:

```
🏢 Office Building = Linux Filesystem

  👑 CEO (root)         → Has a master key to every room
  👷 Employee (user)    → Has a key to their own office only
  👥 Team (group)       → Shares access to one conference room
  🚪 Visitor (others)   → Can only access the lobby

  Each room (file) has a lock (permission) that says:
  ✅ Owner can read, write, enter
  ✅ Team members can read, enter
  ❌ Visitors cannot enter at all
```

---

## 2. Linux User Types

Linux recognizes three broad categories of users. Understanding them is critical before you touch permissions.

### 2.1 The Root User

The **root** user (also called the superuser) has **unrestricted access** to everything on the system. Root can:

- Read, write, and execute any file regardless of permissions
- Create, modify, and delete any user or group
- Change ownership of any file
- Mount and unmount filesystems
- Install and remove software system-wide

```bash
# Root's home directory
/root

# Check if you are root
whoami
# Output: root

# Switch to root (if you have sudo)
sudo su -
```

> ⚠️ **WARNING:** Running day-to-day tasks as root is dangerous. A single typo like `rm -rf /` as root is catastrophic. Always use a normal user and escalate with `sudo` only when needed.

### 2.2 Normal Users

Normal users (also called regular or login users) are human users who log in interactively. They have:

- A home directory under `/home/username`
- A private shell environment
- Access to their own files and files permitted by group or world permissions

```bash
# Normal user's home directory
/home/alice
/home/bob

# Create a new user
sudo useradd -m alice
sudo passwd alice
```

### 2.3 System Users

System users are **non-human** accounts created by the OS or applications to run services. They typically have:

- No login shell (shell set to `/usr/sbin/nologin` or `/bin/false`)
- No home directory, or a service-specific home directory
- Tightly scoped permissions for one job only

```bash
# Examples of system users
www-data   → Apache/Nginx web server
mysql      → MySQL database server
nobody     → Generic unprivileged user
sshd       → OpenSSH daemon

# View system users
cat /etc/passwd | grep nologin
```

### 2.4 UID and GID — The Numbers Behind Users

Linux doesn't actually track users by name internally — it tracks them by **numbers**.

| Term | Full Form | Description |
|---|---|---|
| **UID** | User ID | Unique number assigned to each user |
| **GID** | Group ID | Unique number assigned to each group |

**UID Ranges (typical):**

| Range | Category |
|---|---|
| `0` | Root user always |
| `1 – 999` | System/service users |
| `1000+` | Normal human users |

```bash
# See your UID and GID
id
# Output: uid=1000(alice) gid=1000(alice) groups=1000(alice),27(sudo),1001(developers)

# See UID/GID for any user
id bob

# View the user database
cat /etc/passwd
# Format: username:password:UID:GID:comment:home:shell
# alice:x:1000:1000:Alice Smith:/home/alice:/bin/bash

# View the group database
cat /etc/group
# Format: groupname:password:GID:members
# developers:x:1001:alice,bob
```

### 2.5 Groups — The Collaboration Mechanism

Groups allow multiple users to share permissions without making files world-accessible.

```bash
# Create a group
sudo groupadd developers

# Add a user to a group
sudo usermod -aG developers alice

# Remove a user from a group
sudo gpasswd -d alice developers

# See all groups a user belongs to
groups alice

# See all groups on the system
cat /etc/group
```

---

## 3. File Types in Linux

Before studying permissions, you need to understand **what you're setting permissions on**. Linux represents everything as a file — but not all files are equal.

### 3.1 Identifying File Types with `ls -l`

```bash
ls -l /
```

Output example:
```
total 64
lrwxrwxrwx   1 root root    7 Jan  1 00:00 bin -> usr/bin
drwxr-xr-x   4 root root 4096 Jan  1 00:00 boot
crw-------   1 root root 5, 1 Jan  1 00:00 /dev/console
brw-rw----   1 root disk 8, 0 Jan  1 00:00 /dev/sda
prw-r--r--   1 alice alice   0 Jan  1 00:00 mypipe
-rw-r--r--   1 root root  123 Jan  1 00:00 /etc/hostname
srwxrwxrwx   1 root root    0 Jan  1 00:00 /run/docker.sock
```

The **first character** of each line tells you the file type:

| Character | File Type | Description |
|---|---|---|
| `-` | Regular file | Text, binary, image, script — a normal file |
| `d` | Directory | A folder containing other files |
| `l` | Symbolic link | A pointer/shortcut to another file or directory |
| `c` | Character device | Hardware device read/written one byte at a time (keyboard, terminal) |
| `b` | Block device | Hardware device read/written in blocks (hard disk, USB) |
| `p` | Named pipe (FIFO) | Inter-process communication channel |
| `s` | Socket | Inter-process or network communication endpoint |

### 3.2 File Type Breakdown

**Regular Files (`-`)**
```bash
-rw-r--r-- 1 alice alice  1234 Mar 10 09:00 report.txt
-rwxr-xr-x 1 alice alice  5678 Mar 10 09:00 deploy.sh
-rw-r--r-- 1 root  root  98765 Mar 10 09:00 /etc/passwd
```

**Directories (`d`)**
```bash
drwxr-xr-x 5 alice alice 4096 Mar 10 09:00 projects/
drwx------ 3 alice alice 4096 Mar 10 09:00 .ssh/
drwxrwxrwt 8 root  root  4096 Mar 10 09:00 /tmp/
```

**Symbolic Links (`l`)**
```bash
lrwxrwxrwx 1 root root 7 Jan 1 00:00 /bin -> usr/bin
lrwxrwxrwx 1 root root 19 Jan 1 00:00 /etc/localtime -> ../usr/share/zoneinfo/UTC
```

> 💡 **Note:** Symbolic links always show `lrwxrwxrwx` — the permissions on the symlink itself are irrelevant. What matters are the permissions of the **target** file.

---

## 4. Understanding the Permission Model

This is the heart of Linux permissions. Master this section, and everything else will make sense.

### 4.1 The Permission String

When you run `ls -l`, you see something like this:

```
-rwxr-xr-x  1  alice  developers  4096  Mar 10 09:00  script.sh
```

Let's decode the permission string `-rwxr-xr-x` character by character:

```
 - r w x r - x r - x
 │ │ │ │ │ │ │ │ │ │
 │ └─────┘ └─────┘ └─────┘
 │   User   Group   Others
 │
 └── File type (- = regular file)
```

**Full breakdown:**

```
Position:  [1]  [2][3][4]  [5][6][7]  [8][9][10]
           Type   Owner      Group      Others
           -      r w x      r - x      r - x
```

| Position | Character | Meaning |
|---|---|---|
| 1 | `-` | File type: regular file |
| 2 | `r` | Owner can **read** |
| 3 | `w` | Owner can **write** |
| 4 | `x` | Owner can **execute** |
| 5 | `r` | Group can **read** |
| 6 | `-` | Group **cannot write** |
| 7 | `x` | Group can **execute** |
| 8 | `r` | Others can **read** |
| 9 | `-` | Others **cannot write** |
| 10 | `x` | Others can **execute** |

### 4.2 The Three Permission Types

#### Read (`r`)

| Applied to | Meaning |
|---|---|
| Regular file | Can open and view the contents |
| Directory | Can list the contents with `ls` |

```bash
# Without read permission on a file
cat secret.txt
# Permission denied

# Without read permission on a directory
ls /home/alice/
# Permission denied
```

#### Write (`w`)

| Applied to | Meaning |
|---|---|
| Regular file | Can modify, edit, or truncate the file's contents |
| Directory | Can create, rename, or delete files **inside** the directory |

```bash
# Without write permission on a file
echo "hello" >> readonly.txt
# Permission denied

# Without write permission on a directory
touch /home/alice/newfile.txt
# Permission denied
```

> ⚠️ **Important:** To delete a file, you need **write permission on the directory**, not on the file itself. This surprises many beginners.

#### Execute (`x`)

| Applied to | Meaning |
|---|---|
| Regular file | Can run it as a program or script |
| Directory | Can `cd` into it and access its contents (traverse it) |

```bash
# Without execute on a script
./deploy.sh
# Permission denied

# Without execute on a directory
cd /home/alice/private/
# Permission denied
```

> 💡 **Tip:** You can have a directory with read but no execute permission. `ls` will show filenames but you cannot `cd` into it or open files inside it. Execute on a directory is often called the **"search"** or **"traverse"** bit.

### 4.3 The Three User Categories

#### User (`u`) — The Owner

The **owner** of a file. By default, this is the user who created the file.

#### Group (`g`) — The Group Owner

A file has one **group** associated with it. All users who are members of that group share these permissions.

#### Others (`o`) — Everyone Else

Anyone who is not the file's owner and not a member of the file's group. This is the "world" permission.

### 4.4 Permission Evaluation Order

Linux checks permissions in this exact order and **stops at the first match**:

```
Is the process running as root?
    YES → Access granted (root bypasses all permissions)
    NO  ↓
Is the process running as the file's OWNER?
    YES → Apply OWNER permissions. Done.
    NO  ↓
Is the process running as a member of the file's GROUP?
    YES → Apply GROUP permissions. Done.
    NO  ↓
Apply OTHERS permissions. Done.
```

> ⚠️ **Key insight:** If you are the owner but the owner permissions say `---`, you are **denied** — even if group or others permissions would allow it. Linux evaluates **the matching category only**, not the most permissive one.

### 4.5 Permission Examples

**Example 1: `rwxr-xr-x`**
```
Owner:  rwx → can read, write, execute
Group:  r-x → can read, execute (NOT write)
Others: r-x → can read, execute (NOT write)
```
Typical use: A binary executable or shared script

---

**Example 2: `rw-------`**
```
Owner:  rw- → can read, write (NOT execute)
Group:  --- → no permissions at all
Others: --- → no permissions at all
```
Typical use: A private key file (`~/.ssh/id_rsa`)

---

**Example 3: `rwxrwxrwx`**
```
Owner:  rwx → full access
Group:  rwx → full access
Others: rwx → full access
```
Typical use: NEVER use this in production. It means anyone can do anything.

---

**Example 4: `rw-r--r--`**
```
Owner:  rw- → can read, write
Group:  r-- → can only read
Others: r-- → can only read
```
Typical use: A configuration file like `/etc/hostname`

---

### 4.6 Permission Table

| Permission String | Owner | Group | Others | Common Use Case |
|---|---|---|---|---|
| `rwx------` | Full | None | None | Private executable |
| `rwxr-x---` | Full | Read+Exec | None | Team scripts |
| `rwxr-xr-x` | Full | Read+Exec | Read+Exec | Public binary |
| `rw-------` | Read+Write | None | None | SSH private keys |
| `rw-r--r--` | Read+Write | Read | Read | Public config file |
| `rw-rw----` | Read+Write | Read+Write | None | Shared project file |
| `rw-rw-r--` | Read+Write | Read+Write | Read | Group-editable, world-readable |
| `rwxrwxrwt` | Full | Full | Full+StickyBit | `/tmp` directory |

---

## 5. Numeric (Octal) Permissions

While symbolic permissions (`rwx`) are human-readable, numeric (octal) permissions are faster to type and widely used in scripts and commands.

### 5.1 The 4-2-1 System

Each permission (r, w, x) has a numeric value:

| Permission | Symbol | Value |
|---|---|---|
| Read | `r` | **4** |
| Write | `w` | **2** |
| Execute | `x` | **1** |
| No permission | `-` | **0** |

To calculate the numeric value for one category (user/group/others), you **add** the values of the permissions granted.

```
rwx = 4 + 2 + 1 = 7
rw- = 4 + 2 + 0 = 6
r-x = 4 + 0 + 1 = 5
r-- = 4 + 0 + 0 = 4
-wx = 0 + 2 + 1 = 3
-w- = 0 + 2 + 0 = 2
--x = 0 + 0 + 1 = 1
--- = 0 + 0 + 0 = 0
```

A full permission set is **three digits** — one for User, one for Group, one for Others:

```
chmod 755 file.sh
       │││
       ││└── Others: r-x = 5
       │└─── Group:  r-x = 5
       └──── User:   rwx = 7
```

### 5.2 Complete Octal Reference Table

| Octal | Binary | Symbolic | Meaning |
|---|---|---|---|
| `0` | 000 | `---` | No permissions |
| `1` | 001 | `--x` | Execute only |
| `2` | 010 | `-w-` | Write only |
| `3` | 011 | `-wx` | Write + Execute |
| `4` | 100 | `r--` | Read only |
| `5` | 101 | `r-x` | Read + Execute |
| `6` | 110 | `rw-` | Read + Write |
| `7` | 111 | `rwx` | Read + Write + Execute (Full) |

### 5.3 Common Permission Values

| Octal | Symbolic | Typical Use |
|---|---|---|
| `777` | `rwxrwxrwx` | Everyone has full access — AVOID in production |
| `755` | `rwxr-xr-x` | Public executables, web directories |
| `644` | `rw-r--r--` | Standard files, web content |
| `600` | `rw-------` | Private files, SSH keys |
| `700` | `rwx------` | Private scripts |
| `664` | `rw-rw-r--` | Group-collaborative files |
| `775` | `rwxrwxr-x` | Group-collaborative directories |
| `750` | `rwxr-x---` | Application directories |
| `640` | `rw-r-----` | Sensitive config files |
| `400` | `r--------` | Read-only backups, key material |

### 5.4 Practice: Reading Octal

**Exercise — What does `chmod 764` do?**

```
7 = 4+2+1 = rwx  (Owner gets full access)
6 = 4+2+0 = rw-  (Group gets read+write)
4 = 4+0+0 = r--  (Others get read only)

Result: rwxrw-r--
```

**Exercise — What octal is `rw-r-xr--`?**

```
rw- = 4+2+0 = 6
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4

Result: 654
```

---

## 6. chmod Command — Deep Dive

`chmod` (change mode) is the command used to modify file permissions. It has two modes: **symbolic** and **numeric**.

### 6.1 Syntax

```bash
chmod [OPTIONS] MODE FILE
```

### 6.2 Symbolic Mode

Symbolic mode uses letters and operators to describe changes:

**Operators:**

| Operator | Meaning |
|---|---|
| `+` | Add permission |
| `-` | Remove permission |
| `=` | Set exact permission (replaces existing) |

**User categories:**

| Letter | Category |
|---|---|
| `u` | User (owner) |
| `g` | Group |
| `o` | Others |
| `a` | All (u+g+o) |

**Syntax:**

```bash
chmod [u/g/o/a][+/-/=][r/w/x] filename
```

**Examples:**

```bash
# Add execute for owner
chmod u+x script.sh

# Remove write from others
chmod o-w publicfile.txt

# Add read+write for group
chmod g+rw sharedfile.txt

# Set exact permissions: owner=rwx, group=rx, others=nothing
chmod u=rwx,g=rx,o= script.sh

# Add execute for everyone
chmod a+x deploy.sh

# Remove all permissions for others
chmod o-rwx sensitive.conf

# Give group same permissions as owner
chmod g=u file.txt
```

### 6.3 Numeric Mode

```bash
# Full access for owner, read+execute for group and others
chmod 755 script.sh

# Read+write for owner, read for everyone
chmod 644 index.html

# Private file — owner only
chmod 600 private.key

# Make a directory accessible to group
chmod 750 /opt/app/
```

### 6.4 Recursive chmod

Use `-R` to apply permissions to a directory and all its contents:

```bash
# Set 755 on a directory and everything inside
chmod -R 755 /var/www/html/

# Make all .sh files in a folder executable
find /opt/scripts/ -name "*.sh" -exec chmod +x {} \;

# Set files to 644 and directories to 755 (the correct way to do both)
find /var/www/html -type f -exec chmod 644 {} \;
find /var/www/html -type d -exec chmod 755 {} \;
```

> ⚠️ **Warning:** Never run `chmod -R 777 /var/www/html/`. This allows any user on the system to modify your web files — a severe security hole.

### 6.5 Real-World chmod Examples

**Web Server Permissions**
```bash
# Web root: owned by www-data, group www-data
# Directories need execute to traverse
# Files need read, but not execute (unless PHP, etc.)
sudo chown -R www-data:www-data /var/www/html/
sudo find /var/www/html/ -type d -exec chmod 755 {} \;
sudo find /var/www/html/ -type f -exec chmod 644 {} \;
```

**Making a Shell Script Executable**
```bash
# Create script
nano deploy.sh

# Make it executable by owner only
chmod 700 deploy.sh

# Run it
./deploy.sh
```

**Securing SSH Keys**
```bash
# Private key — must be readable ONLY by owner
chmod 600 ~/.ssh/id_rsa

# Public key — can be world-readable
chmod 644 ~/.ssh/id_rsa.pub

# .ssh directory itself
chmod 700 ~/.ssh/
```

**Application Config File**
```bash
# Config with passwords — owner read/write only
chmod 600 /etc/myapp/config.env

# Config without secrets — readable by app group
chmod 640 /etc/myapp/settings.conf
chown root:appgroup /etc/myapp/settings.conf
```

---

## 7. chown and chgrp — Ownership Management

### 7.1 chown — Change Ownership

`chown` changes the owner (and optionally the group) of a file.

```bash
# Syntax
chown [OPTIONS] OWNER[:GROUP] FILE

# Change owner only
chown alice file.txt

# Change owner AND group
chown alice:developers file.txt

# Change group only (note the colon prefix)
chown :developers file.txt

# Recursive — change everything in a directory
chown -R alice:alice /home/alice/

# Change owner of a symlink itself (not the target)
chown -h alice symlink_name
```

### 7.2 chgrp — Change Group

`chgrp` changes only the group ownership.

```bash
# Syntax
chgrp [OPTIONS] GROUP FILE

# Change group of a file
chgrp developers project.txt

# Change group recursively
chgrp -R www-data /var/www/html/

# Change group of symlink itself
chgrp -h webteam link_to_file
```

### 7.3 Real-World Ownership Examples

**Scenario: Developer Shared Project Folder**
```bash
# Create a group for the team
sudo groupadd webteam

# Add users to the group
sudo usermod -aG webteam alice
sudo usermod -aG webteam bob
sudo usermod -aG webteam charlie

# Set up shared directory
sudo mkdir /srv/webproject
sudo chown root:webteam /srv/webproject
sudo chmod 2775 /srv/webproject
# (2775 = SGID bit + rwxrwxr-x — covered in section 8)
```

**Scenario: Deploying a Node.js App**
```bash
# App files owned by app user, readable by nginx
sudo useradd -r -s /usr/sbin/nologin nodeapp
sudo chown -R nodeapp:nodeapp /opt/myapp/
sudo chmod -R 750 /opt/myapp/

# Log directory writable by app
sudo mkdir /var/log/myapp
sudo chown nodeapp:nodeapp /var/log/myapp
sudo chmod 755 /var/log/myapp
```

**Scenario: Recovering a Misowned Home Directory**
```bash
# If /home/alice got wrong ownership
sudo chown -R alice:alice /home/alice/
```

### 7.4 Understanding the `ls -l` Output in Context

```bash
ls -l /var/www/html/index.html
-rw-r--r-- 1 www-data www-data 1234 Mar 10 09:00 index.html
             │          │
             │          └── Group owner: www-data
             └── User owner: www-data
```

---

## 8. Special Permissions — SUID, SGID, Sticky Bit

Beyond the standard `rwx`, Linux has three special permission bits that enable powerful (and sometimes risky) behaviors.

### 8.1 Overview

| Bit | Name | Octal | Symbol (file) | Symbol (dir) |
|---|---|---|---|---|
| SUID | Set User ID | `4xxx` | `s` in owner execute | N/A |
| SGID | Set Group ID | `2xxx` | `s` in group execute | `s` in group execute |
| Sticky | Sticky Bit | `1xxx` | `t` in others execute | `t` in others execute |

### 8.2 SUID — Set User ID

**What it does:** When set on an **executable file**, the program runs with the **owner's privileges**, not the caller's. If root owns the file, the program runs as root — regardless of who executes it.

**Why it exists:** Some programs need elevated privileges temporarily to do their job.

```bash
# Classic example: /usr/bin/passwd
# Regular users need to change their own password.
# But /etc/shadow (the password file) is only writable by root.
# Solution: passwd has SUID set, so it temporarily runs as root.

ls -l /usr/bin/passwd
-rwsr-xr-x 1 root root 59640 Mar 10 09:00 /usr/bin/passwd
     ^
     └── 's' here = SUID is set (and execute is also set)

# Other common SUID binaries
ls -l /usr/bin/sudo
ls -l /usr/bin/ping
ls -l /usr/bin/su
```

**Setting SUID:**
```bash
# Symbolic
chmod u+s myprogram

# Numeric (4 prefix)
chmod 4755 myprogram

# Viewing files with SUID
find / -perm /4000 -type f 2>/dev/null
```

> ⚠️ **Security Risk:** SUID files are a major attack surface. If a SUID binary has a vulnerability, an attacker can exploit it to gain root. Audit all SUID files regularly.

### 8.3 SGID — Set Group ID

**What it does on files:** The executable runs with the **group's privileges** instead of the user's group.

**What it does on directories:** All new files created inside the directory **inherit the directory's group** instead of the creator's primary group.

```bash
# SGID on a file
ls -l /usr/bin/write
-rwxr-sr-x 1 root tty 14328 Mar 10 09:00 /usr/bin/write
        ^
        └── 's' in group execute position = SGID

# SGID on a directory — extremely useful for shared folders!
ls -ld /srv/webproject
drwxrwsr-x 2 root webteam 4096 Mar 10 09:00 /srv/webproject
       ^
       └── 's' in group execute position = SGID on directory

# Without SGID: Alice creates file.txt → owned by alice:alice
# With SGID:    Alice creates file.txt → owned by alice:webteam
# Everyone in webteam can access it!
```

**Setting SGID:**
```bash
# Symbolic
chmod g+s /srv/shared/

# Numeric (2 prefix)
chmod 2775 /srv/shared/

# Viewing directories with SGID
find / -perm /2000 -type d 2>/dev/null
```

### 8.4 Sticky Bit

**What it does on directories:** Even if a directory is world-writable, users can only **delete or rename files they own**. They cannot delete other users' files.

**Why it exists:** The `/tmp` directory needs to be writable by all users, but we don't want users deleting each other's temporary files.

```bash
# The classic example: /tmp
ls -ld /tmp
drwxrwxrwt 8 root root 4096 Mar 10 09:00 /tmp
          ^
          └── 't' = Sticky Bit is set

# Alice creates /tmp/alice_data
# Bob creates /tmp/bob_data
# Bob CANNOT delete /tmp/alice_data (even though /tmp is 777)
# Only alice (or root) can delete alice's files
```

**Setting Sticky Bit:**
```bash
# Symbolic
chmod +t /tmp/shared_folder/
chmod o+t /tmp/shared_folder/

# Numeric (1 prefix)
chmod 1777 /tmp/shared_folder/

# Removing sticky bit
chmod -t /tmp/shared_folder/
```

### 8.5 Capital S and Capital T

When you see a **capital S** or **capital T** instead of lowercase, it means the special bit is set but the corresponding **execute bit is NOT set**.

```bash
# Lowercase 's' = SUID/SGID + execute is set   ← Normal
# Uppercase 'S' = SUID/SGID, execute NOT set   ← Unusual, probably a mistake

# Lowercase 't' = Sticky + others execute set  ← Normal for /tmp
# Uppercase 'T' = Sticky, others execute NOT set ← Unusual
```

### 8.6 Combined Octal for Special Bits

The special bits add a **4th leading digit**:

```
chmod 4755 file   →  SUID + rwxr-xr-x
chmod 2775 dir    →  SGID + rwxrwxr-x
chmod 1777 dir    →  Sticky + rwxrwxrwx
chmod 6755 file   →  SUID + SGID + rwxr-xr-x
```

---

## 9. Default Permissions and umask

### 9.1 What is umask?

When you create a new file or directory, what permissions does it get by default? The answer depends on `umask` (user file creation mask).

`umask` defines which permissions are **REMOVED** from the system defaults when creating new files/directories.

### 9.2 Default Maximum Permissions

| Type | Maximum Default Permissions | Octal |
|---|---|---|
| File | `rw-rw-rw-` | `666` |
| Directory | `rwxrwxrwx` | `777` |

Why don't files default to `777`? Because files are not usually meant to be executable. The system defaults to `666` for files, then umask removes bits further.

### 9.3 How umask Works — The Calculation

```
Final permissions = Maximum permissions - umask (NOT subtraction — bitwise AND with complement)

Simpler way to think about it:
  Final = Maximum permissions with umask bits REMOVED
```

**Example: umask = 022 (the most common default)**

```
For files:
  Maximum:  666  (rw-rw-rw-)
  umask:    022  (----w--w-)  ← bits to REMOVE
  Result:   644  (rw-r--r--)  ← bits removed from group-write and others-write

For directories:
  Maximum:  777  (rwxrwxrwx)
  umask:    022  (----w--w-)  ← bits to REMOVE
  Result:   755  (rwxr-xr-x)  ← bits removed from group-write and others-write
```

**Another example: umask = 027**

```
For files:
  Maximum:  666  (rw-rw-rw-)
  umask:    027  (----w-rwx)  ← bits to REMOVE
  Result:   640  (rw-r-----)

For directories:
  Maximum:  777  (rwxrwxrwx)
  umask:    027  (----w-rwx)  ← bits to REMOVE
  Result:   750  (rwxr-x---)
```

### 9.4 Common umask Values

| umask | Files created as | Dirs created as | Use Case |
|---|---|---|---|
| `022` | `644` | `755` | Standard — most systems default |
| `027` | `640` | `750` | Security-conscious servers |
| `077` | `600` | `700` | Highly secure (personal data) |
| `002` | `664` | `775` | Collaborative environments |
| `000` | `666` | `777` | Never use in production |

### 9.5 Viewing and Changing umask

```bash
# View current umask (octal)
umask
# Output: 0022

# View current umask (symbolic)
umask -S
# Output: u=rwx,g=rx,o=rx

# Change umask for current session only
umask 027

# Change umask permanently for a user
# Add to ~/.bashrc or ~/.profile:
echo "umask 027" >> ~/.bashrc
source ~/.bashrc

# Change system-wide default umask
# Edit /etc/login.defs:
sudo nano /etc/login.defs
# Find line: UMASK 022
# Change to: UMASK 027
```

> 💡 **Tip:** Most Linux systems set umask to `022` (for regular users) or `027` (for root and service accounts). For a shared development server, `002` makes collaboration easier.

---

## 10. Access Control Lists (ACL)

### 10.1 The Limitation of Standard Permissions

Standard Unix permissions only allow you to set permissions for:
- **One owner** (user)
- **One group**
- **Everyone else**

What if you need this scenario?

```
/srv/project/

alice  → rwx  (project lead)
bob    → rw-  (developer, same group as alice)
carol  → r--  (stakeholder, DIFFERENT group)
dave   → ---  (no access at all)
```

With standard permissions alone, you **cannot** achieve this. That's where **ACL (Access Control Lists)** come in.

### 10.2 What are ACLs?

ACLs extend the standard permission model to allow **per-user and per-group** permissions on any file or directory, without changing the standard owner/group.

### 10.3 Installing ACL Tools

```bash
# Debian/Ubuntu
sudo apt install acl

# RHEL/CentOS/Fedora
sudo yum install acl
# or
sudo dnf install acl

# Check if filesystem is mounted with ACL support
mount | grep acl
# Or check /etc/fstab — look for 'acl' option
# Most modern Linux filesystems (ext4, xfs) have ACL enabled by default
```

### 10.4 getfacl — View ACL

```bash
# View ACL of a file
getfacl /srv/project/

# Output:
# file: srv/project/
# owner: alice
# group: developers
# user::rwx         ← Standard owner permissions
# user:carol:r-x    ← ACL entry: carol gets r-x
# group::rwx        ← Standard group permissions
# mask::rwx         ← Maximum effective permissions
# other::r-x        ← Standard others permissions
```

### 10.5 setfacl — Set ACL

```bash
# Syntax
setfacl [OPTIONS] SPEC FILE

# Grant specific user access
setfacl -m u:carol:r-- /srv/project/report.txt

# Grant specific group access
setfacl -m g:contractors:rx /srv/project/

# Set ACL recursively
setfacl -R -m u:carol:r-x /srv/project/

# Set default ACL (inherited by new files in directory)
setfacl -d -m u:carol:r-x /srv/project/

# Remove a specific user's ACL entry
setfacl -x u:carol /srv/project/report.txt

# Remove ALL ACL entries (revert to standard permissions)
setfacl -b /srv/project/report.txt

# Copy ACL from one file to another
getfacl source.txt | setfacl --set-file=- destination.txt
```

### 10.6 ACL Mask

The **mask** in ACL output defines the **maximum effective permissions** for any named user or group (except the owner). Think of it as an upper bound.

```bash
# If mask is r-x, even if carol has rwx, her effective permission is r-x

# Set mask explicitly
setfacl -m m::rx /srv/project/

# Check effective permissions
getfacl /srv/project/
# user:carol:rwx    #effective:r-x   ← effective is limited by mask
```

### 10.7 Real ACL Scenario

**Problem:** Apache web server needs to read files in alice's home directory, but alice doesn't want to make her whole home world-readable.

```bash
# Instead of chmod 755 /home/alice/ (dangerous!)
# Use ACL to give www-data specific access

setfacl -m u:www-data:r-x /home/alice/
setfacl -m u:www-data:r-x /home/alice/public_site/
setfacl -R -m u:www-data:r-x /home/alice/public_site/

# Verify
getfacl /home/alice/public_site/
```

> 💡 **Tip:** Files with ACLs show a `+` at the end of the permission string in `ls -l`:
> `-rw-r--r--+ 1 alice alice 1234 Mar 10 09:00 file.txt`

---

## 11. Real-World Scenarios

### 11.1 Apache/Nginx Web Server Setup

**Goal:** Web files readable by the server, writable only by the deployment user.

```bash
# Setup
sudo groupadd webteam
sudo useradd -m deployer
sudo usermod -aG webteam deployer
sudo usermod -aG webteam www-data

# Web root setup
sudo mkdir -p /var/www/mysite
sudo chown deployer:webteam /var/www/mysite
sudo chmod 2775 /var/www/mysite    # SGID so new files inherit group

# Deploy files
sudo -u deployer rsync -av ./dist/ /var/www/mysite/

# Set correct permissions after deploy
sudo find /var/www/mysite -type d -exec chmod 2755 {} \;
sudo find /var/www/mysite -type f -exec chmod 644 {} \;

# Nginx config files — root owned, read by nginx
sudo chown root:root /etc/nginx/nginx.conf
sudo chmod 644 /etc/nginx/nginx.conf

# SSL certificates — very sensitive
sudo chmod 600 /etc/ssl/private/mysite.key
sudo chmod 644 /etc/ssl/certs/mysite.crt
```

### 11.2 Shared Development Folder (Team Project)

```bash
# Create team group
sudo groupadd devteam
sudo usermod -aG devteam alice
sudo usermod -aG devteam bob
sudo usermod -aG devteam charlie

# Create shared directory
sudo mkdir /srv/project
sudo chown root:devteam /srv/project
sudo chmod 2775 /srv/project    # SGID ensures new files belong to devteam
                                 # 775 means team can read/write

# Members log out and back in (to pick up group membership)
# Now when alice creates a file:
touch /srv/project/alice_work.txt
ls -l /srv/project/alice_work.txt
# -rw-rw-r-- 1 alice devteam 0 Mar 10 09:00 alice_work.txt
#                    ^^^^^^^
#                    Group inherited from directory due to SGID!
# bob can now edit alice_work.txt
```

### 11.3 Secure File Storage (Credentials, Keys)

```bash
# Application secrets directory
sudo mkdir /etc/myapp
sudo chown root:appgroup /etc/myapp
sudo chmod 750 /etc/myapp        # root+appgroup can enter, others cannot

# Database credentials
sudo touch /etc/myapp/db.conf
sudo chown root:appgroup /etc/myapp/db.conf
sudo chmod 640 /etc/myapp/db.conf   # owner rw, group r, others nothing

# SSH keys for deployment
sudo mkdir /home/deployer/.ssh
sudo chown deployer:deployer /home/deployer/.ssh
sudo chmod 700 /home/deployer/.ssh

sudo touch /home/deployer/.ssh/authorized_keys
sudo chown deployer:deployer /home/deployer/.ssh/authorized_keys
sudo chmod 600 /home/deployer/.ssh/authorized_keys
```

### 11.4 DevOps and CI/CD Use Cases

**Scenario: Jenkins running deployments**

```bash
# Jenkins system user needs to:
# - Read application source code
# - Write to build output directory
# - Execute deploy scripts

sudo useradd -r -s /usr/sbin/nologin jenkins

# Give jenkins read access to source
sudo setfacl -R -m u:jenkins:r-x /srv/repos/myapp/

# Give jenkins write access to build output
sudo mkdir /var/builds/myapp
sudo chown jenkins:jenkins /var/builds/myapp
sudo chmod 755 /var/builds/myapp

# Deploy script owned by root, executable by jenkins
sudo chown root:jenkins /usr/local/bin/deploy_myapp.sh
sudo chmod 750 /usr/local/bin/deploy_myapp.sh
```

**Scenario: Docker socket permissions**

```bash
# Allow a user to run docker without sudo
# (by adding them to the docker group)
sudo usermod -aG docker alice

# The docker socket
ls -l /var/run/docker.sock
# srw-rw---- 1 root docker 0 Mar 10 09:00 /var/run/docker.sock
# Group 'docker' has rw access to the socket
```

**Scenario: Log files — writable by app, readable by monitoring**

```bash
sudo mkdir /var/log/myapp
sudo chown myapp:myapp /var/log/myapp
sudo chmod 750 /var/log/myapp

# Let monitoring user read logs
sudo setfacl -R -m u:prometheus:r-x /var/log/myapp
sudo setfacl -d -m u:prometheus:r-x /var/log/myapp  # default for new files
```

---

## 12. Security Best Practices

### 12.1 The Principle of Least Privilege

> Grant only the minimum permissions required to perform a task. Nothing more.

```bash
# BAD: Give full access "just to be safe"
chmod 777 /var/www/html/
chown nobody:nobody /etc/app/config.conf
chmod 666 /etc/app/config.conf

# GOOD: Give exactly what's needed
chmod 755 /var/www/html/          # Directories: traversable, not writable by others
chmod 644 /var/www/html/index.html  # Files: readable, writable only by owner
chown appuser:appgroup /etc/app/config.conf
chmod 640 /etc/app/config.conf    # Owner rw, group r, others nothing
```

### 12.2 Never Use 777

`chmod 777` means: **any user on the system can read, write, and execute this file.** This is almost never what you want.

```bash
# 777 on a file: any user can overwrite your file, inject malicious code,
# read sensitive data, and execute it.

# 777 on a directory: anyone can create files in it, delete others' files
# (unless sticky bit), and rename things.

# The ONLY legitimate use of 777 is /tmp (with sticky bit → 1777)
# Everything else should have targeted, minimal permissions.
```

### 12.3 Audit SUID and SGID Files

```bash
# Find all SUID files (runs as file owner, possibly root)
find / -perm /4000 -type f 2>/dev/null

# Find all SGID files/directories
find / -perm /2000 2>/dev/null

# Find all world-writable files (excluding /tmp, /proc)
find / -perm /o+w -not -path "/tmp/*" -not -path "/proc/*" 2>/dev/null

# Find all world-writable directories
find / -type d -perm /o+w -not -path "/tmp" -not -path "/proc/*" 2>/dev/null
```

### 12.4 Protect Sensitive Files

```bash
# SSH configuration
chmod 600 ~/.ssh/id_rsa           # Private key
chmod 644 ~/.ssh/id_rsa.pub       # Public key
chmod 700 ~/.ssh/                 # SSH directory
chmod 600 ~/.ssh/authorized_keys  # Authorized keys
chmod 644 ~/.ssh/known_hosts      # Known hosts

# Application secrets
chmod 600 .env
chmod 600 /etc/myapp/secrets.conf

# Cron jobs
chmod 700 /etc/cron.d/
chmod 644 /etc/cron.d/mycronjob   # Root-owned, not writable by others

# Web application uploads directory
# Should NOT be executable
find /var/www/uploads/ -type f -exec chmod 644 {} \;
find /var/www/uploads/ -type d -exec chmod 755 {} \;
# And configure your web server to not execute files in this directory
```

### 12.5 Common Mistakes

| Mistake | Problem | Fix |
|---|---|---|
| `chmod 777` on web files | Anyone can inject malware | `chmod 644` for files, `755` for dirs |
| World-writable `/etc/passwd` | Any user can create root accounts | `chmod 644 /etc/passwd` |
| `.ssh` dir with wrong perms | SSH refuses to connect | `chmod 700 ~/.ssh` |
| Private key with wrong perms | SSH refuses to use it | `chmod 600 ~/.ssh/id_rsa` |
| SUID on custom script | Privilege escalation risk | Never set SUID on shell scripts |
| Forgetting sticky on shared dirs | Users delete each other's files | `chmod +t /shared/dir` |
| Running services as root | Exploit = full system compromise | Create dedicated service users |
| Recursive `777` during debugging | Forgotten, left in production | Use ACLs for targeted access instead |

### 12.6 Regular Permission Audits

```bash
# Baseline check — save expected state
find /var/www -type f -exec stat --format="%n %a %U %G" {} \; > /root/permissions_baseline.txt

# Later, compare against baseline
find /var/www -type f -exec stat --format="%n %a %U %G" {} \; > /tmp/permissions_current.txt
diff /root/permissions_baseline.txt /tmp/permissions_current.txt
```

---

## 13. Troubleshooting Permission Issues

### 13.1 Understanding "Permission Denied"

`Permission denied` is Linux's way of saying: "The permission check failed." Here's how to debug it systematically.

### 13.2 Step-by-Step Debugging

**Step 1: Identify who you are**
```bash
whoami         # Current username
id             # Full user and group info
groups         # All groups you belong to
```

**Step 2: Check the file's permissions and ownership**
```bash
ls -la /path/to/file
# Also check the parent directories!
ls -la /path/to/
ls -la /path/
```

**Step 3: Trace the full path**
```bash
# Every directory in the path needs execute permission for you to traverse it
namei -l /path/to/file
# OR manually:
ls -ld /
ls -ld /path
ls -ld /path/to
ls -ld /path/to/file
```

**Step 4: Check ACLs**
```bash
getfacl /path/to/file
```

**Step 5: Check if filesystem is mounted read-only**
```bash
mount | grep " / "
mount | grep /path
# Look for 'ro' in the options
cat /proc/mounts
```

### 13.3 Common Permission Problems and Fixes

**Problem: `bash: ./script.sh: Permission denied`**
```bash
# Script is not executable
ls -l script.sh
# -rw-r--r-- 1 alice alice 123 Mar 10 09:00 script.sh

# Fix:
chmod +x script.sh
```

**Problem: `Permission denied` when cd into directory**
```bash
# Directory is missing execute bit
ls -ld /home/bob/private/
# drw------- 2 bob bob 4096 Mar 10 09:00 /home/bob/private/
# You don't have execute permission on this directory

# Fix (if you're the owner):
chmod 755 /home/bob/private/
# Or use ACL to grant access to specific users
sudo setfacl -m u:alice:rx /home/bob/private/
```

**Problem: Can read file but can't save changes**
```bash
# No write permission
ls -l /etc/hosts
-rw-r--r-- 1 root root 312 Mar 10 09:00 /etc/hosts

# Fix: use sudo
sudo nano /etc/hosts
```

**Problem: SSH keeps saying `bad permissions` for key**
```bash
# SSH enforces strict permissions on keys
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@ WARNING: UNPROTECTED PRIVATE KEY FILE! @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for 'id_rsa' are too open.

# Fix:
chmod 600 ~/.ssh/id_rsa
chmod 700 ~/.ssh/
```

**Problem: Apache returning 403 Forbidden**
```bash
# Check web root permissions
ls -ld /var/www/html/
namei -l /var/www/html/index.html

# Common fix:
sudo chmod 755 /var/www/html/
sudo chmod 644 /var/www/html/index.html
sudo chown -R www-data:www-data /var/www/html/

# Also check SELinux/AppArmor if on RHEL/Ubuntu Server
sudo setenforce 0  # Temporarily disable SELinux to test (RHEL)
sudo aa-status     # Check AppArmor (Ubuntu)
```

**Problem: `sudo: unable to open ... Permission denied`**
```bash
# /etc/sudoers has wrong permissions
# Fix: use visudo or pkexec
pkexec chmod 0440 /etc/sudoers
```

### 13.4 Useful Diagnostic Commands

```bash
# Trace what permissions are being checked
strace -e trace=open,access,stat command 2>&1 | grep "EACCES\|EPERM"

# Check if a specific user has access to a file
sudo -u www-data cat /var/www/html/config.php

# Run a command as another user to test permissions
su -c "cat /srv/project/data.txt" bob

# Check effective permissions with ACL
getfacl --access /path/to/file
```

---

## 14. Cheat Sheet

### 14.1 Quick Command Reference

```bash
# === VIEWING ===
ls -l file               # Show file permissions
ls -la directory/        # Show all files including hidden
ls -ld directory/        # Show directory's own permissions
stat file                # Detailed file metadata
namei -l /path/to/file   # Show permissions along a path
getfacl file             # Show ACL entries

# === chmod ===
chmod 755 file           # rwxr-xr-x
chmod 644 file           # rw-r--r--
chmod 600 file           # rw-------
chmod 700 file           # rwx------
chmod +x file            # Add execute for everyone
chmod u+x file           # Add execute for owner only
chmod o-w file           # Remove write from others
chmod -R 755 dir/        # Recursive
chmod a=r file           # Set read-only for everyone

# === chown ===
chown alice file         # Change owner to alice
chown alice:devs file    # Change owner and group
chown :devs file         # Change group only
chown -R alice:alice dir # Recursive ownership change

# === chgrp ===
chgrp developers file    # Change group
chgrp -R webteam dir/    # Recursive group change

# === Special Permissions ===
chmod u+s file           # Set SUID
chmod g+s dir/           # Set SGID
chmod +t dir/            # Set Sticky Bit
chmod 4755 file          # SUID + rwxr-xr-x
chmod 2775 dir/          # SGID + rwxrwxr-x
chmod 1777 dir/          # Sticky + rwxrwxrwx

# === umask ===
umask                    # View current umask
umask 022                # Set umask for session
umask -S                 # View in symbolic form

# === ACL ===
getfacl file             # View ACL
setfacl -m u:alice:rwx file   # Give alice rwx
setfacl -m g:devs:rx dir/     # Give group rx
setfacl -R -m u:alice:rx dir/ # Recursive ACL
setfacl -d -m u:alice:rx dir/ # Default ACL (new files inherit)
setfacl -x u:alice file       # Remove alice's ACL entry
setfacl -b file               # Remove ALL ACL entries

# === Audit ===
find / -perm /4000 -type f 2>/dev/null    # All SUID files
find / -perm /2000 -type d 2>/dev/null    # All SGID directories
find / -perm /o+w -type f 2>/dev/null     # World-writable files
```

### 14.2 Permission Quick Reference Table

| Octal | Symbolic | Category | Typical Use |
|---|---|---|---|
| `700` | `rwx------` | Private executable | Owner-only scripts |
| `600` | `rw-------` | Private file | SSH keys, secrets |
| `400` | `r--------` | Read-only private | Backups, certs |
| `755` | `rwxr-xr-x` | Public executable | Binaries, web dirs |
| `644` | `rw-r--r--` | Public file | Web files, configs |
| `664` | `rw-rw-r--` | Group-collaborative | Team source files |
| `775` | `rwxrwxr-x` | Group-collaborative dir | Shared project dirs |
| `750` | `rwxr-x---` | Group-only access | App directories |
| `640` | `rw-r-----` | Group-readable | Sensitive configs |
| `1777` | `rwxrwxrwt` | Public + Sticky | /tmp |
| `2775` | `rwxrwsr-x` | SGID collaborative | Shared project dirs |
| `4755` | `rwsr-xr-x` | SUID binary | passwd, sudo-like |

### 14.3 Numeric Permission Matrix

```
     USER   GROUP  OTHERS
       │       │       │
     r w x   r w x   r w x
     │ │ │   │ │ │   │ │ │
     4 2 1   4 2 1   4 2 1

rwx = 7   rw- = 6   r-x = 5
r-- = 4   -wx = 3   -w- = 2
--x = 1   --- = 0
```

### 14.4 Who Can Do What — Decision Tree

```
Accessing a file?
        │
        ▼
   Are you root?
   ├── YES → Access GRANTED (root bypasses all)
   └── NO
        │
        ▼
   Are you the FILE OWNER?
   ├── YES → Apply OWNER (u) permissions
   └── NO
        │
        ▼
   Are you in the FILE'S GROUP?
   ├── YES → Apply GROUP (g) permissions
   └── NO → Apply OTHERS (o) permissions
```

---

## 15. Practice Exercises

### 15.1 Beginner Exercises

**Exercise 1: Decode permissions**

Look at this output and answer the questions:
```
-rwxr-x--- 1 alice developers 4096 Mar 10 09:00 deploy.sh
```

1. What type of file is this?
2. What can the owner (alice) do?
3. Can members of the `developers` group execute this file?
4. Can the user `bob` (not in `developers`) read this file?
5. What is the octal representation of these permissions?

<details>
<summary>Answers</summary>

1. Regular file (`-`)
2. Read, write, and execute (`rwx`)
3. Yes — group has `r-x` (read + execute)
4. No — others have `---` (no permissions)
5. `750`

</details>

---

**Exercise 2: Apply basic permissions**

Create a file and directory, then set the following:
- File `notes.txt`: owner can read/write, everyone else can only read
- Directory `projects/`: owner has full access, group can read and execute, others have no access

```bash
# Create the files
touch notes.txt
mkdir projects

# Your chmod commands here:
# ??
# ??
```

<details>
<summary>Solution</summary>

```bash
chmod 644 notes.txt
chmod 750 projects/

# Verify
ls -l notes.txt    # Should show: -rw-r--r--
ls -ld projects/   # Should show: drwxr-x---
```

</details>

---

### 15.2 Intermediate Exercises

**Exercise 3: Fix a broken web server**

An Nginx web server is returning 403 Forbidden on all pages. The web root is `/var/www/mysite`. Nginx runs as `www-data`.

```bash
ls -la /var/www/mysite/
# drwx------ 2 alice alice 4096 Mar 10 09:00 .
# drwxr-xr-x 4 root  root  4096 Mar 10 09:00 ..
# -rw------- 1 alice alice 1234 Mar 10 09:00 index.html
```

What is wrong? Write the commands to fix it.

<details>
<summary>Solution</summary>

**Problem:** The directory (`drwx------`) and file (`-rw-------`) are only accessible by alice. Nginx (`www-data`) cannot read them.

**Fix:**
```bash
# Fix directory permissions
sudo chmod 755 /var/www/mysite/

# Fix file permissions
sudo chmod 644 /var/www/mysite/index.html

# Better: change group ownership so nginx can access
sudo chown -R alice:www-data /var/www/mysite/
sudo chmod 750 /var/www/mysite/
sudo chmod 640 /var/www/mysite/index.html
```

</details>

---

**Exercise 4: Set up a shared project directory**

Create a shared directory `/srv/teamwork` for three users: alice, bob, carol. Requirements:
- All three can read and write files
- New files should automatically be accessible to the group
- Users should NOT be able to delete each other's files

```bash
# Create users and group, then set up the directory
```

<details>
<summary>Solution</summary>

```bash
# Create group and add users
sudo groupadd teamwork
sudo usermod -aG teamwork alice
sudo usermod -aG teamwork bob
sudo usermod -aG teamwork carol

# Create directory
sudo mkdir /srv/teamwork

# Set ownership
sudo chown root:teamwork /srv/teamwork

# SGID: new files inherit group
# Sticky: users can't delete each other's files
# 1775 = sticky bit + rwxrwxr-x
sudo chmod 1775 /srv/teamwork

# Verify
ls -ld /srv/teamwork
# drwxrwxr-t 2 root teamwork 4096 Mar 10 09:00 /srv/teamwork
```

</details>

---

### 15.3 Advanced Exercises

**Exercise 5: ACL scenario**

You have a directory `/srv/confidential` owned by `root:hr`. The HR team (group `hr`) has full access. Requirements:
- The auditor user `auditor` needs **read-only** access
- The contractor user `vendor` should have **no access at all** even though they're in the `hr` group

```bash
# Current state:
# drwxrwx--- 2 root hr 4096 Mar 10 09:00 /srv/confidential
# auditor is NOT in the hr group
# vendor IS in the hr group
```

<details>
<summary>Solution</summary>

```bash
# Give auditor read access via ACL (auditor isn't in hr)
sudo setfacl -m u:auditor:r-x /srv/confidential/
sudo setfacl -R -m u:auditor:r-- /srv/confidential/

# Deny vendor specifically via ACL (vendor is in hr but should have no access)
sudo setfacl -m u:vendor:--- /srv/confidential/

# Verify
getfacl /srv/confidential/
# user::rwx
# user:auditor:r-x    ← auditor can read+traverse
# user:vendor:---     ← vendor is denied despite being in hr
# group::rwx          ← hr group still has full access
# mask::rwx
# other::---
```

</details>

---

**Exercise 6: Security audit**

Write a script that:
1. Finds all world-writable files in `/var/www/`
2. Finds all SUID files on the entire system
3. Checks if `/etc/passwd` and `/etc/shadow` have correct permissions
4. Outputs a security report

<details>
<summary>Solution</summary>

```bash
#!/bin/bash
# security_audit.sh

echo "==============================="
echo " Linux Permission Security Audit"
echo " Date: $(date)"
echo "==============================="

echo ""
echo "[1] World-Writable Files in /var/www/"
echo "--------------------------------------"
find /var/www -perm /o+w -type f 2>/dev/null | while read f; do
    echo "  RISK: $f"
    ls -l "$f"
done

echo ""
echo "[2] SUID Files on System"
echo "-------------------------"
find / -perm /4000 -type f 2>/dev/null | while read f; do
    echo "  SUID: $f"
    ls -l "$f"
done

echo ""
echo "[3] Critical File Permissions"
echo "------------------------------"

check_perm() {
    local file=$1
    local expected=$2
    local actual
    actual=$(stat -c "%a" "$file" 2>/dev/null)
    if [ "$actual" = "$expected" ]; then
        echo "  OK  $file ($actual)"
    else
        echo "  WARN $file — expected $expected, got $actual"
    fi
}

check_perm /etc/passwd  644
check_perm /etc/shadow  640
check_perm /etc/sudoers 440
check_perm /etc/hosts   644
check_perm /tmp         1777

echo ""
echo "Audit complete."
```

```bash
chmod +x security_audit.sh
sudo ./security_audit.sh
```

</details>

---

**Exercise 7: umask calculation**

If the current umask is `027`, what permissions will these commands create?

```bash
touch newfile.txt
mkdir newdir/
```

And what umask would you set to create files with `640` and directories with `750`?

<details>
<summary>Solution</summary>

**With umask `027`:**

```
File:      666 - 027 → 640 → rw-r-----
Directory: 777 - 027 → 750 → rwxr-x---
```

**Desired: files=640, dirs=750**
```
Files:      666 - ? = 640   → umask must remove 027  → umask = 027
Dirs:       777 - 027 = 750 ✓

# So umask 027 already achieves this!
umask 027
```

</details>

---

## Congratulations! 🎉

You've completed the full Linux Permissions guide. Here's a summary of what you've mastered:

```
✅ Linux user types (root, normal, system), UID/GID
✅ File types and how to identify them
✅ The rwx permission model for user/group/others
✅ Symbolic and numeric (octal) permission notation
✅ chmod, chown, chgrp commands in depth
✅ Special permissions: SUID, SGID, Sticky Bit
✅ umask and default permission calculation
✅ Access Control Lists (ACL) for fine-grained control
✅ Real-world scenarios for web servers, DevOps, shared dirs
✅ Security best practices and common mistakes
✅ Troubleshooting permission denied errors
```

---

> **Next Steps:**
> - Practice daily: Run `ls -l` everywhere and understand what you see
> - Study SELinux and AppArmor for mandatory access control (MAC)
> - Explore `/etc/sudoers` for privileged command delegation
> - Learn about Linux capabilities (`setcap`, `getcap`) for fine-grained root privileges
> - Set up a test VM and practice all exercises hands-on

---

*Guide Version: 1.0 | Tested on Ubuntu 22.04, Debian 12, RHEL 9 | Last Updated: 2025*
