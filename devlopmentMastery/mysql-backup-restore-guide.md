# MySQL Backup & Restore Master Guide

> **A complete, production-grade reference from beginner to enterprise level**

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [MySQL Backup Types](#2-mysql-backup-types)
3. [Best Industry Standard Approaches](#3-best-industry-standard-approaches)
4. [Installation & Setup](#4-installation--setup)
5. [mysqldump Deep Dive](#5-mysqldump-deep-dive)
6. [mysql Restore Deep Dive](#6-mysql-restore-deep-dive)
7. [GUI Backup Methods](#7-gui-backup-methods)
8. [Automation](#8-automation)
9. [Cloud Backup Strategy](#9-cloud-backup-strategy)
10. [Docker MySQL Backup](#10-docker-mysql-backup)
11. [Prisma + MySQL Backup Strategy](#11-prisma--mysql-backup-strategy)
12. [Security Best Practices](#12-security-best-practices)
13. [Performance Optimization](#13-performance-optimization)
14. [Common Errors & Fixes](#14-common-errors--fixes)
15. [Disaster Recovery Planning](#15-disaster-recovery-planning)
16. [Production Architectures](#16-production-architectures)
17. [Best Practices Checklist](#17-best-practices-checklist)
18. [Cheat Sheet](#18-cheat-sheet)

---

## 1. Introduction

### What Is a Database Backup?

A **database backup** is a copy of your data — schema, rows, indexes, routines, and configuration — stored separately from the primary server so it can be recovered if the original is lost, corrupted, or destroyed.

Think of it as insurance: you hope you never need it, but when disaster strikes, the presence or absence of a good backup determines whether your business survives.

### Why Backups Matter

Data loss happens for many reasons:

- **Human error** — accidental `DROP TABLE` or `DELETE` without a `WHERE` clause
- **Hardware failure** — disk corruption, RAID array failure, SSD failure
- **Ransomware / cyberattack** — malicious encryption of database files
- **Deployment bugs** — a broken migration that deletes or corrupts rows
- **Data center outage** — fire, flood, power failure
- **Cloud provider incidents** — even AWS and GCP have had regional failures

> ⚠️ **Stat:** According to Veeam's 2023 Data Protection Report, 76% of organizations experienced at least one outage in the prior 12 months. 93% of companies that lose their data for 10+ days file for bankruptcy within a year (University of Texas research).

### Real-World Disaster Examples

| Incident | What Happened | Lesson |
|----------|---------------|--------|
| GitLab (2017) | DB admin accidentally deleted production database. Backups were not tested — most were invalid. | Test your restores. Untested backups are not backups. |
| Code Spaces (2014) | Attacker deleted all AWS resources including backups. Company shut down. | Store backups in isolated, access-controlled locations. |
| MySpace (2019) | Server migration corrupted 3 years of user music uploads. No recovery was possible. | Retain multiple backup generations. |
| Pixar (1998) | `rm -rf` accidentally ran on Toy Story 2 render. A remote backup on a team member's home machine saved the film. | Offsite backups save businesses. |

### Backup vs Replication

These are often confused but serve different purposes:

| Feature | Backup | Replication |
|--------|--------|-------------|
| **Purpose** | Point-in-time recovery | High availability / read scaling |
| **Protects against data loss?** | Yes | No — errors replicate too |
| **Protects against `DROP TABLE`?** | Yes | No — propagates immediately |
| **Can restore to past state?** | Yes | No |
| **Storage location** | Separate storage/server | Live replica server |
| **Recovery time** | Minutes to hours | Seconds (failover) |

> **Rule:** Replication is not a backup. You need both.

### RPO and RTO

Two critical concepts used in every enterprise backup strategy:

**RPO — Recovery Point Objective**
The maximum amount of data loss that is acceptable, measured in time.
- RPO = 1 hour means: you can afford to lose up to 1 hour of data
- Determines backup *frequency*

**RTO — Recovery Time Objective**
The maximum acceptable downtime after a failure.
- RTO = 4 hours means: the system must be back online within 4 hours
- Determines backup *type* and *location*

```
Timeline Example:

  Last backup     Failure      Recovery complete
       |              |               |
  ─────┼──────────────┼───────────────┼─────
       │◄── RPO ─────►│               │
                      │◄─── RTO ─────►│
```

Define these numbers before choosing your backup strategy. A financial system might require RPO = 0 minutes and RTO = 30 minutes. A dev blog might accept RPO = 24 hours and RTO = 8 hours.

---

## 2. MySQL Backup Types

### Logical Backups

A logical backup exports the **data and schema as SQL statements** (INSERT, CREATE TABLE, etc.) that can be re-executed on any compatible MySQL instance.

**Tool:** `mysqldump`, `mysqlpump`

**Pros:**
- Portable across MySQL versions and platforms
- Human-readable and editable
- Selective: backup specific databases, tables, or rows
- Compatible with different storage engines

**Cons:**
- Slow for large databases (must read every row)
- Large file sizes (SQL text is verbose)
- Restore is slow (must re-execute all statements)
- Not suitable for very large datasets (100GB+) without optimization

**Use cases:** Development databases, small-to-medium production, cross-server migrations, selective table exports

---

### Physical Backups

A physical backup copies the **actual database files on disk** — InnoDB tablespace files, `.frm` files, binary logs, etc.

**Tools:** Percona XtraBackup, MySQL Enterprise Backup, filesystem snapshot

**Pros:**
- Much faster backup and restore for large databases
- Minimal server load
- Binary-exact copy
- Supports incremental backups efficiently

**Cons:**
- Only compatible with the same MySQL version and OS
- Not human-readable
- Requires disk access or agent
- More complex setup

**Use cases:** Large production databases, high-availability environments, rapid disaster recovery

---

### Full Backups

Captures **all data** in the target scope (entire server, specific database, or specific table) at a single point in time.

```
Full Backup (Monday):
[DB State] ──────────────────────────► [Backup File: 100% of data]
```

**Pros:** Simple to restore, self-contained  
**Cons:** Large, time-consuming, high I/O impact  
**Frequency:** Weekly (combined with incrementals), or daily for smaller DBs

---

### Incremental Backups

Captures only **data changed since the last backup** (full or incremental). MySQL implements this via binary logs.

```
Monday:   [Full Backup]        ──► backup_full_monday.sql
Tuesday:  [Changes since Mon]  ──► backup_inc_tuesday.binlog
Wednesday:[Changes since Tue]  ──► backup_inc_wednesday.binlog
```

**Restore process:** Apply full backup, then replay each incremental in order.

**Pros:** Small backup files, fast backup  
**Cons:** Complex restore (must chain all incrementals), risk of corruption if one file is lost

---

### Differential Backups

Captures all **data changed since the last full backup** (not the last differential).

```
Monday:    [Full Backup]             ──► 100% of data
Tuesday:   [Changes since Monday]   ──► +5% of data
Wednesday: [Changes since Monday]   ──► +12% of data (not just since Tue)
```

**Restore process:** Apply full backup, then apply only the latest differential.

**Pros:** Simpler restore than incremental (only 2 files needed)  
**Cons:** Differentials grow larger over time  

| Type | Backup Speed | Restore Speed | Storage |
|------|-------------|---------------|---------|
| Full | Slowest | Fastest | Largest |
| Incremental | Fastest | Slowest | Smallest |
| Differential | Medium | Medium | Medium |

---

### Snapshots

A **snapshot** captures the database filesystem at a point in time using OS/cloud-level snapshot technology (LVM, AWS EBS, ZFS).

**Pros:** Near-instant backup, no server load, consistent state  
**Cons:** Platform-dependent, requires filesystem support, not portable  
**Use cases:** AWS RDS, Azure SQL, on-prem LVM volumes

---

### Hot vs Cold Backups

| Type | DB State | Risk | Performance Impact |
|------|----------|------|-------------------|
| **Hot** | Running, accepting connections | Low (if done correctly) | Some I/O overhead |
| **Warm** | Running, no writes allowed | Low | Moderate |
| **Cold** | Server stopped | None | Full server downtime |

For production systems, **hot backups are mandatory**. Use `--single-transaction` with InnoDB for consistent hot backups without locking.

---

## 3. Best Industry Standard Approaches

### mysqldump

The built-in MySQL logical backup tool. Available on every MySQL installation.

**Best for:** Development, small-to-medium production (< 10GB), portability, scheduled exports  
**Not ideal for:** Multi-terabyte databases, zero-downtime production requirements

### mysqlpump

An improved, parallelized version of mysqldump introduced in MySQL 5.7.

**Advantages over mysqldump:** Parallel export, progress indicators, compressed output  
**Limitation:** Less mature, fewer options for point-in-time recovery

### MySQL Enterprise Backup (MEB)

Oracle's official physical backup tool, included with MySQL Enterprise Edition.

**Best for:** Enterprise Oracle MySQL users, large databases, incremental backups  
**Cost:** Paid license required

### Percona XtraBackup

The de facto open-source standard for physical MySQL backups. Used by millions of production deployments.

**Best for:** Large databases, hot physical backups, incremental backups  
**Supports:** MySQL, Percona Server, MariaDB  
**Cost:** Free and open source

### Filesystem Snapshots

LVM or ZFS snapshots of the MySQL data directory.

**Best for:** On-premise servers with LVM/ZFS, ultra-fast snapshots  
**Requirement:** MySQL data must be on LVM or ZFS volume; must flush and lock briefly

### Cloud Snapshots

Automated snapshots provided by cloud providers (AWS RDS, Azure, GCP Cloud SQL).

**Best for:** Managed cloud database deployments  
**Limitation:** Tied to the cloud provider, less control over scheduling

### Replication-Based Backups

Take backups from a **replica server** rather than the primary, offloading I/O from production.

**Best for:** High-traffic production environments, zero primary impact backups  
**Requirement:** A replica must be maintained and monitored

---

### Recommended Setup by Scale

#### Startup / Solo Developer

```
Strategy:
  - Daily mysqldump (automated cron job)
  - Compress and upload to S3 or Google Cloud Storage
  - Retain 7 daily + 4 weekly backups
  - Test restore monthly

Tools: mysqldump + cron + AWS S3
```

#### Freelancer / Small Team (< 50GB)

```
Strategy:
  - Daily full mysqldump from replica or off-peak hours
  - Nightly binary log backup for point-in-time recovery
  - Backups stored on separate server + cloud
  - Test restore quarterly with automated verification

Tools: mysqldump + binary logs + S3 with lifecycle policies
```

#### Enterprise / High Availability (> 100GB)

```
Strategy:
  - Weekly full physical backup (Percona XtraBackup)
  - Daily incremental physical backups
  - Continuous binary log shipping to offsite storage
  - Backups taken from dedicated replica
  - Automated daily restore verification
  - Multi-region backup storage
  - RPO target: 5 minutes, RTO target: 30 minutes

Tools: Percona XtraBackup + MySQL replication + S3/GCS with cross-region replication
```

---

## 4. Installation & Setup

### Linux (Ubuntu/Debian)

```bash
# Install MySQL server and client tools
sudo apt update
sudo apt install mysql-server mysql-client -y

# Verify installation
mysql --version
mysqldump --version

# Install Percona XtraBackup (optional, for physical backups)
wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb
sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb
sudo apt update
sudo apt install percona-xtrabackup-80 -y   # For MySQL 8.0
xtrabackup --version
```

### Linux (RHEL/CentOS/Fedora)

```bash
# Install MySQL tools
sudo dnf install mysql mysql-server -y

# Install Percona XtraBackup
sudo yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
sudo yum install percona-xtrabackup-80 -y

# Verify
mysql --version
xtrabackup --version
```

### macOS

```bash
# Install via Homebrew
brew install mysql

# Verify
mysql --version
mysqldump --version

# Start/stop MySQL service
brew services start mysql
brew services stop mysql
```

### Windows

```powershell
# Option 1: Download MySQL Installer from https://dev.mysql.com/downloads/installer/
# Run the installer and select "MySQL Server" + "MySQL Shell"

# Option 2: Chocolatey
choco install mysql -y

# Add MySQL to PATH (if not done by installer)
# Open System Properties > Advanced > Environment Variables
# Add to Path: C:\Program Files\MySQL\MySQL Server 8.0\bin

# Verify in PowerShell
mysql --version
mysqldump --version
```

### Verify Tools Available

```bash
# Check all relevant MySQL tools
which mysql mysqldump mysqlpump mysqlcheck mysqlimport

# Check MySQL server status
mysqladmin -u root -p status

# Check MySQL version (server)
mysql -u root -p -e "SELECT VERSION();"

# Check binary logging (required for point-in-time recovery)
mysql -u root -p -e "SHOW VARIABLES LIKE 'log_bin';"
mysql -u root -p -e "SHOW BINARY LOGS;"
```

---

## 5. mysqldump Deep Dive

### Syntax Overview

```bash
mysqldump [options] [database_name] [table_name] > output_file.sql
```

### Authentication Options

```bash
# Inline password (not recommended for scripts — visible in process list)
mysqldump -u root -pYourPassword dbname > backup.sql

# Prompt for password (safer for interactive use)
mysqldump -u root -p dbname > backup.sql

# Using .my.cnf config file (recommended for scripts)
# Create ~/.my.cnf:
# [client]
# user=backupuser
# password=yourpassword
mysqldump dbname > backup.sql

# Remote server
mysqldump -h 192.168.1.100 -P 3306 -u root -p dbname > backup.sql
```

### Full Database Backup

```bash
# Backup a single database
mysqldump -u root -p myapp_db > myapp_db_backup.sql

# With timestamp in filename
mysqldump -u root -p myapp_db > myapp_db_$(date +%Y%m%d_%H%M%S).sql

# Recommended production flags (InnoDB)
mysqldump \
  --single-transaction \
  --routines \
  --triggers \
  --events \
  --set-gtid-purged=OFF \
  -u root -p myapp_db > myapp_db_full.sql
```

> ⚠️ `--single-transaction` uses a transaction snapshot for InnoDB tables — no table locks needed. Do NOT use with MyISAM tables; use `--lock-tables` instead.

### Single Table Backup

```bash
# Backup one table from a database
mysqldump -u root -p myapp_db users > users_backup.sql

# Multiple specific tables
mysqldump -u root -p myapp_db users orders products > tables_backup.sql
```

### Multiple Database Backup

```bash
# Backup multiple specific databases
mysqldump --databases db1 db2 db3 -u root -p > multi_db_backup.sql

# Note: --databases includes CREATE DATABASE statements
# Without --databases, you need to create the DB manually before restoring
```

### All Databases Backup

```bash
# Backup everything
mysqldump --all-databases -u root -p > all_databases.sql

# All databases with all options
mysqldump \
  --all-databases \
  --single-transaction \
  --routines \
  --triggers \
  --events \
  -u root -p > full_server_backup.sql
```

### Compressed Backup

```bash
# Compress output with gzip (reduces size by 70-90%)
mysqldump -u root -p myapp_db | gzip > myapp_db_$(date +%Y%m%d).sql.gz

# Compress with higher ratio (bzip2)
mysqldump -u root -p myapp_db | bzip2 > myapp_db.sql.bz2

# Use pigz for faster parallel compression
mysqldump -u root -p myapp_db | pigz > myapp_db.sql.gz

# Verify compressed file integrity
gzip -t myapp_db.sql.gz && echo "File OK"
```

### Backup with Routines (Stored Procedures & Functions)

```bash
# Include stored procedures and functions
mysqldump --routines -u root -p myapp_db > myapp_with_routines.sql

# Routines only (no data, no tables)
mysqldump --routines --no-create-info --no-data --no-create-db \
  -u root -p myapp_db > routines_only.sql
```

### Backup with Triggers

```bash
# Triggers are included by default; to explicitly include:
mysqldump --triggers -u root -p myapp_db > myapp_with_triggers.sql

# Skip triggers (useful for migration scenarios)
mysqldump --skip-triggers -u root -p myapp_db > myapp_no_triggers.sql
```

### Backup with Events

```bash
# Include scheduled events (disabled by default)
mysqldump --events -u root -p myapp_db > myapp_with_events.sql
```

### Transaction-Consistent Backup (InnoDB)

```bash
# --single-transaction is the key for consistent hot backups on InnoDB
mysqldump \
  --single-transaction \
  --master-data=2 \
  -u root -p myapp_db > consistent_backup.sql

# --master-data=2: Adds binary log position as a comment
# Needed for point-in-time recovery
```

### Lock Tables (MyISAM)

```bash
# For MyISAM tables — will lock all tables during backup
mysqldump --lock-tables -u root -p myapp_db > myisam_backup.sql

# Lock all tables across all databases (for consistency across DBs)
mysqldump --lock-all-tables -u root -p myapp_db > myapp_locked.sql
```

### Schema-Only Backup (No Data)

```bash
# Export structure without any data
mysqldump --no-data -u root -p myapp_db > schema_only.sql

# Useful for: documentation, migrating to new server, diff comparison
```

### Data-Only Backup (No Schema)

```bash
# Export data without CREATE TABLE statements
mysqldump --no-create-info -u root -p myapp_db > data_only.sql

# Useful for: refreshing data on a pre-existing schema
```

### Advanced Options Reference

```bash
# Add DROP TABLE before CREATE TABLE (safe re-import)
mysqldump --add-drop-table -u root -p myapp_db > safe_backup.sql

# Disable foreign key checks in the dump (for easier restoration)
mysqldump --single-transaction --disable-keys -u root -p myapp_db > backup.sql

# Include a WHERE clause to export partial data
mysqldump -u root -p myapp_db users --where="created_at > '2024-01-01'" > recent_users.sql

# Extended inserts (default, groups multiple rows per INSERT)
mysqldump --extended-insert -u root -p myapp_db > backup.sql

# One INSERT per row (slower but easier to diff/edit)
mysqldump --skip-extended-insert -u root -p myapp_db > readable_backup.sql

# Output in hex for binary data columns
mysqldump --hex-blob -u root -p myapp_db > backup_with_blobs.sql

# Include GTID information (MySQL 5.6+)
mysqldump --set-gtid-purged=ON -u root -p myapp_db > gtid_backup.sql
```

---

## 6. mysql Restore Deep Dive

### Restore Full Database

```bash
# Basic restore
mysql -u root -p myapp_db < backup.sql

# If the database doesn't exist yet, create it first
mysql -u root -p -e "CREATE DATABASE myapp_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
mysql -u root -p myapp_db < backup.sql

# Restore from a backup made with --all-databases or --databases
# (These backups contain CREATE DATABASE statements)
mysql -u root -p < all_databases.sql
```

### Restore a Single Table

```bash
# You must have the table backup made with:
# mysqldump -u root -p myapp_db users > users_backup.sql

# Then restore:
mysql -u root -p myapp_db < users_backup.sql

# To restore a specific table from a full DB dump, extract it first:
grep -n "Table structure for table \`users\`" full_backup.sql
# Note the line numbers, then use sed to extract
# Or use a tool like mysqlslicer for precision extraction
```

### Restore Compressed Backup

```bash
# Restore gzip-compressed backup
gunzip < myapp_db.sql.gz | mysql -u root -p myapp_db

# Or using process substitution
mysql -u root -p myapp_db < <(gunzip -c myapp_db.sql.gz)

# Restore bzip2 backup
bunzip2 < myapp_db.sql.bz2 | mysql -u root -p myapp_db

# Restore with progress using pv
gunzip -c myapp_db.sql.gz | pv | mysql -u root -p myapp_db
```

### Restore Specific Schema Only

```bash
# If your backup contains multiple databases, restore just one schema
mysql -u root -p --one-database target_db < all_databases.sql

# Or use grep to extract just the relevant portion
# (less reliable for large files; use --one-database instead)
```

### Importing SQL Files

```bash
# Using SOURCE inside mysql CLI (interactive)
mysql -u root -p
mysql> USE myapp_db;
mysql> SOURCE /path/to/backup.sql;

# Using redirect (recommended for scripts)
mysql -u root -p myapp_db < /path/to/backup.sql

# Show progress with verbose
mysql -u root -p myapp_db -v < backup.sql

# Ignore errors and continue
mysql -u root -p --force myapp_db < backup.sql

# Suppress output for cleaner script logs
mysql -u root -p myapp_db < backup.sql 2>&1 | grep -v "Warning:"
```

### Point-in-Time Recovery (PITR)

PITR restores data to a specific moment in time using binary logs.

```bash
# Step 1: Restore the last full backup
mysql -u root -p myapp_db < full_backup.sql

# Step 2: Find binary log files after the backup
mysql -u root -p -e "SHOW BINARY LOGS;"

# Step 3: Apply binary logs up to a specific point in time
mysqlbinlog \
  --start-datetime="2024-03-15 09:00:00" \
  --stop-datetime="2024-03-15 11:30:00" \
  /var/lib/mysql/binlog.000042 | mysql -u root -p myapp_db

# Step 4: Verify data integrity after recovery
mysql -u root -p -e "SELECT COUNT(*) FROM myapp_db.users;"
```

### Common Restore Mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| Not creating the DB first | `ERROR 1049: Unknown database` | `CREATE DATABASE dbname;` before restoring |
| Restoring with wrong charset | Data corruption, garbled text | Ensure `CHARACTER SET utf8mb4` matches source |
| Ignoring FK constraints | Insert order violations | Add `SET FOREIGN_KEY_CHECKS=0;` at top of SQL, reset after |
| Restoring while app is running | Partial reads by app | Put app in maintenance mode during restore |
| Not verifying after restore | Silent data corruption | Always run row count and spot checks |
| Restoring to wrong database | Overwrites wrong data | Double-check DB name with `USE` confirmation |

---

## 7. GUI Backup Methods

### MySQL Workbench

**Backup:**
1. Open MySQL Workbench → Connect to server
2. Server → Data Export
3. Select databases/tables to export
4. Choose Export Options:
   - `Dump Structure and Data` (most common)
   - `Dump Data Only`
   - `Dump Structure Only`
5. Enable: Include Stored Procedures, Include Events, Include Triggers
6. Choose: `Export to Self-Contained File` or `Export to Dump Project Folder`
7. Click **Start Export**

**Restore:**
1. Server → Data Import
2. Choose `Import from Self-Contained File`
3. Select target database or `New...` to create one
4. Click **Start Import**

---

### phpMyAdmin

**Backup (Export):**
1. Select database in left panel
2. Click **Export** tab
3. Method: `Quick` (default) or `Custom` (more options)
4. Custom options:
   - Format: SQL
   - Structure: `CREATE TABLE`, `DROP TABLE IF EXISTS`
   - Data: `INSERT`, compression (gzip/zip)
5. Click **Go** → downloads `.sql.gz` file

**Restore (Import):**
1. Select or create target database
2. Click **Import** tab
3. Choose file (up to `upload_max_filesize` PHP limit — often 2–8MB by default)
4. For large files, use `mysql` CLI or increase PHP limits

> ⚠️ phpMyAdmin has a file size upload limit set by PHP configuration. For databases > 50MB, use `mysql` CLI or increase `upload_max_filesize` and `post_max_size` in `php.ini`.

---

### DBeaver

**Backup:**
1. Right-click database → Tools → Dump Database
2. Configure: output format, compression, objects to include
3. Click **Start** — uses native `mysqldump` under the hood

**Restore:**
1. Right-click database → Tools → Restore Database
2. Select dump file
3. Configure options → Click **Start**

DBeaver is excellent for developers — supports SSH tunnels, multiple DBs, and visual query plans.

---

### TablePlus

**Backup:**
1. File → Export → SQL dump
2. Choose scope: entire database, selected tables
3. Options: structure, data, drop-if-exists

**Restore:**
1. File → Import → SQL or JSON
2. Select target connection and database

TablePlus is fast, lightweight, and popular on macOS/Windows.

---

### HeidiSQL

**Backup:**
1. Tools → Export database as SQL
2. Select tables, choose options (DROP + CREATE, INSERT data, etc.)
3. Save to file or clipboard

**Restore:**
1. File → Run SQL file
2. Select `.sql` file → Execute

---

### GUI vs CLI Comparison

| Feature | GUI | CLI |
|---------|-----|-----|
| Learning curve | Low | Medium |
| Automation | Limited | Full |
| Large databases | Slow / unreliable | Recommended |
| Scheduling | Manual or OS-level | Native cron/Task Scheduler |
| CI/CD integration | No | Yes |
| Production-grade | Partial | Yes |
| Free options | Yes | Yes |

**Verdict:** GUIs are great for development and exploration. Production environments should always use CLI-based, automated, scripted backups.

---

## 8. Automation

### Linux Cron Jobs

```bash
# Edit crontab
crontab -e

# Run backup every day at 2:00 AM
0 2 * * * /home/deploy/scripts/mysql_backup.sh

# Run every 6 hours
0 */6 * * * /home/deploy/scripts/mysql_backup.sh

# Run at 2:00 AM Monday (weekly full backup)
0 2 * * 1 /home/deploy/scripts/mysql_full_backup.sh
```

### Backup Shell Script (Linux/macOS)

```bash
#!/bin/bash
# mysql_backup.sh — Production MySQL backup with rotation

# ─── Configuration ────────────────────────────────────────────────────────────
DB_USER="backupuser"
DB_PASS="yourpassword"       # Better: use ~/.my.cnf or secrets manager
DB_NAME="myapp_db"
DB_HOST="127.0.0.1"
BACKUP_DIR="/var/backups/mysql"
RETENTION_DAYS=14            # Keep backups for 14 days
S3_BUCKET="s3://my-company-backups/mysql"
LOG_FILE="/var/log/mysql_backup.log"

# ─── Setup ───────────────────────────────────────────────────────────────────
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
FILENAME="${DB_NAME}_${TIMESTAMP}.sql.gz"
FILEPATH="${BACKUP_DIR}/${FILENAME}"

mkdir -p "$BACKUP_DIR"

log() {
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# ─── Backup ──────────────────────────────────────────────────────────────────
log "Starting backup of ${DB_NAME}"

mysqldump \
  -h "$DB_HOST" \
  -u "$DB_USER" \
  -p"$DB_PASS" \
  --single-transaction \
  --routines \
  --triggers \
  --events \
  --set-gtid-purged=OFF \
  "$DB_NAME" | gzip > "$FILEPATH"

if [ $? -eq 0 ]; then
  log "Backup succeeded: ${FILEPATH} ($(du -sh "$FILEPATH" | cut -f1))"
else
  log "ERROR: Backup FAILED"
  exit 1
fi

# ─── Upload to S3 ────────────────────────────────────────────────────────────
if command -v aws &> /dev/null; then
  aws s3 cp "$FILEPATH" "${S3_BUCKET}/${FILENAME}"
  if [ $? -eq 0 ]; then
    log "Uploaded to S3: ${S3_BUCKET}/${FILENAME}"
  else
    log "WARNING: S3 upload failed"
  fi
fi

# ─── Rotate old backups ──────────────────────────────────────────────────────
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +${RETENTION_DAYS} -delete
log "Removed backups older than ${RETENTION_DAYS} days"

log "Backup process complete"
```

Make the script executable:
```bash
chmod +x /home/deploy/scripts/mysql_backup.sh
```

### Windows Task Scheduler + PowerShell

```powershell
# mysql_backup.ps1 — Windows backup script

$DBUser     = "backupuser"
$DBPass     = "yourpassword"
$DBName     = "myapp_db"
$DBHost     = "127.0.0.1"
$BackupDir  = "C:\MySQLBackups"
$Timestamp  = (Get-Date -Format "yyyyMMdd_HHmmss")
$Filename   = "${DBName}_${Timestamp}.sql"
$Filepath   = Join-Path $BackupDir $Filename

# Create backup directory if it doesn't exist
New-Item -ItemType Directory -Force -Path $BackupDir | Out-Null

# Run mysqldump
$MySQLDumpPath = "C:\Program Files\MySQL\MySQL Server 8.0\bin\mysqldump.exe"
& $MySQLDumpPath -h $DBHost -u $DBUser "-p$DBPass" `
    --single-transaction --routines --triggers --events `
    $DBName | Out-File -FilePath $Filepath -Encoding UTF8

if ($LASTEXITCODE -eq 0) {
    Write-Host "Backup succeeded: $Filepath"

    # Compress with built-in Compress-Archive
    Compress-Archive -Path $Filepath -DestinationPath "${Filepath}.zip"
    Remove-Item $Filepath

    # Rotate backups older than 14 days
    Get-ChildItem $BackupDir -Filter "*.zip" |
        Where-Object { $_.LastWriteTime -lt (Get-Date).AddDays(-14) } |
        Remove-Item

    Write-Host "Old backups cleaned up."
} else {
    Write-Error "Backup FAILED"
    exit 1
}
```

**Schedule in Task Scheduler:**
```powershell
# Register daily backup task at 2:00 AM
$Action  = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-File C:\scripts\mysql_backup.ps1"
$Trigger = New-ScheduledTaskTrigger -Daily -At 2:00AM
Register-ScheduledTask -TaskName "MySQL Daily Backup" -Action $Action -Trigger $Trigger -RunLevel Highest
```

### Backup Rotation & Retention Policies

```bash
# Keep: 7 daily, 4 weekly, 12 monthly backups
#!/bin/bash

BACKUP_DIR="/var/backups/mysql"

# Remove daily backups older than 7 days
find "$BACKUP_DIR/daily" -name "*.sql.gz" -mtime +7 -delete

# Remove weekly backups older than 28 days (4 weeks)
find "$BACKUP_DIR/weekly" -name "*.sql.gz" -mtime +28 -delete

# Remove monthly backups older than 365 days (12 months)
find "$BACKUP_DIR/monthly" -name "*.sql.gz" -mtime +365 -delete
```

---

## 9. Cloud Backup Strategy

### AWS RDS Automated Backups

AWS RDS handles backups automatically with point-in-time recovery:

```bash
# Enable automated backups (via AWS CLI)
aws rds modify-db-instance \
  --db-instance-identifier myapp-prod \
  --backup-retention-period 14 \
  --preferred-backup-window "02:00-03:00" \
  --apply-immediately

# Create manual snapshot
aws rds create-db-snapshot \
  --db-instance-identifier myapp-prod \
  --db-snapshot-identifier myapp-prod-manual-$(date +%Y%m%d)

# List snapshots
aws rds describe-db-snapshots --db-instance-identifier myapp-prod

# Restore from snapshot
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier myapp-prod-restored \
  --db-snapshot-identifier myapp-prod-manual-20240315
```

### Backup MySQL to AWS S3

```bash
#!/bin/bash
# Backup to S3 with server-side encryption

DB_NAME="myapp_db"
S3_BUCKET="s3://my-backups/mysql"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

mysqldump \
  --single-transaction \
  --routines \
  --triggers \
  -u root -p "$DB_NAME" | \
  gzip | \
  aws s3 cp - \
    "${S3_BUCKET}/${DB_NAME}_${TIMESTAMP}.sql.gz" \
    --sse AES256 \
    --storage-class STANDARD_IA

echo "Upload exit code: $?"
```

### S3 Lifecycle Policy (Retention)

```json
{
  "Rules": [
    {
      "ID": "mysql-backup-lifecycle",
      "Status": "Enabled",
      "Filter": { "Prefix": "mysql/" },
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        },
        {
          "Days": 90,
          "StorageClass": "GLACIER"
        }
      ],
      "Expiration": {
        "Days": 365
      }
    }
  ]
}
```

### Google Cloud SQL Backups

```bash
# Enable automated backups
gcloud sql instances patch myapp-prod \
  --backup-start-time="02:00" \
  --enable-bin-log \
  --retained-backups-count=14 \
  --retained-transaction-log-days=7

# Create on-demand backup
gcloud sql backups create --instance=myapp-prod

# List backups
gcloud sql backups list --instance=myapp-prod

# Restore from backup
gcloud sql backups restore BACKUP_ID \
  --restore-instance=myapp-prod
```

### Azure Database for MySQL Backups

Azure takes automatic backups (full weekly, differential daily, transaction logs every 5 min):

```bash
# View backup configuration
az mysql server show \
  --resource-group myResourceGroup \
  --name myapp-mysql-server \
  --query '{backupRetentionDays:storageProfile.backupRetentionDays, \
            geoRedundantBackup:storageProfile.geoRedundantBackup}'

# Restore to point in time
az mysql server restore \
  --resource-group myResourceGroup \
  --name myapp-mysql-restored \
  --source-server myapp-mysql-server \
  --restore-point-in-time "2024-03-15T11:00:00Z"
```

### Backup Encryption Best Practices

```bash
# Encrypt backup file with GPG before uploading
mysqldump -u root -p myapp_db | \
  gzip | \
  gpg --symmetric --cipher-algo AES256 \
  > backup_encrypted.sql.gz.gpg

# Decrypt and restore
gpg --decrypt backup_encrypted.sql.gz.gpg | \
  gunzip | \
  mysql -u root -p myapp_db

# Encrypt using OpenSSL
mysqldump -u root -p myapp_db | \
  gzip | \
  openssl enc -aes-256-cbc -salt -k "$BACKUP_PASSWORD" \
  > backup_encrypted.sql.gz.enc

# Decrypt
openssl enc -d -aes-256-cbc -k "$BACKUP_PASSWORD" \
  < backup_encrypted.sql.gz.enc | \
  gunzip | \
  mysql -u root -p myapp_db
```

---

## 10. Docker MySQL Backup

### Backup Running MySQL Container

```bash
# Get container name/ID
docker ps | grep mysql

# Backup using docker exec
docker exec mysql-container \
  mysqldump \
  -u root -p"$MYSQL_ROOT_PASSWORD" \
  --single-transaction \
  --routines \
  --triggers \
  myapp_db > backup_$(date +%Y%m%d).sql

# Compressed
docker exec mysql-container \
  mysqldump -u root -p"$MYSQL_ROOT_PASSWORD" \
  --single-transaction myapp_db | \
  gzip > backup_$(date +%Y%m%d).sql.gz
```

### Restore to Docker Container

```bash
# Restore SQL file into container
docker exec -i mysql-container \
  mysql -u root -p"$MYSQL_ROOT_PASSWORD" myapp_db \
  < backup.sql

# Restore compressed
gunzip -c backup.sql.gz | \
  docker exec -i mysql-container \
  mysql -u root -p"$MYSQL_ROOT_PASSWORD" myapp_db
```

### Docker Volume Backup

```bash
# Find the volume name
docker inspect mysql-container | grep -A 5 '"Mounts"'

# Backup volume to tar file
docker run --rm \
  --volumes-from mysql-container \
  -v $(pwd):/backup \
  ubuntu \
  tar czf /backup/mysql_volume_backup.tar.gz /var/lib/mysql

# Restore volume from tar
docker run --rm \
  --volumes-from mysql-container \
  -v $(pwd):/backup \
  ubuntu \
  bash -c "cd / && tar xzf /backup/mysql_volume_backup.tar.gz"
```

### Docker Compose Backup Workflow

```yaml
# docker-compose.yml
version: '3.8'
services:
  mysql:
    image: mysql:8.0
    container_name: app_mysql
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
    volumes:
      - mysql_data:/var/lib/mysql
      - ./backups:/backups
    networks:
      - app_network

  backup:
    image: mysql:8.0
    depends_on:
      - mysql
    volumes:
      - ./backups:/backups
    command: >
      bash -c "
        mysqldump -h mysql -u root -p$$MYSQL_ROOT_PASSWORD
        --single-transaction --routines $$MYSQL_DATABASE
        | gzip > /backups/backup_$$(date +%Y%m%d_%H%M%S).sql.gz
        && echo 'Backup complete'
      "
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
    networks:
      - app_network
    profiles:
      - backup

volumes:
  mysql_data:

networks:
  app_network:
```

```bash
# Run backup service on demand
docker compose --profile backup up backup

# Schedule via host cron
0 2 * * * cd /app && docker compose --profile backup up backup
```

---

## 11. Prisma + MySQL Backup Strategy

### Always Backup Before Running Migrations

```bash
#!/bin/bash
# pre-migrate.sh — Run this before any prisma migrate deploy

set -e  # Exit on any error

DB_NAME="myapp_db"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="pre_migration_${TIMESTAMP}.sql.gz"
BACKUP_DIR="./backups"

mkdir -p "$BACKUP_DIR"

echo "📦 Creating pre-migration backup..."
mysqldump \
  --single-transaction \
  --routines \
  --triggers \
  -h "$DB_HOST" \
  -u "$DB_USER" \
  -p"$DB_PASS" \
  "$DB_NAME" | gzip > "${BACKUP_DIR}/${BACKUP_FILE}"

echo "✅ Backup saved: ${BACKUP_DIR}/${BACKUP_FILE}"

echo "🚀 Running Prisma migration..."
npx prisma migrate deploy

echo "✅ Migration complete."
```

### Prisma Migration Safety Workflow

```
Before migration:
1. Run pre-migrate.sh → backup saved
2. Review migration SQL: npx prisma migrate diff
3. Test on staging first
4. Deploy to production
5. Verify application health
6. If failure → rollback using backup
```

### Rollback After Failed Migration

```bash
#!/bin/bash
# rollback.sh — Restore from most recent pre-migration backup

BACKUP_DIR="./backups"
LATEST_BACKUP=$(ls -t ${BACKUP_DIR}/pre_migration_*.sql.gz | head -1)

if [ -z "$LATEST_BACKUP" ]; then
  echo "❌ No backup found in $BACKUP_DIR"
  exit 1
fi

echo "⚠️  Rolling back to: $LATEST_BACKUP"
read -p "Are you sure? This will overwrite the database. (yes/no): " CONFIRM

if [ "$CONFIRM" = "yes" ]; then
  gunzip -c "$LATEST_BACKUP" | \
    mysql -h "$DB_HOST" -u "$DB_USER" -p"$DB_PASS" "$DB_NAME"
  echo "✅ Rollback complete"
else
  echo "Rollback cancelled"
fi
```

### Package.json Integration

```json
{
  "scripts": {
    "db:backup": "bash scripts/pre-migrate.sh --backup-only",
    "db:migrate": "bash scripts/pre-migrate.sh",
    "db:rollback": "bash scripts/rollback.sh",
    "db:migrate:dev": "prisma migrate dev"
  }
}
```

### Prisma Schema Migration Protection Checklist

- [ ] Backup created before every `prisma migrate deploy`
- [ ] Migration reviewed with `prisma migrate diff` before running
- [ ] Staging tested before production
- [ ] Avoid destructive operations in single migration (drop column + add column separately)
- [ ] Never use `prisma migrate reset` in production
- [ ] Enable `shadowDatabaseUrl` for dev migrations only

---

## 12. Security Best Practices

### Dedicated Backup User (Least Privilege)

```sql
-- Create a dedicated backup user
CREATE USER 'backup_user'@'localhost' IDENTIFIED BY 'StrongPassword!';

-- Grant minimal permissions for backup
GRANT SELECT, SHOW VIEW, TRIGGER, LOCK TABLES, PROCESS, REPLICATION CLIENT
  ON *.* TO 'backup_user'@'localhost';

-- For mysqldump with --events
GRANT EVENT ON *.* TO 'backup_user'@'localhost';

FLUSH PRIVILEGES;
```

> ⚠️ Never use the `root` user in automated backup scripts.

### Credential Management

```bash
# Option 1: .my.cnf file (permissions locked to owner)
cat > ~/.my.cnf << EOF
[client]
user=backup_user
password=StrongPassword!
host=127.0.0.1
EOF

chmod 600 ~/.my.cnf   # Only owner can read
chown $(whoami):$(whoami) ~/.my.cnf

# Now use mysqldump without -p flag:
mysqldump myapp_db > backup.sql  # Reads credentials from .my.cnf
```

```bash
# Option 2: Environment variables from .env
source /etc/mysql_backup.env    # Contains DB_PASS=...
mysqldump -u backup_user -p"$DB_PASS" myapp_db > backup.sql
```

```bash
# Option 3: AWS Secrets Manager (enterprise)
DB_PASS=$(aws secretsmanager get-secret-value \
  --secret-id prod/mysql/backup_user \
  --query SecretString \
  --output text | jq -r .password)

mysqldump -u backup_user -p"$DB_PASS" myapp_db > backup.sql
unset DB_PASS   # Clear from memory
```

### Backup File Permissions

```bash
# Restrict backup directory access
chmod 700 /var/backups/mysql
chown mysql_backup_user:mysql_backup_user /var/backups/mysql

# Restrict individual backup files
chmod 600 /var/backups/mysql/*.sql.gz

# Verify permissions
ls -la /var/backups/mysql/
```

### Backup Encryption

```bash
# Encrypt at rest with GPG
gpg --gen-key   # Generate key pair if you don't have one

# Encrypt backup
mysqldump -u backup_user myapp_db | \
  gzip | \
  gpg --encrypt --recipient backup@company.com \
  > backup_encrypted.sql.gz.gpg

# Store private key securely (separate from backup storage!)
gpg --export-secret-keys backup@company.com | \
  gpg --symmetric \
  > backup_private_key.gpg.enc
```

### Ransomware Prevention

```bash
# Use write-once S3 bucket with Object Lock
aws s3api put-object-lock-configuration \
  --bucket my-mysql-backups \
  --object-lock-configuration '{
    "ObjectLockEnabled": "Enabled",
    "Rule": {
      "DefaultRetention": {
        "Mode": "COMPLIANCE",
        "Days": 30
      }
    }
  }'

# This makes backups immutable for 30 days — ransomware cannot delete them
```

**Security Checklist:**
- [ ] Backup user has read-only permissions only
- [ ] Backup credentials stored in `.my.cnf` with `600` permissions, never in scripts
- [ ] Backups encrypted before cloud upload
- [ ] Backup storage is access-controlled (separate IAM role)
- [ ] Offsite/immutable backup copy exists (S3 Object Lock or similar)
- [ ] Backup script logs to append-only log file
- [ ] Backup notifications sent to monitoring channel

---

## 13. Performance Optimization

### Large Database Backup Strategies

```bash
# Stream directly to S3 (no local disk required)
mysqldump \
  --single-transaction \
  -u root -p myapp_db | \
  gzip | \
  aws s3 cp - s3://backups/myapp_db_$(date +%Y%m%d).sql.gz

# Parallel table backup with mysqlpump
mysqlpump \
  --default-parallelism=4 \
  --parallel-schemas=4:myapp_db \
  -u root -p \
  --include-databases=myapp_db \
  > myapp_db_parallel.sql

# Use Percona XtraBackup for large databases (physical)
xtrabackup \
  --backup \
  --user=backup_user \
  --password=password \
  --target-dir=/var/backups/xtrabackup/$(date +%Y%m%d)
```

### Minimizing Downtime & Performance Impact

```bash
# Use --single-transaction for InnoDB (no locks, no downtime)
mysqldump --single-transaction -u root -p myapp_db > backup.sql

# Limit backup I/O with ionice (Linux)
ionice -c 3 mysqldump -u root -p myapp_db > backup.sql

# Reduce CPU priority
nice -n 19 mysqldump -u root -p myapp_db > backup.sql

# Combined low-priority backup
nice -n 19 ionice -c 3 mysqldump \
  --single-transaction \
  -u root -p myapp_db | gzip > backup.sql.gz
```

### Transaction Consistency

```bash
# Check InnoDB engine is being used (required for --single-transaction)
mysql -u root -p -e "
  SELECT TABLE_NAME, ENGINE
  FROM information_schema.TABLES
  WHERE TABLE_SCHEMA = 'myapp_db'
  AND ENGINE != 'InnoDB';"

# If MyISAM tables exist, either convert them or use --lock-tables
ALTER TABLE my_table ENGINE=InnoDB;

# For mixed engine databases, use LOCK TABLES (causes brief lock)
mysqldump --lock-tables -u root -p myapp_db > backup.sql
```

### Streaming & Incremental with Percona XtraBackup

```bash
# Full backup
xtrabackup --backup \
  --user=backup_user \
  --password=password \
  --target-dir=/backups/full_$(date +%Y%m%d)

# Prepare full backup (needed before restore)
xtrabackup --prepare \
  --target-dir=/backups/full_20240315

# Incremental backup
xtrabackup --backup \
  --user=backup_user \
  --password=password \
  --target-dir=/backups/inc_$(date +%Y%m%d) \
  --incremental-basedir=/backups/full_20240315

# Prepare incremental (apply to full)
xtrabackup --prepare \
  --apply-log-only \
  --target-dir=/backups/full_20240315 \
  --incremental-dir=/backups/inc_20240316

# Restore (copy-back to data directory)
systemctl stop mysql
xtrabackup --copy-back --target-dir=/backups/full_20240315
chown -R mysql:mysql /var/lib/mysql
systemctl start mysql
```

---

## 14. Common Errors & Fixes

### Error: Access Denied

```
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
```

**Causes & Fixes:**

```bash
# Wrong password
mysql -u root -p   # Re-enter correct password

# Check user exists and has correct host
mysql -u root -p -e "SELECT user, host FROM mysql.user WHERE user='root';"

# Reset root password (if you've lost it)
# Stop MySQL, start with --skip-grant-tables
sudo systemctl stop mysql
sudo mysqld_safe --skip-grant-tables &
mysql -u root -e "
  ALTER USER 'root'@'localhost' IDENTIFIED BY 'NewPassword';
  FLUSH PRIVILEGES;"
sudo systemctl start mysql
```

### Error: Connection Refused

```
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock'
```

**Fixes:**

```bash
# Check if MySQL is running
sudo systemctl status mysql

# Start MySQL
sudo systemctl start mysql

# Check socket file location
mysql_config --socket
mysqladmin -u root -p --socket=/var/lib/mysql/mysql.sock status

# Check port for TCP connections
mysql -u root -p -h 127.0.0.1 -P 3306
```

### Error: Packet Too Large

```
ERROR 1153 (08S01): Got a packet bigger than 'max_allowed_packet' bytes
```

**Fixes:**

```bash
# Increase max_allowed_packet for restore
mysql -u root -p \
  --max_allowed_packet=512M \
  myapp_db < backup.sql

# Or set in my.cnf permanently
# [mysqld]
# max_allowed_packet=512M

# Or set at runtime
mysql -u root -p -e "SET GLOBAL max_allowed_packet=536870912;"

# Use chunked backup to avoid large packets
mysqldump \
  --max_allowed_packet=512M \
  -u root -p myapp_db > backup.sql
```

### Error: Collation Mismatch

```
ERROR 1253 (42000): COLLATION 'utf8mb4_0900_ai_ci' is not valid for CHARACTER SET 'latin1'
```

**Fixes:**

```bash
# Check source database charset
mysql -u root -p -e "
  SELECT DEFAULT_CHARACTER_SET_NAME, DEFAULT_COLLATION_NAME
  FROM information_schema.SCHEMATA
  WHERE SCHEMA_NAME='myapp_db';"

# Create target DB with matching charset
mysql -u root -p -e "
  CREATE DATABASE myapp_db
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;"

# Convert during restore by replacing collation in dump file
sed 's/utf8mb4_0900_ai_ci/utf8mb4_unicode_ci/g' backup.sql | \
  mysql -u root -p myapp_db
```

### Error: Table Doesn't Exist During Restore

```
ERROR 1146 (42S02): Table 'myapp_db.users' doesn't exist
```

**Fix:** The backup may be data-only (`--no-create-info`). Import schema first, then data:

```bash
# Import schema first
mysqldump --no-data -u root -p source_db | mysql -u root -p target_db

# Then import data
mysqldump --no-create-info -u root -p source_db | mysql -u root -p target_db
```

### Corrupted Dump File

```
ERROR: ASCII '\0' found near 'backup.sql' line 1
```

**Detection & Recovery:**

```bash
# Test gzip integrity
gzip -t backup.sql.gz && echo "OK" || echo "CORRUPTED"

# Check SQL file for validity
mysql -u root -p myapp_db < backup.sql 2>&1 | head -20

# If file is partially recoverable, try to extract what you can
grep -n "INSERT INTO \`users\`" backup.sql | head -5
# Note start line, then extract with sed
sed -n 'START_LINE,END_LINEp' backup.sql > partial_recovery.sql
```

**Prevention:** Always verify backup integrity after creation:

```bash
# Add this to backup script
gzip -t "$BACKUP_FILE" && echo "Backup integrity: OK" || {
  echo "BACKUP CORRUPTED — Sending alert!"
  # send_alert_email or Slack notification
  exit 1
}
```

### Error: Unknown System Variable

```
ERROR 1193 (HY000): Unknown system variable 'gtid_purged'
```

**Fix:** Add `--set-gtid-purged=OFF` to mysqldump command when source uses GTIDs but target doesn't:

```bash
mysqldump --set-gtid-purged=OFF -u root -p myapp_db > backup.sql
```

---

## 15. Disaster Recovery Planning

### The DR Workflow

```
Disaster Recovery Playbook:

1. DETECT
   └─ Alert triggered (monitoring, user report, automated check)

2. ASSESS
   ├─ What failed? (server, DB, data, config?)
   ├─ What is the scope? (one table, one DB, full server?)
   └─ What is the most recent valid backup?

3. ISOLATE
   ├─ Put application in maintenance mode
   └─ Prevent further writes to corrupted data

4. RESTORE
   ├─ Select backup (full + incrementals if needed)
   ├─ Spin up or prepare target server
   ├─ Execute restore procedure
   └─ Apply binary logs for point-in-time recovery (if needed)

5. VERIFY
   ├─ Check row counts match expected values
   ├─ Run application smoke tests
   └─ Confirm data integrity for critical tables

6. RESUME
   ├─ Disable maintenance mode
   ├─ Monitor closely for 30-60 minutes
   └─ Document the incident and timeline

7. REVIEW
   ├─ Post-mortem: root cause analysis
   ├─ Update backup strategy if needed
   └─ Update runbooks with lessons learned
```

### Testing Your Backups (Critical)

> ⚠️ **An untested backup is not a backup.** Test restores regularly.

```bash
#!/bin/bash
# test_restore.sh — Verify backup can be restored

BACKUP_FILE=$1
TEST_DB="restore_test_$(date +%Y%m%d%H%M%S)"

echo "Creating test database: $TEST_DB"
mysql -u root -p -e "CREATE DATABASE $TEST_DB CHARACTER SET utf8mb4;"

echo "Restoring backup..."
gunzip -c "$BACKUP_FILE" | mysql -u root -p "$TEST_DB"

if [ $? -eq 0 ]; then
  echo "✅ Restore successful"

  # Verify row counts
  mysql -u root -p -e "
    SELECT TABLE_NAME, TABLE_ROWS
    FROM information_schema.TABLES
    WHERE TABLE_SCHEMA='$TEST_DB'
    ORDER BY TABLE_ROWS DESC;" | head -20

  echo "✅ Verification complete"
else
  echo "❌ Restore FAILED"
fi

# Cleanup
mysql -u root -p -e "DROP DATABASE $TEST_DB;"
echo "Test database dropped."
```

### Backup Verification Script

```bash
#!/bin/bash
# verify_backup.sh — Run after every backup

BACKUP_FILE=$1

echo "=== Backup Verification ==="

# 1. File exists and is not empty
[ -f "$BACKUP_FILE" ] && [ -s "$BACKUP_FILE" ] && echo "✅ File exists and has content" || { echo "❌ File missing or empty"; exit 1; }

# 2. Integrity check
gzip -t "$BACKUP_FILE" && echo "✅ Gzip integrity OK" || { echo "❌ Gzip corrupted"; exit 1; }

# 3. Contains expected SQL structure
gunzip -c "$BACKUP_FILE" | head -50 | grep -q "MySQL dump" && echo "✅ Valid MySQL dump format" || echo "⚠️  Unexpected file format"

# 4. File size check (fail if < 1KB — likely empty or failed)
SIZE=$(stat -c%s "$BACKUP_FILE")
[ "$SIZE" -gt 1024 ] && echo "✅ File size: $(du -sh "$BACKUP_FILE" | cut -f1)" || { echo "❌ File suspiciously small"; exit 1; }

echo "=== All checks passed ==="
```

### Scheduled Restore Drills

```bash
# Monthly restore drill cron job
0 6 1 * * /opt/scripts/test_restore.sh /var/backups/mysql/latest_backup.sql.gz >> /var/log/restore_drill.log 2>&1
```

---

## 16. Production Architectures

### Master-Replica Replication Setup

```
Production Architecture (Read Scaling + Safe Backups):

                    ┌─────────────────────┐
                    │   Application Layer  │
                    │  (App Servers / API) │
                    └──────────┬──────────┘
                               │
              ┌────────────────┴────────────────┐
              │ Writes                    Reads  │
              ▼                                  ▼
  ┌───────────────────┐            ┌─────────────────────┐
  │   PRIMARY (Master) │──binlog──►│  REPLICA (Standby)  │
  │   (Read/Write)    │            │   (Read-Only)        │
  └───────────────────┘            └──────────┬──────────┘
                                              │
                                    Take backups from replica
                                    (zero impact on primary)
```

### Setting Up Replication for Backup

```sql
-- On Primary: Create replication user
CREATE USER 'repl_user'@'replica_ip' IDENTIFIED BY 'ReplPassword!';
GRANT REPLICATION SLAVE ON *.* TO 'repl_user'@'replica_ip';
FLUSH PRIVILEGES;

-- Get primary binary log position
SHOW MASTER STATUS;

-- On Replica: Configure replication
CHANGE MASTER TO
  MASTER_HOST='primary_ip',
  MASTER_USER='repl_user',
  MASTER_PASSWORD='ReplPassword!',
  MASTER_LOG_FILE='binlog.000001',
  MASTER_LOG_POS=154;

START SLAVE;
SHOW SLAVE STATUS\G
```

### High Availability with MySQL InnoDB Cluster

```
MySQL InnoDB Cluster (Group Replication):

┌──────────────────────────────────────────────────────┐
│                    MySQL Router                       │
│            (Transparent read/write splitting)         │
└──────────────┬───────────────────────────────────────┘
               │
    ┌──────────┴───────────┐
    │                      │
    ▼                      ▼
┌────────┐           ┌────────┐
│ Node 1 │◄─────────►│ Node 2 │
│Primary │  Group    │Standby │
└────┬───┘  Replication└───┬────┘
     │                     │
     └──────────┬──────────┘
                │
           ┌────▼───┐
           │ Node 3 │
           │Standby │
           └────────┘

- Automatic failover
- Synchronous replication
- Minimum 3 nodes for quorum
```

### Zero-Downtime Backup Strategy

```bash
#!/bin/bash
# zero_downtime_backup.sh
# Takes backup from replica — zero impact on primary

REPLICA_HOST="replica.db.internal"
DB_NAME="myapp_db"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# Pause replication briefly for consistent state (optional)
mysql -h "$REPLICA_HOST" -u root -p -e "STOP SLAVE SQL_THREAD;"

# Take backup
mysqldump \
  -h "$REPLICA_HOST" \
  --single-transaction \
  --master-data=2 \
  --routines --triggers --events \
  -u backup_user \
  "$DB_NAME" | gzip > "/backups/${DB_NAME}_${TIMESTAMP}.sql.gz"

# Resume replication
mysql -h "$REPLICA_HOST" -u root -p -e "START SLAVE SQL_THREAD;"

echo "Backup complete: /backups/${DB_NAME}_${TIMESTAMP}.sql.gz"
```

### Dedicated Backup Server Architecture

```
Enterprise Backup Architecture:

  Production Cluster          Backup Infrastructure
  ──────────────────          ─────────────────────
  ┌─────────────┐             ┌──────────────────┐
  │  Primary DB │──binlogs───►│  Backup Server   │
  └──────┬──────┘             │  (Dedicated VM)  │
         │                    └────────┬─────────┘
  ┌──────▼──────┐                      │
  │  Replica DB │──mysqldump──►         │
  └─────────────┘             ┌────────▼─────────┐
                               │  Local Storage   │
                               │  (NFS/SAN)       │
                               └────────┬─────────┘
                                        │
                               ┌────────▼─────────┐
                               │  Cloud Storage   │
                               │  (S3/GCS)        │
                               │  Cross-region    │
                               └──────────────────┘
```

---

## 17. Best Practices Checklist

### Production Backup Checklist

- [ ] Automated daily full backup configured
- [ ] Binary logging enabled for PITR
- [ ] Backup taken from replica (not primary)
- [ ] Backups stored in minimum 2 locations (local + cloud)
- [ ] Cross-region backup replication enabled
- [ ] Backup size monitored — alert if unexpectedly small
- [ ] Backup completion notifications configured
- [ ] RPO and RTO defined and documented
- [ ] Retention policy configured (daily/weekly/monthly)

### Security Checklist

- [ ] Dedicated backup user with minimal permissions
- [ ] No root credentials in scripts
- [ ] Credentials in `.my.cnf` with `chmod 600`
- [ ] Backup files encrypted before cloud upload
- [ ] Backup storage uses separate IAM/access credentials
- [ ] Object Lock / immutable storage for ransomware protection
- [ ] Backup server access restricted by IP/VPN
- [ ] Backup logs reviewed regularly

### Restore Checklist

- [ ] Monthly restore drill scheduled
- [ ] Restore procedure documented and accessible offline
- [ ] Target restore server available or can be provisioned in < 30 min
- [ ] Character set and collation verified before restore
- [ ] Application in maintenance mode before restore
- [ ] Row counts verified after restore
- [ ] Application smoke tests run after restore
- [ ] Incident documented post-restore

### Automation Checklist

- [ ] Cron/Task Scheduler configured with correct user permissions
- [ ] Backup script logs to a persistent file
- [ ] Log rotation configured for backup logs
- [ ] Alert on backup failure (email/Slack/PagerDuty)
- [ ] Backup integrity check runs after each backup
- [ ] Old backups cleaned up automatically (retention policy)
- [ ] Script tested in staging before production

---

## 18. Cheat Sheet

### Essential mysqldump Commands

```bash
# Full database backup
mysqldump -u root -p mydb > backup.sql

# Full backup (production recommended)
mysqldump --single-transaction --routines --triggers --events -u root -p mydb > backup.sql

# All databases
mysqldump --all-databases -u root -p > all_dbs.sql

# Compressed
mysqldump -u root -p mydb | gzip > backup.sql.gz

# Schema only
mysqldump --no-data -u root -p mydb > schema.sql

# Data only
mysqldump --no-create-info -u root -p mydb > data.sql

# Single table
mysqldump -u root -p mydb users > users.sql

# With WHERE
mysqldump -u root -p mydb users --where="active=1" > active_users.sql

# Remote server
mysqldump -h 10.0.0.1 -P 3306 -u root -p mydb > backup.sql

# With binary log position (for PITR)
mysqldump --single-transaction --master-data=2 -u root -p mydb > backup.sql
```

### Restore Commands

```bash
# Basic restore
mysql -u root -p mydb < backup.sql

# Restore compressed
gunzip -c backup.sql.gz | mysql -u root -p mydb

# Create DB then restore
mysql -u root -p -e "CREATE DATABASE mydb;"
mysql -u root -p mydb < backup.sql

# Restore all databases
mysql -u root -p < all_dbs.sql

# Restore from remote
cat backup.sql | mysql -h 10.0.0.1 -u root -p mydb

# Point-in-time recovery
mysqlbinlog --start-datetime="2024-01-15 10:00:00" \
            --stop-datetime="2024-01-15 11:30:00" \
            /var/lib/mysql/binlog.000042 | mysql -u root -p mydb
```

### Troubleshooting Commands

```bash
# Check MySQL is running
sudo systemctl status mysql

# Check disk space
df -h /var/lib/mysql

# Check MySQL error log
sudo tail -100 /var/log/mysql/error.log

# Check binary logs
mysql -u root -p -e "SHOW BINARY LOGS;"
mysql -u root -p -e "SHOW MASTER STATUS;"

# Check running processes
mysql -u root -p -e "SHOW PROCESSLIST;"

# Check replication status
mysql -u root -p -e "SHOW SLAVE STATUS\G"

# Test connectivity
mysqladmin -u root -p ping

# Check table corruption
mysqlcheck --all-databases -u root -p

# Repair tables
mysqlcheck --repair --all-databases -u root -p
```

### Verification Commands

```bash
# Count rows in all tables
mysql -u root -p -e "
  SELECT TABLE_NAME, TABLE_ROWS
  FROM information_schema.TABLES
  WHERE TABLE_SCHEMA='mydb'
  ORDER BY TABLE_ROWS DESC;"

# Check database size
mysql -u root -p -e "
  SELECT
    table_schema AS 'Database',
    ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
  FROM information_schema.TABLES
  GROUP BY table_schema;"

# Verify backup file integrity
gzip -t backup.sql.gz && echo "OK" || echo "CORRUPTED"

# Preview backup contents
gunzip -c backup.sql.gz | head -50

# Check MySQL variables
mysql -u root -p -e "SHOW VARIABLES LIKE 'log_bin';"
mysql -u root -p -e "SHOW VARIABLES LIKE 'max_allowed_packet';"
mysql -u root -p -e "SHOW VARIABLES LIKE 'innodb_buffer_pool_size';"

# Test that a backup was taken from a GTID-enabled server
gunzip -c backup.sql.gz | grep "GTID"
```

---

## Quick Reference: Backup Strategy by Size

| DB Size | Recommended Tool | Frequency | Est. Backup Time |
|---------|-----------------|-----------|-----------------|
| < 1 GB | mysqldump | Every 6h | < 1 min |
| 1–10 GB | mysqldump | Daily | 2–15 min |
| 10–100 GB | mysqlpump or XtraBackup | Daily full + hourly binlog | 15–60 min |
| > 100 GB | Percona XtraBackup | Weekly full + daily incremental | 30–120 min |
| Cloud (RDS/GCS) | Managed snapshots | Automated | Seconds (snapshot) |

---

> **Remember:** A backup you've never tested is just a file. Test your restores, document your procedures, and automate everything that runs more than once.
>
> *Last reviewed: 2025 | Tools: MySQL 8.0+, Percona XtraBackup 8.0, AWS CLI v2*
