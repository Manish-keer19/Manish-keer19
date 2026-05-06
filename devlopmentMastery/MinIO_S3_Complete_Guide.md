# 🪣 MinIO Complete Mastery Guide
### From Beginner to Production — Build Your Own S3-Compatible Storage on a VPS

> **Author:** Senior DevOps Engineer & Backend Architect  
> **Stack:** MinIO · Node.js · Express · NGINX · Linux (Ubuntu) · Docker  
> **Goal:** Run your own object storage, integrate it with Node.js, and scale it to production.

---

## Table of Contents

1. [Introduction to MinIO](#1-introduction-to-minio)
2. [Core Concepts](#2-core-concepts)
3. [Installing MinIO on VPS (Linux)](#3-installing-minio-on-vps-linux)
4. [MinIO Console (Web UI)](#4-minio-console-web-ui)
5. [MinIO Client (mc CLI) — Full Guide](#5-minio-client-mc-cli--full-guide)
6. [Node.js Integration](#6-nodejs-integration)
7. [File Upload System (Backend)](#7-file-upload-system-backend)
8. [Frontend Integration](#8-frontend-integration)
9. [Security Best Practices](#9-security-best-practices)
10. [Production Deployment](#10-production-deployment)
11. [Scaling & Advanced Topics](#11-scaling--advanced-topics)
12. [Comparison with AWS S3](#12-comparison-with-aws-s3)
13. [Troubleshooting](#13-troubleshooting)
14. [Real-World Architecture](#14-real-world-architecture)
15. [Bonus — Docker Setup](#15-bonus--docker-setup)

---

## 1. Introduction to MinIO

### What is MinIO?

MinIO is a **high-performance, S3-compatible object storage system** that you can self-host. It is written in Go, open-source (AGPL v3), and designed for cloud-native workloads.

Think of it as: **"Your own AWS S3, running on your server."**

You store files (called **objects**) in containers (called **buckets**), accessible via an HTTP API that is 100% compatible with the AWS S3 API. Any library or tool that works with S3 — works with MinIO too.

```
┌─────────────────────────────────────────────┐
│              YOUR VPS / SERVER              │
│                                             │
│   ┌─────────────────────────────────────┐   │
│   │           MinIO Server              │   │
│   │                                     │   │
│   │  Bucket: images/                    │   │
│   │    ├── avatar.png                   │   │
│   │    ├── banner.jpg                   │   │
│   │                                     │   │
│   │  Bucket: videos/                    │   │
│   │    ├── intro.mp4                    │   │
│   │                                     │   │
│   │  Bucket: backups/                   │   │
│   │    ├── db-2024-01-01.tar.gz         │   │
│   └─────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

### How it Compares with AWS S3

| Feature | AWS S3 | MinIO |
|---|---|---|
| API Compatibility | Native S3 | 100% S3-compatible |
| Hosting | AWS Cloud | Self-hosted (VPS/on-prem) |
| Cost | Pay per GB + requests | Only server cost |
| Setup | Minutes | 10–30 minutes |
| Scalability | Infinite (managed) | Manual (distributed mode) |
| Latency | Depends on region | Very low (same datacenter) |
| Privacy / GDPR | Data on AWS | Data on your server |
| SDK Support | AWS SDK | AWS SDK (same!) |

### Use Cases

- **Image & media storage** — profile pictures, thumbnails, course videos
- **Document management** — PDFs, reports, invoices
- **App backups** — MongoDB dumps, application logs
- **CDN origin** — serve static files through NGINX
- **ML datasets** — training data, model weights
- **Multi-tenant file systems** — one bucket per user/org

---

## 2. Core Concepts

### Buckets

A **bucket** is like a folder at the top level. You can't nest buckets inside buckets.

```
minio-server/
  ├── images/          ← Bucket
  ├── videos/          ← Bucket
  └── documents/       ← Bucket
```

- Bucket names must be globally unique within your MinIO instance.
- Names are lowercase, 3–63 characters, no spaces.
- Example: `shellelearning-images`, `user-uploads-2024`

### Objects

An **object** = your actual file + its metadata. Objects live inside buckets.

```
images/
  ├── users/avatar-1001.png       ← Object (with prefix path)
  ├── courses/banner-react.jpg    ← Object
  └── thumbnails/yt-thumb.webp    ← Object
```

Objects are referenced by a **key** (the full path within the bucket):
- Bucket: `images`
- Key: `users/avatar-1001.png`

### Object Storage vs File System

```
FILE SYSTEM (traditional)              OBJECT STORAGE (MinIO/S3)
─────────────────────────              ──────────────────────────
/home/uploads/                         Bucket: uploads
  ├── users/                             Key: users/avatar.png
  │   └── avatar.png                     Key: users/profile.jpg
  └── courses/                           Key: courses/banner.jpg
      └── banner.jpg

Nested directories                     Flat namespace with "/" prefix
OS-level permissions                   Policy-based (JSON)
Mounted disk required                  HTTP API access
Hard to scale                          Horizontally scalable
```

**Key differences:**
- Object storage has no real folders — the `/` in a key is just a naming convention.
- Every object is accessed via HTTP, not `fopen()`.
- Objects are immutable — you replace, not patch.
- Each object has metadata: content-type, size, ETag, custom headers.

### Access Key / Secret Key

MinIO uses AWS-style credentials:

- **Access Key** (like a username) — identifies WHO is making the request
- **Secret Key** (like a password) — signs the request cryptographically

```
Access Key:  minioadmin          (default, change in prod!)
Secret Key:  minioadmin123       (default, change in prod!)
```

These are used in your Node.js config, CLI, and SDK — just like AWS credentials.

### Policies & Permissions

MinIO uses **JSON-based bucket policies** (identical to AWS IAM).

**Public read policy** (anyone can download):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": ["s3:GetObject"],
      "Resource": ["arn:aws:s3:::my-bucket/*"]
    }
  ]
}
```

**Private (default):** Only authenticated users with correct keys can access.

**Common permission types:**
- `s3:GetObject` — download/read
- `s3:PutObject` — upload/write
- `s3:DeleteObject` — delete
- `s3:ListBucket` — list contents

---

## 3. Installing MinIO on VPS (Linux)

> **Prerequisites:** Ubuntu 20.04/22.04 VPS, sudo access, at least 1 GB RAM, 10 GB disk.

### Step 1 — Download the MinIO Binary

```bash
# Download latest MinIO binary
wget https://dl.min.io/server/minio/release/linux-amd64/minio

# Make it executable
chmod +x minio

# Move to system PATH
sudo mv minio /usr/local/bin/minio

# Verify installation
minio --version
# Output: minio version RELEASE.2024-XX-XX...
```

### Step 2 — Create Dedicated User and Data Directory

```bash
# Create a system user for MinIO (no login shell)
sudo useradd -r minio-user -s /sbin/nologin

# Create directory for data storage
sudo mkdir -p /mnt/minio-data

# Set ownership
sudo chown minio-user:minio-user /mnt/minio-data

# Verify
ls -la /mnt/
```

### Step 3 — Create Environment File

```bash
# Create the environment configuration file
sudo nano /etc/default/minio
```

Add the following content:

```bash
# /etc/default/minio

# Root credentials (CHANGE THESE IN PRODUCTION!)
MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=StrongPassword@2024!

# Directory where MinIO stores data
MINIO_VOLUMES="/mnt/minio-data"

# MinIO server options
# API will run on port 9000
# Console (Web UI) will run on port 9001
MINIO_OPTS="--console-address :9001"

# Optional: Set your domain for the console
# MINIO_BROWSER_REDIRECT_URL=https://console.yourdomain.com
```

```bash
# Secure the file (only root can read it)
sudo chmod 600 /etc/default/minio
```

### Step 4 — Create systemd Service (Auto-Start)

```bash
sudo nano /etc/systemd/system/minio.service
```

```ini
# /etc/systemd/system/minio.service

[Unit]
Description=MinIO Object Storage Server
Documentation=https://min.io/docs
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/usr/local/bin/minio

[Service]
# Run as dedicated minio user
User=minio-user
Group=minio-user

# Load environment variables from config file
EnvironmentFile=-/etc/default/minio

# Start command — uses $MINIO_OPTS and $MINIO_VOLUMES from env file
ExecStartPre=/bin/bash -c "if [ -z \"${MINIO_VOLUMES}\" ]; then echo \"Variable MINIO_VOLUMES not set\"; exit 1; fi"
ExecStart=/usr/local/bin/minio server $MINIO_OPTS $MINIO_VOLUMES

# Restart policy
Restart=always
RestartSec=5s

# Logging
StandardOutput=journal
StandardError=inherit

# Security hardening
LimitNOFILE=65536
TasksMax=infinity
TimeoutStopSec=infinity
SendSIGKILL=no

[Install]
WantedBy=multi-user.target
```

### Step 5 — Enable and Start MinIO

```bash
# Reload systemd to recognize new service
sudo systemctl daemon-reload

# Enable MinIO to start on boot
sudo systemctl enable minio

# Start the service
sudo systemctl start minio

# Check status
sudo systemctl status minio
```

**Expected output:**
```
● minio.service - MinIO Object Storage Server
     Loaded: loaded (/etc/systemd/system/minio.service; enabled)
     Active: active (running) since ...
```

### Step 6 — Check Logs

```bash
# View live logs
sudo journalctl -u minio -f

# Last 50 lines
sudo journalctl -u minio -n 50
```

### Step 7 — Open Firewall Ports

```bash
# Allow MinIO API port
sudo ufw allow 9000/tcp

# Allow MinIO Web Console port
sudo ufw allow 9001/tcp

# Check rules
sudo ufw status
```

### Verify MinIO is Running

```bash
# Test via curl
curl http://localhost:9000/minio/health/live
# Should return: HTTP 200

curl http://localhost:9000/minio/health/ready
# Should return: HTTP 200
```

### Production-Ready Environment Variables Reference

```bash
# /etc/default/minio (full production example)

# Credentials
MINIO_ROOT_USER=your_access_key_here
MINIO_ROOT_PASSWORD=your_very_strong_secret_here_min_8_chars

# Storage path (can be multiple for distributed)
MINIO_VOLUMES="/mnt/minio-data"

# Server options
MINIO_OPTS="--console-address :9001"

# Domain configuration (when using NGINX reverse proxy)
MINIO_SERVER_URL=https://s3.yourdomain.com
MINIO_BROWSER_REDIRECT_URL=https://console.yourdomain.com

# Region (optional, for S3 SDK compatibility)
MINIO_REGION=us-east-1

# Enable prometheus metrics (optional)
# MINIO_PROMETHEUS_AUTH_TYPE=public
```

---

## 4. MinIO Console (Web UI)

### Accessing the Console

Open your browser and navigate to:

```
http://YOUR_VPS_IP:9001
```

Login with your `MINIO_ROOT_USER` and `MINIO_ROOT_PASSWORD`.

```
┌──────────────────────────────────────────────────────┐
│                  MinIO Console                        │
│                                                       │
│         Username: [ minioadmin            ]          │
│         Password: [ ••••••••••••          ]          │
│                                                       │
│                  [ Login ]                            │
└──────────────────────────────────────────────────────┘
```

### Create a Bucket via Console

1. Click **"Buckets"** in the left sidebar
2. Click **"Create Bucket +"**
3. Enter bucket name (e.g., `shellelearning-images`)
4. Toggle options:
   - **Versioning** — keep older versions of objects
   - **Object Locking** — prevent deletion (for compliance)
5. Click **"Create Bucket"**

### Upload Files via Console

1. Go to **Buckets → your-bucket**
2. Click **"Upload"**
3. Drag and drop files or browse
4. Files appear in the bucket listing instantly

### Manage Users and Policies

**Create a new user:**
1. Go to **Identity → Users**
2. Click **"Create User"**
3. Set username + password
4. Assign a policy (readwrite, readonly, writeonly, or custom)

**Create a custom policy:**
1. Go to **Identity → Policies**
2. Click **"Create Policy"**
3. Write JSON policy (example below)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::shellelearning-images",
        "arn:aws:s3:::shellelearning-images/*"
      ]
    }
  ]
}
```

### Console Feature Overview

```
Dashboard          → Storage usage, object counts, network stats
Buckets            → Create/manage buckets, set policies, browse objects
Identity → Users   → Manage users, assign policies
Identity → Groups  → Group users, assign group-level policies
Identity → Policies→ Create/edit JSON policies
Monitoring         → Real-time metrics, logs, traces
Settings           → Server config, TLS, notifications
```

---

## 5. MinIO Client (mc CLI) — Full Guide

The `mc` CLI is the command-line tool for managing MinIO (and also AWS S3, GCS, etc.).

### Install mc

```bash
# Download mc binary
wget https://dl.min.io/client/mc/release/linux-amd64/mc

# Make executable
chmod +x mc

# Move to PATH
sudo mv mc /usr/local/bin/mc

# Verify
mc --version
# Output: mc version RELEASE.2024-XX-XX...
```

### Configure Alias (Connect to Your MinIO Server)

An **alias** is a shortcut name for your MinIO instance.

```bash
# Syntax:
# mc alias set <ALIAS_NAME> <ENDPOINT_URL> <ACCESS_KEY> <SECRET_KEY>

mc alias set myminio http://localhost:9000 minioadmin StrongPassword@2024!

# For remote VPS:
mc alias set myminio http://YOUR_VPS_IP:9000 minioadmin StrongPassword@2024!

# For HTTPS (production):
mc alias set myminio https://s3.yourdomain.com minioadmin StrongPassword@2024!
```

**Verify the alias works:**
```bash
mc admin info myminio
```

### mc ls — List Buckets and Objects

```bash
# List all buckets
mc ls myminio

# List contents of a bucket
mc ls myminio/shellelearning-images

# List with human-readable sizes
mc ls --summarize myminio/shellelearning-images

# Recursive list (all files, all "subfolders")
mc ls --recursive myminio/shellelearning-images

# List objects with a specific prefix
mc ls myminio/shellelearning-images/users/
```

### mc mb — Make Bucket

```bash
# Create a bucket
mc mb myminio/my-new-bucket

# Create with region (for SDK compatibility)
mc mb --region us-east-1 myminio/my-new-bucket

# Create with object locking enabled
mc mb --with-lock myminio/locked-bucket
```

### mc cp — Copy Files

```bash
# Upload a local file to MinIO
mc cp ./photo.jpg myminio/shellelearning-images/users/avatar.jpg

# Download a file from MinIO
mc cp myminio/shellelearning-images/users/avatar.jpg ./downloaded-avatar.jpg

# Copy within MinIO (between buckets)
mc cp myminio/bucket-a/file.txt myminio/bucket-b/file.txt

# Upload entire folder (recursive)
mc cp --recursive ./uploads/ myminio/shellelearning-images/batch/

# Upload with content-type header
mc cp --attr "Content-Type=image/jpeg" ./photo.jpg myminio/images/photo.jpg
```

### mc rm — Remove Objects

```bash
# Delete a single object
mc rm myminio/shellelearning-images/users/old-avatar.jpg

# Delete multiple objects
mc rm myminio/bucket/file1.txt myminio/bucket/file2.txt

# Delete all objects in a bucket (recursive)
mc rm --recursive --force myminio/shellelearning-images/temp/

# Delete a bucket AND all its objects
mc rb --force myminio/old-bucket
```

### mc admin — Server Administration

```bash
# Server info and health
mc admin info myminio

# Disk usage
mc admin disk myminio

# Create a new user
mc admin user add myminio newuser newpassword123

# List all users
mc admin user list myminio

# Disable a user
mc admin user disable myminio username

# Remove a user
mc admin user remove myminio username

# View server logs (live)
mc admin logs myminio

# Server configuration
mc admin config get myminio

# Restart MinIO server
mc admin service restart myminio
```

### Bucket Policies — Public vs Private

```bash
# Make a bucket publicly readable (anyone can download)
mc anonymous set download myminio/public-assets

# Make bucket public for read AND list
mc anonymous set public myminio/public-assets

# Remove public access (back to private)
mc anonymous set none myminio/public-assets

# View current policy
mc anonymous get myminio/public-assets

# Apply a custom JSON policy from file
mc anonymous set-json ./policy.json myminio/my-bucket

# View current JSON policy
mc anonymous get-json myminio/my-bucket
```

### Generate Presigned URLs (Temporary Share Links)

Presigned URLs let you share a private file temporarily without changing permissions.

```bash
# Generate a download URL valid for 24 hours (1440 minutes)
mc share download --expire 24h myminio/shellelearning-images/users/avatar.jpg

# Generate a download URL valid for 7 days
mc share download --expire 168h myminio/bucket/file.pdf

# Generate an upload URL (so client can upload directly)
mc share upload --expire 1h myminio/uploads/new-file.jpg
```

**Output example:**
```
URL: http://localhost:9000/shellelearning-images/users/avatar.jpg
Expire: 24 hours 0 minutes 0 seconds
Share: http://localhost:9000/shellelearning-images/users/avatar.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&...
```

### Sync Folders

```bash
# Sync local folder to MinIO bucket (upload new/changed)
mc mirror ./local-folder/ myminio/my-bucket/

# Sync and delete files in MinIO that don't exist locally
mc mirror --overwrite --remove ./local-folder/ myminio/my-bucket/

# Sync MinIO bucket to local (download)
mc mirror myminio/my-bucket/ ./local-backup/

# Sync between two MinIO buckets
mc mirror myminio/source-bucket/ myminio/dest-bucket/

# Watch mode (real-time sync)
mc mirror --watch ./local-folder/ myminio/my-bucket/
```

---

## 6. Node.js Integration

MinIO is 100% S3-compatible, so we use the official **AWS SDK for JavaScript v3**.

### Install Dependencies

```bash
npm install @aws-sdk/client-s3 @aws-sdk/s3-request-presigner
```

### Setup Configuration File

```javascript
// config/minio.js

const { S3Client } = require("@aws-sdk/client-s3");

// MinIO configuration using AWS SDK S3Client
const minioClient = new S3Client({
  endpoint: process.env.MINIO_ENDPOINT || "http://localhost:9000",
  region: process.env.MINIO_REGION || "us-east-1",  // Any string works for MinIO
  credentials: {
    accessKeyId: process.env.MINIO_ACCESS_KEY || "minioadmin",
    secretAccessKey: process.env.MINIO_SECRET_KEY || "StrongPassword@2024!",
  },
  // IMPORTANT: This tells AWS SDK NOT to use virtual-hosted-style URLs
  // MinIO requires path-style: http://host:9000/bucket/key
  // (not virtual: http://bucket.host:9000/key)
  forcePathStyle: true,
});

const BUCKET_NAME = process.env.MINIO_BUCKET || "shellelearning-images";

module.exports = { minioClient, BUCKET_NAME };
```

**.env file:**
```env
MINIO_ENDPOINT=http://localhost:9000
MINIO_REGION=us-east-1
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=StrongPassword@2024!
MINIO_BUCKET=shellelearning-images
```

### Upload a File

```javascript
// services/storage.js

const {
  S3Client,
  PutObjectCommand,
  GetObjectCommand,
  DeleteObjectCommand,
  ListObjectsV2Command,
} = require("@aws-sdk/client-s3");
const { getSignedUrl } = require("@aws-sdk/s3-request-presigner");
const { minioClient, BUCKET_NAME } = require("../config/minio");
const fs = require("fs");
const path = require("path");

/**
 * Upload a file from local filesystem to MinIO
 * @param {string} localFilePath - Path to local file
 * @param {string} objectKey - Destination key in bucket (e.g., "users/avatar.jpg")
 * @param {string} contentType - MIME type (e.g., "image/jpeg")
 */
async function uploadFile(localFilePath, objectKey, contentType) {
  // Read the file into a buffer
  const fileBuffer = fs.readFileSync(localFilePath);
  const fileSize = fs.statSync(localFilePath).size;

  const command = new PutObjectCommand({
    Bucket: BUCKET_NAME,
    Key: objectKey,
    Body: fileBuffer,
    ContentType: contentType,
    ContentLength: fileSize,
  });

  const response = await minioClient.send(command);
  console.log(`✅ Uploaded: ${objectKey}`);
  return response;
}
```

### Upload Image (from Buffer — most common in Express)

```javascript
/**
 * Upload a file buffer directly to MinIO
 * Used when you have the file in memory (e.g., from Multer memoryStorage)
 * @param {Buffer} buffer - File data as Buffer
 * @param {string} objectKey - Destination key in bucket
 * @param {string} mimeType - MIME type (e.g., "image/png")
 */
async function uploadBuffer(buffer, objectKey, mimeType) {
  const command = new PutObjectCommand({
    Bucket: BUCKET_NAME,
    Key: objectKey,            // e.g., "users/avatar-1001.jpg"
    Body: buffer,              // File data as Buffer
    ContentType: mimeType,     // "image/jpeg", "image/png", etc.
    ContentLength: buffer.length,
  });

  await minioClient.send(command);

  // Return the public URL (if bucket is public)
  const publicUrl = `${process.env.MINIO_ENDPOINT}/${BUCKET_NAME}/${objectKey}`;
  return publicUrl;
}
```

### Get / Download a File

```javascript
/**
 * Download an object from MinIO as a Buffer
 * @param {string} objectKey - Key of the object to download
 * @returns {Buffer} File data
 */
async function getFile(objectKey) {
  const command = new GetObjectCommand({
    Bucket: BUCKET_NAME,
    Key: objectKey,
  });

  const response = await minioClient.send(command);

  // response.Body is a ReadableStream, convert to Buffer
  const chunks = [];
  for await (const chunk of response.Body) {
    chunks.push(chunk);
  }

  return Buffer.concat(chunks);
}

/**
 * Stream a file directly to an HTTP response (Express)
 * Better for large files — doesn't load entire file into memory
 * @param {string} objectKey - Object key in MinIO
 * @param {object} res - Express response object
 */
async function streamFile(objectKey, res) {
  const command = new GetObjectCommand({
    Bucket: BUCKET_NAME,
    Key: objectKey,
  });

  const response = await minioClient.send(command);

  // Set content type header from MinIO metadata
  res.setHeader("Content-Type", response.ContentType);
  res.setHeader("Content-Length", response.ContentLength);

  // Pipe the stream directly to HTTP response
  response.Body.pipe(res);
}
```

### Delete a File

```javascript
/**
 * Delete a single object from MinIO
 * @param {string} objectKey - Key of the object to delete
 */
async function deleteFile(objectKey) {
  const command = new DeleteObjectCommand({
    Bucket: BUCKET_NAME,
    Key: objectKey,
  });

  await minioClient.send(command);
  console.log(`🗑️ Deleted: ${objectKey}`);
}
```

### List Files

```javascript
/**
 * List all objects in the bucket (or with a prefix)
 * @param {string} prefix - Optional prefix filter (e.g., "users/")
 * @returns {Array} List of objects with key, size, lastModified
 */
async function listFiles(prefix = "") {
  const command = new ListObjectsV2Command({
    Bucket: BUCKET_NAME,
    Prefix: prefix,       // Filter by prefix (like a folder path)
    MaxKeys: 100,         // Limit results per page
  });

  const response = await minioClient.send(command);

  // Map to a cleaner format
  const files = (response.Contents || []).map((obj) => ({
    key: obj.Key,
    size: obj.Size,
    lastModified: obj.LastModified,
    eTag: obj.ETag,
    // Generate a URL for each file
    url: `${process.env.MINIO_ENDPOINT}/${BUCKET_NAME}/${obj.Key}`,
  }));

  return files;
}
```

### Generate Presigned URL

```javascript
/**
 * Generate a temporary presigned URL for downloading a private file
 * The URL expires after the specified time and requires no credentials
 * @param {string} objectKey - Object key in MinIO
 * @param {number} expiresInSeconds - URL expiry time (default: 1 hour)
 * @returns {string} Presigned URL
 */
async function getPresignedDownloadUrl(objectKey, expiresInSeconds = 3600) {
  const command = new GetObjectCommand({
    Bucket: BUCKET_NAME,
    Key: objectKey,
  });

  // Generate signed URL that expires
  const url = await getSignedUrl(minioClient, command, {
    expiresIn: expiresInSeconds,
  });

  return url;
}

/**
 * Generate a presigned URL for direct client upload
 * Client uploads directly to MinIO — your backend never touches the file data
 * @param {string} objectKey - Where to store the file in MinIO
 * @param {number} expiresInSeconds - URL expiry time
 * @returns {string} Presigned upload URL
 */
async function getPresignedUploadUrl(objectKey, expiresInSeconds = 300) {
  const { PutObjectCommand } = require("@aws-sdk/client-s3");

  const command = new PutObjectCommand({
    Bucket: BUCKET_NAME,
    Key: objectKey,
  });

  const url = await getSignedUrl(minioClient, command, {
    expiresIn: expiresInSeconds,
  });

  return url;
}

module.exports = {
  uploadFile,
  uploadBuffer,
  getFile,
  streamFile,
  deleteFile,
  listFiles,
  getPresignedDownloadUrl,
  getPresignedUploadUrl,
};
```

---

## 7. File Upload System (Backend)

### Setup: Express + Multer

```bash
npm install express multer uuid dotenv
```

```javascript
// server.js

const express = require("express");
const multer = require("multer");
const { v4: uuidv4 } = require("uuid");
const path = require("path");
const {
  uploadBuffer,
  deleteFile,
  listFiles,
  getPresignedDownloadUrl,
} = require("./services/storage");

const app = express();
app.use(express.json());

// ─── Multer Configuration ─────────────────────────────────────────────────────

// Store files in memory (not disk) so we can pass buffer directly to MinIO
const storage = multer.memoryStorage();

// File filter — only allow images and documents
const fileFilter = (req, file, cb) => {
  const allowedMimeTypes = [
    "image/jpeg",
    "image/png",
    "image/webp",
    "image/gif",
    "application/pdf",
    "video/mp4",
  ];

  if (allowedMimeTypes.includes(file.mimetype)) {
    cb(null, true);  // Accept file
  } else {
    cb(new Error(`File type ${file.mimetype} is not allowed`), false);
  }
};

const upload = multer({
  storage,
  fileFilter,
  limits: {
    fileSize: 50 * 1024 * 1024, // 50 MB limit
  },
});

// ─── Routes ───────────────────────────────────────────────────────────────────

/**
 * POST /upload/image
 * Upload a single image
 */
app.post("/upload/image", upload.single("image"), async (req, res) => {
  try {
    if (!req.file) {
      return res.status(400).json({ error: "No file uploaded" });
    }

    const file = req.file;

    // Generate a unique key using UUID to avoid collisions
    const ext = path.extname(file.originalname); // e.g., ".jpg"
    const objectKey = `images/${uuidv4()}${ext}`; // e.g., "images/uuid.jpg"

    // Upload buffer to MinIO
    const url = await uploadBuffer(file.buffer, objectKey, file.mimetype);

    res.json({
      success: true,
      message: "Image uploaded successfully",
      data: {
        key: objectKey,
        url,
        originalName: file.originalname,
        size: file.size,
        mimeType: file.mimetype,
      },
    });
  } catch (error) {
    console.error("Upload error:", error);
    res.status(500).json({ error: error.message });
  }
});

/**
 * POST /upload/multiple
 * Upload up to 10 files at once
 */
app.post("/upload/multiple", upload.array("files", 10), async (req, res) => {
  try {
    if (!req.files || req.files.length === 0) {
      return res.status(400).json({ error: "No files uploaded" });
    }

    // Upload all files in parallel
    const uploadPromises = req.files.map(async (file) => {
      const ext = path.extname(file.originalname);
      const objectKey = `uploads/${uuidv4()}${ext}`;
      const url = await uploadBuffer(file.buffer, objectKey, file.mimetype);
      return { key: objectKey, url, originalName: file.originalname };
    });

    const uploadedFiles = await Promise.all(uploadPromises);

    res.json({
      success: true,
      message: `${uploadedFiles.length} files uploaded`,
      data: uploadedFiles,
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

/**
 * DELETE /files/:key
 * Delete a file from MinIO
 */
app.delete("/files/:key(*)", async (req, res) => {
  try {
    const objectKey = req.params.key; // e.g., "images/uuid.jpg"
    await deleteFile(objectKey);
    res.json({ success: true, message: "File deleted" });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

/**
 * GET /files
 * List all files (with optional prefix filter)
 */
app.get("/files", async (req, res) => {
  try {
    const prefix = req.query.prefix || ""; // e.g., ?prefix=images/
    const files = await listFiles(prefix);
    res.json({ success: true, count: files.length, data: files });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

/**
 * GET /files/signed/:key
 * Get a temporary presigned download URL
 */
app.get("/files/signed/:key(*)", async (req, res) => {
  try {
    const objectKey = req.params.key;
    const expires = parseInt(req.query.expires) || 3600; // Default: 1 hour
    const url = await getPresignedDownloadUrl(objectKey, expires);
    res.json({ success: true, url, expiresIn: expires });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// ─── Error Handler ────────────────────────────────────────────────────────────

app.use((err, req, res, next) => {
  if (err instanceof multer.MulterError) {
    if (err.code === "LIMIT_FILE_SIZE") {
      return res.status(413).json({ error: "File too large. Maximum size is 50MB." });
    }
    return res.status(400).json({ error: err.message });
  }
  res.status(500).json({ error: err.message });
});

// ─── Start Server ─────────────────────────────────────────────────────────────

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`🚀 Server running on port ${PORT}`);
});
```

### Handling Large Files (Multipart Upload)

For files larger than 100MB, use multipart upload:

```javascript
// services/largeUpload.js

const {
  CreateMultipartUploadCommand,
  UploadPartCommand,
  CompleteMultipartUploadCommand,
  AbortMultipartUploadCommand,
} = require("@aws-sdk/client-s3");
const { minioClient, BUCKET_NAME } = require("../config/minio");

const PART_SIZE = 10 * 1024 * 1024; // 10MB per part (minimum 5MB for S3)

/**
 * Upload a large file using multipart upload
 * @param {Buffer} buffer - Full file buffer
 * @param {string} objectKey - Destination key
 * @param {string} contentType - MIME type
 */
async function uploadLargeFile(buffer, objectKey, contentType) {
  let uploadId;

  try {
    // Step 1: Initiate multipart upload
    const createResponse = await minioClient.send(
      new CreateMultipartUploadCommand({
        Bucket: BUCKET_NAME,
        Key: objectKey,
        ContentType: contentType,
      })
    );
    uploadId = createResponse.UploadId;

    // Step 2: Split buffer into parts and upload each
    const parts = [];
    const totalParts = Math.ceil(buffer.length / PART_SIZE);

    for (let i = 0; i < totalParts; i++) {
      const start = i * PART_SIZE;
      const end = Math.min(start + PART_SIZE, buffer.length);
      const partBuffer = buffer.slice(start, end);
      const partNumber = i + 1;

      const partResponse = await minioClient.send(
        new UploadPartCommand({
          Bucket: BUCKET_NAME,
          Key: objectKey,
          UploadId: uploadId,
          PartNumber: partNumber,
          Body: partBuffer,
        })
      );

      parts.push({ PartNumber: partNumber, ETag: partResponse.ETag });
      console.log(`Uploaded part ${partNumber}/${totalParts}`);
    }

    // Step 3: Complete the multipart upload
    await minioClient.send(
      new CompleteMultipartUploadCommand({
        Bucket: BUCKET_NAME,
        Key: objectKey,
        UploadId: uploadId,
        MultipartUpload: { Parts: parts },
      })
    );

    console.log(`✅ Large file upload complete: ${objectKey}`);
    return `${process.env.MINIO_ENDPOINT}/${BUCKET_NAME}/${objectKey}`;

  } catch (error) {
    // Abort the multipart upload if something fails
    if (uploadId) {
      await minioClient.send(
        new AbortMultipartUploadCommand({
          Bucket: BUCKET_NAME,
          Key: objectKey,
          UploadId: uploadId,
        })
      );
    }
    throw error;
  }
}

module.exports = { uploadLargeFile };
```

---

## 8. Frontend Integration

### Direct Upload via Backend (Recommended for Most Cases)

```jsx
// React component — uploads via your Express backend

import React, { useState } from "react";
import axios from "axios";

export default function FileUploader() {
  const [file, setFile] = useState(null);
  const [preview, setPreview] = useState(null);
  const [uploading, setUploading] = useState(false);
  const [result, setResult] = useState(null);
  const [error, setError] = useState(null);

  const handleFileSelect = (e) => {
    const selectedFile = e.target.files[0];
    setFile(selectedFile);

    // Show image preview
    if (selectedFile && selectedFile.type.startsWith("image/")) {
      const reader = new FileReader();
      reader.onload = (ev) => setPreview(ev.target.result);
      reader.readAsDataURL(selectedFile);
    }
  };

  const handleUpload = async () => {
    if (!file) return;

    const formData = new FormData();
    formData.append("image", file);

    setUploading(true);
    setError(null);

    try {
      const response = await axios.post(
        "http://localhost:3000/upload/image",
        formData,
        {
          headers: { "Content-Type": "multipart/form-data" },
          // Track upload progress
          onUploadProgress: (progressEvent) => {
            const percent = Math.round(
              (progressEvent.loaded * 100) / progressEvent.total
            );
            console.log(`Upload progress: ${percent}%`);
          },
        }
      );

      setResult(response.data.data);
    } catch (err) {
      setError(err.response?.data?.error || "Upload failed");
    } finally {
      setUploading(false);
    }
  };

  return (
    <div style={{ padding: 20 }}>
      <h2>Upload Image to MinIO</h2>

      <input
        type="file"
        accept="image/*"
        onChange={handleFileSelect}
      />

      {preview && (
        <img
          src={preview}
          alt="Preview"
          style={{ width: 200, marginTop: 10, display: "block" }}
        />
      )}

      <button
        onClick={handleUpload}
        disabled={!file || uploading}
        style={{ marginTop: 10 }}
      >
        {uploading ? "Uploading..." : "Upload"}
      </button>

      {result && (
        <div style={{ marginTop: 10, color: "green" }}>
          <p>✅ Uploaded!</p>
          <p>URL: <a href={result.url}>{result.url}</a></p>
          <p>Key: {result.key}</p>
        </div>
      )}

      {error && <p style={{ color: "red" }}>❌ {error}</p>}
    </div>
  );
}
```

### Direct Upload to MinIO (Presigned URL — Advanced)

This approach lets the client upload directly to MinIO, bypassing your backend server. Better for large files and reduces server load.

```jsx
// React component — direct upload to MinIO using presigned URL

import React, { useState } from "react";
import axios from "axios";

export default function DirectUploader() {
  const [file, setFile] = useState(null);
  const [uploading, setUploading] = useState(false);
  const [progress, setProgress] = useState(0);
  const [result, setResult] = useState(null);

  const handleUpload = async () => {
    if (!file) return;
    setUploading(true);

    try {
      // Step 1: Ask your backend for a presigned upload URL
      const { data } = await axios.post("/api/presigned-upload", {
        filename: file.name,
        contentType: file.type,
      });

      const { uploadUrl, objectKey } = data;

      // Step 2: Upload directly to MinIO using the presigned URL
      // Note: PUT request, not POST, and no authentication headers needed
      await axios.put(uploadUrl, file, {
        headers: {
          "Content-Type": file.type,
        },
        onUploadProgress: (e) => {
          setProgress(Math.round((e.loaded * 100) / e.total));
        },
      });

      // Step 3: Notify your backend that upload is complete (optional)
      await axios.post("/api/upload-complete", { objectKey });

      setResult({ objectKey });
    } catch (err) {
      console.error(err);
    } finally {
      setUploading(false);
    }
  };

  return (
    <div>
      <input type="file" onChange={(e) => setFile(e.target.files[0])} />
      <button onClick={handleUpload} disabled={uploading}>
        {uploading ? `Uploading... ${progress}%` : "Upload Direct"}
      </button>
      {result && <p>Done! Key: {result.objectKey}</p>}
    </div>
  );
}
```

**Backend route for presigned URL generation:**

```javascript
// Express route to generate presigned upload URL
app.post("/api/presigned-upload", async (req, res) => {
  const { filename, contentType } = req.body;
  const ext = path.extname(filename);
  const objectKey = `uploads/${uuidv4()}${ext}`;

  const url = await getPresignedUploadUrl(objectKey, 300); // 5 min expiry
  res.json({ uploadUrl: url, objectKey });
});
```

---

## 9. Security Best Practices

### 1. Never Use Default Credentials

```bash
# IMMEDIATELY change defaults after install
# Edit /etc/default/minio and set strong credentials:
MINIO_ROOT_USER=your_custom_admin_username
MINIO_ROOT_PASSWORD=Tr0ub4dor&3_veryLong$ecurePassw0rd
```

### 2. Keep Buckets Private by Default

```bash
# Default policy is private — never use this in production:
mc anonymous set public myminio/production-bucket  # ❌ BAD

# Instead, keep private and use presigned URLs:
mc anonymous set none myminio/production-bucket    # ✅ GOOD
```

### 3. Use Presigned URLs for All File Access

```javascript
// BAD — exposing MinIO port 9000 directly
const url = "http://localhost:9000/private-bucket/file.pdf";

// GOOD — generate a time-limited URL
const url = await getPresignedDownloadUrl("private-bucket/file.pdf", 3600);
// URL expires in 1 hour and only works for this specific file
```

### 4. Separate Users with Minimal Permissions

Create one user per service/application with only the permissions it needs:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject"],
      "Resource": ["arn:aws:s3:::uploads/*"]
    }
  ]
}
```

```bash
# Create user for your Node.js app
mc admin user add myminio nodeapp-user RandomSecretPass123!

# Create and attach minimal policy
mc admin policy create myminio nodeapp-policy ./nodeapp-policy.json
mc admin policy attach myminio nodeapp-policy --user nodeapp-user
```

### 5. HTTPS Setup with NGINX + SSL

```nginx
# /etc/nginx/sites-available/minio

# MinIO API — s3.yourdomain.com
server {
    listen 443 ssl;
    server_name s3.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/s3.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/s3.yourdomain.com/privkey.pem;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000" always;
    add_header X-Content-Type-Options nosniff;

    # Allow large uploads
    client_max_body_size 500M;

    location / {
        proxy_pass http://localhost:9000;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Required for presigned URLs
        proxy_set_header X-Forwarded-Host $http_host;

        # Long timeouts for large uploads
        proxy_connect_timeout 300;
        proxy_send_timeout 300;
        proxy_read_timeout 300;
        send_timeout 300;
    }
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name s3.yourdomain.com;
    return 301 https://$host$request_uri;
}
```

### 6. Environment Variables — Never Hardcode

```javascript
// ❌ BAD — credentials in code
const client = new S3Client({
  credentials: {
    accessKeyId: "minioadmin",
    secretAccessKey: "password123",
  },
});

// ✅ GOOD — from environment
const client = new S3Client({
  credentials: {
    accessKeyId: process.env.MINIO_ACCESS_KEY,
    secretAccessKey: process.env.MINIO_SECRET_KEY,
  },
});
```

```bash
# Always add .env to .gitignore
echo ".env" >> .gitignore
```

### 7. Validate File Types Before Upload

```javascript
// Validate by MIME type AND file extension (double check)
function validateFile(file) {
  const allowedMimes = new Set([
    "image/jpeg", "image/png", "image/webp",
    "application/pdf",
  ]);
  const allowedExtensions = new Set([".jpg", ".jpeg", ".png", ".webp", ".pdf"]);
  const ext = path.extname(file.originalname).toLowerCase();

  if (!allowedMimes.has(file.mimetype)) {
    throw new Error(`MIME type ${file.mimetype} not allowed`);
  }
  if (!allowedExtensions.has(ext)) {
    throw new Error(`Extension ${ext} not allowed`);
  }
}
```

---

## 10. Production Deployment

### Architecture Overview

```
Internet
    │
    ▼
[Cloudflare / DNS]
    │
    ▼
[NGINX Reverse Proxy]  ← Port 80/443 (HTTPS)
    │               │
    ▼               ▼
[Node.js App]   [MinIO Console]
  Port 3000       Port 9001 (internal only)
    │
    ▼
[MinIO Server]  ← Port 9000 (internal only)
    │
    ▼
[/mnt/minio-data]  ← Persistent disk storage
```

### NGINX Full Configuration

```nginx
# /etc/nginx/sites-available/shellelearning

# MinIO API
server {
    listen 443 ssl http2;
    server_name s3.shellelearningacademy.com;

    ssl_certificate /etc/letsencrypt/live/s3.shellelearningacademy.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/s3.shellelearningacademy.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    # Support large file uploads (500MB)
    client_max_body_size 500M;

    # Disable proxy buffering for streaming
    proxy_buffering off;

    location / {
        proxy_pass http://127.0.0.1:9000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        proxy_connect_timeout 300;
        proxy_send_timeout 300;
        proxy_read_timeout 300;
    }
}

# MinIO Console (restrict access to admin IPs in production)
server {
    listen 443 ssl http2;
    server_name console.shellelearningacademy.com;

    ssl_certificate /etc/letsencrypt/live/console.shellelearningacademy.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/console.shellelearningacademy.com/privkey.pem;

    # Optional: Restrict console to your IP only
    # allow 103.x.x.x;  # Your office IP
    # deny all;

    location / {
        proxy_pass http://127.0.0.1:9001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

# HTTP → HTTPS redirect
server {
    listen 80;
    server_name s3.shellelearningacademy.com console.shellelearningacademy.com;
    return 301 https://$host$request_uri;
}
```

### Enable HTTPS with Let's Encrypt

```bash
# Install Certbot
sudo apt update
sudo apt install certbot python3-certbot-nginx -y

# Obtain certificate (auto-configures NGINX)
sudo certbot --nginx -d s3.yourdomain.com -d console.yourdomain.com

# Auto-renewal is set up by Certbot automatically
# Verify with:
sudo systemctl status certbot.timer

# Test renewal
sudo certbot renew --dry-run
```

### Update MinIO to Use Domain

```bash
# Edit /etc/default/minio
sudo nano /etc/default/minio
```

Add:
```bash
MINIO_SERVER_URL=https://s3.yourdomain.com
MINIO_BROWSER_REDIRECT_URL=https://console.yourdomain.com
```

```bash
# Restart MinIO
sudo systemctl restart minio
```

### Firewall Setup (UFW)

```bash
# Only allow ports 80 and 443 publicly
# MinIO ports (9000, 9001) should be internal only

sudo ufw default deny incoming
sudo ufw default allow outgoing

# SSH (IMPORTANT — don't lock yourself out)
sudo ufw allow 22/tcp

# Web traffic (NGINX)
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Block direct MinIO access from internet
# (9000 and 9001 should NOT be in ufw allow list)

sudo ufw enable
sudo ufw status verbose
```

### Backup Strategy

```bash
#!/bin/bash
# /opt/scripts/backup-minio.sh
# Run this via cron every day

BACKUP_DIR="/backup/minio-$(date +%Y-%m-%d)"
MINIO_DATA="/mnt/minio-data"

# Create backup directory
mkdir -p $BACKUP_DIR

# Option 1: Mirror to remote bucket using mc
mc mirror myminio/ myminio-backup/

# Option 2: rsync to another disk
rsync -avz --progress $MINIO_DATA/ $BACKUP_DIR/

# Option 3: Compress and archive
tar -czf /backup/minio-$(date +%Y-%m-%d).tar.gz $MINIO_DATA

# Keep only last 7 days of backups
find /backup -name "minio-*.tar.gz" -mtime +7 -delete

echo "Backup complete: $BACKUP_DIR"
```

```bash
# Add to cron (run daily at 2 AM)
crontab -e
# Add:
0 2 * * * /opt/scripts/backup-minio.sh >> /var/log/minio-backup.log 2>&1
```

---

## 11. Scaling & Advanced Topics

### Distributed MinIO Setup

For production workloads requiring high availability, MinIO supports distributed mode across multiple servers/disks.

**Minimum: 4 nodes, 4 disks each (16 drives total)**

```bash
# On each node, set the same environment:
# /etc/default/minio on all nodes

MINIO_ROOT_USER=admin
MINIO_ROOT_PASSWORD=StrongPass123!

# List all nodes and their disk paths
MINIO_VOLUMES="http://minio-node-{1...4}:9000/mnt/disk{1...4}"
MINIO_OPTS="--console-address :9001"
```

```bash
# Start MinIO on each node (they find each other automatically)
sudo systemctl start minio
```

MinIO uses **erasure coding** — data is spread across drives with parity. If up to N/2 drives fail, your data is still safe and readable.

```
Node 1          Node 2          Node 3          Node 4
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│ disk 1   │   │ disk 1   │   │ disk 1   │   │ disk 1   │
│ disk 2   │   │ disk 2   │   │ disk 2   │   │ disk 2   │
│ disk 3   │   │ disk 3   │   │ disk 3   │   │ disk 3   │
│ disk 4   │   │ disk 4   │   │ disk 4   │   │ disk 4   │
└──────────┘   └──────────┘   └──────────┘   └──────────┘
```

### Load Balancing with NGINX

```nginx
# NGINX upstream for MinIO cluster
upstream minio_backend {
    least_conn;  # Route to least busy server
    server minio-node-1:9000;
    server minio-node-2:9000;
    server minio-node-3:9000;
    server minio-node-4:9000;
}

server {
    listen 443 ssl;
    server_name s3.yourdomain.com;

    location / {
        proxy_pass http://minio_backend;
        proxy_set_header Host $http_host;
    }
}
```

### Object Lifecycle Management

Auto-delete old files, or transition to different storage classes:

```bash
# Set lifecycle rule via mc CLI
# Delete objects older than 30 days in "temp-uploads" bucket
mc ilm rule add myminio/temp-uploads \
  --expire-days 30

# Expire objects with specific prefix after 7 days
mc ilm rule add myminio/my-bucket \
  --prefix "temp/" \
  --expire-days 7

# View lifecycle rules
mc ilm rule ls myminio/my-bucket
```

Programmatically via SDK:

```javascript
const { PutBucketLifecycleConfigurationCommand } = require("@aws-sdk/client-s3");

await minioClient.send(new PutBucketLifecycleConfigurationCommand({
  Bucket: "temp-uploads",
  LifecycleConfiguration: {
    Rules: [
      {
        ID: "expire-temp-30days",
        Status: "Enabled",
        Filter: { Prefix: "temp/" },
        Expiration: { Days: 30 },
      },
    ],
  },
}));
```

### Versioning

Keep all versions of an object — useful for document management and undo features.

```bash
# Enable versioning on a bucket
mc version enable myminio/documents

# List all versions of an object
mc ls --versions myminio/documents/report.pdf

# Restore a specific version
mc cp --version-id "version-id-here" \
  myminio/documents/report.pdf \
  myminio/documents/report-restored.pdf
```

---

## 12. Comparison with AWS S3

### Feature Comparison

| Feature | AWS S3 | MinIO |
|---|---|---|
| API Compatibility | Native | 100% S3-compatible |
| Cost | $0.023/GB storage + requests | Server cost only |
| Data Sovereignty | AWS datacenters | Your server |
| Latency | Regional (50–200ms) | Same datacenter (<10ms) |
| Setup Complexity | Minimal | 30 min |
| Maintenance | Zero (managed) | You manage updates |
| Scalability | Unlimited | Limited by your hardware |
| SLA / Uptime | 99.999999999% (11 nines) | Depends on your setup |
| CDN Integration | CloudFront native | Manual NGINX/Cloudflare |
| Compliance | SOC2, HIPAA, etc. | Self-managed |
| Bandwidth | Egress costs $0.09/GB | Usually unlimited on VPS |

### When to Use MinIO

**Choose MinIO when:**
- You have cost-sensitive workloads (lots of data, high bandwidth)
- Data residency/privacy requirements (e.g., GDPR, medical data)
- Development/staging environments
- Internal tools and corporate intranet
- Learning and building your own infrastructure
- Existing VPS with spare disk space
- ShellElearning Academy — your own platform, full control

**Choose AWS S3 when:**
- You need infinite scale without effort
- Global CDN via CloudFront is critical
- Zero infrastructure management
- Compliance certifications (SOC2, HIPAA, FedRAMP)
- Multi-region replication is needed out-of-the-box
- Your team is small and DevOps resources are limited

### Cost Example

```
Scenario: 1TB stored, 500GB download/month

AWS S3:
  Storage:   1000 GB × $0.023 = $23/month
  Egress:     500 GB × $0.09  = $45/month
  Requests:  ~1M requests      = $5/month
  TOTAL: ~$73/month

MinIO on VPS (Hetzner/DigitalOcean):
  8 vCPU, 16GB RAM, 2TB disk VPS = ~$20–40/month
  Bandwidth: Usually included
  TOTAL: ~$20–40/month
  
  SAVINGS: $33–53/month (~$400–630/year)
```

---

## 13. Troubleshooting

### Common Errors and Fixes

**1. Connection Refused**
```bash
# Symptom:
# Error: connect ECONNREFUSED 127.0.0.1:9000

# Check if MinIO is running:
sudo systemctl status minio

# Check what's on port 9000:
sudo lsof -i :9000

# Check logs:
sudo journalctl -u minio -n 50
```

**2. Signature Mismatch (403 Error)**
```bash
# Symptom:
# SignatureDoesNotMatch: The request signature we calculated does not match...

# Cause: Wrong credentials OR forcePathStyle not set

# Fix 1: Check credentials in .env
# Fix 2: Add forcePathStyle: true to S3Client config
const client = new S3Client({
  forcePathStyle: true,  // MUST be true for MinIO
  // ...
});
```

**3. NoSuchBucket (404)**
```bash
# Symptom: The specified bucket does not exist

# Create the bucket:
mc mb myminio/your-bucket-name

# Or verify bucket name (case-sensitive, no uppercase):
mc ls myminio
```

**4. Access Denied (403)**
```bash
# Symptom: AccessDenied

# Check if bucket is private and you're using correct credentials
mc anonymous get myminio/bucket-name

# Verify your policy allows the action
mc admin policy list myminio
mc admin user info myminio yourusername
```

**5. Large File Upload Failing**
```bash
# Symptom: Upload hangs or times out

# Check NGINX client_max_body_size:
sudo nano /etc/nginx/sites-available/minio
# Ensure: client_max_body_size 500M;

# Check NGINX timeouts:
proxy_connect_timeout 300;
proxy_send_timeout 300;
proxy_read_timeout 300;

# Reload NGINX
sudo nginx -t && sudo systemctl reload nginx
```

**6. MinIO Won't Start After Config Change**
```bash
# Check syntax of environment file:
sudo bash -n /etc/default/minio

# Check for permission issues on data dir:
ls -la /mnt/minio-data
sudo chown -R minio-user:minio-user /mnt/minio-data

# View detailed startup logs:
sudo journalctl -u minio -n 100 --no-pager
```

**7. Presigned URL Not Working**
```bash
# Cause: MINIO_SERVER_URL not set (URLs contain localhost instead of domain)

# Fix: Add to /etc/default/minio:
MINIO_SERVER_URL=https://s3.yourdomain.com

# Restart:
sudo systemctl restart minio
```

### Debug Commands Reference

```bash
# Health check
curl -I http://localhost:9000/minio/health/live

# Check disk usage
mc du myminio/

# View server info
mc admin info myminio

# View live logs
mc admin logs myminio

# Trace all API requests (debug mode)
mc admin trace myminio

# Check system logs
sudo journalctl -u minio -f

# Test credentials
mc alias list
mc ls myminio/

# Check network ports
sudo ss -tlnp | grep -E '9000|9001'
```

---

## 14. Real-World Architecture

### ShellElearning Academy — Complete System

```
                              INTERNET
                                  │
                    ┌─────────────▼────────────┐
                    │      Cloudflare DNS       │
                    │  shellelearningacademy.com│
                    └─────────────┬────────────┘
                                  │ HTTPS
                    ┌─────────────▼────────────┐
                    │        VPS Server         │
                    │   (Ubuntu 22.04 LTS)      │
                    │                           │
                    │  ┌─────────────────────┐  │
                    │  │   NGINX (Port 443)   │  │
                    │  │  Reverse Proxy + SSL │  │
                    │  └──────┬──────┬───────┘  │
                    │         │      │           │
                    │         ▼      ▼           │
                    │  ┌──────────┐ ┌─────────┐  │
                    │  │Node.js   │ │ MinIO   │  │
                    │  │Express   │ │Console  │  │
                    │  │Port 3000 │ │Port 9001│  │
                    │  └────┬─────┘ └─────────┘  │
                    │       │                    │
                    │       ▼                    │
                    │  ┌──────────┐              │
                    │  │  MinIO   │              │
                    │  │  Server  │              │
                    │  │Port 9000 │              │
                    │  └────┬─────┘              │
                    │       │                    │
                    │       ▼                    │
                    │  ┌──────────────────────┐  │
                    │  │   /mnt/minio-data     │  │
                    │  │  ┌────────────────┐  │  │
                    │  │  │Bucket: images  │  │  │
                    │  │  │Bucket: videos  │  │  │
                    │  │  │Bucket: docs    │  │  │
                    │  │  └────────────────┘  │  │
                    │  └──────────────────────┘  │
                    │                           │
                    │  ┌──────────────────────┐  │
                    │  │    MongoDB            │  │
                    │  │    Port 27017         │  │
                    │  └──────────────────────┘  │
                    └───────────────────────────┘
```

### Request Flow — File Upload

```
User Browser
    │
    │ POST /api/courses/upload-banner
    │ (multipart/form-data with image)
    ▼
NGINX (Port 443)
    │
    │ Proxied to Node.js
    ▼
Express Server (Port 3000)
    │
    │ Multer parses the file → Buffer in memory
    │ Validates MIME type, size
    │ Generates UUID key: "course-banners/uuid.jpg"
    ▼
MinIO SDK (uploadBuffer)
    │
    │ PutObjectCommand → localhost:9000
    ▼
MinIO Server
    │
    │ Writes to /mnt/minio-data
    ▼
Response → { key, url } → Express → NGINX → Browser
    │
    │ Store URL in MongoDB course document
    ▼
✅ Done — Image accessible via https://s3.shellelearningacademy.com/course-banners/uuid.jpg
```

### Request Flow — Serving a Private File

```
User requests a private certificate PDF
    │
    ▼
GET /api/certificates/download/cert-123
    │
    ▼
Express checks auth (JWT) → user owns cert-123 ✓
    │
    ▼
getPresignedDownloadUrl("certificates/cert-123.pdf", 300)
    │
    ▼
MinIO generates signed URL (expires in 5 min)
    │
    ▼
Express returns: { url: "https://s3.domain.com/...?X-Amz-Signature=..." }
    │
    ▼
Browser redirects to presigned URL
    │
    ▼
MinIO validates signature → serves PDF directly
    │
    ▼
URL expires in 5 minutes — cannot be reused or shared
```

---

## 15. Bonus — Docker Setup

### Single Container (Development)

```yaml
# docker-compose.yml

version: "3.8"

services:
  minio:
    image: minio/minio:latest
    container_name: minio
    command: server /data --console-address ":9001"
    ports:
      - "9000:9000"    # API
      - "9001:9001"    # Console
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin123
    volumes:
      - minio_data:/data  # Persist data in Docker volume
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:
  minio_data:
    driver: local
```

```bash
# Start MinIO with Docker
docker-compose up -d

# View logs
docker-compose logs -f minio

# Stop
docker-compose down
```

### Full Stack: MinIO + Node.js + MongoDB

```yaml
# docker-compose.yml (full stack)

version: "3.8"

services:
  # ── MongoDB ──────────────────────────────────────────
  mongodb:
    image: mongo:7
    container_name: mongodb
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: mongopassword
    volumes:
      - mongodb_data:/data/db
    restart: unless-stopped

  # ── MinIO ─────────────────────────────────────────────
  minio:
    image: minio/minio:latest
    container_name: minio
    command: server /data --console-address ":9001"
    ports:
      - "9000:9000"
      - "9001:9001"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: StrongPassword@2024!
    volumes:
      - minio_data:/data
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 10s
      retries: 3

  # ── MinIO Bucket Init (one-time setup) ────────────────
  minio-init:
    image: minio/mc:latest
    container_name: minio-init
    depends_on:
      minio:
        condition: service_healthy
    # Create bucket on first start
    entrypoint: >
      /bin/sh -c "
        mc alias set myminio http://minio:9000 minioadmin StrongPassword@2024!;
        mc mb --ignore-existing myminio/shellelearning-images;
        mc mb --ignore-existing myminio/shellelearning-videos;
        mc mb --ignore-existing myminio/shellelearning-docs;
        echo 'Buckets created!';
        exit 0;
      "

  # ── Node.js App ───────────────────────────────────────
  api:
    build: .
    container_name: nodeapi
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: development
      MONGODB_URI: mongodb://admin:mongopassword@mongodb:27017/ShellElearningDB?authSource=admin
      MINIO_ENDPOINT: http://minio:9000
      MINIO_ACCESS_KEY: minioadmin
      MINIO_SECRET_KEY: StrongPassword@2024!
      MINIO_BUCKET: shellelearning-images
    depends_on:
      - mongodb
      - minio
    volumes:
      - .:/app
      - /app/node_modules  # Don't override node_modules
    restart: unless-stopped

volumes:
  mongodb_data:
  minio_data:
```

```dockerfile
# Dockerfile (for Node.js app)

FROM node:20-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```

```bash
# Start the full stack
docker-compose up -d

# View all containers
docker-compose ps

# View logs from all services
docker-compose logs -f

# Only MinIO logs
docker-compose logs -f minio

# Rebuild and restart app after code changes
docker-compose up -d --build api

# Stop everything
docker-compose down

# Stop and remove all data (fresh start)
docker-compose down -v
```

### Initialize Buckets on Startup (Script)

```bash
#!/bin/bash
# scripts/init-minio.sh
# Run after MinIO starts to create required buckets

MINIO_ENDPOINT="http://localhost:9000"
MINIO_ACCESS_KEY="minioadmin"
MINIO_SECRET_KEY="StrongPassword@2024!"

# Wait for MinIO to be ready
until curl -sf $MINIO_ENDPOINT/minio/health/live; do
  echo "Waiting for MinIO..."
  sleep 2
done

echo "MinIO is ready, initializing buckets..."

mc alias set myminio $MINIO_ENDPOINT $MINIO_ACCESS_KEY $MINIO_SECRET_KEY

# Create buckets
mc mb --ignore-existing myminio/shellelearning-images
mc mb --ignore-existing myminio/shellelearning-videos
mc mb --ignore-existing myminio/shellelearning-docs
mc mb --ignore-existing myminio/shellelearning-backups

# Set public read for images bucket
mc anonymous set download myminio/shellelearning-images

echo "✅ MinIO initialized successfully!"
```

---

## Quick Reference Cheatsheet

```
┌─────────────────────────────────────────────────────────────────┐
│                    MinIO Quick Reference                         │
├─────────────────────────┬───────────────────────────────────────┤
│ START SERVICE           │ sudo systemctl start minio             │
│ STOP SERVICE            │ sudo systemctl stop minio              │
│ STATUS                  │ sudo systemctl status minio            │
│ LOGS                    │ sudo journalctl -u minio -f            │
├─────────────────────────┼───────────────────────────────────────┤
│ ADD ALIAS               │ mc alias set local http://IP:9000 u p  │
│ LIST BUCKETS            │ mc ls local                            │
│ CREATE BUCKET           │ mc mb local/bucket-name                │
│ UPLOAD FILE             │ mc cp ./file.jpg local/bucket/key.jpg  │
│ DOWNLOAD FILE           │ mc cp local/bucket/key.jpg ./file.jpg  │
│ DELETE FILE             │ mc rm local/bucket/key.jpg             │
│ LIST FILES              │ mc ls local/bucket/                    │
│ SYNC FOLDER             │ mc mirror ./folder/ local/bucket/      │
│ PUBLIC ACCESS           │ mc anonymous set download local/bucket │
│ PRIVATE ACCESS          │ mc anonymous set none local/bucket     │
│ PRESIGNED URL           │ mc share download --expire 24h local/… │
├─────────────────────────┼───────────────────────────────────────┤
│ WEB CONSOLE             │ http://YOUR_IP:9001                    │
│ API ENDPOINT            │ http://YOUR_IP:9000                    │
│ HEALTH CHECK            │ curl http://IP:9000/minio/health/live  │
└─────────────────────────┴───────────────────────────────────────┘
```

---

## Summary — What You Can Now Do

By completing this guide, you can:

- **Install and run** a production-grade MinIO server on any Ubuntu VPS
- **Manage storage** with the web console and `mc` CLI
- **Integrate MinIO** with Node.js/Express using the AWS S3 SDK
- **Handle file uploads** — single files, multiple files, large files
- **Secure your system** with private buckets, presigned URLs, and HTTPS
- **Deploy to production** with NGINX reverse proxy and Let's Encrypt SSL
- **Scale** from single-node to a distributed cluster
- **Containerize** everything with Docker and docker-compose
- **Choose** intelligently between MinIO and AWS S3 for your use case

---

*Guide version: 2024 · MinIO RELEASE.2024+ · Node.js 20 · AWS SDK v3*
