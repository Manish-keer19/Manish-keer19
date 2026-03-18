# 🔐 SSH & SCP Complete Guide for DevOps Engineers

> **A beginner-friendly yet professional reference for mastering SSH and SCP in real-world DevOps workflows.**

---

## 📋 Table of Contents

1. [Introduction to SSH and OpenSSH](#1-introduction-to-ssh-and-openssh)
2. [Installing and Enabling SSH Server on Ubuntu](#2-installing-and-enabling-ssh-server-on-ubuntu)
3. [Connecting to a Remote Server Using SSH](#3-connecting-to-a-remote-server-using-ssh)
4. [SSH Key-Based Authentication](#4-ssh-key-based-authentication)
5. [Step-by-Step: Generate SSH Keys](#5-step-by-step-generate-ssh-keys)
6. [Copy Public Key to Server](#6-copy-public-key-to-server)
7. [File Permissions Required for SSH](#7-file-permissions-required-for-ssh)
8. [Disabling Password Authentication Securely](#8-disabling-password-authentication-securely)
9. [The `ssh -i` Option Explained](#9-the-ssh--i-option-explained)
10. [SCP Command Usage](#10-scp-command-usage)
11. [Common Errors and Fixes](#11-common-errors-and-fixes)
12. [Best Practices for SSH Security](#12-best-practices-for-ssh-security)
13. [Real-World CI/CD Usage with SSH](#13-real-world-cicd-usage-with-ssh)

---

## 1. Introduction to SSH and OpenSSH

### What is SSH?

**SSH (Secure Shell)** is a cryptographic network protocol used to securely access and manage remote systems over an unsecured network (like the internet). It replaces older, insecure protocols like Telnet and rlogin.

SSH provides:
- **Encrypted communication** — all data is encrypted in transit
- **Authentication** — verifies the identity of users and servers
- **Integrity** — ensures data hasn't been tampered with

```
[Your Machine] ──── Encrypted SSH Tunnel ────▶ [Remote Server]
     Client                                        Server (sshd)
```

### What is OpenSSH?

**OpenSSH** is the most widely used open-source implementation of the SSH protocol. It comes pre-installed on most Linux distributions and macOS.

OpenSSH includes:

| Tool | Purpose |
|------|---------|
| `ssh` | SSH client — connect to remote servers |
| `sshd` | SSH daemon — the server-side service |
| `ssh-keygen` | Generate SSH key pairs |
| `ssh-copy-id` | Copy public keys to remote servers |
| `ssh-agent` | Key management agent |
| `scp` | Secure file copy over SSH |
| `sftp` | Secure FTP over SSH |

### SSH Protocol Versions

| Version | Status | Notes |
|---------|--------|-------|
| SSH-1 | ❌ Deprecated | Vulnerable — never use |
| SSH-2 | ✅ Current | Secure, use always |

> **DevOps Note:** In production environments, SSH is the backbone of remote server management, deployment pipelines, container orchestration (Kubernetes nodes), and secure inter-service communication.

---

## 2. Installing and Enabling SSH Server on Ubuntu

### Check if SSH is Already Installed

```bash
# Check if OpenSSH server is installed
dpkg -l | grep openssh-server

# Check if SSH service is running
systemctl status ssh
```

### Install OpenSSH Server

```bash
# Update package lists
sudo apt update

# Install OpenSSH server
sudo apt install openssh-server -y
```

### Enable and Start the SSH Service

```bash
# Start SSH service immediately
sudo systemctl start ssh

# Enable SSH to start automatically on boot
sudo systemctl enable ssh

# Verify the service is active
sudo systemctl status ssh
```

**Expected output:**
```
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2025-01-01 10:00:00 UTC; 5min ago
```

### Check SSH is Listening on Port 22

```bash
# Check which port SSH is listening on
sudo ss -tlnp | grep sshd

# Alternative using netstat
sudo netstat -tlnp | grep :22
```

### Allow SSH Through UFW Firewall

```bash
# Allow SSH connections
sudo ufw allow ssh

# Or allow specific port
sudo ufw allow 22/tcp

# Enable UFW if not already active
sudo ufw enable

# Check firewall status
sudo ufw status
```

### Configuration File Location

```bash
# Main SSH server config file
/etc/ssh/sshd_config

# Always backup before editing
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

# Reload SSH after config changes (no disconnection)
sudo systemctl reload ssh

# Or restart SSH (existing sessions stay, new config applies)
sudo systemctl restart ssh
```

> ⚠️ **Warning:** Always test SSH config before reloading: `sudo sshd -t`  
> A syntax error in `sshd_config` can lock you out of the server!

---

## 3. Connecting to a Remote Server Using SSH

### Basic SSH Syntax

```bash
ssh [OPTIONS] user@hostname_or_ip
```

### Common Connection Examples

```bash
# Connect with username and IP
ssh john@192.168.1.100

# Connect with username and domain
ssh john@myserver.example.com

# Connect to a non-standard port (not 22)
ssh -p 2222 john@192.168.1.100

# Connect with verbose output (useful for debugging)
ssh -v john@192.168.1.100

# Even more verbose (level 2 and 3)
ssh -vv john@192.168.1.100
ssh -vvv john@192.168.1.100
```

### SSH Config File (Shortcut for Frequent Connections)

Instead of typing long SSH commands, use the SSH config file:

```bash
# Edit or create the SSH config file
nano ~/.ssh/config
```

Add a host entry:

```
# ~/.ssh/config

Host myserver
    HostName 192.168.1.100
    User john
    Port 22
    IdentityFile ~/.ssh/id_rsa

Host prod-server
    HostName prod.example.com
    User deploy
    Port 2222
    IdentityFile ~/.ssh/prod_key
```

Now connect simply with:

```bash
ssh myserver
ssh prod-server
```

### First-Time Connection: Host Key Verification

When connecting to a server for the first time, SSH shows a fingerprint warning:

```
The authenticity of host '192.168.1.100 (192.168.1.100)' can't be established.
ED25519 key fingerprint is SHA256:abc123def456...
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Type `yes` to add the server to `~/.ssh/known_hosts`. This prevents **man-in-the-middle attacks** — if the fingerprint changes unexpectedly, SSH will warn you.

---

## 4. SSH Key-Based Authentication

### Overview

Instead of using a password, SSH key-based authentication uses a **cryptographic key pair**:

```
┌─────────────────────┐         ┌──────────────────────┐
│      YOUR PC        │         │    REMOTE SERVER      │
│                     │         │                       │
│  🔑 Private Key     │◄───────▶│  🔓 Public Key        │
│  (Keep SECRET!)     │  Trust  │  (Safe to share)      │
│  ~/.ssh/id_rsa      │         │  ~/.ssh/authorized_   │
│                     │         │  keys                 │
└─────────────────────┘         └──────────────────────┘
```

### Private Key vs Public Key

| Property | Private Key | Public Key |
|----------|------------|------------|
| **File** | `~/.ssh/id_rsa` | `~/.ssh/id_rsa.pub` |
| **Location** | Your machine only | Remote server |
| **Shareable** | ❌ Never share | ✅ Safe to share |
| **Purpose** | Proves your identity | Stored on server to verify you |
| **If lost** | Security breach — revoke immediately | No security risk |
| **Passphrase** | Should be protected with one | Not applicable |

### How SSH Key Authentication Works Internally

Here is the step-by-step cryptographic handshake:

```
Step 1: Client sends connection request
        [Your Machine] ──── "I want to connect as john" ────▶ [Server]

Step 2: Server generates a random challenge
        [Server] generates a random number (challenge)
        Encrypts it with your PUBLIC KEY
        [Server] ──── "Decrypt this to prove you're john" ────▶ [Your Machine]

Step 3: Client decrypts the challenge using PRIVATE KEY
        Only you can decrypt it (because only you have the private key)
        [Your Machine] ──── "Here's the decrypted challenge" ────▶ [Server]

Step 4: Server verifies and grants access
        [Server] checks: "Yes, that matches — access granted!" ✅
```

This means:
- **Your password never travels over the network**
- Even if someone intercepts the traffic, they cannot authenticate
- This is why key-based auth is far more secure than passwords

### Supported Key Algorithms

| Algorithm | Key Size | Security | Recommended Use |
|-----------|----------|----------|----------------|
| `ed25519` | 256-bit (fixed) | ⭐⭐⭐⭐⭐ Best | Modern default — use this |
| `ecdsa` | 256/384/521-bit | ⭐⭐⭐⭐ | Good, slightly older |
| `rsa` | 2048-4096-bit | ⭐⭐⭐ | Legacy systems only |
| `dsa` | 1024-bit | ❌ Broken | Never use |

---

## 5. Step-by-Step: Generate SSH Keys

### Generate an Ed25519 Key (Recommended)

```bash
# Generate Ed25519 key pair (modern, secure, fast)
ssh-keygen -t ed25519 -C "your_email@example.com"
```

### Generate an RSA Key (Legacy Systems)

```bash
# Generate 4096-bit RSA key (for older servers)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

### Interactive Prompts Explained

```bash
$ ssh-keygen -t ed25519 -C "john@example.com"

Generating public/private ed25519 key pair.

# Where to save — press Enter for default
Enter file in which to save the key (/home/john/.ssh/id_ed25519): 
#   ↑ Press Enter (or type a custom path like /home/john/.ssh/myserver_key)

# Add a passphrase — HIGHLY RECOMMENDED for security
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
#   ↑ This passphrase protects your private key if someone steals your machine

Your identification has been saved in /home/john/.ssh/id_ed25519
Your public key has been saved in /home/john/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:AbCdEf1234567890... john@example.com
The key's randomart image is:
+--[ED25519 256]--+
|       o+.       |
|      . +o       |
|       o. .      |
+----[SHA256]-----+
```

### View Your Generated Keys

```bash
# List SSH key files
ls -la ~/.ssh/

# View your PUBLIC key (safe to copy and share)
cat ~/.ssh/id_ed25519.pub
# Output: ssh-ed25519 AAAAC3NzaC1lZD... john@example.com

# Never print your private key, but view its permissions
ls -la ~/.ssh/id_ed25519
# Should show: -rw------- (600 permissions)
```

### Generate a Named Key (Multiple Servers)

```bash
# Generate a key specifically for a production server
ssh-keygen -t ed25519 -f ~/.ssh/prod_server_key -C "prod-server-deploy"

# This creates:
# ~/.ssh/prod_server_key       ← private key
# ~/.ssh/prod_server_key.pub   ← public key
```

---

## 6. Copy Public Key to Server

### Method 1: Using `ssh-copy-id` (Easiest)

```bash
# Basic syntax
ssh-copy-id user@remote_server

# Example
ssh-copy-id john@192.168.1.100

# With a specific key file
ssh-copy-id -i ~/.ssh/id_ed25519.pub john@192.168.1.100

# With a non-standard port
ssh-copy-id -p 2222 john@192.168.1.100
```

`ssh-copy-id` automatically:
1. Reads your public key
2. Connects to the remote server via password
3. Appends the key to `~/.ssh/authorized_keys` on the server
4. Sets correct permissions

### Method 2: Manual Copy (When ssh-copy-id is Unavailable)

```bash
# Step 1: Display your public key on local machine
cat ~/.ssh/id_ed25519.pub
# Copy the entire output

# Step 2: Log into the remote server
ssh john@192.168.1.100

# Step 3: On the remote server, create the .ssh directory
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Step 4: Add your public key to authorized_keys
echo "ssh-ed25519 AAAAC3NzaC1lZD... john@example.com" >> ~/.ssh/authorized_keys

# Step 5: Set correct permissions
chmod 600 ~/.ssh/authorized_keys

# Step 6: Exit and test
exit
ssh john@192.168.1.100   # Should not ask for password
```

### Method 3: Using Pipe (One-Liner)

```bash
# Pipe the public key directly to the server
cat ~/.ssh/id_ed25519.pub | ssh john@192.168.1.100 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

### Verify the Key Was Added

```bash
# On the remote server
cat ~/.ssh/authorized_keys

# You should see your public key listed
# ssh-ed25519 AAAAC3NzaC1lZD... john@example.com
```

---

## 7. File Permissions Required for SSH

SSH is **very strict about file permissions**. Wrong permissions will cause authentication to silently fail.

### Required Permissions Table

| File/Directory | Permission | Numeric | Reason |
|---------------|-----------|---------|--------|
| `~/.ssh/` | `drwx------` | `700` | Only owner can read/write/execute |
| `~/.ssh/authorized_keys` | `-rw-------` | `600` | Only owner can read/write |
| `~/.ssh/id_rsa` (private) | `-rw-------` | `600` | Only owner can read/write |
| `~/.ssh/id_ed25519` (private) | `-rw-------` | `600` | Only owner can read/write |
| `~/.ssh/id_rsa.pub` (public) | `-rw-r--r--` | `644` | Others can read (it's public) |
| `~/.ssh/config` | `-rw-------` | `600` | Only owner can read/write |
| Home directory `~/` | `drwxr-xr-x` | `755` | Group/others must NOT have write access |

### Fix Permissions Quickly

```bash
# Fix all SSH permissions at once
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_rsa
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_rsa.pub
chmod 644 ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/config

# Fix home directory permissions
chmod 755 ~
```

### Check Current Permissions

```bash
# View all SSH files with permissions
ls -la ~/.ssh/

# Example correct output:
# drwx------  2 john john 4096 Jan  1 10:00 .
# -rw-------  1 john john  573 Jan  1 10:00 authorized_keys
# -rw-------  1 john john  411 Jan  1 10:00 id_ed25519
# -rw-r--r--  1 john john   98 Jan  1 10:00 id_ed25519.pub
# -rw-------  1 john john  147 Jan  1 10:00 config
```

> **Why does SSH care about permissions?**  
> If your `authorized_keys` file is writable by others, an attacker could add their key and gain access. SSH refuses to use keys from insecure locations to prevent this.

---

## 8. Disabling Password Authentication Securely

Once SSH key-based authentication is working, disable password login to eliminate brute-force attacks entirely.

### ⚠️ Pre-requisites Before Disabling Passwords

```bash
# 1. Make sure you can log in with your SSH key FIRST
ssh john@192.168.1.100
# If this works without a password prompt — you're ready

# 2. Keep a backup session open while editing
# Open a second terminal connected to the server before making changes
```

### Edit the SSH Daemon Configuration

```bash
# Open the SSH config file
sudo nano /etc/ssh/sshd_config
```

Find and change these settings:

```bash
# /etc/ssh/sshd_config

# Disable password authentication
PasswordAuthentication no

# Disable empty passwords
PermitEmptyPasswords no

# Disable challenge-response (PAM-based passwords)
ChallengeResponseAuthentication no

# Optionally disable root login entirely (recommended)
PermitRootLogin no

# Or allow root login with key only (if root access needed)
# PermitRootLogin prohibit-password

# Keep public key auth enabled
PubkeyAuthentication yes

# Set authorized keys file location
AuthorizedKeysFile .ssh/authorized_keys
```

### Test Config and Reload

```bash
# Test config for syntax errors BEFORE reloading
sudo sshd -t

# If no errors, reload SSH daemon
sudo systemctl reload ssh

# Verify the changes
sudo grep -E "PasswordAuthentication|PermitRootLogin|PubkeyAuthentication" /etc/ssh/sshd_config
```

### Test That Password Auth is Disabled

```bash
# From another terminal, try to connect with password
ssh -o PreferredAuthentications=password john@192.168.1.100

# Expected result:
# Permission denied (publickey)
```

> **Recovery Plan:** If you're locked out, access the server via cloud provider console (AWS EC2 console, DigitalOcean recovery console, etc.) to re-enable password auth temporarily.

---

## 9. The `ssh -i` Option Explained

### What Does `-i` Mean?

The `-i` flag stands for **identity file** — it tells SSH which specific private key file to use for authentication.

```bash
ssh -i /path/to/private_key user@server
```

### When Do You Need `-i`?

By default, SSH automatically tries these keys in order:
- `~/.ssh/id_ed25519`
- `~/.ssh/id_ecdsa`
- `~/.ssh/id_rsa`

You need `-i` when:
- Your key has a **custom name** (not the default)
- You have **multiple keys** for different servers
- The key is stored in a **non-default location** (e.g., CI/CD pipelines, downloaded `.pem` files)

### Examples

```bash
# Use a specific named key
ssh -i ~/.ssh/prod_server_key john@192.168.1.100

# Use an AWS PEM file
ssh -i ~/Downloads/aws-key.pem ubuntu@ec2-54-123-45-67.compute-1.amazonaws.com

# Combine with port and other options
ssh -i ~/.ssh/deploy_key -p 2222 deploy@staging.example.com

# Use with SCP
scp -i ~/.ssh/prod_key file.txt john@192.168.1.100:/home/john/

# Use with a relative path
ssh -i ./server_key deploy@myserver.com
```

### Fix PEM File Permissions (AWS)

```bash
# AWS PEM keys must have 400 or 600 permissions
chmod 400 ~/Downloads/aws-key.pem

# Or
chmod 600 ~/Downloads/aws-key.pem
```

### Add Key to ssh-agent (Avoid Repeating -i)

```bash
# Start the SSH agent
eval "$(ssh-agent -s)"

# Add your key (prompts for passphrase if set)
ssh-add ~/.ssh/prod_server_key

# List loaded keys
ssh-add -l

# Now connect without -i
ssh john@192.168.1.100

# Remove all keys from agent
ssh-add -D
```

---

## 10. SCP Command Usage

**SCP (Secure Copy Protocol)** uses SSH to securely transfer files between machines. Think of it as `cp` but over a network with encryption.

### Basic SCP Syntax

```bash
scp [OPTIONS] source destination
```

Remote paths are written as: `user@host:/path/to/file`

### 10.1 Copy File — Local to Remote

```bash
# Basic file upload
scp localfile.txt john@192.168.1.100:/home/john/

# Upload to a specific remote path and rename
scp report.pdf john@192.168.1.100:/var/www/html/docs/report_2025.pdf

# Upload with non-standard SSH port
scp -P 2222 config.yml deploy@myserver.com:/etc/app/

# Upload multiple files
scp file1.txt file2.txt john@192.168.1.100:/home/john/uploads/

# Upload with verbose output
scp -v app.zip john@192.168.1.100:/tmp/
```

> **Note:** SCP uses `-P` (uppercase) for port, while SSH uses `-p` (lowercase).

### 10.2 Copy Folder Recursively Using `-r`

```bash
# Copy entire folder to remote server
scp -r ./my_project john@192.168.1.100:/home/john/

# Copy a folder with specific port
scp -r -P 2222 ./dist deploy@myserver.com:/var/www/html/

# Copy and preserve timestamps/permissions
scp -r -p ./configs john@192.168.1.100:/etc/app/

# Upload a build folder in CI/CD context
scp -r ./build ubuntu@ec2-server.compute.amazonaws.com:/var/www/myapp/
```

### 10.3 Copy From Remote to Local (Download)

```bash
# Download a file from remote to current directory
scp john@192.168.1.100:/var/log/app.log ./

# Download with specific local name
scp john@192.168.1.100:/home/john/data.csv ~/Downloads/server_data.csv

# Download entire folder from remote
scp -r john@192.168.1.100:/var/www/html/backup ./local_backup/

# Download from non-standard port
scp -P 2222 john@myserver.com:/etc/nginx/nginx.conf ./nginx_backup.conf
```

### 10.4 Using SCP With SSH Key (`-i`)

```bash
# Download using a specific SSH key
scp -i ~/.ssh/prod_key john@192.168.1.100:/var/log/app.log ./

# Upload using AWS PEM key
scp -i ~/aws-key.pem ./deploy.zip ubuntu@ec2-54-123.compute-1.amazonaws.com:/home/ubuntu/

# Upload folder using key
scp -i ~/.ssh/deploy_key -r ./dist deploy@192.168.1.100:/var/www/app/

# Upload with key and non-default port
scp -i ~/.ssh/mykey -P 2222 file.txt user@server.com:/tmp/
```

### SCP Options Reference

| Option | Description | Example |
|--------|------------|---------|
| `-r` | Recursive (copy folders) | `scp -r ./folder user@host:/path` |
| `-P` | Specify port (uppercase!) | `scp -P 2222 file user@host:/path` |
| `-i` | Identity file (SSH key) | `scp -i ~/.ssh/key file user@host:/path` |
| `-p` | Preserve timestamps/perms | `scp -p file user@host:/path` |
| `-v` | Verbose output | `scp -v file user@host:/path` |
| `-C` | Compress during transfer | `scp -C large_file user@host:/path` |
| `-l` | Limit bandwidth (Kbps) | `scp -l 1000 file user@host:/path` |
| `-q` | Quiet mode (no progress) | `scp -q file user@host:/path` |

> **Modern Alternative:** For production use, consider `rsync` over SSH — it's faster for large transfers because it only copies changed files:
> ```bash
> rsync -avz -e "ssh -i ~/.ssh/mykey" ./dist/ user@server:/var/www/app/
> ```

---

## 11. Common Errors and Fixes

### Error 1: Permission Denied (publickey)

```
john@192.168.1.100: Permission denied (publickey)
```

**Causes and Fixes:**

```bash
# Fix 1: Check if public key is in authorized_keys on server
ssh john@192.168.1.100 cat ~/.ssh/authorized_keys

# Fix 2: Fix wrong permissions on server
ssh john@192.168.1.100
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# Fix 3: Use correct key file explicitly
ssh -i ~/.ssh/correct_key john@192.168.1.100

# Fix 4: Check if SSH is using the right key
ssh -vvv john@192.168.1.100
# Look for "Offering public key:" lines in verbose output

# Fix 5: Confirm PasswordAuthentication is not set to "no" prematurely
# Check /etc/ssh/sshd_config on the server
```

### Error 2: WARNING: UNPROTECTED PRIVATE KEY FILE!

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for '/home/john/.ssh/id_rsa' are too open.
```

**Fix:**

```bash
# Fix private key permissions
chmod 600 ~/.ssh/id_rsa
chmod 600 ~/.ssh/id_ed25519

# Fix .ssh directory permissions
chmod 700 ~/.ssh
```

### Error 3: Invalid Format / Load Key Error

```
Load key "/home/john/.ssh/id_rsa": invalid format
```

**Causes and Fixes:**

```bash
# Cause 1: File is corrupted or not a valid key
# Check if it looks like a real key
head -1 ~/.ssh/id_rsa
# Should show: -----BEGIN OPENSSH PRIVATE KEY-----

# Cause 2: Windows line endings (CRLF instead of LF) — common when downloading from CI/CD
# Fix with dos2unix
sudo apt install dos2unix
dos2unix ~/.ssh/id_rsa

# Or with sed
sed -i 's/\r//' ~/.ssh/id_rsa

# Cause 3: Key was encoded (base64) and not decoded properly
# Decode if needed
base64 -d encoded_key > id_rsa
chmod 600 id_rsa

# Cause 4: Passphrase entered wrong
ssh -i ~/.ssh/id_rsa john@server  # Will re-prompt for passphrase
```

### Error 4: Connection Refused

```
ssh: connect to host 192.168.1.100 port 22: Connection refused
```

**Fixes:**

```bash
# Fix 1: Check if SSH is running on the server
sudo systemctl status ssh

# Fix 2: Start SSH if it's down
sudo systemctl start ssh

# Fix 3: Check if server is listening on the right port
sudo ss -tlnp | grep sshd

# Fix 4: Check if firewall is blocking port 22
sudo ufw status
sudo ufw allow 22/tcp

# Fix 5: Server might be on a different port
ssh -p 2222 john@192.168.1.100
```

### Error 5: Host Key Verification Failed

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
```

**Causes and Fixes:**

```bash
# Cause: Server was reinstalled, IP changed, or legitimate host key rotation
# If you're SURE it's a legitimate change:

# Fix 1: Remove old key for that host
ssh-keygen -R 192.168.1.100

# Fix 2: Or manually remove from known_hosts
nano ~/.ssh/known_hosts
# Find and delete the line with the old server IP/hostname

# Fix 3: Connect again (will prompt to add new fingerprint)
ssh john@192.168.1.100
```

> ⚠️ **Security Warning:** If the host key changed unexpectedly on a production server, investigate before proceeding — it could indicate a man-in-the-middle attack.

### Error 6: Too Many Authentication Failures

```
Received disconnect from 192.168.1.100: Too many authentication failures
```

**Fix:**

```bash
# SSH is trying too many keys from ssh-agent
# Specify the exact key to use
ssh -o IdentitiesOnly=yes -i ~/.ssh/specific_key john@192.168.1.100

# Or in ~/.ssh/config
Host myserver
    HostName 192.168.1.100
    User john
    IdentityFile ~/.ssh/specific_key
    IdentitiesOnly yes
```

### Error 7: SCP "No Such File or Directory"

```
scp: /remote/path/file.txt: No such file or directory
```

**Fixes:**

```bash
# Fix 1: Create the remote directory first
ssh john@server "mkdir -p /remote/path/"
scp file.txt john@server:/remote/path/

# Fix 2: Check if remote path exists
ssh john@server "ls /remote/path/"

# Fix 3: Ensure you have write permissions on remote directory
ssh john@server "ls -la /remote/"
```

---

## 12. Best Practices for SSH Security

### 1. Use Ed25519 Keys

```bash
# Always prefer Ed25519 over RSA
ssh-keygen -t ed25519 -C "purpose-description"
```

### 2. Always Set a Passphrase on Private Keys

```bash
# Add passphrase to existing key
ssh-keygen -p -f ~/.ssh/id_ed25519
```

### 3. Change the Default SSH Port

```bash
# In /etc/ssh/sshd_config
Port 2222   # Use any port between 1024–65535
```

This reduces automated bot attacks significantly.

### 4. Disable Root Login

```bash
# In /etc/ssh/sshd_config
PermitRootLogin no
```

### 5. Limit User Access

```bash
# Allow only specific users
AllowUsers john deploy ci-user

# Or allow specific groups
AllowGroups ssh-users developers
```

### 6. Set Login Grace Time and Retry Limits

```bash
# In /etc/ssh/sshd_config
LoginGraceTime 30
MaxAuthTries 3
MaxSessions 5
```

### 7. Use Fail2Ban to Block Brute-Force Attacks

```bash
# Install fail2ban
sudo apt install fail2ban -y

# Create local config
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local

# SSH jail settings
[sshd]
enabled  = true
port     = 22
maxretry = 3
bantime  = 3600   # Ban for 1 hour
findtime = 600    # Within 10 minutes
```

### 8. Use SSH Certificate Authority (Advanced)

```bash
# For large teams, use CA-signed certificates instead of individual authorized_keys
# Generate CA key
ssh-keygen -t ed25519 -f ~/.ssh/ca_key -C "Team CA"

# Sign a user's public key
ssh-keygen -s ~/.ssh/ca_key -I "john@company.com" -n john -V +52w ~/.ssh/id_ed25519.pub
# Creates: ~/.ssh/id_ed25519-cert.pub (valid for 52 weeks)
```

### 9. Regular Key Rotation

- Rotate SSH keys every 6–12 months
- Immediately revoke keys of departing team members
- Audit `authorized_keys` files quarterly

### 10. Monitor SSH Access Logs

```bash
# View SSH authentication logs
sudo journalctl -u ssh -f

# On older Ubuntu/Debian
sudo tail -f /var/log/auth.log | grep sshd

# Check who's logged in right now
who
w
last | head -20
```

---

## 13. Real-World CI/CD Usage with SSH

### 13.1 GitHub Actions — Deploy via SSH

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Build application
        run: npm ci && npm run build

      - name: Setup SSH Key
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_ed25519
          chmod 600 ~/.ssh/id_ed25519
          # Disable strict host checking for automation
          ssh-keyscan -H ${{ secrets.SERVER_HOST }} >> ~/.ssh/known_hosts

      - name: Deploy files with SCP
        run: |
          scp -i ~/.ssh/id_ed25519 -r ./dist deploy@${{ secrets.SERVER_HOST }}:/var/www/app/

      - name: Restart application
        run: |
          ssh -i ~/.ssh/id_ed25519 deploy@${{ secrets.SERVER_HOST }} \
            "cd /var/www/app && pm2 restart app"
```

**Set GitHub Secrets:**
- `SSH_PRIVATE_KEY` — contents of your private key
- `SERVER_HOST` — your server IP or domain

### 13.2 GitLab CI/CD — SSH Deployment

```yaml
# .gitlab-ci.yml
stages:
  - build
  - deploy

deploy_production:
  stage: deploy
  image: ubuntu:22.04
  before_script:
    - apt-get update -qy && apt-get install -y openssh-client
    - mkdir -p ~/.ssh
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' > ~/.ssh/id_ed25519
    - chmod 600 ~/.ssh/id_ed25519
    - ssh-keyscan -H $SERVER_HOST >> ~/.ssh/known_hosts
  script:
    - scp -i ~/.ssh/id_ed25519 -r ./dist deploy@$SERVER_HOST:/var/www/app/
    - ssh -i ~/.ssh/id_ed25519 deploy@$SERVER_HOST "sudo systemctl restart myapp"
  environment:
    name: production
  only:
    - main
```

### 13.3 Using a Dedicated Deploy Key (Recommended Pattern)

```bash
# 1. Generate a deploy-specific key (no passphrase for automation)
ssh-keygen -t ed25519 -f ~/.ssh/ci_deploy_key -N "" -C "github-actions-deploy"

# 2. Add PUBLIC key to server's authorized_keys
ssh-copy-id -i ~/.ssh/ci_deploy_key.pub deploy@production-server.com

# 3. Add PRIVATE key contents as CI/CD secret variable

# 4. Restrict what the deploy key can do (via authorized_keys command restriction)
# On the server, in ~/.ssh/authorized_keys:
command="cd /var/www/app && git pull",no-port-forwarding,no-X11-forwarding,no-agent-forwarding ssh-ed25519 AAAAC3... ci-deploy-key
```

### 13.4 SSH Bastion Host / Jump Host Pattern

In production, servers are often not directly accessible. You use a **bastion host** as a secure gateway:

```
Internet ──▶ Bastion Host (Jump Server) ──▶ Internal Server
                 (Public IP)                   (Private IP only)
```

```bash
# Connect through a jump/bastion host
ssh -J bastion@jump.example.com john@internal-server-192.168.10.5

# In ~/.ssh/config for permanent setup
Host internal-app
    HostName 192.168.10.5
    User john
    ProxyJump bastion@jump.example.com
    IdentityFile ~/.ssh/deploy_key

# Now just run:
ssh internal-app
scp -r ./deploy internal-app:/var/www/
```

### 13.5 SSH Tunneling for Database Access

```bash
# Forward local port 5433 to remote PostgreSQL port 5432
# Useful for accessing production DB securely without opening it to internet
ssh -L 5433:localhost:5432 john@db-server.example.com -N

# Now connect to production DB locally:
psql -h localhost -p 5433 -U dbuser myapp_production

# Run in background
ssh -L 5433:localhost:5432 john@db-server.example.com -N -f
```

### 13.6 Ansible SSH Configuration (Infrastructure as Code)

```yaml
# ansible.cfg
[defaults]
inventory = hosts.ini
remote_user = deploy
private_key_file = ~/.ssh/ansible_key

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
```

```ini
# hosts.ini
[web_servers]
web1 ansible_host=192.168.1.101
web2 ansible_host=192.168.1.102

[db_servers]
db1 ansible_host=192.168.1.201 ansible_ssh_private_key_file=~/.ssh/db_key
```

```bash
# Deploy with Ansible over SSH
ansible-playbook -i hosts.ini deploy.yml --private-key ~/.ssh/ansible_key
```

---

## 📚 Reference Summary

### Quick Command Cheatsheet

```bash
# ─── SSH ───────────────────────────────────────────────────
ssh user@host                          # Basic connection
ssh -p 2222 user@host                 # Custom port
ssh -i ~/.ssh/key user@host           # Specific key
ssh -J jump@bastion user@host         # Via jump host
ssh -L 8080:localhost:80 user@host    # Local port forward
ssh -v user@host                      # Verbose / debug

# ─── KEY MANAGEMENT ────────────────────────────────────────
ssh-keygen -t ed25519 -C "email"      # Generate key
ssh-copy-id user@host                 # Copy key to server
ssh-keygen -R hostname                # Remove known host
ssh-add ~/.ssh/key                    # Add key to agent
ssh-add -l                            # List agent keys

# ─── SCP ───────────────────────────────────────────────────
scp file.txt user@host:/path/         # Upload file
scp -r ./folder user@host:/path/      # Upload folder
scp user@host:/path/file.txt ./       # Download file
scp -r user@host:/path/folder ./      # Download folder
scp -i ~/.ssh/key file user@host:/p   # Upload with key
scp -P 2222 file user@host:/path/     # Custom port

# ─── PERMISSIONS ───────────────────────────────────────────
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub

# ─── SSHD CONFIG ───────────────────────────────────────────
sudo sshd -t                          # Test config
sudo systemctl reload ssh             # Reload config
sudo journalctl -u ssh -f             # View SSH logs
```

---

## 📖 Recommended References

| Resource | URL | Description |
|----------|-----|-------------|
| OpenSSH Official Docs | https://www.openssh.com/manual.html | Complete man pages and official documentation |
| SSH Academy | https://www.ssh.com/academy/ssh | Comprehensive beginner to advanced SSH guides |
| DigitalOcean SSH Tutorials | https://www.digitalocean.com/community/tutorials | Practical SSH and SCP guides with examples |
| NIST SP 800-190 | https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-190.pdf | Security guidelines for SSH in enterprise |
| Ansible SSH Guide | https://docs.ansible.com/ansible/latest/inventory_guide/connection_details.html | SSH in Infrastructure as Code context |
| GitHub Actions SSH | https://docs.github.com/en/actions/use-cases-and-examples/deploying/deploying-with-github-actions | Official CI/CD deployment docs |
| Mozilla SSH Guidelines | https://infosec.mozilla.org/guidelines/openssh | Hardened SSH configuration recommendations |
| `man ssh` | Run `man ssh` in terminal | Full local manual for SSH client |
| `man sshd_config` | Run `man sshd_config` in terminal | All server configuration options |

---

> **Document Version:** 1.0  
> **Last Updated:** 2025  
> **Target Audience:** Junior DevOps Engineers, Backend Developers, System Administrators  
> **Skill Level:** Beginner → Intermediate → Advanced  

---

*Happy deploying! 🚀 — If something breaks, `ssh -vvv` is your best friend.*
