# 🐧 Linux Mastery for DevOps: From Zero to Production Hero

> **Role:** Senior Linux Kernel Engineer + DevOps Architect
> **Goal:** Teach you Linux from absolute first principles to advanced kernel tuning and production debugging.

---

## 📚 Table of Contents
1. [Phase 1: Linux Foundations (Internals)](#phase-1-linux-foundations-absolute-beginner)
2. [Phase 2: File & Directory Management](#phase-2-file--directory-management)
3. [Phase 3: Viewing & Processing Files](#phase-3-viewing--processing-files-critical)
4. [Phase 4: Permissions & Ownership](#phase-4-permissions--ownership-security-core)
5. [Phase 5: Search & Text Processing](#phase-5-search--text-processing-devops-superpower)
6. [Phase 6: Process & System Management](#phase-6-process--system-management)
7. [Phase 7: Networking](#phase-7-networking-production-critical)
8. [Phase 8: Package Management](#phase-8-package-management)
9. [Phase 9: Shell Scripting](#phase-9-shell-scripting-automation-core)
10. [Phase 10: Advanced Linux](#phase-10-advanced-linux-real-engineer-level)
11. [Phase 11: Linux for DevOps & Cloud](#phase-11-linux-for-devops--cloud)
12. [Phase 12: Linux Internals](#phase-12-linux-internals-advanced)

---

## Phase 1: Linux Foundations (Absolute Beginner)

### 1. What is Linux? (Internals Overview)

**Concept:** 
Linux is NOT just an Operating System (OS). strictly speaking, **Linux is the Kernel**. The "OS" you use (Ubuntu, CentOS, Debian) is a distribution that bundles the Kernel with user-space tools (like GNU utilities).

*   **Kernel:** The core brain. Talks directly to hardware (CPU, RAM, Disk).
*   **User Space:** Where you (the user) and your applications ("Apps") live. You cannot touch hardware directly; you must ask the Kernel.
*   **System Calls:** The protocol/language User Space uses to ask the Kernel to do things (e.g., "Open this file", "Send this packet").

**Why Linux Dominates DevOps:**
*   **Open Source:** Free and auditable.
*   **Stability:** Servers run for years without rebooting.
*   **Automation:** Everything is a text file or CLI command, making it easy to automate.

**Architecture Diagram:**

```ascii
      User (You)
         ↓
   Shell (bash/zsh)   <-- Command Line Interface
         ↓
    System Calls      <-- The API to the Kernel (e.g., open(), fork(), read())
         ↓
 ┌─────────────────┐
 │   Linux Kernel  │  <-- Process Mgmt, Memory Mgmt, Hardware Drivers
 └────────┬────────┘
          ↓
      Hardware        <-- CPU, RAM, Disk, Network Interface
```

### 2. Linux File System Deep Dive

**Concept:**
In Windows, you have drives (`C:\`, `D:\`). In Linux, **everything starts from Root (`/`)**. There are no drive letters. Drives are "mounted" to folders within this tree.

**"Everything is a file":**
In Linux, not just text documents are files.
*   A directory is a file.
*   Your mouse is a file (`/dev/input/mouse0`).
*   A running process is a directory of files (`/proc/PID`).

**The Hierarchy Standard (FHS) Diagram:**

```ascii
/ (Root)
├── bin     → Essential user binaries (ls, cp, cat)
├── sbin    → System binaries (reboot, ip) - mostly for root
├── etc     → Configuration files (nginx.conf, sshd_config)
├── var     → Variable data (logs, websites, caches)
│   ├── log   → System logs
│   └── www   → Web server files
├── home    → User home directories (/home/manish)
├── root    → Home directory for the Superuser (Root)
├── proc    → Virtual filesystem for Kernel/Process info (in RAM)
├── dev     → Device files (hard drives, terminals)
└── tmp     → Temporary files (cleared on reboot)
```

**Essential Navigation Commands:**

#### `pwd` (Print Working Directory)
*   **What:** Tells you exactly where you are in the directory tree.
*   **Why:** It's easy to get lost in deep folder structures.
*   **Usage:** `pwd`

#### `ls` (List)
*   **What:** Lists files in the current directory.
*   **Options:**
    *   `-l`: Long listing (shows permissions, owner, size, date). **ALWAYS USE THIS.**
    *   `-a`: All files (shows hidden files starting with `.`).
    *   `-h`: Human-readable sizes (KB, MB instead of bytes).
*   **Real-world:** `ls -lah` is the muscle memory command for every DevOps engineer.

#### `cd` (Change Directory)
*   **What:** Moves you to a different folder.
*   **Tricks:**
    *   `cd ..` → Go up one level.
    *   `cd ~` or just `cd` → Go to your home folder.
    *   `cd -` → Go back to the *previous* directory (Undo).

#### `tree`
*   **What:** Shows directory structure as a tree.
*   **Why:** Great for understanding project structures.

**Real-world Example:**
```bash
$ cd /var/log
$ pwd
/var/log
$ ls -lh syslog
-rw-r----- 1 syslog adm 2.5M Dec 18 10:00 syslog
```

---

## Phase 2: File & Directory Management

**Lifecycle of a File:**
```ascii
Create → Modify → Move/Rename → Backup (Copy) → Delete
```

### 1. `touch`
*   **What:** Creates an empty file OR updates the timestamp of an existing file.
*   **Why:** Quickly create placeholder files or trigger file-watcher events.
*   **Syntax:** `touch filename`
*   **Example:** `touch .gitignore`

### 2. `mkdir` (Make Directory)
*   **What:** Creates folders.
*   **Why:** Organizing project structures.
*   **Common Mistake:** Trying to create a nested folder without `-p`.
*   **Syntax:** `mkdir dirname`
*   **Pro Tip:** Use `-p` (parents) to create full paths.
    *   `mkdir -p app/src/components` (Creates app, then src, then components).

### 3. `cp` (Copy)
*   **What:** Copies files or directories.
*   **Syntax:** `cp source destination`
*   **Options:**
    *   `-r`: Recursive (Mandatory for directories).
    *   `-a`: Archive (Preserves permissions, timestamps, symbolic links). **CRITICAL FOR BACKUPS.**
*   **DevOps Use Case:** `cp -a /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak` (Backing up config before editing).

### 4. `mv` (Move / Rename)
*   **What:** Moves a file. In Linux, **Renaming is moving**.
*   **Syntax:** `mv source destination`
*   **Example:** `mv config.txt config.yaml` (Renaming).
*   **Example:** `mv build/app /var/www/html/` (Deploying).

### 5. `rm` (Remove)
*   **What:** Deletes files forever. **NO RECYCLE BIN.**
*   **Syntax:** `rm filename`
*   **Options:**
    *   `-r`: Recursive (for folders).
    *   `-f`: Force (no confirmation prompts).
*   **DANGER ZONE:** `rm -rf /` will destroy your OS. Always double-check your path.
*   **DevOps Use Case:** Cleaning up old artifacts. `rm -rf ./dist/`

### 6. `stat`
*   **What:** Detailed low-level file info.
*   **Why:** Debugging timestamps (Access vs Modify vs Change).
*   **Example:** `stat index.js`

---

## Phase 3: Viewing & Processing Files (CRITICAL)

**The "Log File" Workflow:**
```ascii
Log File Generates Data
       ↓
    tail -f           <-- Watch it live
       ↓
     grep             <-- Filter for errors
       ↓
    less              <-- Analyze history safely
```

### 1. `cat` (Concatenate)
*   **What:** Dumps the *entire* file content to the screen.
*   **Why:** Quick view of small files.
*   **Mistake:** Running `cat` on a 10GB log file. It will freeze your terminal.
*   **Use:** `cat config.json`

### 2. `less`
*   **What:** A pager. Opens file allowing scrolling (Up/Down). Does NOT load the whole file into RAM.
*   **Why:** Viewing massive logs safely.
*   **Navigation:**
    *   `Space`: Page down.
    *   `b`: Page up.
    *   `/search_term`: Search inside file.
    *   `q`: Quit.

### 3. `head` & `tail`
*   **What:** View the beginning (`head`) or end (`tail`) of a file.
*   **Why:** Checking CSV headers or seeing the latest log entry.
*   **Real-world Magic:** `tail -f` (Follow).
    *   Keeps the stream open and prints new lines as they are written.
    *   **DevOps Use:** `tail -f /var/log/nginx/access.log` (Watch live traffic).

### 4. `wc` (Word Count)
*   **What:** Counts lines, words, and characters.
*   **Why:** verifying data integrity or counting occurrences.
*   **Flag:** `-l` (Lines only).
*   **Example:** `cat access.log | grep "404" | wc -l` (Count how many 404 errors occurred).

---

## Phase 4: Permissions & Ownership (SECURITY CORE)

**The Security Model:**
Linux is a multi-user system. Every file belongs to a User and a Group, and has permissions for the Owner, the Group, and Everyone Else.

**Permission Structure Diagram:**

```ascii
 -rwxr-x---
 ││││││││││
 │││││││││└── Others (World) - Can they Read/Write/Execute?
 ││││││└─── Group - Can members of the group Read/Write/Execute?
 │││└─── Owner - Can the owner Read/Write/Execute?
 │└──── file type (- for file, d for directory)
```

**Understanding "rwx":**
*   **r (Read - 4):** View file contents / List directory contents.
*   **w (Write - 2):** Modify file / Create or Delete files in a directory.
*   **x (Execute - 1):** Run file as a program / Enter (cd) a directory.

### 1. `chmod` (Change Mode)
*   **What:** Changes the permissions of a file.
*   **Why:** Securing secrets or making scripts executable.
*   **Symbolic Mode:** `chmod u+x script.sh` (Give **U**ser e**X**ecute).
*   **Numeric Mode (The DevOps Standard):**
    *   **7 (4+2+1):** Read + Write + Execute.
    *   **6 (4+2):** Read + Write.
    *   **4:** Read only.
    *   **0:** No permissions.
*   **Common Patterns:**
    *   `755` (Owner: All, Group: Read/Exec, Others: Read/Exec) ← Standard for executables/dirs.
    *   `644` (Owner: RW, Group: R, Others: R) ← Standard for text files.
    *   `600` (Owner: RW, Others: None) ← **CRITICAL FOR SSH KEYS & SECRETS.**
    *   `400` (Owner: Read Only) ← Extra secure read-only keys.

### 2. `chown` (Change Owner)
*   **What:** Changes who owns the file.
*   **Syntax:** `chown user:group file`
*   **Recursive:** `chown -R user:group directory/`
*   **DevOps Use Case:** Nginx runs as `www-data`. If you upload a file as `root`, Nginx can't read it. Fix: `chown -R www-data:www-data /var/www/html`.

### 3. `umask`
*   **What:** Sets default permissions for *new* files.
*   **Why:** Security baseline.

---

## Phase 5: Search & Text Processing (DevOps SUPERPOWER)

**The "Pipeline" Philosophy:**
Join small tools together to do big things.

**Pipeline Diagram:**
```ascii
Input Data → Filter (grep) → Transform (awk) → Sort/Unique → Final Output
```

### 1. `grep` (Global Regular Expression Print)
*   **What:** Finds text inside files. The most used tool in DevOps.
*   **Syntax:** `grep "search_term" file`
*   **Options:**
    *   `-r`: Recursive (search in folders).
    *   `-i`: Case insensitive.
    *   `-v`: Invert match (show lines that do **NOT** match).
    *   `-E`: Extended Regex (allows fancy patterns).
*   **DevOps Example:**
    *   `grep -r "TODO" ./src` (Find all TODOs in source code).
    *   `grep "ERROR" /var/log/syslog` (Find errors).

### 2. `find`
*   **What:** Finds *files* based on metadata (name, size, time).
*   **Syntax:** `find [where] [criteria] [action]`
*   **Examples:**
    *   `find . -name "*.log"` (Find all log files).
    *   `find /var/log -mtime +7` (Find logs older than 7 days).
    *   `find . -size +100M` (Find files larger than 100MB).
*   **Production Tip:** combine with `-exec` or `xargs` to delete old logs.

### 3. `awk`
*   **What:** A programming language for text processing. Best for **Columns**.
*   **Concept:** Breaks lines into fields (`$1`, `$2`, `$3`...).
*   **Example:** `ls -l | awk '{print $1, $9}'` (Print permissions and filename).
*   **DevOps Example:** Extracting IP addresses from logs.

### 4. `sed` (Stream Editor)
*   **What:** search and replace text in a stream.
*   **Syntax:** `sed 's/find/replace/g' filename`
*   **DevOps Use Case:** Replacing version numbers in config files. `sed -i 's/v1.0/v1.1/g' config.yaml` (The `-i` edits the file in-place).

### 5. `sort` & `uniq`
*   **What:** Sort lines and remove duplicates.
*   **Combo:** `sort | uniq -c` (Count occurrences).
*   **Real World:** Count hits per IP in access log.
    *   `cat access.log | awk '{print $1}' | sort | uniq -c | sort -nr`

---

## Phase 6: Process & System Management

**Process Model Diagram:**
```ascii
User Request
    ↓
 Application (e.g., Nginx)
    ↓
  Process (PID: 1234)
    ↓
 System Resources (Allocated by Kernel)
  ├─ CPU Cycles
  └─ RAM Pages
```

### 1. `ps` (Process Status)
*   **What:** Snapshot of running processes.
*   **Command:** `ps aux`
    *   `a`: All users.
    *   `u`: User friendly format.
    *   `x`: Processes without terminal (background daemons).
*   **Why:** "Is my server running?"

### 2. `top` / `htop`
*   **What:** Live task manager.
*   **Why:** Troubleshooting high CPU/RAM usage.
*   **Look For:**
    *   **Load Average:** (1min, 5min, 15min). If load > number of CPU cores, system is overloaded.
    *   **Zombies:** Dead processes waiting for parent to acknowledge them.

### 3. `kill`
*   **What:** Sends a signal to a process.
*   **Syntax:** `kill [PID]` (Sends SIGTERM - polite request to stop).
*   **The Nuclear Option:** `kill -9 [PID]` (SIGKILL - access revoked immediately, no cleanup allowed). Use only if frozen.
*   **Mistake:** Killing random high-CPU processes without knowing what they are.

### 4. `df` (Disk Free)
*   **What:** Shows disk space usage.
*   **Flag:** `-h` (Human readable).
*   **Why:** "No space left on device" errors.
*   **Output:** Shows Usage% per mount point.

### 5. `du` (Disk Usage)
*   **What:** Shows size of files/folders.
*   **Usage:** `du -sh ./folder` (Summary, Human-readable).
*   **Why:** Finding *what* is using all the space when `df` says full.

### 6. `free`
*   **What:** Shows RAM usage.
*   **Flag:** `-h`.
*   **Concept:** "Buff/cache" is RAM used to speed up disk access. It's "available" if apps need it. Don't panic if "Free" is low, check "Available".

### 7. `uptime`
*   **What:** How long system has been running + Load Averages.
*   **Why:** Quick health check.

---

## Phase 7: Networking (Production Critical)

**Flow Diagram:**
```ascii
           Request (Port 80/443)
Client ───────────────────────────► Load Balancer
                                        │
                                        ▼
                                   Web Server (Nginx)
                                        │
                                        ▼
                                    App Server (Node/Python)
                                        │
                                        ▼
                                     Database (Port 5432)
```

### 1. `ip` (The new ifconfig)
*   **What:** IP configuration. Old `ifconfig` is deprecated.
*   **Commands:**
    *   `ip a` (addr): Show IP addresses.
    *   `ip r` (route): Show routing table (Gateway).
*   **Why:** "What is my IP?"

### 2. `ping`
*   **What:** Checks connectivity.
*   **Command:** `ping google.com`
*   **Why:** "Is the internet working?" or "Can I reach the DB?"

### 3. `curl` & `wget`
*   **What:** Transfer data from URLs.
*   **`curl`:** Great for testing APIs.
    *   `curl -v https://api.example.com` (Verbose, shows headers).
    *   `curl -X POST -d '{"json":true}' http://localhost:3000`
*   **`wget`:** Great for downloading files.
    *   `wget https://example.com/file.zip`

### 4. `netstat` / `ss`
*   **What:** Network statistics (Open ports). `ss` is the modern replacement.
*   **Command:** `ss -tulpn`
    *   `t`: TCP
    *   `u`: UDP
    *   `l`: Listening sockets
    *   `p`: Process name/PID
    *   `n`: Numeric ports
*   **DevOps Use:** "Why is my app not reachable?" -> Check if it's listening on the port.

### 5. `traceroute`
*   **What:** Traces the path packets take to a host.
*   **Why:** Diagnosing latency or routing issues.

---

## Phase 8: Package Management

**Concept:**
Linux apps come from **Repositories**. You don't google ".exe" installers.

**Flow:**
```ascii
Official Repo (Ubuntu) ──► Package Manager (apt) ──► Your System
```

### 1. `apt` (Debian/Ubuntu)
*   **`apt update`**: Refreshes the list of available packages (Does NOT install).
*   **`apt upgrade`**: Actually installs new versions of packages.
*   **`apt install [package]`**: Installs software.
    *   `apt install nginx`
*   **`apt remove [package]`**: Removes software.

### 2. `yum` / `dnf` (RHEL/CentOS)
*   **Same concepts, different commands.**
*   `yum install nginx`

**Production Tip:**
Always run `apt update` before installing to avoid "Package not found" errors.

---

## Phase 9: Shell Scripting (AUTOMATION CORE)

**Why Script?**
If you have to do it twice, automate it.

### Basics

1.  **Shebang (`#!/bin/bash`):** Tells Linux "This is a bash script".
2.  **Variables:** `NAME="Manish"` (No spaces around `=`).
3.  **Use Variables:** `echo $NAME`

### Example: Production Health Check Script

```bash
#!/bin/bash

# Define key variables
LOG_FILE="/var/log/syslog"
SEARCH_TERM="ERROR"

echo "Starting Health Check..."

# Check if log file exists
if [ -f "$LOG_FILE" ]; then
    echo "Log file found."
    
    # Count errors
    ERROR_COUNT=$(grep -c "$SEARCH_TERM" "$LOG_FILE")
    
    if [ "$ERROR_COUNT" -gt 0 ]; then
        echo "Alert! Found $ERROR_COUNT errors."
    else
        echo "System Healthy. No errors."
    fi
else
    echo "Critical: Log file missing!"
    exit 1
fi
```

### Loops
**For Loop (Batch Processing):**
```bash
for file in *.log; do
    echo "Archiving $file..."
    mv "$file" "$file.bak"
done
```

---

## Phase 10: Advanced Linux (REAL ENGINEER LEVEL)

### 1. Systemd & Services
*   **What:** `systemd` is the Init system (PID 1). It starts everything else.
*   **`systemctl` Command:**
    *   `systemctl start nginx`: Start a service.
    *   `systemctl enable nginx`: Start on boot (Auto-start).
    *   `systemctl status nginx`: Check if running.
    *   `systemctl restart nginx`: Restart.
*   **Logs:** `journalctl -u nginx` (View specific service logs).

### 2. Cron Jobs (Scheduling)
*   **What:** Run scripts at specific times (like Windows Task Scheduler).
*   **Syntax:** `crontab -e`
*   **Format:**
    ```
    # m h  dom mon dow   command
      0 5   *   *   *    /home/user/backup.sh
    ```
    (Runs at 5:00 AM every day).

### 3. Environment Variables
*   **What:** Secrets or configs passed to apps without hardcoding.
*   **Command:** `export DB_PASS="secret"`
*   **View:** `env` or `printenv`.

### 4. File Descriptors (STDIN, STDOUT, STDERR)
*   **0:** Standard Input (Keyboard).
*   **1:** Standard Output (Screen).
*   **2:** Standard Error (Error logs).
*   **Redirection:**
    *   `> file`: Keep Output (Overwrite).
    *   `>> file`: Append Output.
    *   `2> error.log`: Redirect Errors only.
    *   `&> all.log`: Redirect Both Output and Errors.

---

## Phase 11: Linux for DevOps & Cloud

**The Modern Flow:**
```ascii
Developer → Git Push → CI/CD Pipeline → Build Docker Image → Deploy to Server
```

### 1. Linux + Docker
*   Docker **is** Linux. It uses **Namespaces** (isolation) and **Cgroups** (resource limits) from the Kernel.
*   A "Container" is just a process with a mask on.

### 2. SSH Hardening (Critical Security)
**Never leave default settings.**
1.  **Disable Root Login:** Edit `/etc/ssh/sshd_config` → `PermitRootLogin no`.
2.  **Disable Password Auth:** Use Key-based auth only (`PasswordAuthentication no`).
3.  **Restart SSH:** `systemctl restart sshd`.

---

## Phase 12: Linux Internals (ADVANCED)

**How It Actually Works:**

```ascii
      Application (Node.js)
              │
      malloc() / read()   <-- System Calls
              │
        Kernel Space
    ┌───────────────────┐
    │ Virtual Memory    │ <-- Maps RAM addresses
    │ Process Scheduler │ <-- Decides who runs on CPU
    │ File System       │ <-- Talks to Disk
    └─────────┬─────────┘
              │
          Hardware
```

1.  **Syscalls:** The bridge between User apps and Kernel. `strace` is the tool to watch this.
    *   `strace -p [PID]`: Watch every call a process makes (Debugging magic).
2.  **Virtual Memory:** Apps think they have continuous RAM. Kernel maps this to fragmented physical RAM blocks (Pages).
3.  **Load Average:** Technically, it's the number of processes in the "Runnable" or "Uninterruptible Sleep" (Waiting for Disk I/O) state.

---

# 🚀 Conclusion

You have graduated from "I use Windows" to **Linux Engineer**.
*   **Practice:** Don't memorize. DO.
*   **Break things:** You learn most when you fix a broken server.
*   **Next Steps:** Master Docker Networking and Bash Scripting in depth.

**End of Guide.**
