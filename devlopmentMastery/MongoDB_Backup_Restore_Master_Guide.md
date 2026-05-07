# 🍃 MongoDB Backup & Restore Master Guide

> **A complete, production-grade reference for developers, DevOps engineers, and architects.**
> Covers beginner to advanced strategies, real-world automation, cloud integration, and enterprise architecture.

---

## 📋 Table of Contents

1. [Introduction](#1-introduction)
2. [Types of MongoDB Backups](#2-types-of-mongodb-backups)
3. [Best Industry Standard Approach](#3-best-industry-standard-approach)
4. [Installation](#4-installation)
5. [Connection Strings](#5-connection-strings)
6. [mongodump Deep Dive](#6-mongodump-deep-dive)
7. [mongorestore Deep Dive](#7-mongorestore-deep-dive)
8. [Automation](#8-automation)
9. [Cloud Backup Strategy](#9-cloud-backup-strategy)
10. [Docker MongoDB Backup](#10-docker-mongodb-backup)
11. [Prisma + MongoDB Backup Strategy](#11-prisma--mongodb-backup-strategy)
12. [Backup Verification](#12-backup-verification)
13. [Security Best Practices](#13-security-best-practices)
14. [Common Errors & Fixes](#14-common-errors--fixes)
15. [Production Architecture](#15-production-architecture)
16. [Real-World Recommendations](#16-real-world-recommendations)
17. [Best Practices Checklist](#17-best-practices-checklist)
18. [Cheat Sheet](#18-cheat-sheet)

---

## 1. Introduction

### What is a Backup?

A **backup** is a copy of your database data stored separately from the primary system. It allows you to recover from data loss, corruption, accidental deletions, hardware failures, or security incidents.

In MongoDB, a backup captures:
- **Documents** (JSON-like BSON records)
- **Indexes** (for query performance)
- **Schemas and validation rules**
- **Metadata** (user roles, configuration)

### Why Backup Matters

Your MongoDB database is likely the most valuable asset in your application. Without backups:

- A single `db.dropDatabase()` can wipe everything permanently
- A developer mistake can delete thousands of records in seconds
- Hardware failure can make your data permanently inaccessible
- Ransomware can encrypt your entire database and demand payment
- Cloud provider outages (yes, they happen) can take your data with them

> **Backup is not optional — it is a fundamental engineering responsibility.**

### Risks of Not Taking Backups

| Risk | Impact | Likelihood |
|------|--------|------------|
| Developer accidental deletion | Total data loss | High |
| Application bug corrupting data | Partial/full data corruption | Medium |
| Ransomware attack | Full encryption/loss | Medium |
| Hardware/disk failure | Full data loss | Medium |
| Cloud provider incident | Temporary or permanent loss | Low |
| Malicious insider action | Targeted data deletion | Low |

### Real Production Failure Examples

**GitLab (2017)**
A sysadmin ran `rm -rf` on the wrong server, deleting ~300GB of production PostgreSQL data. They had 5 backup methods but none worked properly. They lost 6 hours of data and had ~18 hours of downtime. The incident was streamed live because they had nothing to hide.

**MongoHQ Breach (2013)**
A MongoDB hosting provider's database was accessed by attackers. Customers who had no backups lost everything.

**Typical Startup Story (Common)**
A developer runs `db.users.drop()` in the production MongoDB shell, thinking they were in the development environment. No backup exists. The company loses all user data. The startup may not survive.

**Lesson**: Even the best teams make mistakes. Backups are your insurance policy.

---

## 2. Types of MongoDB Backups

### 2.1 mongodump / mongorestore

`mongodump` is MongoDB's official command-line backup tool that exports data in **BSON format** (Binary JSON).

**How it works:**
```
mongodump reads documents → converts to BSON → writes to disk files
mongorestore reads BSON files → inserts back into MongoDB
```

**Advantages:**
- Free and built into MongoDB Database Tools
- Works on all MongoDB versions and deployments
- Selective backup (specific DB, collection, or query filter)
- Human-portable: dump directory can be archived and moved

**Disadvantages:**
- Does **not** capture in-progress transactions atomically
- Slow for very large databases (reads all documents one by one)
- Dump files can be large (no native compression unless `--gzip` used)
- Not suitable for point-in-time recovery without oplog

**When to use:**
- Development and staging environments
- Small to medium databases (under 50GB)
- Pre-migration snapshots
- Manual one-off backups before schema changes

**Production Recommendation:** Use for supplementary/manual backups. Combine with oplog for PITR.

---

### 2.2 Filesystem Snapshots

A **filesystem snapshot** captures the exact state of the disk volume at a moment in time using the OS or cloud provider's snapshot feature (LVM snapshots on Linux, EBS snapshots on AWS).

**How it works:**
```
Freeze MongoDB writes (or use journaling) → Take volume snapshot → Resume writes
```

**Advantages:**
- Near-instantaneous (copy-on-write technology)
- Consistent snapshot of the entire data directory
- No performance degradation during backup
- Ideal for large databases (100GB+)

**Disadvantages:**
- Requires proper journal flush or `db.fsyncLock()` for consistency
- More complex to restore (requires volume attach + mongod startup)
- Cloud-specific (AWS EBS, Azure Managed Disks, GCP Persistent Disk)

**When to use:**
- Large production databases
- When RPO (Recovery Point Objective) needs to be minimal
- When integrated with cloud infrastructure

**Production Recommendation:** Best for large, cloud-hosted deployments.

---

### 2.3 Atlas Cloud Backups

MongoDB Atlas (the official cloud service) provides **automated cloud backups** as a managed feature.

**How it works:**
```
Atlas monitors replica sets → takes consistent snapshots → stores in Atlas-managed storage
```

**Advantages:**
- Fully managed — no manual intervention
- Point-in-time recovery down to the second
- Oplog-based continuous backup
- Queryable backups (inspect data before restoring)
- Encryption at rest and in transit by default

**Disadvantages:**
- Requires MongoDB Atlas (paid service)
- Less control over backup storage location
- Cost increases with storage and frequency

**When to use:**
- Any production workload on Atlas
- When you want zero-ops backup management

**Production Recommendation:** **The gold standard for production.** If you can afford Atlas, use this.

---

### 2.4 Replica Set Backups

MongoDB replica sets consist of a **primary** and one or more **secondaries**. You can take backups from a secondary to avoid impacting the primary.

**How it works:**
```
Secondary node is "hidden" → mongodump runs against secondary → primary is unaffected
```

**Advantages:**
- Zero impact on primary (production traffic)
- Consistent point-in-time snapshot if using `--oplog`
- Works with any backup method (mongodump, filesystem snapshot)

**Disadvantages:**
- Secondary may be slightly behind primary (replication lag)
- Requires replica set configuration

**When to use:**
- Any production deployment with replica sets

**Production Recommendation:** Always back up from a secondary, never from the primary.

---

### 2.5 Oplog Backups

The **oplog** (operations log) is a special capped collection in MongoDB that records every write operation. It is the foundation of replication.

**How it works:**
```
mongodump --oplog captures operations during the dump → restore uses oplog to replay changes
```

**Advantages:**
- Enables consistent, point-in-time backups
- Captures changes that happen during the dump process
- Foundation for continuous backup strategies

**Disadvantages:**
- Oplog is a capped collection — it can roll over if writes are frequent
- More complex to manage
- Requires replica set

**When to use:**
- Whenever you need consistency across collections during backup
- Combined with regular mongodump for production-quality backups

**Production Recommendation:** Always use `--oplog` in production mongodump commands.

---

### 2.6 Point-in-Time Recovery (PITR)

**PITR** allows you to restore your database to any specific moment in the past — not just the last backup.

**How it works:**
```
Base Snapshot → + Oplog entries → Restore to exact timestamp
Example:
Snapshot at 2:00 AM → + Oplog 2:00 AM to 3:47 PM → Database state at 3:47 PM
```

**Advantages:**
- Most precise recovery option
- Protects against data corruption introduced at a known time
- Minimizes RPO to near-zero

**Disadvantages:**
- Requires continuous oplog archiving
- Complex to implement manually
- Best used with Atlas or a managed backup service

**When to use:**
- Enterprise applications
- Financial, medical, or legal systems requiring audit trails

**Production Recommendation:** Use Atlas PITR or implement oplog tailing with a tool like `mongo-oplog-backup`.

---

### 2.7 Incremental Backups

**Incremental backups** only copy data that has **changed** since the last backup.

**How it works:**
```
Full backup on Sunday → Incremental Monday (only Monday's changes) → Incremental Tuesday...
```

**Advantages:**
- Much smaller backup size
- Faster backup windows
- Lower storage costs

**Disadvantages:**
- More complex restore process (need full backup + all incrementals)
- MongoDB doesn't natively support incremental dump — requires oplog or Ops Manager

**When to use:**
- Large databases where full backups are impractical
- Enterprise deployments with MongoDB Ops Manager or Atlas

**Production Recommendation:** Use MongoDB Ops Manager or Atlas for incremental backups.

---

### 2.8 Export/Import JSON

`mongoexport` and `mongoimport` work with **JSON or CSV** format (human-readable).

```bash
# Export
mongoexport --db mydb --collection users --out users.json

# Import
mongoimport --db mydb --collection users --file users.json
```

**Advantages:**
- Human-readable format
- Easy to inspect, edit, and transform
- Cross-database compatibility

**Disadvantages:**
- Lossy — loses BSON-specific types (ObjectId, Date become strings)
- Slow for large collections
- Not recommended for full database backup/restore

**When to use:**
- Data migration between different databases
- Sharing sample data with teams
- Seeding development databases

**Production Recommendation:** Do NOT use for production backups. Use only for data portability.

---

### 2.9 BSON Backup

`mongodump` outputs **BSON** files natively. BSON (Binary JSON) is MongoDB's native data format.

```
backup/
├── mydb/
│   ├── users.bson         # Document data
│   ├── users.metadata.json # Index info
│   ├── orders.bson
│   └── orders.metadata.json
```

**Advantages:**
- Lossless — preserves all BSON types (ObjectId, Date, Binary, etc.)
- Compact binary format
- Native to MongoDB — perfect fidelity

**Disadvantages:**
- Not human-readable
- Requires `mongorestore` to load

**When to use:**
- All production backup scenarios
- Any time you need a lossless copy

**Production Recommendation:** Always prefer BSON backup over JSON export.

---

### Backup Types Comparison Table

| Type | Consistency | Speed | Size | PITR | Complexity | Best For |
|------|------------|-------|------|------|------------|----------|
| mongodump | Medium | Slow | Large | No (without oplog) | Low | Manual/Dev |
| mongodump + oplog | High | Slow | Large | Yes | Medium | Small Prod |
| Filesystem Snapshot | High | Fast | Full | No | Medium | Large Prod |
| Atlas Backup | High | Managed | Managed | Yes | None | Cloud Prod |
| Incremental | High | Fast | Small | Partial | High | Enterprise |
| JSON Export | Low | Slow | Variable | No | Low | Dev/Migration |

---

## 3. Best Industry Standard Approach

### Why mongodump is the Standard Baseline

`mongodump` is used because:
1. It ships free with MongoDB Database Tools
2. Works without cloud infrastructure
3. Familiar to every MongoDB developer
4. Can be scripted and automated easily
5. Produces portable BSON files that can be archived anywhere

It is the **minimum viable backup** for any MongoDB deployment. Even if you have Atlas backups, developers should still know mongodump.

### Why Atlas Backups are Best for Production

Atlas managed backups are superior because:
1. **Automatic scheduling** — no cron jobs to manage
2. **Oplog-based PITR** — restore to any second in the past
3. **Consistency guaranteed** — Atlas coordinates snapshot across replica sets
4. **Queryable backup** — inspect data in a backup without full restore
5. **Encryption built-in** — at-rest and in-transit by default
6. **Tested recovery** — Atlas tests backup integrity automatically
7. **Compliance-ready** — meets SOC2, HIPAA, GDPR requirements

### Recommended Strategy for Startups

```
┌─────────────────────────────────────────────────────────────┐
│                   STARTUP BACKUP STRATEGY                   │
├─────────────────────────────────────────────────────────────┤
│  Daily mongodump → Compressed → Upload to AWS S3 / GCS      │
│  + Atlas M10 or above with Atlas Backups enabled            │
│  + Pre-deployment manual dump before any migration          │
│  + Weekly restore test to staging environment               │
└─────────────────────────────────────────────────────────────┘
```

**Cost:** Low (S3 storage is cheap; Atlas M10 ~$57/month)
**Effort:** Low (automate with cron or GitHub Actions)
**Protection:** High

### Recommended Strategy for Enterprise Systems

```
┌────────────────────────────────────────────────────────────────────┐
│                  ENTERPRISE BACKUP STRATEGY                        │
├────────────────────────────────────────────────────────────────────┤
│  Atlas Dedicated Cluster (M30+) with:                              │
│   - Continuous cloud backup (PITR to the second)                   │
│   - Cross-region snapshot replication                              │
│   - Queryable backups                                              │
│   - Automated restore testing via Ops Manager                      │
│                                                                    │
│  + Daily filesystem snapshot of hidden replica set member          │
│  + Oplog archiving to separate S3 bucket (30-day retention)        │
│  + Weekly DR drill — full restore to isolated environment          │
│  + Backup encryption with KMS-managed keys                         │
│  + Immutable S3 backup buckets (WORM — Write Once Read Many)       │
└────────────────────────────────────────────────────────────────────┘
```

---

## 4. Installation

### MongoDB Database Tools

The tools (`mongodump`, `mongorestore`, `mongoexport`, `mongoimport`) are part of the **MongoDB Database Tools** package — separate from the MongoDB server.

---

### Windows Installation

**Method 1: MongoDB Installer (Recommended)**
1. Download MongoDB Community Server from [mongodb.com/try/download/community](https://www.mongodb.com/try/download/community)
2. During installation, check **"Install MongoDB Compass"** and **"MongoDB Database Tools"**
3. Tools are installed to: `C:\Program Files\MongoDB\Tools\<version>\bin\`

**Method 2: Standalone Tools**
1. Go to [mongodb.com/try/download/database-tools](https://www.mongodb.com/try/download/database-tools)
2. Choose **Windows**, **zip** package
3. Extract to `C:\mongodb-tools\bin\`

**Setting Environment Variables (Windows):**
```powershell
# Run in PowerShell as Administrator
$toolsPath = "C:\Program Files\MongoDB\Tools\100\bin"
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";$toolsPath", "Machine")
```

Or manually:
1. Open **System Properties** → **Advanced** → **Environment Variables**
2. Under **System variables**, find `Path` → Edit → New
3. Add: `C:\Program Files\MongoDB\Tools\100\bin`

**Verify:**
```powershell
mongodump --version
mongorestore --version
```

---

### Linux Installation (Ubuntu/Debian)

```bash
# Step 1: Import MongoDB GPG key
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
  sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor

# Step 2: Add repository
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] \
  https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | \
  sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

# Step 3: Update and install
sudo apt-get update
sudo apt-get install -y mongodb-database-tools

# Verify installation
mongodump --version
mongorestore --version
```

**For RHEL/CentOS/Fedora:**
```bash
# Create repo file
sudo tee /etc/yum.repos.d/mongodb-org-7.0.repo << 'EOF'
[mongodb-org-7.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/7.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-7.0.asc
EOF

sudo yum install -y mongodb-database-tools
```

**Environment Setup (Linux):**
```bash
# Add to ~/.bashrc or ~/.zshrc if not already in PATH
echo 'export PATH="/usr/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Verify
which mongodump
mongodump --version
```

---

### macOS Installation

**Method 1: Homebrew (Recommended)**
```bash
# Install Homebrew if not already installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Add MongoDB tap
brew tap mongodb/brew

# Install MongoDB Database Tools
brew install mongodb-database-tools

# Verify
mongodump --version
mongorestore --version
```

**Method 2: Manual Download**
```bash
# Download from mongodb.com, then:
tar -zxvf mongodb-database-tools-macos-*.tgz
sudo mv mongodb-database-tools-*/bin/* /usr/local/bin/
```

**Environment Setup (macOS):**
```bash
# For Homebrew installations, tools are automatically in PATH
# For manual installs, add to ~/.zshrc:
echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

---

### Checking Versions

```bash
# Check tool versions
mongodump --version
mongorestore --version
mongoexport --version
mongoimport --version

# Check MongoDB server version (if running locally)
mongosh --eval "db.version()"
```

> ⚠️ **Important:** Always match your Database Tools version to your MongoDB server version. Mismatches can cause compatibility issues, especially during restore.

---

## 5. Connection Strings

### MongoDB URI Structure

A MongoDB connection string (URI) follows this structure:

```
mongodb://[username:password@]host[:port][/database][?options]
```

**Full example with all components:**
```
mongodb://admin:P%40ssw0rd@db.example.com:27017/myapp?authSource=admin&ssl=true
```

| Part | Example | Description |
|------|---------|-------------|
| `mongodb://` | `mongodb://` | Protocol scheme |
| `username` | `admin` | Database user |
| `password` | `P%40ssw0rd` | URL-encoded password |
| `host` | `db.example.com` | Server hostname or IP |
| `port` | `27017` | MongoDB port (default: 27017) |
| `/database` | `/myapp` | Default database |
| `?authSource=admin` | `?authSource=admin` | Auth database |
| `&ssl=true` | `&ssl=true` | Enable TLS/SSL |

### Replica Set URI

```
mongodb://user:pass@host1:27017,host2:27017,host3:27017/db?replicaSet=myReplicaSet&authSource=admin
```

### Atlas URI Structure

```
mongodb+srv://username:password@cluster0.xxxxx.mongodb.net/mydb?retryWrites=true&w=majority
```

The `+srv` indicates DNS SRV lookup — Atlas uses this to automatically discover cluster members.

---

### Username/Password Encoding

**Critical Rule:** Special characters in usernames or passwords **must be percent-encoded** (URL-encoded).

If you don't encode, MongoDB will misparse the URI and throw authentication errors.

**Common special characters and their encodings:**

| Character | Encoded | Common In |
|-----------|---------|-----------|
| `@` | `%40` | Email-style usernames |
| `:` | `%3A` | Passwords with colons |
| `/` | `%2F` | Passwords with slashes |
| `?` | `%3F` | Random-generated passwords |
| `#` | `%23` | Passwords with hash |
| `%` | `%25` | Passwords with percent |
| `+` | `%2B` | Passwords with plus |
| ` ` | `%20` | Spaces (avoid!) |

**The `%40` Usage for `@`**

This is the most common mistake. If your password is `my@pass`:

```bash
# WRONG — MongoDB thinks "my" is the username host
mongodb://admin:my@pass@localhost:27017

# CORRECT — @ is encoded as %40
mongodb://admin:my%40pass@localhost:27017
```

**How to encode passwords safely:**

```bash
# Python one-liner to URL-encode a string
python3 -c "import urllib.parse; print(urllib.parse.quote('my@p@ss:w0rd!', safe=''))"
# Output: my%40p%40ss%3Aw0rd%21

# Node.js
node -e "console.log(encodeURIComponent('my@p@ss:w0rd!'))"
# Output: my%40p%40ss%3Aw0rd!
```

---

### Common URI Mistakes

**Mistake 1: Unencoded special characters**
```bash
# WRONG
mongodb://admin:pass@word@localhost:27017

# RIGHT
mongodb://admin:pass%40word@localhost:27017
```

**Mistake 2: Wrong authSource**
```bash
# WRONG — user exists in 'admin' but you're not specifying authSource
mongodb://admin:password@localhost:27017/myapp

# RIGHT
mongodb://admin:password@localhost:27017/myapp?authSource=admin
```

**Mistake 3: Missing port for non-default deployments**
```bash
# WRONG (if running on 27018)
mongodb://admin:password@localhost/myapp

# RIGHT
mongodb://admin:password@localhost:27018/myapp
```

**Mistake 4: Using Atlas URI without `+srv`**
```bash
# WRONG for Atlas
mongodb://user:pass@cluster0.xxxxx.mongodb.net/mydb

# RIGHT for Atlas
mongodb+srv://user:pass@cluster0.xxxxx.mongodb.net/mydb
```

---

### URI Security Best Practices

```bash
# NEVER hardcode URIs in shell scripts
# WRONG:
mongodump --uri="mongodb://admin:mypassword@localhost:27017" --out=/backup

# RIGHT: Use environment variable
export MONGODB_URI="mongodb://admin:mypassword@localhost:27017"
mongodump --uri="$MONGODB_URI" --out=/backup

# Store in .env file (never commit to git)
echo "MONGODB_URI=mongodb://admin:mypassword@localhost:27017" >> .env
echo ".env" >> .gitignore
```

---

## 6. mongodump Deep Dive

### Basic Syntax

```bash
mongodump [options] [--uri <connectionString> | --host <hostname> --port <port>]
```

---

### Full Database Backup

```bash
# Backup ALL databases to default ./dump/ directory
mongodump

# Backup ALL databases to a specific directory
mongodump --out /var/backups/mongodb/$(date +%Y-%m-%d)
```

**Output structure:**
```
dump/
├── admin/
│   ├── system.users.bson
│   └── system.users.metadata.json
├── myapp/
│   ├── users.bson
│   ├── users.metadata.json
│   ├── orders.bson
│   └── orders.metadata.json
└── oplog.bson   # (if --oplog was used)
```

---

### Single Database Backup

```bash
# Backup only the 'myapp' database
mongodump --db myapp --out /var/backups/mongodb/

# With URI
mongodump --uri="mongodb://localhost:27017" --db myapp --out /backup/myapp
```

---

### Single Collection Backup

```bash
# Backup only the 'users' collection from 'myapp'
mongodump --db myapp --collection users --out /backup/

# With a query filter (backup users created after a date)
mongodump --db myapp --collection users \
  --query '{"createdAt": {"$gt": {"$date": "2024-01-01T00:00:00Z"}}}' \
  --out /backup/
```

---

### Compressed Backup (Recommended)

```bash
# Archive format (single file) — best for storage and transfer
mongodump --db myapp --archive=/backup/myapp_$(date +%Y-%m-%d).archive

# Gzip compressed archive (smaller file size)
mongodump --db myapp --archive=/backup/myapp_$(date +%Y-%m-%d).archive.gz --gzip

# Gzip compress all files in a directory
mongodump --db myapp --out /backup/myapp/ --gzip
```

> 💡 **Tip:** The `--archive` format is preferred for cloud storage uploads — single file, easier to transfer and manage.

---

### Oplog Backup (Production Recommended)

```bash
# Capture oplog entries during dump for consistency
# This is CRITICAL for multi-collection consistent backups
mongodump --oplog --out /backup/myapp_$(date +%Y-%m-%d)

# With archive and gzip
mongodump --oplog --archive=/backup/full_$(date +%Y-%m-%d_%H%M).archive.gz --gzip
```

**What `--oplog` does:**
```
Dump starts at T1 → Documents are read → Writes happen during dump → oplog.bson captures those writes → Restore replays oplog → Database is consistent to T2
```

> ⚠️ **Warning:** `--oplog` only works against a replica set member (primary or secondary), not a standalone mongod.

---

### Remote Cluster Backup

```bash
# Backup a remote MongoDB instance
mongodump --host db.production.com --port 27017 \
  --username admin --password mypassword \
  --authenticationDatabase admin \
  --db myapp \
  --out /backup/remote_$(date +%Y-%m-%d)

# Using full URI (preferred for complex setups)
mongodump --uri="mongodb://admin:mypassword@db.production.com:27017/?authSource=admin" \
  --db myapp \
  --out /backup/
```

---

### Atlas Backup via mongodump

```bash
# Get connection string from Atlas: Cluster → Connect → Connect your application
# Replace <password> with your actual password

mongodump --uri="mongodb+srv://admin:myP%40ssword@cluster0.abc123.mongodb.net" \
  --db myapp \
  --out /backup/atlas_$(date +%Y-%m-%d) \
  --ssl \
  --authenticationDatabase admin

# Atlas with oplog (only works if Atlas allows oplog access)
# Note: Atlas managed backups are generally preferred over this approach
mongodump --uri="mongodb+srv://admin:pass@cluster0.abc123.mongodb.net" \
  --oplog \
  --archive=/backup/atlas_full.archive.gz \
  --gzip
```

---

### Authentication Options

```bash
# Username/password via flags
mongodump --host localhost --port 27017 \
  --username myuser \
  --password mypassword \
  --authenticationDatabase admin \
  --db myapp --out /backup/

# SCRAM-SHA-256 (default for MongoDB 4.0+)
mongodump --uri="mongodb://user:pass@host:27017/db?authSource=admin&authMechanism=SCRAM-SHA-256"

# X.509 certificate authentication
mongodump --host localhost --port 27017 \
  --ssl \
  --sslCAFile /etc/ssl/ca.pem \
  --sslClientCertificateKeyFile /etc/ssl/client.pem \
  --authenticationMechanism MONGODB-X509 \
  --out /backup/
```

---

### Backup Options Summary Table

| Flag | Description | Example |
|------|-------------|---------|
| `--uri` | Full connection string | `--uri="mongodb://..."` |
| `--host` | Server hostname | `--host=db.example.com` |
| `--port` | Server port | `--port=27017` |
| `--db` | Specific database | `--db=myapp` |
| `--collection` | Specific collection | `--collection=users` |
| `--out` | Output directory | `--out=/backup/` |
| `--archive` | Single archive file | `--archive=backup.archive` |
| `--gzip` | Enable gzip compression | `--gzip` |
| `--oplog` | Include oplog | `--oplog` |
| `--query` | Filter documents | `--query='{"active":true}'` |
| `--username` | Auth username | `--username=admin` |
| `--password` | Auth password | `--password=secret` |
| `--authenticationDatabase` | Auth source DB | `--authenticationDatabase=admin` |
| `--ssl` | Enable TLS/SSL | `--ssl` |
| `--numParallelCollections` | Parallel threads | `--numParallelCollections=4` |
| `--excludeCollection` | Exclude collection | `--excludeCollection=logs` |
| `--readPreference` | Read from secondary | `--readPreference=secondary` |

---

## 7. mongorestore Deep Dive

### Basic Syntax

```bash
mongorestore [options] [<directory> | --archive <file>]
```

---

### Restoring a Full Database

```bash
# Restore all databases from dump directory
mongorestore /backup/dump/

# Restore from a specific dump directory
mongorestore --dir /backup/2024-01-15/

# Using URI
mongorestore --uri="mongodb://localhost:27017" /backup/dump/
```

---

### Restoring a Specific Collection

```bash
# Restore only users collection to myapp database
mongorestore --db myapp --collection users /backup/dump/myapp/users.bson

# Restore from archive, specific collection
mongorestore --uri="mongodb://localhost:27017" \
  --archive=/backup/myapp.archive \
  --nsInclude="myapp.users"
```

---

### The `--drop` Option (Important)

By default, `mongorestore` **merges** data — it inserts documents alongside existing ones. This can cause duplicate key errors.

```bash
# Drop existing collection before restoring (clean restore)
mongorestore --drop /backup/dump/

# Drop specific collection
mongorestore --db myapp --collection users --drop /backup/dump/myapp/users.bson
```

> ⚠️ **Warning:** `--drop` is destructive. Always confirm you have the right source and target before using it.

---

### Restore to Another Database

```bash
# Restore 'myapp' backup into 'myapp_restored' database
mongorestore --nsFrom="myapp.*" --nsTo="myapp_restored.*" /backup/dump/

# Restore single collection to a different database and collection name
mongorestore --nsFrom="myapp.users" --nsTo="backup.users_jan15" \
  /backup/dump/myapp/users.bson
```

---

### Restoring Compressed Backups

```bash
# Restore from gzip-compressed directory
mongorestore --gzip /backup/dump_gzipped/

# Restore from archive file
mongorestore --archive=/backup/myapp_2024-01-15.archive

# Restore from gzip archive
mongorestore --archive=/backup/myapp_2024-01-15.archive.gz --gzip

# Restore from gzip archive to specific database
mongorestore --uri="mongodb://localhost:27017" \
  --archive=/backup/myapp_2024-01-15.archive.gz \
  --gzip \
  --db myapp_restored \
  --drop
```

---

### Restoring with Oplog Replay

```bash
# Restore with oplog replay (for consistent PITR)
mongorestore --oplogReplay /backup/dump/

# Restore with oplog up to a specific timestamp
# oplogLimit format: <seconds>:<ordinal>
mongorestore --oplogReplay --oplogLimit 1705276800:0 /backup/dump/
```

---

### Restoring Atlas Backups

1. Download the backup snapshot from Atlas UI (Atlas → Cluster → Backup → Download Snapshot)
2. Extract the downloaded archive
3. Restore:

```bash
# Restore downloaded Atlas snapshot
mongorestore --uri="mongodb://localhost:27017" \
  --archive=/downloads/atlas_snapshot.archive.gz \
  --gzip \
  --drop

# Restore Atlas backup to another Atlas cluster
mongorestore --uri="mongodb+srv://admin:pass@cluster-restored.abc.mongodb.net" \
  --archive=/downloads/atlas_snapshot.archive.gz \
  --gzip \
  --ssl \
  --drop
```

---

### mongorestore Options Summary

| Flag | Description |
|------|-------------|
| `--uri` | Connection string |
| `--db` | Target database name |
| `--collection` | Target collection name |
| `--drop` | Drop before restoring |
| `--archive` | Read from archive file |
| `--gzip` | Decompress gzip |
| `--oplogReplay` | Replay oplog entries |
| `--oplogLimit` | Replay oplog up to timestamp |
| `--nsFrom` | Source namespace pattern |
| `--nsTo` | Target namespace pattern |
| `--nsInclude` | Only restore matching namespaces |
| `--nsExclude` | Skip matching namespaces |
| `--numParallelCollections` | Parallel restore threads |
| `--noIndexRestore` | Skip index restoration |
| `--stopOnError` | Halt on first error |
| `--preserveUUID` | Preserve collection UUIDs |

---

## 8. Automation

### Linux/macOS: Cron Job Backup

Cron runs scheduled tasks on Unix-like systems.

**Creating the backup script:**

```bash
# Create backup script
cat > /usr/local/bin/mongodb_backup.sh << 'EOF'
#!/bin/bash

# ============================================
# MongoDB Backup Script
# ============================================

# Configuration
MONGODB_URI="${MONGODB_URI:-mongodb://localhost:27017}"
BACKUP_DIR="/var/backups/mongodb"
DB_NAME="${DB_NAME:-myapp}"
DATE=$(date +%Y-%m-%d_%H%M%S)
BACKUP_NAME="backup_${DB_NAME}_${DATE}"
RETENTION_DAYS=7

# Create backup directory
mkdir -p "$BACKUP_DIR"

echo "[$(date)] Starting MongoDB backup..."

# Run mongodump
mongodump \
  --uri="$MONGODB_URI" \
  --db="$DB_NAME" \
  --archive="${BACKUP_DIR}/${BACKUP_NAME}.archive.gz" \
  --gzip \
  --oplog

if [ $? -eq 0 ]; then
  echo "[$(date)] Backup successful: ${BACKUP_NAME}.archive.gz"
else
  echo "[$(date)] BACKUP FAILED!" >&2
  exit 1
fi

# Delete backups older than RETENTION_DAYS
find "$BACKUP_DIR" -name "backup_*.archive.gz" -mtime +$RETENTION_DAYS -delete
echo "[$(date)] Cleaned up backups older than ${RETENTION_DAYS} days"

echo "[$(date)] Backup complete."
EOF

chmod +x /usr/local/bin/mongodb_backup.sh
```

**Setting up the cron job:**

```bash
# Open crontab editor
crontab -e

# Add backup schedules:

# Daily backup at 2:00 AM
0 2 * * * /usr/local/bin/mongodb_backup.sh >> /var/log/mongodb_backup.log 2>&1

# Every 6 hours
0 */6 * * * /usr/local/bin/mongodb_backup.sh >> /var/log/mongodb_backup.log 2>&1

# Every hour (for critical systems)
0 * * * * /usr/local/bin/mongodb_backup.sh >> /var/log/mongodb_backup.log 2>&1

# Weekly full backup on Sunday at 1:00 AM
0 1 * * 0 /usr/local/bin/mongodb_backup_full.sh >> /var/log/mongodb_backup.log 2>&1
```

**Cron syntax reference:**
```
* * * * * command
│ │ │ │ │
│ │ │ │ └─ Day of week (0-7, 0 and 7 are Sunday)
│ │ │ └─── Month (1-12)
│ │ └───── Day of month (1-31)
│ └─────── Hour (0-23)
└───────── Minute (0-59)
```

---

### Windows Task Scheduler

```powershell
# Create backup script (save as C:\Scripts\mongodb_backup.ps1)
$ErrorActionPreference = "Stop"
$date = Get-Date -Format "yyyy-MM-dd_HHmmss"
$backupDir = "C:\Backups\MongoDB"
$dbName = "myapp"
$mongoUri = $env:MONGODB_URI

# Create directory if not exists
New-Item -ItemType Directory -Force -Path $backupDir | Out-Null

# Run mongodump
& mongodump `
  --uri="$mongoUri" `
  --db=$dbName `
  --archive="$backupDir\backup_${dbName}_${date}.archive.gz" `
  --gzip

if ($LASTEXITCODE -ne 0) {
  Write-Error "Backup failed!"
  exit 1
}

Write-Host "Backup complete: backup_${dbName}_${date}.archive.gz"

# Cleanup old backups (keep last 7 days)
Get-ChildItem -Path $backupDir -Filter "*.archive.gz" |
  Where-Object { $_.LastWriteTime -lt (Get-Date).AddDays(-7) } |
  Remove-Item

# Register the scheduled task
$action = New-ScheduledTaskAction `
  -Execute "powershell.exe" `
  -Argument "-NonInteractive -File C:\Scripts\mongodb_backup.ps1"

$trigger = New-ScheduledTaskTrigger -Daily -At "02:00AM"

$settings = New-ScheduledTaskSettingsSet `
  -ExecutionTimeLimit (New-TimeSpan -Hours 2) `
  -RestartCount 3 `
  -RestartInterval (New-TimeSpan -Minutes 10)

Register-ScheduledTask `
  -TaskName "MongoDBBackup" `
  -Action $action `
  -Trigger $trigger `
  -Settings $settings `
  -RunLevel Highest `
  -Force
```

---

### Backup Script with Cloud Upload (S3)

```bash
#!/bin/bash
# mongodb_backup_s3.sh — Backup MongoDB and upload to AWS S3

set -euo pipefail

# Config
MONGODB_URI="${MONGODB_URI}"
DB_NAME="${DB_NAME:-myapp}"
S3_BUCKET="${S3_BUCKET:-my-company-mongodb-backups}"
S3_PREFIX="backups/${DB_NAME}"
BACKUP_DIR="/tmp/mongodb_backups"
DATE=$(date +%Y-%m-%d_%H%M%S)
ARCHIVE_NAME="backup_${DB_NAME}_${DATE}.archive.gz"
LOCAL_PATH="${BACKUP_DIR}/${ARCHIVE_NAME}"

mkdir -p "$BACKUP_DIR"

echo "=== MongoDB Backup Started: $(date) ==="

# 1. Create backup
mongodump \
  --uri="$MONGODB_URI" \
  --db="$DB_NAME" \
  --archive="$LOCAL_PATH" \
  --gzip \
  --oplog

echo "Backup created: $ARCHIVE_NAME ($(du -sh "$LOCAL_PATH" | cut -f1))"

# 2. Upload to S3
aws s3 cp "$LOCAL_PATH" "s3://${S3_BUCKET}/${S3_PREFIX}/${ARCHIVE_NAME}" \
  --storage-class STANDARD_IA \
  --sse aws:kms

echo "Uploaded to s3://${S3_BUCKET}/${S3_PREFIX}/${ARCHIVE_NAME}"

# 3. Cleanup local file
rm -f "$LOCAL_PATH"

# 4. Delete S3 objects older than 30 days
CUTOFF=$(date -d "30 days ago" +%Y-%m-%d 2>/dev/null || date -v-30d +%Y-%m-%d)
aws s3 ls "s3://${S3_BUCKET}/${S3_PREFIX}/" | \
  awk '{print $4}' | \
  while read key; do
    FILE_DATE=$(echo "$key" | grep -oP '\d{4}-\d{2}-\d{2}' | head -1)
    if [[ "$FILE_DATE" < "$CUTOFF" ]]; then
      aws s3 rm "s3://${S3_BUCKET}/${S3_PREFIX}/${key}"
      echo "Deleted old backup: $key"
    fi
  done

echo "=== Backup Complete: $(date) ==="
```

---

## 9. Cloud Backup Strategy

### AWS S3 Backup Workflow

```
MongoDB → mongodump → Local Archive → AWS S3 → (optional) Glacier
```

**S3 bucket setup:**
```bash
# Create S3 bucket
aws s3 mb s3://mycompany-mongodb-backups --region us-east-1

# Enable versioning (protects against accidental deletion)
aws s3api put-bucket-versioning \
  --bucket mycompany-mongodb-backups \
  --versioning-configuration Status=Enabled

# Enable server-side encryption
aws s3api put-bucket-encryption \
  --bucket mycompany-mongodb-backups \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "aws:kms"
      }
    }]
  }'

# Block all public access
aws s3api put-public-access-block \
  --bucket mycompany-mongodb-backups \
  --public-access-block-configuration \
    "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"

# Set lifecycle policy (transition to Glacier after 30 days, delete after 1 year)
aws s3api put-bucket-lifecycle-configuration \
  --bucket mycompany-mongodb-backups \
  --lifecycle-configuration '{
    "Rules": [{
      "ID": "MongoDBBackupRetention",
      "Status": "Enabled",
      "Filter": {"Prefix": "backups/"},
      "Transitions": [
        {"Days": 30, "StorageClass": "STANDARD_IA"},
        {"Days": 90, "StorageClass": "GLACIER"}
      ],
      "Expiration": {"Days": 365}
    }]
  }'
```

---

### Google Cloud Storage (GCS)

```bash
# Install gsutil / gcloud CLI
# Then authenticate:
gcloud auth login

# Create bucket
gsutil mb -l us-central1 gs://mycompany-mongodb-backups

# Enable versioning
gsutil versioning set on gs://mycompany-mongodb-backups

# Upload backup
gsutil cp /backup/myapp_backup.archive.gz \
  gs://mycompany-mongodb-backups/backups/$(date +%Y/%m/%d)/

# Set lifecycle policy (delete after 90 days)
cat > lifecycle.json << 'EOF'
{
  "rule": [{
    "action": {"type": "Delete"},
    "condition": {"age": 90}
  }]
}
EOF
gsutil lifecycle set lifecycle.json gs://mycompany-mongodb-backups
```

---

### Azure Blob Storage

```bash
# Install Azure CLI
# Login
az login

# Create storage account and container
az storage account create \
  --name mycompanydbbackups \
  --resource-group myResourceGroup \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2

az storage container create \
  --name mongodb-backups \
  --account-name mycompanydbbackups

# Upload backup
az storage blob upload \
  --account-name mycompanydbbackups \
  --container-name mongodb-backups \
  --name "backups/$(date +%Y/%m/%d)/backup.archive.gz" \
  --file /backup/myapp_backup.archive.gz \
  --tier Cool
```

---

### Backup Retention Strategy

```
┌─────────────────────────────────────────────────┐
│           RECOMMENDED RETENTION TIERS           │
├──────────────┬───────────────┬──────────────────┤
│  Frequency   │  Retention    │  Storage Tier    │
├──────────────┼───────────────┼──────────────────┤
│  Hourly      │  24 hours     │  Hot (S3 Standard│
│  Daily       │  30 days      │  Warm (S3-IA)    │
│  Weekly      │  90 days      │  Cold (Glacier)  │
│  Monthly     │  1 year       │  Archive         │
│  Yearly      │  7 years      │  Deep Archive    │
└──────────────┴───────────────┴──────────────────┘
```

---

### Disaster Recovery Planning

**RTO and RPO Definitions:**
- **RPO (Recovery Point Objective):** Maximum acceptable data loss (how old can the backup be?)
- **RTO (Recovery Time Objective):** Maximum acceptable downtime (how long can restore take?)

| Backup Strategy | RPO | RTO | Cost |
|----------------|-----|-----|------|
| Daily mongodump | 24 hours | 2-4 hours | Low |
| Hourly mongodump + S3 | 1 hour | 1-2 hours | Medium |
| Atlas PITR | Seconds | 30-60 min | High |
| Replica set failover | Near-zero | Minutes | High |

**DR Checklist:**
- [ ] Backup stored in different geographic region
- [ ] Restore procedure documented and tested
- [ ] Team trained on restore process
- [ ] RTO and RPO defined and agreed upon
- [ ] Quarterly DR drills scheduled

---

## 10. Docker MongoDB Backup

### Backup from a Running Container

```bash
# Get the container name or ID
docker ps | grep mongo

# Basic backup from container
docker exec <container_name> mongodump \
  --out /tmp/backup/

# Copy backup from container to host
docker cp <container_name>:/tmp/backup ./mongodb_backup

# One-liner: backup directly to host
docker exec <container_name> mongodump \
  --archive \
  --gzip | gzip > ./mongodb_backup_$(date +%Y-%m-%d).archive.gz
```

### Backup with Authentication

```bash
# Backup authenticated MongoDB container
docker exec myapp_mongo mongodump \
  --username admin \
  --password mypassword \
  --authenticationDatabase admin \
  --db myapp \
  --archive \
  --gzip > ./backup_$(date +%Y-%m-%d).archive.gz
```

---

### Docker Compose Backup

```yaml
# docker-compose.yml
version: '3.8'
services:
  mongodb:
    image: mongo:7.0
    container_name: myapp_mongo
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_ROOT_PASSWORD}
    volumes:
      - mongo_data:/data/db
      - ./backups:/backups  # Mount backup directory
    ports:
      - "27017:27017"

  mongo-backup:
    image: mongo:7.0
    depends_on:
      - mongodb
    volumes:
      - ./backups:/backups
    environment:
      MONGO_URI: mongodb://admin:${MONGO_ROOT_PASSWORD}@mongodb:27017
    command: >
      sh -c "mongodump
        --uri=$$MONGO_URI
        --authenticationDatabase=admin
        --archive=/backups/backup_$$(date +%Y-%m-%d_%H%M%S).archive.gz
        --gzip
        && echo 'Backup complete'"
    profiles:
      - backup  # Only runs when explicitly called

volumes:
  mongo_data:
```

```bash
# Run backup service
docker compose run --rm mongo-backup

# Schedule with cron (runs docker compose from host)
# Add to crontab:
0 2 * * * cd /app && docker compose run --rm mongo-backup >> /var/log/mongodb_backup.log 2>&1
```

---

### Volume Backup (Filesystem Level)

```bash
# Stop the container (or use a secondary if replica set)
docker stop myapp_mongo

# Backup the volume using a temporary container
docker run --rm \
  -v myapp_mongo_data:/data/db \
  -v $(pwd)/volume_backup:/backup \
  mongo:7.0 \
  tar czf /backup/mongo_volume_$(date +%Y-%m-%d).tar.gz -C /data/db .

# Restart the container
docker start myapp_mongo
```

---

### Container Restore

```bash
# Restore from archive file into running container
docker exec -i myapp_mongo mongorestore \
  --uri="mongodb://admin:password@localhost:27017" \
  --authenticationDatabase=admin \
  --drop \
  --archive \
  --gzip < ./backup_2024-01-15.archive.gz

# Restore volume from tar backup
docker stop myapp_mongo

docker run --rm \
  -v myapp_mongo_data:/data/db \
  -v $(pwd)/volume_backup:/backup \
  mongo:7.0 \
  sh -c "cd /data/db && tar xzf /backup/mongo_volume_2024-01-15.tar.gz"

docker start myapp_mongo
```

---

## 11. Prisma + MongoDB Backup Strategy

### How Prisma Works with MongoDB

**Prisma** is an ORM (Object-Relational Mapper) that supports MongoDB as a data source. In production, Prisma:
- Connects to MongoDB via a connection string in `.env`
- Runs `prisma migrate` (for relational) or `prisma db push` (for MongoDB)
- `prisma db push` syncs schema and indexes to MongoDB

```
schema.prisma
     ↓ (prisma db push)
MongoDB Collections + Indexes
```

**Key difference:** Prisma with MongoDB does NOT have traditional migrations like SQL. Instead, `prisma db push` modifies the live database directly.

> ⚠️ **Critical:** `prisma db push --force-reset` will **wipe your entire database**. Always backup before running this command.

---

### Safest Backup Workflow Before Migration

```bash
# Step 1: Backup before any schema change
mongodump \
  --uri="$MONGODB_URI" \
  --archive="./backups/pre_migration_$(date +%Y-%m-%d_%H%M%S).archive.gz" \
  --gzip

# Step 2: Record current Prisma schema
cp prisma/schema.prisma ./backups/schema_before_$(date +%Y-%m-%d).prisma

# Step 3: Test the migration on a copy first
mongorestore \
  --uri="mongodb://localhost:27017" \
  --nsFrom="myapp.*" \
  --nsTo="myapp_staging.*" \
  --archive="./backups/pre_migration_backup.archive.gz" \
  --gzip

# Switch DATABASE_URL to staging database
DATABASE_URL="mongodb://localhost:27017/myapp_staging" npx prisma db push

# Step 4: If staging test passes, run on production
npx prisma db push
```

---

### migrate reset Safety

```bash
# DANGER ZONE: This command WIPES your database
# prisma migrate reset  ← DO NOT RUN IN PRODUCTION

# Safe procedure:
# 1. Backup FIRST
mongodump --uri="$MONGODB_URI" --archive=./backup_before_reset.archive.gz --gzip

# 2. Verify backup
mongorestore --uri="mongodb://localhost:27017/verify_test" \
  --archive=./backup_before_reset.archive.gz --gzip --drop
mongosh --eval "use verify_test; db.users.countDocuments()" mongodb://localhost:27017

# 3. Only THEN run the reset
npx prisma migrate reset
```

---

### Development Workflow

```bash
# 1. Seed development database
npx prisma db seed

# 2. Before testing destructive operations
mongodump --uri="$MONGODB_URI" --db=myapp_dev --archive=./dev_backup.archive.gz --gzip

# 3. Safely experiment
npx prisma db push --force-reset
npx prisma db seed

# 4. If you need to revert
mongorestore --uri="$MONGODB_URI" --db=myapp_dev \
  --archive=./dev_backup.archive.gz --gzip --drop
```

---

### Production Workflow (Prisma + MongoDB)

```bash
# Production deployment script
#!/bin/bash
set -e

echo "=== Pre-deployment backup ==="
mongodump \
  --uri="$MONGODB_URI" \
  --archive="./releases/backup_$(date +%Y%m%d_%H%M%S).archive.gz" \
  --gzip

echo "=== Verify backup integrity ==="
mongorestore --uri="mongodb://localhost:27017/verify_$(date +%s)" \
  --archive="$(ls -t ./releases/*.archive.gz | head -1)" \
  --gzip --drop
mongosh --eval "db.users.countDocuments()" \
  mongodb://localhost:27017/verify_$(date +%s)

echo "=== Run Prisma schema push ==="
npx prisma db push

echo "=== Deployment complete ==="
```

---

## 12. Backup Verification

### Why Verification Matters

> **An untested backup is not a backup — it's a hope.**

A backup file that:
- Contains 0 bytes
- Is corrupted mid-transfer
- Was created from the wrong database
- Cannot be restored due to version mismatch

...is completely worthless when disaster strikes. Verify regularly.

---

### Testing Backup Integrity

```bash
# 1. Check file exists and has reasonable size
ls -lh /backup/myapp_backup.archive.gz
# Expect: non-zero size comparable to previous backups

# 2. Test archive integrity (gzip)
gzip -t /backup/myapp_backup.archive.gz && echo "Archive OK" || echo "CORRUPTED!"

# 3. List contents without restoring
mongorestore \
  --archive=/backup/myapp_backup.archive.gz \
  --gzip \
  --dryRun \
  --verbose

# 4. Check mongodump exit code in scripts
mongodump [...] ; echo "Exit code: $?"
# Exit code 0 = success, non-zero = failure
```

---

### Restore Testing (Full Verification)

```bash
#!/bin/bash
# verify_backup.sh — Full backup verification script

BACKUP_FILE="$1"
TEST_DB="backup_verify_$(date +%s)"
MONGODB_TEST_URI="${MONGODB_TEST_URI:-mongodb://localhost:27017}"

if [ -z "$BACKUP_FILE" ]; then
  echo "Usage: $0 <backup_file>"
  exit 1
fi

echo "=== Testing backup: $BACKUP_FILE ==="

# Step 1: Check file integrity
gzip -t "$BACKUP_FILE"
echo "✓ Gzip integrity check passed"

# Step 2: Restore to test database
mongorestore \
  --uri="$MONGODB_TEST_URI" \
  --nsTo="${TEST_DB}.*" \
  --archive="$BACKUP_FILE" \
  --gzip \
  --drop

echo "✓ Restore successful"

# Step 3: Count documents in key collections
USERS=$(mongosh --quiet --eval \
  "use ${TEST_DB}; db.users.countDocuments()" \
  "$MONGODB_TEST_URI")
echo "✓ Users collection: $USERS documents"

ORDERS=$(mongosh --quiet --eval \
  "use ${TEST_DB}; db.orders.countDocuments()" \
  "$MONGODB_TEST_URI")
echo "✓ Orders collection: $ORDERS documents"

# Step 4: Verify minimum document counts (adjust thresholds)
MIN_USERS=100
if [ "$USERS" -lt "$MIN_USERS" ]; then
  echo "⚠️  WARNING: users count ($USERS) below threshold ($MIN_USERS)"
fi

# Step 5: Cleanup test database
mongosh --quiet --eval \
  "use ${TEST_DB}; db.dropDatabase()" \
  "$MONGODB_TEST_URI"

echo "✓ Test database cleaned up"
echo "=== Verification PASSED ==="
```

---

### Checksum Verification

```bash
# Generate checksum when creating backup
sha256sum /backup/myapp_backup.archive.gz > /backup/myapp_backup.archive.gz.sha256

# Verify checksum before restore
sha256sum -c /backup/myapp_backup.archive.gz.sha256
# Output: myapp_backup.archive.gz: OK

# Store checksum in S3 alongside backup
aws s3 cp /backup/myapp_backup.archive.gz s3://my-bucket/backups/
aws s3 cp /backup/myapp_backup.archive.gz.sha256 s3://my-bucket/backups/
```

---

## 13. Security Best Practices

### Encryption at Rest

```bash
# MongoDB native encryption (requires MongoDB Enterprise or Atlas)
# In mongod.conf:
security:
  enableEncryption: true
  encryptionKeyFile: /etc/mongodb/encryption.key

# For community edition: encrypt at the filesystem/volume level
# AWS EBS: enable encryption when creating volume
# Linux: use LUKS full-disk encryption
cryptsetup luksFormat /dev/sdb
cryptsetup luksOpen /dev/sdb mongodb_data
mkfs.ext4 /dev/mapper/mongodb_data
```

### Encryption in Transit

```bash
# Always use TLS/SSL for remote connections
mongodump \
  --uri="mongodb://admin:pass@remote-host:27017" \
  --ssl \
  --sslCAFile /etc/ssl/mongodb-ca.pem \
  --db myapp \
  --archive=backup.archive.gz \
  --gzip
```

---

### Secret Management

```bash
# NEVER store credentials in scripts directly

# Method 1: Environment variables
export MONGODB_URI="mongodb://admin:pass@localhost:27017"
mongodump --uri="$MONGODB_URI" ...

# Method 2: .env file (never commit to git)
cat > .env << 'EOF'
MONGODB_URI=mongodb://admin:pass@localhost:27017
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG
EOF

# Load in script
source .env
mongodump --uri="$MONGODB_URI" ...

# Method 3: AWS Secrets Manager
SECRET=$(aws secretsmanager get-secret-value \
  --secret-id prod/mongodb/uri \
  --query SecretString \
  --output text)
mongodump --uri="$SECRET" ...

# Method 4: HashiCorp Vault
MONGO_URI=$(vault kv get -field=uri secret/mongodb/prod)
mongodump --uri="$MONGO_URI" ...
```

---

### .env Protection

```bash
# Always add .env to .gitignore
echo ".env" >> .gitignore
echo ".env.*" >> .gitignore
echo "*.archive.gz" >> .gitignore
echo "/backup/" >> .gitignore

# Set restrictive permissions
chmod 600 .env
chown root:root .env  # Only root can read/write

# Scan for accidentally committed secrets
git log --all --full-history -- "*.env"
# Use tools like git-secrets, truffleHog, or gitleaks
```

---

### Access Control

```bash
# Create dedicated backup user with minimal permissions
mongosh --eval '
db.createUser({
  user: "backup_agent",
  pwd: "strongBackupPassword123!",
  roles: [
    { role: "backup", db: "admin" },      // For mongodump
    { role: "restore", db: "admin" },     // For mongorestore
    { role: "readAnyDatabase", db: "admin" }
  ]
})
' mongodb://admin:adminpass@localhost:27017/admin

# Never use the root admin user for automated backups
# Create role-specific users for each operation
```

---

### Backup File Permissions

```bash
# Restrict backup directory permissions
chmod 700 /var/backups/mongodb
chown mongodb:mongodb /var/backups/mongodb

# Restrict individual backup files
chmod 600 /var/backups/mongodb/*.archive.gz

# Verify permissions
ls -la /var/backups/mongodb/
```

---

### Ransomware Protection

```
┌─────────────────────────────────────────────────────────────────┐
│                  RANSOMWARE-PROOF BACKUP STRATEGY              │
├─────────────────────────────────────────────────────────────────┤
│  1. Immutable backups: S3 Object Lock (WORM mode)               │
│  2. Offline copy: Tape or disconnected drive (air-gapped)       │
│  3. Cross-region replication: Geographic separation             │
│  4. Separate AWS account: Backups in isolated account           │
│  5. Limited access: Backup agent cannot delete backups          │
│  6. MFA delete: Require MFA to delete S3 objects               │
└─────────────────────────────────────────────────────────────────┘
```

```bash
# Enable S3 Object Lock (WORM — Write Once Read Many)
aws s3api put-object-lock-configuration \
  --bucket mycompany-mongodb-backups \
  --object-lock-configuration '{
    "ObjectLockEnabled": "Enabled",
    "Rule": {
      "DefaultRetention": {
        "Mode": "COMPLIANCE",
        "Days": 30
      }
    }
  }'
# Note: Once set to COMPLIANCE mode, even AWS admins cannot delete files
```

---

## 14. Common Errors & Fixes

### Error 1: Authentication Failed

```
Error: command find requires authentication
Error: Authentication failed.
```

**Causes & Fixes:**
```bash
# Cause 1: Wrong credentials
# Fix: Verify username/password and authSource
mongodump --uri="mongodb://admin:correctpass@localhost:27017/?authSource=admin"

# Cause 2: Wrong authenticationDatabase
mongodump --host localhost --username admin --password pass \
  --authenticationDatabase admin  # Must match where user was created

# Cause 3: User doesn't have backup role
mongosh --eval 'db.grantRolesToUser("myuser", [{role:"backup", db:"admin"}])' \
  mongodb://admin:pass@localhost:27017/admin
```

---

### Error 2: URI Encoding Issue

```
Error: error parsing command line options: couldn't parse connection string
Error: error: invalid character in password
```

**Cause & Fix:**
```bash
# Cause: Special characters in password not encoded
# Password: my@Pass#123!

# Fix: Encode special characters
# @ → %40, # → %23, ! → %21
mongodump --uri="mongodb://admin:my%40Pass%23123%21@localhost:27017"

# Encode using Python:
python3 -c "import urllib.parse; print(urllib.parse.quote('my@Pass#123!', safe=''))"
```

---

### Error 3: Connection Timeout

```
Error: connection() error occurred during connection handshake: dial tcp: i/o timeout
Error: connection refused
```

**Cause & Fix:**
```bash
# Cause 1: MongoDB not running
sudo systemctl status mongod
sudo systemctl start mongod

# Cause 2: Wrong host/port
mongodump --host=127.0.0.1 --port=27017  # Try IP instead of hostname

# Cause 3: Firewall blocking
sudo ufw allow 27017/tcp
# Or check security group rules (AWS) / NSG rules (Azure)

# Cause 4: Atlas IP whitelist
# Go to Atlas → Network Access → Add IP Address → Add your IP

# Cause 5: Increase timeout
mongodump --uri="mongodb://host:27017/?connectTimeoutMS=30000&socketTimeoutMS=30000"
```

---

### Error 4: Restore Conflicts (Duplicate Keys)

```
Error: E11000 duplicate key error collection: myapp.users index: _id_ dup key
```

**Cause & Fix:**
```bash
# Cause: Restoring into a collection that already has data
# Fix 1: Use --drop to clear before restoring
mongorestore --drop /backup/dump/

# Fix 2: Restore to a new database/collection
mongorestore --nsFrom="myapp.*" --nsTo="myapp_restore.*" /backup/dump/

# Fix 3: Use --stopOnError to diagnose which documents conflict
mongorestore --stopOnError /backup/dump/
```

---

### Error 5: Dump Corruption

```
Error: failed to read BSON: invalid BSON
Error: cannot restore from corrupted archive
```

**Cause & Fix:**
```bash
# Cause 1: Transfer corruption — verify checksum
sha256sum -c backup.archive.gz.sha256

# Cause 2: Incomplete backup — check file size
ls -lh backup.archive.gz
# Compare with expected size from previous backups

# Cause 3: Disk full during backup
df -h  # Check available disk space

# Fix: Always verify gzip integrity
gzip -t backup.archive.gz && echo "OK" || echo "CORRUPTED"

# Prevention: Use --archive to create atomic single file
mongodump --archive=backup.archive.gz --gzip
# Avoids partial file issues with directory-based backup
```

---

### Error 6: Version Mismatch

```
Error: collection version does not match
Warning: mongodump created by mongodump 4.4.x and this is 7.0.x
```

**Cause & Fix:**
```bash
# Check versions
mongodump --version
mongorestore --version
mongosh --eval "db.version()" mongodb://localhost:27017

# Rules:
# - Tools should match server version (minor version match)
# - mongorestore can generally restore older backups
# - Never restore a NEWER backup to an OLDER server

# Fix: Match tool versions
# On Ubuntu:
sudo apt-get install -y mongodb-database-tools=100.9.x

# For Atlas: Download the specific version from mongodb.com/try/download/database-tools
```

---

### Error 7: Disk Space During Backup

```
Error: write /backup/dump/mydb/collection.bson: no space left on device
```

**Fix:**
```bash
# Check disk space
df -h

# Check backup directory size estimate (dry run)
mongodump --uri="$MONGODB_URI" --db=myapp --dryRun

# Use compression to reduce space
mongodump --archive=/backup/backup.archive.gz --gzip
# Gzip typically reduces size by 60-80%

# Stream directly to S3 without local storage
mongodump --archive --gzip | aws s3 cp - s3://my-bucket/backup.archive.gz
```

---

### Quick Error Reference Table

| Error | Likely Cause | Quick Fix |
|-------|-------------|-----------|
| `Authentication failed` | Wrong credentials/authSource | Check `--authenticationDatabase admin` |
| `connection refused` | MongoDB not running | `systemctl start mongod` |
| `i/o timeout` | Firewall/network | Check firewall, IP whitelist |
| `invalid character in password` | Unencoded special chars | URL-encode password |
| `duplicate key error` | Data already exists | Add `--drop` flag |
| `no space left on device` | Disk full | Use `--gzip`, free space |
| `invalid BSON` | Corrupted archive | Verify checksum, re-download |
| `version mismatch` | Tool/server version mismatch | Match tool version to server |

---

## 15. Production Architecture

### Enterprise Backup Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                    ENTERPRISE MONGODB BACKUP ARCHITECTURE            │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  PRIMARY CLUSTER (Region A)                                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                 │
│  │  Primary    │  │ Secondary 1 │  │ Secondary 2 │                 │
│  │  (Traffic)  │  │  (Reads)    │  │  (Backup)   │◄── mongodump    │
│  └─────────────┘  └─────────────┘  └─────────────┘                 │
│         │                │                                           │
│         └────────────────┘                                           │
│                  │ Replication                                        │
│                                                                      │
│  REPLICA (Region B - DR)                                             │
│  ┌─────────────┐  ┌─────────────┐                                   │
│  │ Secondary 3 │  │ Secondary 4 │◄── Filesystem Snapshot            │
│  │  (Standby)  │  │  (Backup)   │                                   │
│  └─────────────┘  └─────────────┘                                   │
│                                                                      │
│  BACKUP STORAGE                                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  S3 Bucket (Region A) → Cross-region replication            │   │
│  │  → S3 Bucket (Region B) [Immutable, WORM]                   │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  MONITORING & ALERTING                                               │
│  ┌────────────────────────────────────┐                             │
│  │  Backup success/failure → PagerDuty│                             │
│  │  Backup age alerts → Slack/Email   │                             │
│  │  Storage usage alerts → CloudWatch │                             │
│  └────────────────────────────────────┘                             │
└──────────────────────────────────────────────────────────────────────┘
```

---

### High Availability Configuration

A MongoDB **replica set** provides:
- **Automatic failover** if primary goes down
- **Read scaling** from secondaries
- **Zero-downtime backup** from a hidden secondary

```bash
# mongod.conf for a replica set member
replication:
  replSetName: "rs0"
  oplogSizeMB: 10240  # 10GB oplog for high-write systems

# Initialize replica set
mongosh --eval '
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "mongo1:27017", priority: 2 },
    { _id: 1, host: "mongo2:27017", priority: 1 },
    { _id: 2, host: "mongo3:27017", priority: 0, hidden: true, votes: 0 }
    // Hidden member: only used for backups, gets no traffic
  ]
})
'

# Run backup against hidden member (priority 0, hidden)
mongodump --host mongo3:27017 \
  --oplog \
  --archive=/backup/$(date +%Y-%m-%d).archive.gz \
  --gzip
```

---

### Sharding Backup Strategy

Sharded clusters require special care because data is distributed across shards.

```
┌────────────────────────────────────────────────────────────────┐
│                  SHARDED CLUSTER BACKUP                        │
├────────────────────────────────────────────────────────────────┤
│  WARNING: Do NOT use mongodump against a sharded cluster       │
│  without stopping the balancer first!                          │
│  Inconsistent backup if chunks are migrating during dump.      │
└────────────────────────────────────────────────────────────────┘
```

```bash
# Step 1: Disable the balancer
mongosh --eval 'sh.stopBalancer()' mongodb://mongos:27017

# Verify balancer is stopped
mongosh --eval 'sh.getBalancerState()' mongodb://mongos:27017

# Step 2: Backup config server first
mongodump --host configsvr1:27017 --db config \
  --out /backup/config_$(date +%Y-%m-%d)/

# Step 3: Backup each shard
mongodump --host shard1_primary:27017 \
  --oplog --out /backup/shard1_$(date +%Y-%m-%d)/

mongodump --host shard2_primary:27017 \
  --oplog --out /backup/shard2_$(date +%Y-%m-%d)/

# Step 4: Re-enable balancer
mongosh --eval 'sh.startBalancer()' mongodb://mongos:27017
```

> ⚠️ **Production Recommendation:** For sharded clusters, use **MongoDB Ops Manager** or **Atlas** for backup. Manual sharded cluster backup is complex and error-prone.

---

### Zero Downtime Backup Strategy

```bash
# Zero-downtime backup: use hidden replica set member
# The hidden member has hidden: true and priority: 0
# It receives no client traffic but is a full replica

# Configure hidden member in rs.conf
mongosh --eval '
cfg = rs.conf();
cfg.members[2].hidden = true;
cfg.members[2].priority = 0;
cfg.members[2].votes = 0;
rs.reconfig(cfg);
'

# Backup from hidden member — zero impact on production
mongodump \
  --host hidden-mongo:27017 \
  --readPreference secondary \
  --oplog \
  --archive=/backup/$(date +%Y-%m-%d).archive.gz \
  --gzip
```

---

## 16. Real-World Recommendations

### Startup Setup (< 5 employees, < 50GB data)

```
┌─────────────────────────────────────────────────────────────┐
│  STARTUP SETUP                                              │
│  Budget: ~$20-100/month                                     │
│  Complexity: Low                                            │
├─────────────────────────────────────────────────────────────┤
│  Infrastructure:                                            │
│  - MongoDB Atlas M10 cluster with Atlas Backups enabled     │
│  - Backup frequency: Daily                                  │
│  - Retention: 7 days                                        │
│                                                             │
│  Supplemental:                                              │
│  - Pre-deployment mongodump (manual, before each release)   │
│  - Store in S3 bucket (< $1/month for small databases)      │
│                                                             │
│  Process:                                                   │
│  - Developer manually runs backup before migrations         │
│  - Atlas handles everything else automatically              │
└─────────────────────────────────────────────────────────────┘
```

---

### Freelancer Setup (Personal Projects)

```bash
# Simple, free backup for personal/freelance projects

# Weekly cron backup to local + external drive
0 3 * * 0 mongodump --uri="$MONGODB_URI" \
  --archive=/backups/weekly_$(date +%Y-%m-%d).archive.gz --gzip

# Upload to free tier Backblaze B2 (much cheaper than S3)
0 4 * * 0 b2 upload-file my-bucket /backups/weekly_$(date +%Y-%m-%d).archive.gz \
  backups/weekly_$(date +%Y-%m-%d).archive.gz

# Keep last 4 weekly backups locally
find /backups -name "weekly_*.archive.gz" -mtime +28 -delete
```

---

### Enterprise Setup (100+ employees, > 500GB data)

```
┌─────────────────────────────────────────────────────────────────────┐
│  ENTERPRISE SETUP                                                   │
│  Budget: $500-5000/month                                            │
│  Complexity: High                                                   │
├─────────────────────────────────────────────────────────────────────┤
│  Infrastructure:                                                    │
│  - Atlas Dedicated M50+ with Continuous Cloud Backup               │
│  - PITR enabled (RPO: seconds)                                      │
│  - Cross-region backup replication                                  │
│  - Ops Manager for on-prem or hybrid setups                         │
│                                                                     │
│  Backup Strategy:                                                   │
│  - Hourly: Oplog archiving                                          │
│  - Daily: Full snapshot + S3 cross-region replication               │
│  - Weekly: Full backup to Glacier/cold storage                      │
│  - Monthly: Archive to immutable WORM storage                       │
│                                                                     │
│  Governance:                                                        │
│  - Backup policy documented in runbooks                             │
│  - Quarterly DR drills with RTO measurement                         │
│  - Backup access auditing enabled                                   │
│  - Encryption with KMS-managed keys                                 │
│  - SOC2/HIPAA compliance verification                               │
└─────────────────────────────────────────────────────────────────────┘
```

---

### Low-Budget Setup (Free/Near-Free)

```bash
# Free backup strategy using community tools

# 1. mongodump to local disk (free)
mongodump --archive=/backup/daily.archive.gz --gzip

# 2. Upload to Backblaze B2 (~$0.006/GB/month vs AWS $0.023/GB)
pip install b2
b2 authorize-account <keyID> <applicationKey>
b2 upload-file my-backup-bucket /backup/daily.archive.gz \
  daily/$(date +%Y-%m-%d).archive.gz

# 3. Or use rclone to sync to Google Drive (free 15GB)
rclone copy /backup/daily.archive.gz gdrive:mongodb-backups/

# 4. Or sync to your own NAS/server via rsync
rsync -avz /backup/daily.archive.gz backup@192.168.1.50:/nas/mongodb/
```

---

### Safest Setup (Zero Risk Tolerance)

```
┌─────────────────────────────────────────────────────────────────────┐
│  SAFEST SETUP                                                       │
│  Budget: No limit                                                   │
├─────────────────────────────────────────────────────────────────────┤
│  Multiple independent backup systems:                               │
│                                                                     │
│  Layer 1: Atlas PITR (seconds RPO)                                  │
│  Layer 2: Hourly mongodump → S3 (Region A) with WORM               │
│  Layer 3: Daily filesystem snapshot (Region B cross-region)         │
│  Layer 4: Weekly backup to air-gapped offline storage               │
│  Layer 5: Monthly archive to physical media (tape)                  │
│                                                                     │
│  Verification:                                                      │
│  - Weekly automated restore test to isolated environment            │
│  - Monthly full DR drill                                            │
│  - Checksum verification on all backups                             │
│  - Backup integrity alerts in PagerDuty                             │
│                                                                     │
│  "3-2-1 Rule":                                                      │
│  ✓ 3 copies of data                                                 │
│  ✓ 2 different storage media                                        │
│  ✓ 1 offsite copy                                                   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 17. Best Practices Checklist

### Backup Creation

- [ ] Backups run automatically on a schedule (daily minimum)
- [ ] `--oplog` flag used for production mongodump backups
- [ ] `--gzip` compression used to reduce storage size
- [ ] `--archive` format used for easier cloud storage
- [ ] Backups taken from a secondary/hidden replica set member
- [ ] Backup script exits with non-zero code on failure
- [ ] Backup failures trigger alerts (email, Slack, PagerDuty)
- [ ] Backup file sizes monitored for anomalous shrinkage
- [ ] Timestamps included in backup file names
- [ ] SHA256 checksum generated alongside each backup

### Backup Storage

- [ ] Backups stored in a different location than the database
- [ ] Backups stored in a different geographic region (cloud)
- [ ] S3 versioning or equivalent enabled
- [ ] Immutable/WORM storage enabled for critical backups
- [ ] Backup storage is encrypted at rest
- [ ] Retention policy defined and enforced (e.g., 7 daily, 4 weekly, 12 monthly)
- [ ] Old backups automatically deleted per retention policy
- [ ] Storage costs monitored

### Security

- [ ] Database credentials stored in environment variables, not scripts
- [ ] `.env` files added to `.gitignore`
- [ ] Backup agent uses a dedicated MongoDB user with minimal roles
- [ ] Backup files have restrictive permissions (600 or 400)
- [ ] Backup directory has restrictive permissions (700)
- [ ] TLS/SSL used for all remote backup connections
- [ ] Backup credentials rotated regularly

### Verification

- [ ] Backup integrity checked (gzip -t) after every backup
- [ ] Backup restore tested at least monthly
- [ ] Document count verification after test restore
- [ ] Restore procedure is documented in a runbook
- [ ] Team knows how to perform emergency restore
- [ ] RTO and RPO defined and documented
- [ ] DR drills scheduled quarterly

### Operations

- [ ] Backup logs stored and reviewed regularly
- [ ] Backup monitoring dashboard exists
- [ ] Pre-deployment backup procedure followed
- [ ] Database version documented (for version-compatible restore)
- [ ] Backup retention costs reviewed quarterly
- [ ] Atlas backup policies reviewed after major infrastructure changes

---

## 18. Cheat Sheet

### Most Useful Backup Commands

```bash
# Full backup (all databases), compressed archive
mongodump --uri="$MONGODB_URI" \
  --archive=backup_$(date +%Y-%m-%d).archive.gz \
  --gzip --oplog

# Single database backup
mongodump --uri="$MONGODB_URI" --db myapp \
  --archive=myapp_$(date +%Y-%m-%d).archive.gz --gzip

# Single collection backup
mongodump --uri="$MONGODB_URI" --db myapp --collection users \
  --archive=users_$(date +%Y-%m-%d).archive.gz --gzip

# Atlas backup
mongodump \
  --uri="mongodb+srv://user:pass%40word@cluster.abc.mongodb.net" \
  --db myapp --archive=atlas_backup.archive.gz --gzip

# Stream backup directly to S3 (no local storage needed)
mongodump --uri="$MONGODB_URI" --archive --gzip | \
  aws s3 cp - s3://my-bucket/backups/backup_$(date +%Y-%m-%d).archive.gz

# Backup from Docker container
docker exec myapp_mongo mongodump --archive --gzip | \
  gzip > backup_$(date +%Y-%m-%d).archive.gz

# Verify backup integrity
gzip -t backup.archive.gz && echo "OK" || echo "CORRUPTED"
sha256sum backup.archive.gz > backup.archive.gz.sha256
```

---

### Quick Restore Commands

```bash
# Restore full backup (merge — doesn't delete existing)
mongorestore --uri="$MONGODB_URI" \
  --archive=backup.archive.gz --gzip

# Restore with drop (clean restore — recommended)
mongorestore --uri="$MONGODB_URI" \
  --archive=backup.archive.gz --gzip --drop

# Restore single collection
mongorestore --uri="$MONGODB_URI" \
  --archive=backup.archive.gz --gzip \
  --nsInclude="myapp.users" --drop

# Restore to a different database
mongorestore --uri="$MONGODB_URI" \
  --archive=backup.archive.gz --gzip \
  --nsFrom="myapp.*" --nsTo="myapp_restore.*"

# Restore from Docker
docker exec -i myapp_mongo mongorestore \
  --uri="mongodb://admin:pass@localhost:27017" \
  --authenticationDatabase=admin --drop \
  --archive --gzip < backup.archive.gz

# Dry run (test without actually restoring)
mongorestore --uri="$MONGODB_URI" \
  --archive=backup.archive.gz --gzip --dryRun
```

---

### Troubleshooting Commands

```bash
# Check MongoDB connection
mongosh "mongodb://admin:pass@localhost:27017/admin" --eval "db.runCommand({ping:1})"

# Check MongoDB server version
mongosh --eval "db.version()" mongodb://localhost:27017

# Check tool versions
mongodump --version && mongorestore --version

# Check replica set status
mongosh --eval "rs.status()" mongodb://localhost:27017

# Check oplog size and usage
mongosh --eval "
  use local;
  var stats = db.oplog.rs.stats();
  print('Oplog size:', stats.maxSize / 1024 / 1024, 'MB');
  print('Oplog used:', stats.size / 1024 / 1024, 'MB');
" mongodb://admin:pass@localhost:27017

# List databases
mongosh --eval "db.adminCommand({listDatabases:1})" mongodb://admin:pass@localhost:27017

# Check disk space
df -h /var/backups/

# Check MongoDB logs
tail -100 /var/log/mongodb/mongod.log

# Test AWS S3 access
aws s3 ls s3://my-backup-bucket/

# URL-encode a password
python3 -c "import urllib.parse; print(urllib.parse.quote('my@Pass!', safe=''))"

# Count documents after restore (verify)
mongosh --eval "
  use myapp;
  db.getCollectionNames().forEach(function(col) {
    print(col + ':', db[col].countDocuments());
  });
" mongodb://localhost:27017

# Force a specific atlas backup
# (Done via Atlas UI: Cluster → ... → Take Snapshot Now)
```

---

### Environment Variables Template

```bash
# .env template — copy and fill in your values
# NEVER commit this file to git

# MongoDB
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/mydb?retryWrites=true&w=majority
DB_NAME=myapp

# AWS S3
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
AWS_DEFAULT_REGION=us-east-1
S3_BUCKET=mycompany-mongodb-backups

# Backup config
BACKUP_DIR=/var/backups/mongodb
RETENTION_DAYS=7
```

---

### Quick Reference: Backup Strategy by Tier

| Tier | Daily Backup | Retention | Storage | Cost/Month |
|------|-------------|-----------|---------|------------|
| Free | mongodump → local | 7 days | Local disk | $0 |
| Starter | mongodump → S3 | 30 days | S3 Standard-IA | ~$2-10 |
| Professional | Atlas M10 Backup | 7 days PITR | Atlas Managed | ~$30-50 |
| Business | Atlas M30 + S3 | 30 days PITR | Atlas + S3 | ~$150-300 |
| Enterprise | Atlas + Ops Manager | 90 days PITR | Multi-region | $500+ |

---

> **Final Note:** No backup strategy is perfect, but any backup strategy is infinitely better than none. Start simple, automate early, and verify regularly. The best backup is the one you actually have when you need it.

---

*Guide Version: 1.0 | MongoDB Database Tools: 100.x | MongoDB Server: 7.x*
*Last Updated: 2024*
