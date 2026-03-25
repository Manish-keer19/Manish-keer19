# How to Connect a Domain to a VPS Hosted Application (Different IP)
## Beginner to Advanced Guide

> **Who is this guide for?**
> Whether you've just deployed your first Node.js app on a VPS and are staring at a raw IP address, or you're an experienced developer looking to set up multi-domain Nginx configurations with SSL — this guide covers it all, end to end.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [How DNS Works](#2-how-dns-works)
3. [Prerequisites](#3-prerequisites)
4. [Point Your Domain to VPS IP](#4-point-your-domain-to-vps-ip)
5. [Verify Domain Connection](#5-verify-domain-connection)
6. [Configure Backend Server](#6-configure-backend-server)
7. [Setup Nginx Reverse Proxy](#7-setup-nginx-reverse-proxy)
8. [Connect Frontend with Domain](#8-connect-frontend-with-domain)
9. [Enable HTTPS (SSL)](#9-enable-https-ssl)
10. [Multiple Projects on Same VPS (Advanced)](#10-multiple-projects-on-same-vps-advanced)
11. [Using Different VPS for Frontend & Backend](#11-using-different-vps-for-frontend--backend)
12. [Security Best Practices](#12-security-best-practices)
13. [Common Errors & Fixes](#13-common-errors--fixes)
14. [Debugging Tools](#14-debugging-tools)
15. [Advanced Topics](#15-advanced-topics)
16. [Architecture Diagram](#16-architecture-diagram)
17. [Conclusion](#17-conclusion)

---

## 1. Introduction

### What is a Domain Name?

A **domain name** is a human-readable address used to identify a website or service on the internet — for example, `shellelearningacademy.com` or `google.com`. Without domain names, users would need to remember raw IP addresses like `192.168.1.105` to reach a website. Domain names are registered through **registrars** (companies like GoDaddy, Namecheap, Google Domains) and are managed via the **Domain Name System (DNS)**.

### What is a VPS (Virtual Private Server)?

A **VPS** is a virtualized server hosted in a data center, giving you dedicated (or near-dedicated) compute resources — CPU, RAM, and storage — at a fraction of the cost of a physical server. Popular VPS providers include:

| Provider | Starting Price | Specialty |
|---|---|---|
| DigitalOcean (Droplets) | ~$6/month | Developer-friendly |
| AWS EC2 | Pay-as-you-go | Enterprise scale |
| Vultr | ~$2.50/month | Budget-friendly |
| Linode (Akamai) | ~$5/month | Reliable uptime |
| Hetzner | ~$4/month | European, very affordable |

Your VPS has a **public IP address** (e.g., `142.93.50.18`) assigned to it. This is the address the internet uses to reach your server.

### How Domains Connect to Servers via IP Addresses

The connection between a domain name and a server IP is managed through **DNS records**. You essentially tell the DNS system: *"When someone looks up `shellelearningacademy.com`, send them to IP `142.93.50.18`."*

This mapping is done by adding an **A Record** in your domain provider's DNS panel.

### Real-World Architecture

Here is how a user's request flows from their browser to your running application:

```
User types: https://shellelearningacademy.com
      |
      v
[ Browser ] ──────────────────────────────────────────────────────────────────
      |
      | (1) DNS Lookup: "What is the IP for shellelearningacademy.com?"
      v
[ DNS Resolver ]  ─── checks nameservers ──>  [ Authoritative DNS Server ]
      |                                              |
      |  (2) Returns: "142.93.50.18"                |
      <──────────────────────────────────────────────
      |
      | (3) HTTP/HTTPS request to 142.93.50.18
      v
[ VPS Server: 142.93.50.18 ]
      |
      | (4) Nginx listens on port 80/443
      v
[ Nginx Reverse Proxy ]
      |
      | (5) Proxy pass to localhost:3000
      v
[ Your App: Node.js / Express / React SSR ]
      |
      | (6) Response returned back up the chain
      v
[ Browser renders the page ]
```

---

## 2. How DNS Works

### What is DNS?

**DNS (Domain Name System)** is the internet's phonebook. It translates human-readable domain names into machine-readable IP addresses. DNS is distributed — there is no single central server; instead, thousands of DNS servers around the world work together.

### DNS Record Types

#### A Record (Address Record)
Maps a **domain or subdomain directly to an IPv4 address**.

```
Type:  A
Host:  @          (represents the root domain, e.g., example.com)
Value: 142.93.50.18
TTL:   3600
```

| Host Value | Resolves |
|---|---|
| `@` | `example.com` |
| `www` | `www.example.com` |
| `api` | `api.example.com` |
| `*` | any subdomain (wildcard) |

#### AAAA Record
Same as A Record but for **IPv6 addresses**.

```
Type:  AAAA
Host:  @
Value: 2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

#### CNAME Record (Canonical Name)
Maps a **subdomain to another domain name** (not directly to an IP). Useful for pointing `www` to the root domain.

```
Type:  CNAME
Host:  www
Value: shellelearningacademy.com    ← points to the A record
TTL:   3600
```

> ⚠️ **Important:** A CNAME record cannot coexist with other records on the same host. Do NOT set a CNAME on `@` (root domain). Use an A record for `@` and CNAME for `www`.

#### MX Record
Directs email to your mail server. Not covered in depth here, but important — don't delete these if they exist.

#### TXT Record
Stores text data — commonly used for domain verification (Google Search Console, SSL providers) and SPF/DKIM records for email.

#### NS Record (Nameserver Record)
Specifies **which DNS servers are authoritative** for your domain. Changing nameservers moves DNS control to another provider (e.g., Cloudflare).

```
Type:  NS
Value: ns1.digitalocean.com
Value: ns2.digitalocean.com
Value: ns3.digitalocean.com
```

### TTL (Time to Live)

**TTL** is a value (in seconds) that tells DNS resolvers how long to **cache** a DNS record before checking for updates.

| TTL Value | Duration | Use Case |
|---|---|---|
| 300 | 5 minutes | Testing / frequent changes |
| 3600 | 1 hour | General use |
| 86400 | 24 hours | Stable production domains |

> 💡 **Tip:** Before making DNS changes, lower your TTL to `300` (5 minutes). After changes are confirmed working, raise it back to `3600` or higher to reduce DNS lookup latency.

### DNS Propagation

When you update a DNS record, the change doesn't take effect instantly worldwide. DNS resolvers around the world cache the old record until its TTL expires. This process is called **DNS propagation**.

- **Typical time:** 15 minutes to 48 hours
- **Average:** Most locations update within 1–4 hours
- **Check propagation:** Use [https://www.whatsmydns.net](https://www.whatsmydns.net) to see propagation status from different countries

---

## 3. Prerequisites

Before you begin, make sure you have all of the following:

### A Registered Domain Name

You need a domain purchased from a registrar. Popular options:

| Registrar | Website | Notes |
|---|---|---|
| Namecheap | namecheap.com | Cheap, good UI, free WhoisGuard |
| GoDaddy | godaddy.com | Popular, slightly pricier |
| Google Domains / Squarespace | domains.google | Clean interface |
| Cloudflare Registrar | cloudflare.com | At-cost pricing, no markup |
| Porkbun | porkbun.com | Budget-friendly, good UI |

### A VPS with a Public IP Address

Your VPS must have a static, public IPv4 address. Confirm this by logging in:

```bash
curl ifconfig.me
# or
hostname -I
```

Note down your VPS IP — you'll need it for DNS setup.

### Your Application Running on the VPS

Your backend or frontend app should already be running. Examples:

```bash
# Node.js / Express backend running on port 3000
node server.js
# or
pm2 start server.js

# React frontend served via a static build
npm run build
# (served via Nginx or serve package)
```

### Basic Linux CLI Knowledge

You should be comfortable with:

```bash
ssh user@your-vps-ip      # Connect to VPS
ls, cd, cat, nano/vim     # File navigation
sudo                       # Run commands as root
systemctl                  # Manage services
```

---

## 4. Point Your Domain to VPS IP

This section walks you through setting up DNS records at your domain registrar's control panel.

### Step 1 — Log In to Your Domain Registrar

Go to your registrar's website and navigate to **DNS Management** or **DNS Settings** for your domain.

- **Namecheap:** Dashboard → Domain List → Manage → Advanced DNS
- **GoDaddy:** My Domains → DNS → Manage Zones
- **Cloudflare:** Select domain → DNS → Records

### Step 2 — Add an A Record for Root Domain

| Field | Value |
|---|---|
| Type | A |
| Host / Name | `@` |
| Value / Points To | `YOUR_VPS_IP` (e.g., `142.93.50.18`) |
| TTL | 3600 (or Auto) |

Click **Save / Add Record**.

### Step 3 — Add an A Record for `www` Subdomain

**Option A — Point `www` directly to VPS IP (Recommended for simplicity):**

| Field | Value |
|---|---|
| Type | A |
| Host / Name | `www` |
| Value / Points To | `YOUR_VPS_IP` |
| TTL | 3600 |

**Option B — Point `www` using CNAME to root domain:**

| Field | Value |
|---|---|
| Type | CNAME |
| Host / Name | `www` |
| Value / Points To | `shellelearningacademy.com` (your root domain) |
| TTL | 3600 |

> 💡 Option A is simpler and avoids an extra DNS lookup hop. Use Option B if your root domain IP might change and you want `www` to follow automatically.

### Step 4 — Add Optional Subdomains

If you have separate services (API, admin panel, etc.), add additional A records:

| Type | Host | Value |
|---|---|---|
| A | `api` | `YOUR_VPS_IP` |
| A | `admin` | `YOUR_VPS_IP` |
| A | `app` | `YOUR_VPS_IP` |

### Step 5 — Save All Changes

After saving, wait for DNS propagation. Use the verification tools in the next section to confirm.

---

## 5. Verify Domain Connection

Once DNS records are saved, use these tools to verify propagation.

### Ping

```bash
ping yourdomain.com
```

Expected output (once propagated):
```
PING yourdomain.com (142.93.50.18): 56 data bytes
64 bytes from 142.93.50.18: icmp_seq=0 ttl=55 time=12.3 ms
```

If it resolves to your VPS IP — DNS is working ✅

### nslookup

```bash
nslookup yourdomain.com
```

Expected output:
```
Server:         8.8.8.8
Address:        8.8.8.8#53

Non-authoritative answer:
Name:   yourdomain.com
Address: 142.93.50.18
```

### dig (More Detailed)

```bash
dig yourdomain.com A
```

Expected output section:
```
;; ANSWER SECTION:
yourdomain.com.   3600  IN  A  142.93.50.18
```

Check a specific subdomain:
```bash
dig api.yourdomain.com A
dig www.yourdomain.com A
```

Check which nameservers are authoritative:
```bash
dig yourdomain.com NS
```

### Online Tools

- [https://www.whatsmydns.net](https://www.whatsmydns.net) — Check DNS propagation worldwide
- [https://dnschecker.org](https://dnschecker.org) — Visual global DNS check
- [https://mxtoolbox.com](https://mxtoolbox.com) — Advanced DNS diagnostics

---

## 6. Configure Backend Server

### Run Backend on Localhost

Your Node.js/Express app should listen on a local port (e.g., `3000`). Do **not** expose your app directly on port 80/443 — use Nginx as a reverse proxy instead.

**Example Express server (`server.js`):**

```javascript
const express = require('express');
const app = express();

const PORT = process.env.PORT || 3000;

app.listen(PORT, '0.0.0.0', () => {
  console.log(`Server running on port ${PORT}`);
});
```

> ⚠️ Binding to `0.0.0.0` means the app listens on all network interfaces — including localhost and the VPS's public IP. This is necessary for Nginx to reach it internally.

### Why Not Expose Backend Directly?

| Concern | Direct Port Exposure | Via Nginx Reverse Proxy |
|---|---|---|
| SSL/HTTPS | Difficult to manage | Easy with Certbot |
| Multiple apps on same server | Not possible on one port | Yes — virtual hosts |
| Security | App process faces the internet | App stays on localhost |
| Logging & rate limiting | Minimal | Built into Nginx |
| Static file serving | Not efficient | Nginx is highly optimized |

### Bind to Correct Address

```javascript
// ✅ Listen on all interfaces
app.listen(3000, '0.0.0.0');

// ✅ Or just port (also binds to 0.0.0.0 by default in Node)
app.listen(3000);

// ❌ Only localhost — Nginx may fail to connect depending on config
app.listen(3000, '127.0.0.1');
```

### Use PM2 for Process Management

```bash
# Install PM2
npm install -g pm2

# Start your app
pm2 start server.js --name "myapp"

# Auto-restart on server reboot
pm2 startup
pm2 save

# View logs
pm2 logs myapp

# Check status
pm2 status
```

---

## 7. Setup Nginx Reverse Proxy

### What is a Reverse Proxy?

A **reverse proxy** sits in front of your application and forwards incoming requests to it. Nginx listens on port 80 (HTTP) and 443 (HTTPS) and routes traffic to the correct app based on the domain name.

### Step 1 — Install Nginx

```bash
# Ubuntu / Debian
sudo apt update
sudo apt install nginx -y

# CentOS / RHEL
sudo yum install epel-release -y
sudo yum install nginx -y

# Start and enable
sudo systemctl start nginx
sudo systemctl enable nginx

# Verify
sudo systemctl status nginx
```

### Step 2 — Understand Nginx File Structure

```
/etc/nginx/
├── nginx.conf                  # Main config file
├── sites-available/            # Config files for each site
│   └── yourdomain.com          # Your site config (inactive)
├── sites-enabled/              # Symlinks to active sites
│   └── yourdomain.com -> ../sites-available/yourdomain.com
└── conf.d/                     # Alternative location for configs
```

### Step 3 — Create a Server Block

```bash
sudo nano /etc/nginx/sites-available/yourdomain.com
```

**Basic reverse proxy config:**

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_cache_bypass $http_upgrade;
    }
}
```

### Step 4 — Enable the Site

```bash
# Create symlink to enable the site
sudo ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled/

# Test config for syntax errors
sudo nginx -t

# Expected output:
# nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
# nginx: configuration file /etc/nginx/nginx.conf test is successful

# Reload Nginx to apply changes
sudo systemctl reload nginx
```

### Step 5 — Allow HTTP Through Firewall

```bash
sudo ufw allow 'Nginx Full'
# or individually:
sudo ufw allow 80
sudo ufw allow 443
```

### Nginx Config Reference

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    # Logging
    access_log /var/log/nginx/yourdomain.access.log;
    error_log  /var/log/nginx/yourdomain.error.log;

    # Max upload size
    client_max_body_size 50M;

    # Proxy to backend
    location /api/ {
        proxy_pass http://localhost:3000/;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Serve static frontend files
    location / {
        root /var/www/yourdomain.com/build;
        index index.html;
        try_files $uri $uri/ /index.html;  # For React Router / SPA
    }
}
```

---

## 8. Connect Frontend with Domain

### Option A — Serve React Build via Nginx (Recommended for Production)

```bash
# Build your React app
npm run build

# Copy build to web root
sudo mkdir -p /var/www/yourdomain.com
sudo cp -r build/* /var/www/yourdomain.com/

# Set permissions
sudo chown -R www-data:www-data /var/www/yourdomain.com
```

Nginx config to serve static files:

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    root /var/www/yourdomain.com;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

> The `try_files` directive is **essential for Single Page Applications (SPAs)** — it ensures that React Router works correctly by falling back to `index.html` for all routes.

### Option B — Run React Dev Server (Not Recommended for Production)

```bash
npm start  # Runs on port 3000 by default
```

Then proxy it via Nginx — but this is only suitable for development.

### Configure API Base URL in Frontend

In your React app, avoid hardcoding `localhost` as the API URL. Use environment variables:

**`.env.production`:**
```env
REACT_APP_API_URL=https://api.yourdomain.com
```

**`.env.development`:**
```env
REACT_APP_API_URL=http://localhost:5000
```

**In your code:**
```javascript
const API_BASE = process.env.REACT_APP_API_URL;

const response = await fetch(`${API_BASE}/api/users`);
```

### Handle CORS on the Backend

When your frontend on `https://yourdomain.com` makes requests to `https://api.yourdomain.com`, the browser enforces **CORS (Cross-Origin Resource Sharing)** policies.

**Express.js CORS setup:**

```bash
npm install cors
```

```javascript
const cors = require('cors');

const allowedOrigins = [
  'https://yourdomain.com',
  'https://www.yourdomain.com',
  'http://localhost:3000'  // For development
];

app.use(cors({
  origin: function (origin, callback) {
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('CORS blocked: ' + origin));
    }
  },
  credentials: true,     // Allow cookies / Authorization headers
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

> ⚠️ Never use `cors({ origin: '*' })` in production if your API handles authenticated requests — this allows any website to make credentialed requests to your API.

---

## 9. Enable HTTPS (SSL)

### Why HTTPS?

- **Security:** Encrypts all data between user and server
- **SEO:** Google ranks HTTPS sites higher
- **Browser trust:** Modern browsers warn users about HTTP sites
- **Cookies/Auth:** Secure cookies require HTTPS

### Step 1 — Install Certbot

Certbot is the official tool for obtaining free **Let's Encrypt** SSL certificates.

```bash
# Ubuntu 20.04+
sudo apt update
sudo apt install certbot python3-certbot-nginx -y

# Ubuntu 18.04
sudo apt install certbot python-certbot-nginx -y

# CentOS / RHEL
sudo yum install certbot python3-certbot-nginx -y
```

### Step 2 — Obtain and Install SSL Certificate

```bash
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

Follow the prompts:
1. Enter your email address
2. Agree to Terms of Service
3. Choose whether to share email with EFF (optional)
4. Certbot will automatically modify your Nginx config to add HTTPS

### Step 3 — Verify SSL Configuration

After Certbot runs, your Nginx config will look similar to this:

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    return 301 https://$host$request_uri;  # Redirect HTTP → HTTPS
}

server {
    listen 443 ssl;
    server_name yourdomain.com www.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Step 4 — Test HTTPS

```bash
curl -I https://yourdomain.com
```

Expected response:
```
HTTP/2 200
server: nginx
...
```

Also check certificate details:
```bash
openssl s_client -connect yourdomain.com:443 -servername yourdomain.com
```

### Step 5 — Enable Auto-Renewal

Let's Encrypt certificates expire every **90 days**. Certbot sets up a cron job automatically:

```bash
# Test dry run (no actual renewal)
sudo certbot renew --dry-run

# Check the timer
sudo systemctl status certbot.timer

# View the cron entry
sudo crontab -l
```

Manually add a cron job if it's not already set:

```bash
sudo crontab -e
```

Add this line:

```cron
0 12 * * * /usr/bin/certbot renew --quiet && systemctl reload nginx
```

> This runs at noon every day — Certbot only renews if the cert is within 30 days of expiry.

---

## 10. Multiple Projects on Same VPS (Advanced)

You can host multiple apps on a single VPS using **Nginx server blocks** (also called virtual hosts). Each domain/subdomain gets its own config block.

### Project Layout

```
VPS (142.93.50.18)
├── App 1: shellelearningacademy.com → port 3000 (main website)
├── App 2: api.shellelearningacademy.com → port 5000 (REST API)
├── App 3: admin.shellelearningacademy.com → port 4000 (admin panel)
└── App 4: blog.shellelearningacademy.com → port 8000 (blog)
```

### DNS Setup

Add A records for each subdomain pointing to the **same VPS IP**:

| Type | Host | Value |
|---|---|---|
| A | `@` | `142.93.50.18` |
| A | `www` | `142.93.50.18` |
| A | `api` | `142.93.50.18` |
| A | `admin` | `142.93.50.18` |

### Create Separate Nginx Config Files

```bash
# App 1 - Main website
sudo nano /etc/nginx/sites-available/shellelearningacademy.com
```

```nginx
server {
    listen 443 ssl;
    server_name shellelearningacademy.com www.shellelearningacademy.com;

    ssl_certificate /etc/letsencrypt/live/shellelearningacademy.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/shellelearningacademy.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 80;
    server_name shellelearningacademy.com www.shellelearningacademy.com;
    return 301 https://$host$request_uri;
}
```

```bash
# App 2 - API
sudo nano /etc/nginx/sites-available/api.shellelearningacademy.com
```

```nginx
server {
    listen 443 ssl;
    server_name api.shellelearningacademy.com;

    ssl_certificate /etc/letsencrypt/live/api.shellelearningacademy.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.shellelearningacademy.com/privkey.pem;

    # Allow larger file uploads (for APIs with file handling)
    client_max_body_size 100M;

    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 80;
    server_name api.shellelearningacademy.com;
    return 301 https://$host$request_uri;
}
```

### Enable All Sites

```bash
# Enable all site configs
sudo ln -s /etc/nginx/sites-available/shellelearningacademy.com /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/api.shellelearningacademy.com /etc/nginx/sites-enabled/

# Generate SSL for each subdomain separately
sudo certbot --nginx -d shellelearningacademy.com -d www.shellelearningacademy.com
sudo certbot --nginx -d api.shellelearningacademy.com

# Test and reload
sudo nginx -t
sudo systemctl reload nginx
```

### PM2 Ecosystem Config

Manage all apps simultaneously with PM2:

```javascript
// ecosystem.config.js
module.exports = {
  apps: [
    {
      name: "main-app",
      script: "./apps/main/server.js",
      env: { PORT: 3000, NODE_ENV: "production" }
    },
    {
      name: "api-server",
      script: "./apps/api/server.js",
      env: { PORT: 5000, NODE_ENV: "production" }
    },
    {
      name: "admin-panel",
      script: "./apps/admin/server.js",
      env: { PORT: 4000, NODE_ENV: "production" }
    }
  ]
};
```

```bash
pm2 start ecosystem.config.js
pm2 save
```

---

## 11. Using Different VPS for Frontend & Backend

In larger architectures, you may want to separate your frontend and backend onto different servers for scalability, independent deployment, or team separation.

### Architecture Overview

```
DNS Setup:
  shellelearningacademy.com     → A Record → 142.93.50.18  (Frontend VPS)
  api.shellelearningacademy.com → A Record → 68.183.42.55  (Backend VPS)
```

### Frontend VPS Setup (IP: 142.93.50.18)

Serve the React build:

```nginx
# /etc/nginx/sites-available/shellelearningacademy.com
server {
    listen 443 ssl;
    server_name shellelearningacademy.com www.shellelearningacademy.com;

    ssl_certificate /etc/letsencrypt/live/shellelearningacademy.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/shellelearningacademy.com/privkey.pem;

    root /var/www/shellelearning/build;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

### Backend VPS Setup (IP: 68.183.42.55)

```nginx
# /etc/nginx/sites-available/api.shellelearningacademy.com
server {
    listen 443 ssl;
    server_name api.shellelearningacademy.com;

    ssl_certificate /etc/letsencrypt/live/api.shellelearningacademy.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.shellelearningacademy.com/privkey.pem;

    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Frontend Environment Config

```env
# .env.production (on the frontend VPS)
REACT_APP_API_URL=https://api.shellelearningacademy.com
```

### CORS on Backend

Make sure the backend explicitly allows the frontend domain:

```javascript
app.use(cors({
  origin: [
    'https://shellelearningacademy.com',
    'https://www.shellelearningacademy.com'
  ],
  credentials: true
}));
```

### DNS Record Summary

| Type | Host | IP / Value | Server |
|---|---|---|---|
| A | `@` | `142.93.50.18` | Frontend VPS |
| A | `www` | `142.93.50.18` | Frontend VPS |
| A | `api` | `68.183.42.55` | Backend VPS |

---

## 12. Security Best Practices

### 1. Configure UFW Firewall

```bash
# Install UFW
sudo apt install ufw -y

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (IMPORTANT: do this before enabling UFW)
sudo ufw allow ssh        # or: sudo ufw allow 22

# Allow web traffic
sudo ufw allow 80         # HTTP
sudo ufw allow 443        # HTTPS

# Enable firewall
sudo ufw enable

# Check status
sudo ufw status verbose
```

> ⚠️ Always allow SSH **before** enabling UFW or you'll lock yourself out of the server.

### 2. Close All Unused Ports

```bash
# View open ports
sudo ss -tlnp

# Remove an allowed rule
sudo ufw deny 3000      # Don't expose app ports directly
sudo ufw deny 5000
sudo ufw deny 27017     # MongoDB — never expose to internet
```

### 3. Disable Root SSH Login

```bash
sudo nano /etc/ssh/sshd_config
```

Change or add:
```
PermitRootLogin no
PasswordAuthentication no     # Only allow SSH key auth
PubkeyAuthentication yes
```

```bash
sudo systemctl restart sshd
```

### 4. Use SSH Key Authentication

On your **local machine**:

```bash
# Generate SSH key pair
ssh-keygen -t ed25519 -C "your@email.com"

# Copy public key to VPS
ssh-copy-id username@your-vps-ip
```

Or manually:
```bash
# On VPS: paste your public key
mkdir -p ~/.ssh
nano ~/.ssh/authorized_keys
# Paste contents of ~/.ssh/id_ed25519.pub

chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### 5. Fail2Ban — Block Brute Force Attacks

```bash
sudo apt install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

Create a local config:
```bash
sudo nano /etc/fail2ban/jail.local
```

```ini
[DEFAULT]
bantime = 1h
findtime = 10m
maxretry = 5

[sshd]
enabled = true
port = ssh

[nginx-http-auth]
enabled = true
```

```bash
sudo systemctl restart fail2ban

# Check banned IPs
sudo fail2ban-client status sshd
```

### 6. Keep System Updated

```bash
sudo apt update && sudo apt upgrade -y

# Enable automatic security updates
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure unattended-upgrades
```

### 7. Nginx Security Headers

Add to your Nginx server block:

```nginx
# Security headers
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "no-referrer-when-downgrade" always;
add_header Content-Security-Policy "default-src 'self' https: data: 'unsafe-inline' 'unsafe-eval';" always;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```

### Security Checklist

| Task | Status |
|---|---|
| UFW firewall configured | ✅ |
| Only ports 22, 80, 443 open | ✅ |
| Root login disabled | ✅ |
| SSH key auth only | ✅ |
| Fail2Ban installed | ✅ |
| HTTPS enabled | ✅ |
| Auto-updates enabled | ✅ |
| MongoDB not exposed to internet | ✅ |
| App not running as root | ✅ |

---

## 13. Common Errors & Fixes

### Error: Domain Not Resolving

**Symptom:** `ping yourdomain.com` doesn't return your VPS IP.

**Fixes:**
```bash
# 1. Check DNS propagation
dig yourdomain.com A @8.8.8.8       # Google's DNS
dig yourdomain.com A @1.1.1.1       # Cloudflare's DNS

# 2. Verify A record exists at registrar
# Log into your domain panel and double-check the record

# 3. Flush local DNS cache
# Linux:
sudo systemd-resolve --flush-caches
# macOS:
sudo dscacheutil -flushcache

# 4. Wait — propagation can take up to 48 hours
```

---

### Error: DNS Propagation Delay

**Symptom:** Domain resolves in some regions but not others.

**Fix:** This is normal. Use [whatsmydns.net](https://www.whatsmydns.net) to monitor propagation globally. Reduce TTL to 300 **before** making DNS changes next time.

---

### Error: Nginx 502 Bad Gateway

**Symptom:** `502 Bad Gateway` when visiting your domain.

**Meaning:** Nginx can't reach your backend app.

**Fixes:**
```bash
# 1. Is your app running?
pm2 status
# or
ps aux | grep node

# 2. Is it on the correct port?
sudo ss -tlnp | grep 3000

# 3. Test app locally on VPS
curl http://localhost:3000

# 4. Check Nginx proxy_pass port matches app port
cat /etc/nginx/sites-available/yourdomain.com | grep proxy_pass

# 5. Check Nginx error log
sudo tail -f /var/log/nginx/error.log
```

---

### Error: Nginx Config Syntax Error

**Symptom:** `sudo nginx -t` returns an error.

**Fixes:**
```bash
# Run test to see exact error line
sudo nginx -t

# Common mistakes:
# - Missing semicolon at end of directive
# - Typo in server_name
# - Wrong file path in ssl_certificate
# - Unclosed {} block

# Check a specific config file
sudo nginx -c /etc/nginx/nginx.conf -t
```

---

### Error: Port Already in Use

**Symptom:** App fails to start — `EADDRINUSE: address already in use :::3000`

**Fixes:**
```bash
# Find what's using the port
sudo lsof -i :3000
# or
sudo ss -tlnp | grep 3000

# Kill the process
sudo kill -9 <PID>

# Or just restart with PM2
pm2 restart myapp
```

---

### Error: SSL Certificate Not Working / Let's Encrypt Fails

**Symptom:** Certbot fails with `Connection refused` or domain verification error.

**Fixes:**
```bash
# 1. Ensure port 80 is open (Certbot uses HTTP-01 challenge)
sudo ufw allow 80

# 2. Ensure Nginx is running
sudo systemctl status nginx

# 3. Ensure domain is pointing to THIS server's IP
dig yourdomain.com A

# 4. Run Certbot with verbose output
sudo certbot --nginx -d yourdomain.com --verbose

# 5. Check Certbot logs
sudo cat /var/log/letsencrypt/letsencrypt.log
```

---

### Error: Mixed Content Warning

**Symptom:** HTTPS site works but browser shows "Not Secure" warnings for some resources.

**Cause:** Page loaded over HTTPS is requesting resources (images, scripts, API calls) over HTTP.

**Fix in React:**
```javascript
// ❌ Bad
const API_URL = 'http://api.yourdomain.com';

// ✅ Good
const API_URL = 'https://api.yourdomain.com';
```

---

## 14. Debugging Tools

### `dig` — DNS Lookup

```bash
# Basic A record lookup
dig yourdomain.com A

# Short output
dig yourdomain.com A +short

# Check specific DNS server
dig @8.8.8.8 yourdomain.com A

# Check all record types
dig yourdomain.com ANY

# Check TTL
dig yourdomain.com A | grep -A2 "ANSWER SECTION"
```

---

### `nslookup` — Quick DNS Check

```bash
# Basic lookup
nslookup yourdomain.com

# Check with specific DNS server
nslookup yourdomain.com 8.8.8.8

# Reverse lookup (IP → domain)
nslookup 142.93.50.18
```

---

### `curl` — Test HTTP/HTTPS Responses

```bash
# Basic request
curl https://yourdomain.com

# Show response headers only
curl -I https://yourdomain.com

# Follow redirects
curl -IL https://yourdomain.com

# Verbose — full request/response details
curl -v https://yourdomain.com

# Test specific endpoint
curl -X GET https://api.yourdomain.com/api/users

# POST request
curl -X POST https://api.yourdomain.com/api/login \
  -H "Content-Type: application/json" \
  -d '{"email": "test@example.com", "password": "secret"}'

# Check SSL certificate
curl -vI https://yourdomain.com 2>&1 | grep -E "SSL|certificate|expire"
```

---

### `netstat` / `ss` — Open Ports and Connections

```bash
# List all listening TCP ports
sudo ss -tlnp

# List with process names
sudo ss -tlnp | grep LISTEN

# Find what's using a specific port
sudo ss -tlnp | grep :3000

# Legacy netstat (if installed)
sudo netstat -tlnp
```

---

### `systemctl` — Service Management

```bash
# Check Nginx status
sudo systemctl status nginx

# Check error logs
sudo journalctl -u nginx -n 50 --no-pager

# Reload after config change
sudo systemctl reload nginx

# Full restart
sudo systemctl restart nginx

# Check PM2 process
pm2 list
pm2 logs
pm2 monit    # Real-time monitoring dashboard
```

---

### `tail` — Live Log Monitoring

```bash
# Watch Nginx access logs in real time
sudo tail -f /var/log/nginx/access.log

# Watch Nginx error logs
sudo tail -f /var/log/nginx/error.log

# Watch app logs (PM2)
pm2 logs myapp --lines 100

# System logs
sudo journalctl -f
```

---

## 15. Advanced Topics

### Load Balancing with Nginx

Distribute traffic across multiple app instances for high availability:

```nginx
upstream myapp_cluster {
    least_conn;                        # Routing strategy: least connections
    server localhost:3000;
    server localhost:3001;
    server localhost:3002;
}

server {
    listen 443 ssl;
    server_name yourdomain.com;

    location / {
        proxy_pass http://myapp_cluster;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

**Load balancing strategies:**

| Strategy | Directive | Description |
|---|---|---|
| Round Robin | (default) | Requests distributed evenly |
| Least Connections | `least_conn` | Send to server with fewest active connections |
| IP Hash | `ip_hash` | Same client always goes to same server (useful for sessions) |
| Weighted | `weight=N` | Distribute proportionally by weight |

---

### Cloudflare as CDN and DNS

Cloudflare acts as a **proxy and CDN** in front of your server:

1. Change nameservers at your registrar to Cloudflare's NS
2. Cloudflare handles DNS, CDN, and DDoS protection
3. Your actual server IP is **hidden** from attackers

**Benefits:**
- Free CDN with global edge caching
- DDoS protection
- SSL termination at the edge
- Analytics and firewall rules
- Page Rules for redirects, caching
- Bot protection

```
User → Cloudflare Edge (CDN) → Your VPS
```

> **Important:** When using Cloudflare proxy (orange cloud ☁️), your SSL cert on the VPS should use "Full (Strict)" mode in Cloudflare's SSL/TLS settings. This requires a valid cert on your origin server too.

---

### Nginx Reverse Proxy Caching

Cache backend responses to reduce load:

```nginx
# Define cache zone (in http block, outside server block)
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=1g
                 inactive=60m use_temp_path=off;

server {
    listen 443 ssl;
    server_name yourdomain.com;

    location / {
        proxy_cache my_cache;
        proxy_cache_valid 200 1h;              # Cache 200 responses for 1 hour
        proxy_cache_valid 404 1m;              # Cache 404s for 1 minute
        proxy_cache_use_stale error timeout;   # Use stale cache on error
        proxy_cache_key "$scheme$request_method$host$request_uri";

        add_header X-Cache-Status $upstream_cache_status;  # Debug header

        proxy_pass http://localhost:3000;
    }
}
```

---

### Rate Limiting

Protect your API from abuse:

```nginx
# Define rate limit zone (in http block)
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;

server {
    location /api/ {
        limit_req zone=api_limit burst=20 nodelay;
        limit_req_status 429;

        proxy_pass http://localhost:5000/;
    }
}
```

This allows **10 requests/second** with a burst of 20. Excess requests get a `429 Too Many Requests` response.

---

### WebSocket Support

For real-time apps (Socket.io, WebSockets):

```nginx
server {
    listen 443 ssl;
    server_name yourdomain.com;

    location /socket.io/ {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;

        # These headers are REQUIRED for WebSocket upgrade
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;

        # Increase timeout for long-lived connections
        proxy_read_timeout 86400;
        proxy_send_timeout 86400;
    }
}
```

---

### Gzip Compression

Reduce response size and improve load times:

```nginx
# In /etc/nginx/nginx.conf, inside http block
gzip on;
gzip_vary on;
gzip_min_length 1024;
gzip_proxied any;
gzip_comp_level 6;
gzip_types
    text/plain
    text/css
    text/xml
    text/javascript
    application/json
    application/javascript
    application/xml+rss
    application/atom+xml
    image/svg+xml;
```

---

## 16. Architecture Diagram

### Basic Single VPS Setup

```
┌──────────────────────────────────────────────────────────────────┐
│                         USER'S BROWSER                           │
│                  https://shellelearningacademy.com               │
└──────────────────────────┬───────────────────────────────────────┘
                           │ DNS Query
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│                      DNS RESOLUTION                              │
│                                                                  │
│  Recursive Resolver → Root DNS → TLD DNS (.com) → Authoritative │
│                                                                  │
│  shellelearningacademy.com  ──A Record──▶  142.93.50.18         │
└──────────────────────────┬───────────────────────────────────────┘
                           │ HTTPS Request to 142.93.50.18:443
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│                   VPS SERVER (142.93.50.18)                      │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                   NGINX (Port 80 / 443)                    │  │
│  │                                                            │  │
│  │  SSL Termination ✓   Gzip ✓   Rate Limiting ✓             │  │
│  │  Access Logs ✓       Security Headers ✓                   │  │
│  └──────────────────────────┬─────────────────────────────── ┘  │
│                             │ proxy_pass                         │
│                             │                                    │
│         ┌───────────────────┼────────────────────┐              │
│         │                   │                    │              │
│         ▼                   ▼                    ▼              │
│  ┌─────────────┐   ┌─────────────────┐   ┌────────────────┐    │
│  │  React App  │   │  Node.js / API  │   │ Admin Panel    │    │
│  │  Port 3000  │   │  Port 5000      │   │ Port 4000      │    │
│  └─────────────┘   └────────┬────────┘   └────────────────┘    │
│                             │                                    │
│                             ▼                                    │
│                    ┌────────────────┐                           │
│                    │   MongoDB      │                           │
│                    │  Port 27017    │                           │
│                    │ (localhost only)│                          │
│                    └────────────────┘                           │
└──────────────────────────────────────────────────────────────────┘
```

### Multi-VPS Architecture (Frontend + Backend Separated)

```
                    ┌───────────────────┐
                    │   USER BROWSER    │
                    └────────┬──────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
     shellelearning.com  api.shell..com  admin.shell..com
              │              │
              ▼              ▼
     A: 142.93.50.18   A: 68.183.42.55
              │              │
              ▼              ▼
   ┌──────────────────┐ ┌──────────────────┐
   │  FRONTEND VPS    │ │  BACKEND VPS     │
   │  142.93.50.18    │ │  68.183.42.55    │
   │                  │ │                  │
   │  Nginx           │ │  Nginx           │
   │  React Build     │ │  Node.js API     │
   │  Static Files    │ │  MongoDB         │
   └──────────────────┘ └──────────────────┘
```

### DNS + Nginx Request Flow (Detailed)

```
1. User types: https://shellelearningacademy.com
                         │
2. Browser DNS lookup ───┘
                         │
3. DNS returns: 142.93.50.18
                         │
4. Browser connects to 142.93.50.18:443
                         │
5. Nginx SSL handshake (Let's Encrypt cert)
                         │
6. Nginx matches server_name → routes to backend
                         │
7. proxy_pass → localhost:3000
                         │
8. Express/Node processes request
                         │
9. Response flows back:  Node → Nginx → Browser
                         │
10. Browser renders page ┘
```

---

## 17. Conclusion

### Summary of the Full Setup

You've now walked through the complete journey of connecting a domain to a VPS-hosted application — from DNS fundamentals to production-grade Nginx configuration, SSL, security hardening, and advanced scaling patterns.

Here's a condensed checklist of the entire production setup:

```
DOMAIN & DNS
  ✅ A Record: @ → VPS IP
  ✅ A Record: www → VPS IP
  ✅ A Record: api → VPS IP (if separate)
  ✅ DNS propagation verified with dig / nslookup

VPS & APP
  ✅ App running via PM2 on localhost port
  ✅ App bound to 0.0.0.0
  ✅ PM2 configured to restart on reboot

NGINX
  ✅ Nginx installed and running
  ✅ Server block created in sites-available
  ✅ Symlink created in sites-enabled
  ✅ nginx -t passes without errors
  ✅ proxy_pass pointing to correct port

SSL
  ✅ Certbot installed
  ✅ SSL certificate issued for domain
  ✅ HTTP → HTTPS redirect in place
  ✅ Auto-renewal cron/timer configured

SECURITY
  ✅ UFW firewall enabled
  ✅ Only ports 22, 80, 443 open
  ✅ Root login disabled
  ✅ SSH key auth only
  ✅ Fail2Ban installed
  ✅ Sensitive ports (MongoDB 27017, app ports) blocked
```

### Best Practices for Production Deployment

**Configuration Management:**
- Store all Nginx configs in version control (Git)
- Use environment variables — never hardcode secrets in code
- Separate `.env.development` and `.env.production`

**Monitoring:**
- Set up uptime monitoring ([UptimeRobot](https://uptimerobot.com) is free)
- Use PM2's monitoring or a tool like [Netdata](https://netdata.cloud)
- Enable error alerting so you know before users do

**Backups:**
- Automate database backups (MongoDB `mongodump`) with cron
- Store backups off-server (S3, Backblaze B2, etc.)
- Test restores regularly — untested backups are not real backups

**Deployments:**
- Automate deploys via CI/CD (GitHub Actions, etc.)
- Use blue-green or rolling deploys to achieve zero downtime
- Always run `nginx -t` before `systemctl reload nginx`

**Scaling:**
- When a single VPS isn't enough, add a load balancer
- Use Cloudflare for global CDN and DDoS protection
- Consider managed databases (MongoDB Atlas) to offload database ops

---

> **You've got the full picture.** From a raw domain name and a blank VPS to a secure, SSL-enabled, reverse-proxied production deployment — this guide covers it all. The same principles apply whether you're hosting a personal portfolio, a MERN stack SaaS, or an e-learning platform like Shell E-Learning Academy. Happy deploying! 🚀
