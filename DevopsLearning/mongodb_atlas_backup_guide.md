# Automated MongoDB Atlas Backup System on Ubuntu VPS

A complete, self-contained reference for setting up a daily automated backup of a MongoDB Atlas database to Google Drive, from a fresh Ubuntu VPS.

---

## 1. Overview

This guide sets up a fully automated daily backup pipeline:

- MongoDB Atlas is dumped daily using `mongodump`.
- The dump is compressed into a single `.archive.gz` file.
- The file is uploaded to Google Drive using `rclone`.
- Once the upload succeeds, the local copy is deleted to save VPS disk space.
- If the upload fails, the local copy is kept so no data is lost.
- The whole process runs unattended every day via `cron`, at **12:00 AM IST**.

Environment assumptions used throughout this document:

| Item | Value |
|---|---|
| VPS OS | Ubuntu Linux |
| VPS user | `manish` |
| VPS timezone | UTC (unchanged) |
| Backup time (IST) | 12:00 AM |
| Backup time (UTC / cron) | 18:30 |
| mongodump version | 100.17.0 |
| mongosh required? | No |
| Local backup folder | `/home/manish/mongodb_backup` |
| Backup filename pattern | `mongodb-backup-YYYY-MM-DD.archive.gz` |
| Cloud destination | Google Drive, folder `MongoDB Backups` |
| Upload tool | rclone, remote name `gdrive` |
| Cron log file | `/home/manish/mongodb_backup.log` |

---

## 2. Architecture / Workflow

```
MongoDB Atlas
      │
      │  mongodump --uri="..." --archive --gzip
      ▼
Ubuntu VPS (system timezone: UTC)
      │
      ▼
/home/manish/mongodb_backup/mongodb-backup-YYYY-MM-DD.archive.gz
      │
      │  rclone copyto (upload only today's file)
      ▼
Google Drive → "MongoDB Backups" folder
      │
      ▼
 Upload successful?
   ┌───────┴───────┐
  YES              NO
   │                │
   ▼                ▼
Delete local     Keep local
 backup file      backup file
```

Every box in this chain is independent and replaceable: Atlas doesn't know about the VPS, the VPS doesn't know about Google Drive credentials beyond what rclone stores, and the cron job just glues these steps together with exit-code checks.

---

## 3. Why `mongosh` Is Not Required

`mongosh` is MongoDB's interactive shell — it's used for running queries, exploring collections, and administering a database by hand. It is **not** used by `mongodump`.

`mongodump` is a standalone binary from the MongoDB Database Tools package. It connects directly to a MongoDB URI, reads data, and writes it to a dump/archive file. Since this pipeline never queries or inspects data interactively on the VPS — it only performs a full dump — `mongosh` adds nothing to the workflow and does not need to be installed.

---

## 4. Installing / Checking `mongodump`

Check whether it's already installed and see its version:

```bash
mongodump --version
```

Expected output includes a line similar to:

```
mongodump version: 100.17.0
```

If it's not installed, install the MongoDB Database Tools package on Ubuntu:

```bash
# Download the MongoDB Database Tools .deb package (check mongodb.com for the latest version/URL for your Ubuntu release)
curl -O https://fastdl.mongodb.org/tools/db/mongodb-database-tools-ubuntu2204-x86_64-100.17.0.deb

# Install it
sudo dpkg -i mongodb-database-tools-ubuntu2204-x86_64-100.17.0.deb
```

Since this environment already has `mongodump` 100.17.0 installed, this step is just for future reference or reinstalling on a new VPS.

---

## 5. MongoDB Atlas Network Access

Atlas blocks all connections by default. The VPS's public IP must be explicitly allowed:

1. Log in to [MongoDB Atlas](https://cloud.mongodb.com).
2. Open your project → **Network Access** (left sidebar).
3. Click **Add IP Address**.
4. Enter the VPS's public IP address (find it on the VPS with `curl ifconfig.me`).
5. Save. It can take a minute to become active.

**Note:** Adding `0.0.0.0/0` (allow from anywhere) is sometimes used for convenience but is a security risk — it allows connection attempts from any IP on the internet (they'd still need valid credentials, but it widens the attack surface). Prefer allow-listing the specific VPS IP.

---

## 6. Getting the MongoDB Atlas Connection String

1. In Atlas, go to your Cluster → **Connect**.
2. Choose **Drivers** (or "Connect your application").
3. Copy the connection string — it looks like:

```
mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority
```

4. Replace `<username>` and `<password>` with your actual database user credentials (not your Atlas login).

Throughout this document, this full string is referred to as:

```
YOUR_MONGODB_ATLAS_CONNECTION_STRING
```

Never paste the real string into scripts in plain text long-term — see [Section 26](#26-security-issue-of-hardcoding-the-mongodb-uri) for the secure alternative.

---

## 7. Manually Testing `mongodump`

Before automating anything, confirm the connection works:

```bash
mongodump --uri="YOUR_MONGODB_ATLAS_CONNECTION_STRING" --archive=test-backup.archive --gzip
```

- `--uri` tells mongodump where to connect (Atlas cluster + credentials).
- `--archive=test-backup.archive` writes the entire dump into a single file instead of a folder of BSON files.
- `--gzip` compresses that archive as it's written.

If it succeeds, you'll see progress lines ending in something like `done dumping ...`, and a `test-backup.archive` file will exist. Delete it afterward:

```bash
rm test-backup.archive
```

---

## 8. Creating `/home/manish/mongodb_backup`

```bash
mkdir -p /home/manish/mongodb_backup
```

`-p` creates the directory without error if it already exists, and creates any missing parent directories too.

---

## 9. Creating the Compressed Backup File

The naming pattern uses the current date:

```bash
DATE=$(date +%F)
mongodump --uri="YOUR_MONGODB_ATLAS_CONNECTION_STRING" \
  --archive="/home/manish/mongodb_backup/mongodb-backup-${DATE}.archive.gz" \
  --gzip
```

- `date +%F` outputs the date as `YYYY-MM-DD`.
- The resulting filename matches the required pattern, e.g. `mongodb-backup-2026-08-15.archive.gz`.

---

## 10. Google Drive Free Storage Considerations

A standard free Google account includes 15 GB of storage, shared across Gmail, Google Photos, and Drive. Points to plan around:

- Each daily backup consumes space; without cleanup, storage will eventually run out.
- Compressed MongoDB dumps are usually small for modest databases, but growth over months adds up.
- This is why [Section 45](#45-recommended-retention-strategy) recommends a retention policy (e.g., keep only the last 30 days).
- If the account is shared with other data (photos, documents), monitor total usage, not just the backup folder.
- Google Workspace accounts have separate, often larger or pooled quotas — check your plan if this is a business account.

---

## 11. What rclone Is

`rclone` is a command-line program that syncs and transfers files between local storage and dozens of cloud storage providers (Google Drive, S3, Dropbox, Backblaze, etc.). It behaves similarly to `rsync`/`cp`, but the "remote" side is a cloud service instead of another local path. Here it's used purely to push one file at a time from the VPS to a Google Drive folder.

---

## 12. Installing rclone on Ubuntu

```bash
sudo apt update
sudo apt install -y rclone
```

Or, for the latest version directly from rclone's installer:

```bash
sudo -v ; curl https://rclone.org/install.sh | sudo bash
```

Verify:

```bash
rclone version
```

---

## 13. Configuring rclone for Google Drive

Start the interactive configuration wizard:

```bash
rclone config
```

This walks through creating a new "remote" (a named connection profile) called `gdrive`, pointing at Google Drive.

---

## 14. Key rclone Configuration Concepts Explained

| Term | Meaning |
|---|---|
| **client_id** | An OAuth application identifier registered with Google. Identifies *which application* is requesting access. You can leave this blank to use rclone's shared default app, or supply your own for higher API rate limits. |
| **client_secret** | The secret paired with a custom client_id, proving the application's identity to Google. Only needed if you supplied your own client_id. Leave blank to use rclone's default. |
| **scope** | The permission level requested from Google Drive. `drive` = full access to all files; `drive.readonly` = read-only; `drive.file` = access only to files created by the app. For backups, full `drive` access is typical so rclone can create/delete files. |
| **service account** | A non-human Google account (JSON key file) used for server-to-server auth without a browser login. More setup overhead; useful for fully headless automation across many machines. Not required for a single VPS — this guide uses standard OAuth login instead. |
| **advanced config** | Optional extra settings (custom endpoints, team drive IDs, etc.). Answer **No** unless you have a specific need (e.g., a Shared Drive). |
| **auto config** | Whether rclone should try to open a browser *on the same machine* to complete Google login. On a VPS with no browser/GUI, this must be **No**. |
| **remote authorization** | The alternative flow used when auto config is No: you authorize on a machine that *does* have a browser (like your Windows PC), then paste the resulting token back into the VPS. |

---

## 15. Authorizing Google Drive from Windows (VPS Has No Browser)

Because the VPS is headless, Google's OAuth login page can't open there. Instead:

1. **On the VPS**, run `rclone config` and proceed until it asks about auto config — answer **No**.
2. rclone will print a command to run elsewhere, looking like:
   ```
   rclone authorize "drive"
   ```
3. **On your Windows PC**, install rclone too (download from rclone.org, or `winget install Rclone.Rclone`).
4. **On Windows**, open a terminal (PowerShell or Command Prompt) and run the exact `rclone authorize "drive" "..."` command shown by the VPS (see Section 16).
5. This opens your default browser on Windows, asking you to log in with **YOUR_GOOGLE_ACCOUNT** and grant access.
6. After approving, rclone (on Windows) prints a config token in the terminal.
7. Copy that entire token and paste it back into the waiting prompt on the **VPS** (see Section 17).

This way, the actual Google login happens on a machine with a browser, while the resulting credentials still end up stored on the VPS.

---

## 16. Explaining the Command: `rclone authorize "drive" "..."`

```bash
rclone authorize "drive" "SOME_LONG_BASE64_STRING"
```

- `authorize` is a standalone rclone subcommand that performs just the OAuth browser flow and prints the resulting token — it doesn't require a prior config.
- `"drive"` tells it which backend/service to authorize against (Google Drive).
- The long base64-looking string (auto-generated by the VPS-side config wizard) encodes the specific OAuth parameters (like the requested scope) so the Windows-side authorization matches what the VPS asked for.

You run this exact command (copied verbatim from the VPS output) on the Windows machine, not on the VPS.

---

## 17. Where the Config Token Is Pasted on the VPS

After completing the browser login on Windows, `rclone authorize` prints a block of text starting with something like:

```
{"access_token":"...","token_type":"Bearer","refresh_token":"...","expiry":"..."}
```

Back on the **VPS**, the `rclone config` wizard is sitting at a prompt like:

```
Option config_token.
Paste your token here.
```

Paste that entire JSON-looking token (copied from the Windows terminal) into this VPS prompt, then press Enter. rclone stores it inside `~/.config/rclone/rclone.conf` on the VPS, associated with the `gdrive` remote name.

---

## 18. Testing the Connection: `rclone lsd gdrive:`

```bash
rclone lsd gdrive:
```

- `lsd` lists directories (not files) at the given remote path.
- `gdrive:` refers to the root of the configured `gdrive` remote (the colon separates the remote name from the path inside it).

A successful run lists the top-level folders in that Google Drive account, confirming the remote is correctly authorized.

---

## 19. Creating the `MongoDB Backups` Folder

```bash
rclone mkdir "gdrive:MongoDB Backups"
```

- `mkdir` creates a folder on the remote if it doesn't already exist (no error if it already does).
- Quoting `"gdrive:MongoDB Backups"` is necessary because the folder name contains a space.

---

## 20. Uploading a Test File

```bash
echo "test upload" > /home/manish/test.txt
rclone copy /home/manish/test.txt "gdrive:MongoDB Backups"
```

Then verify it arrived:

```bash
rclone ls "gdrive:MongoDB Backups"
```

Clean up afterward:

```bash
rm /home/manish/test.txt
rclone delete "gdrive:MongoDB Backups/test.txt"
```

---

## 21. Why `rclone copy mongodb_backup gdrive:"MongoDB Backups"` Can Fail

If your current working directory is **already** `~/mongodb_backup`, then running:

```bash
rclone copy mongodb_backup gdrive:"MongoDB Backups"
```

fails (or behaves unexpectedly) because rclone looks for a subdirectory literally named `mongodb_backup` *inside* the current directory — which doesn't exist, since you're already standing inside it. There is no `~/mongodb_backup/mongodb_backup`.

Fixes:
- Use `.` to mean "the current directory": `rclone copy . gdrive:"MongoDB Backups"`
- Or use the full/absolute path regardless of where you're standing: `rclone copy /home/manish/mongodb_backup gdrive:"MongoDB Backups"`

The final script (Section 24) avoids this ambiguity entirely by always using absolute paths and uploading a single named file rather than a directory.

---

## 22. `rclone copy` vs `rclone copyto`

| Command | Behavior |
|---|---|
| `rclone copy SRC DEST` | Copies the *contents* of SRC into DEST. If SRC is a directory, its files land inside DEST, keeping their own names. If DEST doesn't exist, it's created as a directory. |
| `rclone copyto SRC DEST` | Copies SRC to the exact path/name given by DEST. If SRC is a single file, DEST is treated as the exact destination filename — useful for uploading one file with full control over its remote name. |

For this pipeline, uploading **one specific file** with a known name, `copyto` is the more precise and predictable choice:

```bash
rclone copyto /home/manish/mongodb_backup/mongodb-backup-2026-08-15.archive.gz \
  "gdrive:MongoDB Backups/mongodb-backup-2026-08-15.archive.gz"
```

---

## 23. Why the Script Should Upload Only Today's File

If the script instead ran `rclone copy` (or `sync`) on the *entire* `mongodb_backup` folder every day, it would:

- Re-upload files that are already safely stored in Drive, wasting bandwidth and time.
- Make the "delete after successful upload" logic ambiguous — which files succeeded, which didn't?
- Risk accidentally deleting or overwriting backups if `sync` (which mirrors and deletes extras) were used instead of `copy`.

Uploading exactly **one, explicitly named file per run** keeps the success/failure check simple: one upload command, one exit code, one file to delete or keep.

---

## 24. Complete Backup Script

Save this as `/home/manish/mongodb_backup.sh`:

```bash
#!/bin/bash
#
# mongodb_backup.sh
# Dumps MongoDB Atlas, compresses it, uploads to Google Drive,
# and deletes the local copy only if the upload succeeds.

set -u  # Treat unset variables as an error (helps catch typos early)

# ---- Configuration ----
BACKUP_DIR="/home/manish/mongodb_backup"
DATE=$(date +%F)                                  # e.g. 2026-08-15
FILENAME="mongodb-backup-${DATE}.archive.gz"
LOCAL_PATH="${BACKUP_DIR}/${FILENAME}"
REMOTE_PATH="gdrive:MongoDB Backups/${FILENAME}"

# Load the MongoDB URI from an external, restricted-permission file
# instead of hardcoding it in this script (see Section 26/27).
source /home/manish/.mongodb_backup_env   # defines MONGODB_URI="..."

echo "===== Backup started: $(date) ====="

# ---- Step 1: Ensure backup directory exists ----
mkdir -p "$BACKUP_DIR"

# ---- Step 2: Run mongodump against MongoDB Atlas ----
echo "Running mongodump..."
mongodump --uri="$MONGODB_URI" --archive="$LOCAL_PATH" --gzip

# Capture mongodump's exit status immediately (must be the very next line)
MONGODUMP_STATUS=$?

if [ $MONGODUMP_STATUS -ne 0 ]; then
    echo "ERROR: mongodump failed with exit code $MONGODUMP_STATUS. Aborting."
    exit 1
fi

echo "mongodump completed successfully: $LOCAL_PATH"

# ---- Step 3: Upload only today's file to Google Drive ----
echo "Uploading to Google Drive..."
rclone copyto "$LOCAL_PATH" "$REMOTE_PATH"

RCLONE_STATUS=$?

# ---- Step 4: Delete local file ONLY if upload succeeded ----
if [ $RCLONE_STATUS -eq 0 ]; then
    echo "Upload successful. Deleting local backup to save disk space."
    rm -f "$LOCAL_PATH"
else
    echo "ERROR: rclone upload failed with exit code $RCLONE_STATUS."
    echo "Local backup KEPT at: $LOCAL_PATH"
fi

echo "===== Backup finished: $(date) ====="
```

Make it executable:

```bash
chmod 700 /home/manish/mongodb_backup.sh
```

---

## 25. Script Requirements Checklist

This script satisfies:

- [x] Creates today's backup with today's date in the filename
- [x] Connects to MongoDB Atlas (not a local Mongo instance)
- [x] Uses `--archive` (single-file format)
- [x] Uses `--gzip` (compression)
- [x] Uploads only today's file (via `copyto` with an explicit filename)
- [x] Checks `mongodump`'s exit status before continuing
- [x] Checks `rclone`'s exit status before deleting
- [x] Deletes the local file only after a successful upload
- [x] Keeps the local file if the upload fails
- [x] Prints clear `echo` status messages throughout

---

## 26. Security Issue of Hardcoding the MongoDB URI

The MongoDB Atlas connection string contains a **username and password**. If it's written directly inside `mongodb_backup.sh`:

- Anyone who can read the script (other users on a shared VPS, a misconfigured backup of the script itself, a Git repo it accidentally gets committed to) gets full database credentials.
- Shell history, log files, or `ps aux` output could also leak it if passed as a raw command-line argument in some contexts.
- Rotating the password later means hunting down and editing the script instead of updating one central place.

---

## 27. Recommended Secure Method for Storing Credentials

Store the URI in a separate file that only the owning user can read, and `source` it from the script:

```bash
# Create the credentials file
nano /home/manish/.mongodb_backup_env
```

Contents:

```bash
MONGODB_URI="YOUR_MONGODB_ATLAS_CONNECTION_STRING"
```

Lock down its permissions so only `manish` can read it:

```bash
chmod 600 /home/manish/.mongodb_backup_env
```

- `chmod 600` = read/write for the owner only, nothing for group or others.
- The backup script loads it with `source /home/manish/.mongodb_backup_env`, which sets `MONGODB_URI` as an environment variable for that script run only.

This keeps the secret in exactly one place, outside of version control, and separately permissioned from the script logic itself.

---

## 28. Explaining `chmod 700` for the Backup Script

```bash
chmod 700 /home/manish/mongodb_backup.sh
```

`700` breaks down as three permission digits (owner / group / others):

| Digit | Who | Permission | Meaning |
|---|---|---|---|
| 7 | Owner (`manish`) | rwx | Read, write, and **execute** |
| 0 | Group | — | No access at all |
| 0 | Others | — | No access at all |

Since the script indirectly touches database credentials (by sourcing the env file) and runs automatically, restricting it to the owner only — with execute permission so it can actually run — is the appropriate minimum.

---

## 29. Manually Testing the Complete Script

Run it directly, the same way cron eventually will:

```bash
/home/manish/mongodb_backup.sh
```

Watch the terminal for the `echo` messages defined in the script. Then confirm:

```bash
ls -lh /home/manish/mongodb_backup/          # should be empty if upload succeeded
rclone ls "gdrive:MongoDB Backups"           # should show today's file
```

If something fails, the local `.archive.gz` file will still be sitting in `mongodb_backup/`, which is itself a signal that the upload step didn't complete.

---

## 30. Creating the Cron Job from Scratch

```bash
crontab -e
```

- This opens the current user's (`manish`'s) personal crontab in a text editor (first run may ask you to choose an editor, e.g. `nano`).
- Add this line at the bottom of the file:

```
30 18 * * * /home/manish/mongodb_backup.sh >> /home/manish/mongodb_backup.log 2>&1
```

- Save and exit. Cron picks up the new schedule automatically — no restart needed.

---

## 31. Breaking Down `30 18 * * *`

Cron's five fields, in order, are: **minute, hour, day-of-month, month, day-of-week**.

| Field | Value | Meaning |
|---|---|---|
| Minute | `30` | At the 30th minute of the hour |
| Hour | `18` | At hour 18 (6 PM) in 24-hour format |
| Day of month | `*` | Every day of the month |
| Month | `*` | Every month |
| Day of week | `*` | Every day of the week |

Combined: **"run at 18:30, every day, every month, regardless of weekday."**

---

## 32. Why 18:30 UTC Equals 00:00 IST

India Standard Time is **UTC+5:30** — fixed year-round, with no daylight saving adjustments.

```
18:30 UTC + 5:30 = 24:00 → wraps to 00:00 the next day, IST
```

So a cron job firing at 18:30 on the VPS's UTC clock corresponds to exactly midnight (the start of the next day) in India — matching the requested "12:00 AM IST" backup time.

---

## 33. Why the VPS Timezone Should Remain UTC

- UTC never changes for daylight saving, so the cron schedule's meaning never silently shifts.
- Most server software, logs, and cloud provider dashboards default to and expect UTC — keeping the VPS on UTC avoids confusing cross-referencing later.
- Changing the system timezone to IST would require recalculating cron times whenever the server, its logs, or any other scheduled task is reasoned about — UTC + a documented offset (like this guide does) is simpler and less error-prone than changing system-wide state.
- Other services or scripts on the same VPS may implicitly assume UTC; changing it system-wide could have unintended side effects elsewhere.

---

## 34. Explaining `>> /home/manish/mongodb_backup.log 2>&1`

```
>> /home/manish/mongodb_backup.log 2>&1
```

- `>>` appends output to the file (rather than `>`, which would overwrite the log every single run).
- `2>&1` redirects file descriptor `2` (stderr) into wherever file descriptor `1` (stdout) is currently pointing — which, because it comes *after* the `>>` redirect, is now the log file.
- Order matters: `2>&1 >> file` would NOT work the same way, because stderr would be redirected to the terminal/wherever stdout was pointing *at that moment*, before stdout gets redirected to the file.

Net effect: both normal script output and any error messages land in the same log file.

---

## 35. Linux File Descriptors

| Descriptor | Name | Purpose |
|---|---|---|
| `0` | stdin | Standard input — where a program reads input from (keyboard, pipe, etc.) |
| `1` | stdout | Standard output — where normal program output goes |
| `2` | stderr | Standard error — where error/diagnostic messages go, separately from normal output |

These are numbered "streams" every process has by default; redirection operators (`>`, `>>`, `2>&1`) let you control where each one is sent.

---

## 36. Why Cron Logs Should Contain Both Output and Errors

Cron jobs run unattended — there's no terminal for anyone to watch in real time. If only stdout were captured:

- A failed `mongodump` or `rclone` command's error message would be silently discarded.
- You'd only discover a problem when you noticed backups were missing from Drive (or the disk filling up), with no clue why.

Capturing both streams in one log (Section 34) means a single `cat` or `tail` of the log file tells the whole story of any run — success or failure.

---

## 37. Verifying the Cron Job: `crontab -l`

```bash
crontab -l
```

Lists all cron jobs currently scheduled for the logged-in user (`manish`). Confirm the backup line appears exactly as intended, with no typos in the path or schedule.

---

## 38. Checking the Cron Service: `systemctl status cron`

```bash
systemctl status cron
```

Shows whether the system's cron daemon (`cron`/`cronjob` background service) is currently running. Look for `Active: active (running)` in the output. If cron isn't running, none of your scheduled jobs — no matter how correctly configured — will ever fire.

---

## 39. Enabling Cron If Necessary

If it's inactive or not enabled at boot:

```bash
sudo systemctl start cron     # start it now
sudo systemctl enable cron    # ensure it starts automatically on every reboot
```

---

## 40. Checking the Backup Log

View the entire log:

```bash
cat /home/manish/mongodb_backup.log
```

View just the most recent run(s):

```bash
tail -20 /home/manish/mongodb_backup.log
```

`tail -20` shows the last 20 lines — handy for checking only the latest run without scrolling through a long history.

---

## 41. Verifying the Backup Exists in Google Drive

From the VPS (or any machine with rclone configured against the same account):

```bash
rclone ls "gdrive:MongoDB Backups"
```

Lists every file in the folder along with its size. Confirm today's filename (`mongodb-backup-YYYY-MM-DD.archive.gz`) is present. Alternatively, log in to Google Drive in a browser and open the **MongoDB Backups** folder directly.

---

## 42. What Happens If the VPS Disk Becomes Full

If `/home` runs out of space:

- `mongodump` may fail partway through, producing an incomplete or corrupt archive.
- The script's status check would catch the mongodump failure and abort before attempting an upload of a broken file — but the underlying disk-full condition still needs manual attention.
- Other unrelated services on the VPS (web server, database, logs) could also start failing once the disk is full, since it's a whole-filesystem condition, not something scoped to just the backup folder.

---

## 43. Why Deleting the Local Backup After Success Prevents Disk Filling

Each daily dump occupies real disk space on the VPS. If backups were never deleted locally:

- The `mongodb_backup` folder would grow by one file every day, indefinitely.
- Eventually it would consume all available disk space on the VPS, potentially crashing other services sharing that disk.

Deleting the file immediately after a **confirmed successful** upload means the VPS only ever holds, at most, one day's backup locally (and zero, most of the time) — the durable copy lives in Google Drive instead.

---

## 44. Why Google Drive Can Still Eventually Become Full

Moving the storage problem to Google Drive doesn't eliminate it — it just changes where it happens:

- Google Drive accounts have finite quotas (commonly 15 GB on free tiers, see Section 10).
- Every daily upload adds a new file; with no cleanup, the *cloud* folder grows forever instead of the local one.
- Once quota is reached, `rclone copyto` uploads will start failing — and per the script's logic, that means local files will start accumulating again too (since local deletion only happens after a successful upload).

This is why a retention policy on the Drive side (next section) is still necessary even though local space is already handled.

---

## 45. Recommended Retention Strategy

A simple, common approach: **keep the last 30 days of backups, delete anything older.**

- 30 days gives enough history to recover from a problem discovered days or even a few weeks later.
- It bounds storage growth to a predictable maximum (30 files' worth of backups, not unlimited).
- Adjust the number based on your actual backup size and available quota — e.g., 14 or 60 days if that fits your needs better.

---

## 46. Optional: Automatically Deleting Google Drive Backups Older Than 30 Days

`rclone` can delete remote files older than a given age:

```bash
rclone delete "gdrive:MongoDB Backups" --min-age 30d
```

- `--min-age 30d` targets only files whose modification time is 30 days or older.
- `delete` removes matching files but leaves the folder itself and newer files intact.

To wire this into the daily script (append near the end of `mongodb_backup.sh`, after the upload block):

```bash
# ---- Step 5 (optional): Clean up backups older than 30 days on Google Drive ----
echo "Removing Google Drive backups older than 30 days..."
rclone delete "gdrive:MongoDB Backups" --min-age 30d
```

---

## 47. Risks of Deleting Old Backups — Test First

**Warning:** Deletion commands are irreversible through rclone itself (Google Drive's own Trash may offer a short-lived safety net, but don't rely on it).

Before enabling automatic deletion in the daily script:

1. **Dry-run it first** to see exactly what *would* be deleted, without deleting anything:
   ```bash
   rclone delete "gdrive:MongoDB Backups" --min-age 30d --dry-run
   ```
2. Review the listed files carefully — confirm the age filter is behaving as expected.
3. Only remove `--dry-run` once you're confident the correct (and only the correct) files are targeted.
4. Consider keeping this step manual, or running it weekly rather than daily, so you have a chance to notice if something looks wrong before it compounds.

---

## 48. Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| `mongodump: command not found` | Database Tools not installed, or not in `$PATH` | Reinstall per Section 4; check `which mongodump` |
| Authentication failed | Wrong username/password in URI, or user lacks permissions on the target database | Re-copy the connection string from Atlas; verify the database user's roles |
| MongoDB Atlas network timeout | VPS can't reach Atlas at all (firewall, DNS, outbound blocked) | Test with `curl -v https://cloud.mongodb.com`; check VPS outbound rules |
| IP not allowed in Atlas | VPS's public IP isn't in Atlas Network Access list | Re-check `curl ifconfig.me` and re-add it per Section 5 (IPs can change if not static) |
| `rclone: remote not found` | Remote wasn't named `gdrive`, or config wasn't saved | Run `rclone config show` to list configured remotes |
| Google Drive authorization problems | Token expired, revoked, or authorize step was interrupted | Re-run `rclone config reconnect gdrive:` or redo Section 13–17 |
| Permission denied | Script isn't executable, or files owned by wrong user | `chmod 700` the script; check `ls -l` ownership |
| Cron not running | `cron` service stopped | Section 38–39: check and (re)start/enable it |
| Cron running at the wrong time | Timezone confusion, or wrong UTC offset calculated | Confirm VPS timezone with `timedatectl`; re-verify the UTC math in Section 32 |
| Backup uploaded but local file not deleted | rclone reported non-zero exit despite an apparent success (e.g., a partial/flaky upload), or script logic issue | Check the log for the exact `rclone` exit code and message around that run |

---

## 49. Quick Setup (Essential Commands Only)

```bash
# 1. Install mongodump (if needed)
curl -O https://fastdl.mongodb.org/tools/db/mongodb-database-tools-ubuntu2204-x86_64-100.17.0.deb
sudo dpkg -i mongodb-database-tools-ubuntu2204-x86_64-100.17.0.deb

# 2. Install rclone
sudo apt update && sudo apt install -y rclone

# 3. Configure Google Drive remote (interactive — see Sections 13-17)
rclone config

# 4. Create backup folder
mkdir -p /home/manish/mongodb_backup

# 5. Store credentials securely
nano /home/manish/.mongodb_backup_env   # MONGODB_URI="YOUR_MONGODB_ATLAS_CONNECTION_STRING"
chmod 600 /home/manish/.mongodb_backup_env

# 6. Create and secure the backup script (paste content from Section 24)
nano /home/manish/mongodb_backup.sh
chmod 700 /home/manish/mongodb_backup.sh

# 7. Test manually
/home/manish/mongodb_backup.sh

# 8. Schedule via cron
crontab -e
# add: 30 18 * * * /home/manish/mongodb_backup.sh >> /home/manish/mongodb_backup.log 2>&1
```

---

## 50. Daily Operation (What Happens Automatically)

Every day at 18:30 UTC (00:00 IST):

1. Cron triggers `/home/manish/mongodb_backup.sh`.
2. The script loads the MongoDB URI from the protected env file.
3. `mongodump` connects to Atlas and writes a compressed archive named with today's date.
4. If the dump fails, the script logs the error and stops — nothing is uploaded or deleted.
5. If the dump succeeds, `rclone` uploads that one file to the `MongoDB Backups` folder on Google Drive.
6. If the upload succeeds, the local file is deleted, freeing VPS disk space.
7. If the upload fails, the local file is kept as a fallback copy.
8. All of the above status messages are appended to `/home/manish/mongodb_backup.log`.

No manual action is needed on a normal day. Periodically check the log and Google Drive folder (see Sections 40–41) to confirm things are healthy.

---

## 51. Disaster Recovery — Restoring a Backup

If the live MongoDB Atlas database is lost, corrupted, or needs to be rolled back:

1. **Download the desired backup file from Google Drive** to the machine you'll restore from (this can be the VPS, or your local machine — anywhere with `mongorestore` and network access to Atlas):
   ```bash
   rclone copy "gdrive:MongoDB Backups/mongodb-backup-2026-08-15.archive.gz" .
   ```
2. **Restore it into Atlas** using `mongorestore` (see Section 52–53 below).

---

## 52. How `mongorestore` Works with a Gzip Archive

`mongorestore` is the counterpart to `mongodump`. Given the same kind of single-file archive that `mongodump --archive --gzip` produced, it can read it back with matching flags:

- `--archive=FILE` tells it to read from a single archive file instead of a directory of BSON files.
- `--gzip` tells it the archive is gzip-compressed (must match how it was created).
- `--uri` tells it which MongoDB deployment to restore *into*.

Internally, mongorestore decompresses the stream and replays the collections/documents it contains into the target database.

---

## 53. Example Restore Commands

> **⚠️ WARNING — DESTRUCTIVE POTENTIAL:** Restoring can overwrite or duplicate existing data in the target database if used carelessly. By default, `mongorestore` will error on existing collections rather than silently wiping them, but flags like `--drop` will **delete existing collections before restoring** — an irreversible action if you don't have another backup. Always double-check which database/cluster the `--uri` points to before running any restore command, especially in production.

Restore into a database, keeping any existing data (fails if collections already exist and would conflict):

```bash
mongorestore --uri="YOUR_MONGODB_ATLAS_CONNECTION_STRING" \
  --archive="mongodb-backup-2026-08-15.archive.gz" \
  --gzip
```

Restore and **replace** existing collections (drops them first — use with caution, ideally against a test/staging cluster first):

```bash
mongorestore --uri="YOUR_MONGODB_ATLAS_CONNECTION_STRING" \
  --archive="mongodb-backup-2026-08-15.archive.gz" \
  --gzip \
  --drop
```

Restore into a *different* database name than the original (useful for testing a backup without touching production):

```bash
mongorestore --uri="YOUR_MONGODB_ATLAS_CONNECTION_STRING" \
  --archive="mongodb-backup-2026-08-15.archive.gz" \
  --gzip \
  --nsFrom="originalDbName.*" \
  --nsTo="restoreTestDbName.*"
```

---

## 54. Security Checklist

- [ ] MongoDB URI is stored in `~/.mongodb_backup_env`, not hardcoded in the script.
- [ ] `.mongodb_backup_env` has permissions `600` (owner read/write only).
- [ ] `mongodb_backup.sh` has permissions `700` (owner read/write/execute only).
- [ ] Atlas Network Access allows only the VPS's specific IP (not `0.0.0.0/0`, unless there's a strong reason).
- [ ] The Atlas database user used for backups has only the permissions it needs (read access is sufficient for `mongodump`).
- [ ] rclone's config file (`~/.config/rclone/rclone.conf`, which stores the Drive OAuth token) is not shared, committed to Git, or backed up somewhere public.
- [ ] No credentials appear in `mongodb_backup.log` (the script only echoes status messages, never the URI itself).

---

## 55. Verification Checklist

- [ ] `mongodump --version` shows the expected version.
- [ ] `rclone lsd gdrive:` lists folders successfully.
- [ ] `rclone ls "gdrive:MongoDB Backups"` shows the folder exists.
- [ ] Manual run of `/home/manish/mongodb_backup.sh` completes without errors.
- [ ] After a manual run, `mongodb_backup/` is empty and Drive has the new file.
- [ ] `crontab -l` shows the correct schedule line.
- [ ] `systemctl status cron` shows the service active.
- [ ] After 24+ hours, `/home/manish/mongodb_backup.log` shows a new automatic run with success messages.
- [ ] The backup filename date in Drive matches the expected day (confirming the UTC/IST timing is correct).

---

## 56. Useful Commands

```bash
# Check mongodump version
mongodump --version

# Check VPS current time and timezone
timedatectl

# Check VPS public IP (for Atlas Network Access)
curl ifconfig.me

# List configured rclone remotes
rclone config show

# List files in the Drive backup folder
rclone ls "gdrive:MongoDB Backups"

# View full backup log
cat /home/manish/mongodb_backup.log

# View latest log lines
tail -20 /home/manish/mongodb_backup.log

# List current cron jobs
crontab -l

# Edit cron jobs
crontab -e

# Check cron service status
systemctl status cron

# Manually run the backup right now
/home/manish/mongodb_backup.sh

# Dry-run a retention cleanup (see what would be deleted, without deleting)
rclone delete "gdrive:MongoDB Backups" --min-age 30d --dry-run
```

---

## 57. Cheat Sheet (One Page)

**Setup:**
```bash
sudo apt install -y rclone
rclone config                          # create remote named: gdrive
mkdir -p /home/manish/mongodb_backup
nano /home/manish/.mongodb_backup_env  # MONGODB_URI="..."
chmod 600 /home/manish/.mongodb_backup_env
nano /home/manish/mongodb_backup.sh    # paste script from Section 24
chmod 700 /home/manish/mongodb_backup.sh
```

**Schedule:**
```bash
crontab -e
# 30 18 * * * /home/manish/mongodb_backup.sh >> /home/manish/mongodb_backup.log 2>&1
```

**Test:**
```bash
/home/manish/mongodb_backup.sh
rclone ls "gdrive:MongoDB Backups"
tail -20 /home/manish/mongodb_backup.log
```

**Monitor:**
```bash
crontab -l
systemctl status cron
cat /home/manish/mongodb_backup.log
```

**Restore (⚠️ can overwrite data — verify target first):**
```bash
rclone copy "gdrive:MongoDB Backups/mongodb-backup-YYYY-MM-DD.archive.gz" .
mongorestore --uri="YOUR_MONGODB_ATLAS_CONNECTION_STRING" \
  --archive="mongodb-backup-YYYY-MM-DD.archive.gz" --gzip
```

**Key facts:**
- Cron `30 18 * * *` = 18:30 UTC = 00:00 IST (UTC+5:30, no DST).
- Keep VPS on UTC — don't change system timezone.
- Local backup is deleted only after a confirmed successful upload.
- Retention: delete Drive backups older than 30 days — always `--dry-run` first.
