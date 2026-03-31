# NGINX Mastery Guide
## Beginner → Advanced → Production → Expert → Master

> **A Complete Industry-Level Handbook for Engineers, Architects, and DevOps Professionals**
>
> Written at the level of a Senior DevOps Engineer with 15+ years of production experience.
> This handbook covers everything from your first `nginx -t` to designing globally distributed, fault-tolerant infrastructure.

---

## 📋 Table of Contents

- [Level 1 — Beginner](#level-1--beginner)
  - [Chapter 1: What is NGINX?](#chapter-1-what-is-nginx)
  - [Chapter 2: Installation](#chapter-2-installation)
  - [Chapter 3: Directory Structure](#chapter-3-directory-structure)
  - [Chapter 4: Basic Configuration](#chapter-4-basic-configuration)
  - [Chapter 5: Serving Static Files](#chapter-5-serving-static-files)
- [Level 2 — Intermediate](#level-2--intermediate)
  - [Chapter 6: Reverse Proxy](#chapter-6-reverse-proxy)
  - [Chapter 7: Load Balancing](#chapter-7-load-balancing)
  - [Chapter 8: Caching](#chapter-8-caching)
  - [Chapter 9: Gzip Compression](#chapter-9-gzip-compression)
  - [Chapter 10: SSL / HTTPS Setup](#chapter-10-ssl--https-setup)
- [Level 3 — Advanced](#level-3--advanced)
  - [Chapter 11: Performance Tuning](#chapter-11-performance-tuning)
  - [Chapter 12: Rate Limiting](#chapter-12-rate-limiting)
  - [Chapter 13: Security Hardening](#chapter-13-security-hardening)
  - [Chapter 14: Logging Deep Dive](#chapter-14-logging-deep-dive)
- [Level 4 — Production](#level-4--production)
  - [Chapter 15: High Availability Setup](#chapter-15-high-availability-setup)
  - [Chapter 16: Load Balancing Strategies](#chapter-16-load-balancing-strategies)
  - [Chapter 17: NGINX with Docker](#chapter-17-nginx-with-docker)
  - [Chapter 18: CDN Integration](#chapter-18-cdn-integration)
  - [Chapter 19: Monitoring](#chapter-19-monitoring)
- [Level 5 — Expert](#level-5--expert)
  - [Chapter 20: NGINX Internals](#chapter-20-nginx-internals)
  - [Chapter 21: Worker Processes](#chapter-21-worker-processes)
  - [Chapter 22: Event Loop](#chapter-22-event-loop)
  - [Chapter 23: Memory Management](#chapter-23-memory-management)
- [Level 6 — Master](#level-6--master)
  - [Chapter 24: Large-Scale Architecture](#chapter-24-large-scale-architecture)
  - [Chapter 25: Multi-Region Deployment](#chapter-25-multi-region-deployment)
  - [Chapter 26: Failover Strategies](#chapter-26-failover-strategies)
  - [Chapter 27: Enterprise Design Patterns](#chapter-27-enterprise-design-patterns)
- [Real-World Projects](#real-world-projects)
  - [Project 1: Static Website Hosting](#project-1-static-website-hosting)
  - [Project 2: Reverse Proxy for Node.js App](#project-2-reverse-proxy-for-nodejs-app)
  - [Project 3: Load Balanced Application](#project-3-load-balanced-application)
  - [Project 4: Production NGINX Setup](#project-4-production-nginx-setup)
- [Debugging Section](#debugging-section)
- [Production Folder Structure](#production-folder-structure)
- [Security Best Practices Reference](#security-best-practices-reference)

---

# Level 1 — Beginner

---

## Chapter 1: What is NGINX?

### What You Will Learn
- The history and purpose of NGINX
- How NGINX differs from Apache
- Core use cases in modern infrastructure
- The mental model of NGINX as a traffic controller

### Why This Exists

Before NGINX, Apache was the dominant web server. Apache used a **process-per-connection** model — it spawned a new process (or thread) for every incoming HTTP request. This worked fine in the early internet era when traffic was low, but as the web scaled, Apache began to struggle. On a server handling 10,000 simultaneous connections, Apache would spawn 10,000 processes — a memory and CPU nightmare known as the **C10K problem**.

In 2002, **Igor Sysoev**, a Russian software engineer, began writing NGINX to solve this problem. The first public release was in 2004. NGINX took a radically different approach: instead of spawning a process per connection, it used an **event-driven, asynchronous, non-blocking architecture** that could handle thousands of connections in a single worker process.

NGINX (pronounced "engine-x") became one of the most downloaded software projects in history. As of 2024, it powers over **34% of all websites** on the internet, including Netflix, Airbnb, GitHub, Dropbox, and thousands of enterprise systems.

### How It Works Internally

NGINX operates on the following core principles:

1. **Master Process**: Reads configuration, manages workers, handles signals
2. **Worker Processes**: Handle actual connections using an event loop
3. **Event-Driven I/O**: Uses `epoll` (Linux), `kqueue` (BSD), or `select` to monitor many connections simultaneously without blocking
4. **Non-Blocking Sockets**: Connections do not block the worker — it moves on to the next event while waiting for I/O

```
┌────────────────────────────────────────┐
│            NGINX Process Model         │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │        Master Process            │  │
│  │  - Reads nginx.conf              │  │
│  │  - Spawns Worker Processes       │  │
│  │  - Handles SIGHUP (reload)       │  │
│  └──────────┬───────────────────────┘  │
│             │                          │
│    ┌────────┼────────┐                 │
│    ▼        ▼        ▼                 │
│  ┌────┐  ┌────┐  ┌────┐               │
│  │ W1 │  │ W2 │  │ W3 │  Workers      │
│  │    │  │    │  │    │               │
│  └────┘  └────┘  └────┘               │
│  Each handles thousands of            │
│  connections via event loop           │
└────────────────────────────────────────┘
```

### Architecture Explanation

NGINX can function in multiple roles simultaneously:

| Role | Description |
|------|-------------|
| **Web Server** | Serves static files (HTML, CSS, JS, images) directly from disk |
| **Reverse Proxy** | Forwards client requests to backend application servers |
| **Load Balancer** | Distributes traffic across multiple backend instances |
| **API Gateway** | Routes, authenticates, and rate-limits API requests |
| **SSL Terminator** | Handles HTTPS/TLS so backends don't have to |
| **Cache** | Stores backend responses and serves them without hitting upstream |
| **Media Streaming** | Streams video/audio using HTTP, HLS, RTMP |

### Real-World Example

Consider a modern web application:

```
                    Internet
                       │
                       ▼
              ┌─────────────────┐
              │   NGINX Server  │  ← Handles SSL, serves static files,
              │  (Port 80/443)  │    rate limits, proxies to backend
              └────────┬────────┘
                       │
           ┌───────────┼───────────┐
           ▼           ▼           ▼
      ┌─────────┐ ┌─────────┐ ┌─────────┐
      │  App 1  │ │  App 2  │ │  App 3  │  ← Node.js / Python / Java
      └─────────┘ └─────────┘ └─────────┘
           │           │           │
           └───────────┴───────────┘
                       │
                       ▼
              ┌─────────────────┐
              │    Database     │  ← PostgreSQL / MySQL / MongoDB
              └─────────────────┘
```

### Commands

```bash
# Check NGINX version
nginx -v

# Check detailed version with compile flags
nginx -V

# Test configuration syntax
nginx -t

# Start NGINX
sudo systemctl start nginx

# Stop NGINX
sudo systemctl stop nginx

# Reload configuration (zero-downtime)
sudo systemctl reload nginx

# Full restart
sudo systemctl restart nginx

# Enable NGINX on boot
sudo systemctl enable nginx

# Check status
sudo systemctl status nginx
```

### Production Use Case

At Netflix, NGINX is used as the edge proxy to handle video streaming requests, SSL termination, and traffic routing across regions. Each NGINX instance handles hundreds of thousands of concurrent connections using minimal RAM — something impossible with a thread-per-connection server.

### Common Mistakes

- Confusing **restart** (downtime) with **reload** (zero-downtime) — always use `reload` in production
- Thinking NGINX is "just a web server" — it is a complete traffic management platform
- Ignoring the master/worker process model — understand it before tuning `worker_processes`

### Best Practices

- Always run `nginx -t` before any reload
- Understand that NGINX is stateless by design — state belongs in Redis/Memcached
- Use NGINX as the single entry point for all traffic into your infrastructure
- Never expose backend application servers directly to the internet

### Hands-On Tasks

1. Draw your current infrastructure diagram and identify where NGINX fits
2. Visit `nginx.org/en/docs` and read the Beginner's Guide
3. Identify 3 production websites that use NGINX (use `curl -I https://example.com` and check the `Server:` header)

### Debugging Section

```bash
# Is NGINX running?
ps aux | grep nginx

# What ports is NGINX listening on?
sudo ss -tlnp | grep nginx

# Check NGINX error log in real time
sudo tail -f /var/log/nginx/error.log
```

---

## Chapter 2: Installation

### What You Will Learn
- Installing NGINX on Ubuntu/Debian, CentOS/RHEL, and from source
- Installing NGINX Plus (commercial)
- Verifying a successful installation
- Running NGINX inside Docker

### Why This Exists

NGINX must be installed before it can do anything. The installation method matters: package manager installations are fast and maintainable but may not include all modules. Source compilation gives you full control but requires manual maintenance.

### How It Works Internally

When you install NGINX via a package manager, you get a pre-compiled binary with a specific set of modules built in. When you compile from source, you choose exactly which modules to include using `--with-*` flags.

### Installation on Ubuntu/Debian

```bash
# Update package index
sudo apt update

# Install NGINX
sudo apt install nginx -y

# Verify installation
nginx -v
# Output: nginx version: nginx/1.24.0

# Check if running
sudo systemctl status nginx

# Enable on boot
sudo systemctl enable nginx
```

**Install from the official NGINX repository (recommended for latest stable):**

```bash
# Install prerequisites
sudo apt install -y curl gnupg2 ca-certificates lsb-release ubuntu-keyring

# Import NGINX signing key
curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
    | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null

# Add stable repo
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
http://nginx.org/packages/ubuntu $(lsb_release -cs) nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list

# Install
sudo apt update
sudo apt install nginx -y
```

### Installation on CentOS/RHEL/Rocky Linux

```bash
# Create repo file
sudo tee /etc/yum.repos.d/nginx.repo << 'EOF'
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
EOF

# Install
sudo yum install nginx -y

# Start and enable
sudo systemctl start nginx
sudo systemctl enable nginx

# Open firewall (if using firewalld)
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

### Compiling NGINX from Source

Compiling from source is necessary when you need custom modules (e.g., `ngx_http_geoip2_module`, `nginx-rtmp-module`) or specific compile-time optimizations.

```bash
# Install build dependencies
sudo apt install -y build-essential libpcre3 libpcre3-dev zlib1g zlib1g-dev \
    libssl-dev libgd-dev libgeoip-dev

# Download NGINX source
cd /tmp
wget http://nginx.org/download/nginx-1.24.0.tar.gz
tar -zxvf nginx-1.24.0.tar.gz
cd nginx-1.24.0

# Configure with common modules
./configure \
    --prefix=/etc/nginx \
    --sbin-path=/usr/sbin/nginx \
    --modules-path=/usr/lib64/nginx/modules \
    --conf-path=/etc/nginx/nginx.conf \
    --error-log-path=/var/log/nginx/error.log \
    --http-log-path=/var/log/nginx/access.log \
    --pid-path=/var/run/nginx.pid \
    --with-http_ssl_module \
    --with-http_v2_module \
    --with-http_gzip_static_module \
    --with-http_stub_status_module \
    --with-http_realip_module \
    --with-http_auth_request_module \
    --with-threads \
    --with-stream \
    --with-stream_ssl_module \
    --with-pcre \
    --with-file-aio

# Compile
make -j$(nproc)

# Install
sudo make install

# Verify
nginx -V
```

### Running NGINX in Docker

```bash
# Pull official image
docker pull nginx:latest

# Run a basic container
docker run -d \
    --name nginx-test \
    -p 80:80 \
    nginx:latest

# Run with custom config and content
docker run -d \
    --name nginx-prod \
    -p 80:80 \
    -p 443:443 \
    -v /path/to/nginx.conf:/etc/nginx/nginx.conf:ro \
    -v /path/to/html:/usr/share/nginx/html:ro \
    -v /path/to/ssl:/etc/nginx/ssl:ro \
    nginx:latest

# Verify it's running
docker ps
curl -I http://localhost
```

### Verifying the Installation

```bash
# 1. Version check
nginx -v

# 2. Configuration test
sudo nginx -t
# Expected: nginx: configuration file /etc/nginx/nginx.conf test is successful

# 3. HTTP test
curl -I http://localhost
# Expected: HTTP/1.1 200 OK

# 4. Browser test
# Open http://YOUR_SERVER_IP — you should see the NGINX welcome page
```

### Common Mistakes

- Not adding NGINX to systemd startup (`systemctl enable nginx`)
- Installing an old version from OS package repos instead of the official NGINX repo
- Forgetting to open firewall ports 80 and 443

### Best Practices

- Always install from the official NGINX repository for security patches
- In production, pin your NGINX version to prevent unexpected upgrades
- Document your compile flags if building from source — you'll need them for upgrades

---

## Chapter 3: Directory Structure

### What You Will Learn
- Every important file and directory in an NGINX installation
- The role of `nginx.conf`, `conf.d/`, `sites-available/`, `sites-enabled/`
- How include directives connect configuration files
- The standard production folder layout

### Why This Exists

NGINX's configuration is entirely file-based. Understanding the directory structure is foundational — without it, you cannot find configuration, logs, or SSL certificates when debugging production incidents.

### NGINX Directory Layout (Ubuntu/Debian)

```
/etc/nginx/                         ← Main configuration directory
├── nginx.conf                      ← Main configuration file (root)
├── conf.d/                         ← Drop-in configuration files
│   ├── default.conf                ← Default server block
│   └── *.conf                      ← Additional virtual hosts
├── sites-available/                ← All defined virtual hosts (Debian convention)
│   ├── default                     ← Default site
│   └── myapp.conf                  ← Your application
├── sites-enabled/                  ← Symlinks to active virtual hosts
│   └── myapp.conf -> ../sites-available/myapp.conf
├── snippets/                       ← Reusable config snippets
│   ├── fastcgi-php.conf
│   └── snakeoil.conf
├── modules-available/              ← Available dynamic modules
├── modules-enabled/                ← Enabled dynamic modules
├── mime.types                      ← MIME type mappings
├── koi-utf                         ← Character encoding maps
├── koi-win
├── win-utf
└── fastcgi_params                  ← FastCGI parameter defaults

/var/log/nginx/                     ← Log files
├── access.log                      ← HTTP access log
└── error.log                       ← Error log

/var/www/html/                      ← Default web root
└── index.nginx-debian.html         ← Default welcome page

/usr/share/nginx/html/              ← Alternative default web root (RHEL)
└── index.html

/run/nginx.pid                      ← Master process ID
/usr/lib/nginx/modules/             ← Dynamic modules
```

### File-by-File Explanation

**`/etc/nginx/nginx.conf`** — The root configuration file. Every other file is included from here. The master process reads this file on start and reload.

**`conf.d/*.conf`** — Files in this directory are automatically included by the default `nginx.conf`. Used for virtual host definitions on RHEL-based systems.

**`sites-available/`** — Convention used on Debian/Ubuntu systems. Contains all your site configurations, whether enabled or not. This is where you create new site files.

**`sites-enabled/`** — Contains symlinks to files in `sites-available/`. Only configurations symlinked here are active.

```bash
# Enable a site (Debian/Ubuntu pattern)
sudo ln -s /etc/nginx/sites-available/myapp.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx

# Disable a site
sudo rm /etc/nginx/sites-enabled/myapp.conf
sudo nginx -t
sudo systemctl reload nginx
```

**`mime.types`** — Maps file extensions to MIME types. This is how NGINX knows to send `Content-Type: text/javascript` for `.js` files.

```bash
# View the include in nginx.conf
grep mime /etc/nginx/nginx.conf
# http {
#     include /etc/nginx/mime.types;
```

### The `include` Directive

The `include` directive is how NGINX connects multiple configuration files:

```nginx
# In nginx.conf
http {
    include       /etc/nginx/mime.types;
    include       /etc/nginx/conf.d/*.conf;
    include       /etc/nginx/sites-enabled/*;
}
```

This design allows you to split configuration across many files, making large configurations manageable.

### Production Folder Structure (Recommended)

```
/etc/nginx/
├── nginx.conf                   ← Minimal root config, includes everything
├── conf.d/
│   ├── upstream.conf            ← All upstream definitions
│   ├── cache.conf               ← Cache zone definitions
│   └── gzip.conf                ← Compression settings
├── sites-available/
│   ├── api.example.com.conf     ← API service
│   ├── app.example.com.conf     ← Main web app
│   └── static.example.com.conf  ← Static asset server
├── sites-enabled/               ← Symlinks only
├── snippets/
│   ├── ssl-params.conf          ← Shared SSL parameters
│   ├── security-headers.conf    ← HTTP security headers
│   └── proxy-params.conf        ← Shared proxy headers
└── ssl/
    ├── example.com/
    │   ├── fullchain.pem
    │   └── privkey.pem
    └── dhparam.pem

/var/log/nginx/
├── access.log                   ← Combined access log
├── error.log                    ← Error log
├── api.example.com-access.log   ← Per-vhost access log
└── api.example.com-error.log    ← Per-vhost error log
```

### Commands

```bash
# View current configuration structure
ls -la /etc/nginx/

# View active includes
grep -r "include" /etc/nginx/nginx.conf

# Check what sites are enabled
ls -la /etc/nginx/sites-enabled/

# Find all config files
find /etc/nginx -name "*.conf" | sort

# View NGINX's compiled-in defaults
nginx -V 2>&1 | tr -- - '\n' | grep path
```

### Practical Exercise

1. List all files under `/etc/nginx/` and understand each one's purpose
2. Open `/etc/nginx/nginx.conf` and identify every `include` directive
3. Create a `snippets/security-headers.conf` file and include it in your server block

### Debugging Section

```bash
# Find syntax errors including in included files
sudo nginx -t

# Dump the full effective configuration (after all includes are resolved)
sudo nginx -T

# Find which config file handles a specific server name
grep -r "server_name" /etc/nginx/
```

---

## Chapter 4: Basic Configuration

### What You Will Learn
- The structure of `nginx.conf` — contexts, directives, blocks
- `main`, `events`, `http`, `server`, and `location` contexts
- How directives inherit from parent contexts
- Writing your first server block

### Why This Exists

The configuration file is the brain of NGINX. Every behavior — what to serve, how to route, what to log, how to compress — is defined here. Understanding the configuration language is the single most important skill in NGINX mastery.

### How It Works Internally

NGINX configuration is hierarchical. It has **contexts** (blocks) and **directives** (instructions). Directives in outer contexts are inherited by inner contexts unless overridden.

```
nginx.conf structure:
─────────────────────
main context (global)
│   ├── worker_processes
│   ├── error_log
│   └── pid
│
├── events { }
│   └── worker_connections
│
└── http { }
    ├── include mime.types
    ├── access_log
    ├── gzip
    │
    ├── upstream { }           ← backend groups
    │
    └── server { }             ← virtual host
        ├── listen
        ├── server_name
        ├── access_log
        │
        └── location / { }     ← URL matching
            ├── root
            └── index
```

### The Default `nginx.conf` — Annotated

```nginx
# ─────────────────────────────────────────────
# MAIN CONTEXT (global settings)
# ─────────────────────────────────────────────

# How many worker processes to spawn.
# 'auto' = number of CPU cores
worker_processes auto;

# Error log location and level
# Levels: debug, info, notice, warn, error, crit, alert, emerg
error_log /var/log/nginx/error.log warn;

# PID file for the master process
pid /run/nginx.pid;

# Load dynamic modules
include /etc/nginx/modules-enabled/*.conf;


# ─────────────────────────────────────────────
# EVENTS CONTEXT
# Controls connection processing behavior
# ─────────────────────────────────────────────
events {
    # Max simultaneous connections per worker
    worker_connections 1024;

    # Accept multiple connections at once (recommended)
    multi_accept on;

    # Connection processing method (epoll = Linux best practice)
    use epoll;
}


# ─────────────────────────────────────────────
# HTTP CONTEXT
# All web server / proxy configuration
# ─────────────────────────────────────────────
http {
    # Include MIME types mapping
    include       /etc/nginx/mime.types;

    # Default MIME type for unknown file extensions
    default_type  application/octet-stream;

    # Define log format
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    # Global access log
    access_log /var/log/nginx/access.log main;

    # Efficient file sending (uses sendfile() syscall)
    sendfile        on;

    # Optimize TCP packet sending
    tcp_nopush      on;
    tcp_nodelay     on;

    # How long to keep a connection open for reuse
    keepalive_timeout 65;

    # Hide NGINX version in error pages and Server header
    server_tokens off;

    # Include virtual host configurations
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

### Writing Your First Server Block

```nginx
# /etc/nginx/sites-available/mysite.conf

server {
    # Listen on port 80 (HTTP)
    listen 80;

    # The domain name(s) this server block responds to
    server_name example.com www.example.com;

    # Document root (where files are served from)
    root /var/www/example.com;

    # Default index files
    index index.html index.htm;

    # Access and error logs for this virtual host
    access_log /var/log/nginx/example.com-access.log;
    error_log  /var/log/nginx/example.com-error.log;

    # Main location block
    location / {
        # Try to serve requested file, then directory, then 404
        try_files $uri $uri/ =404;
    }
}
```

### Location Block Matching

Location blocks are the most important concept in NGINX configuration. They match incoming request URIs and determine what to do with them.

```nginx
# Matching priority (highest to lowest):
# 1. Exact match          (=)
# 2. Preferential prefix  (^~)
# 3. Regex match          (~, ~*)
# 4. Prefix match         (none)

server {
    # 1. Exact match — highest priority
    location = /favicon.ico {
        log_not_found off;
        access_log off;
    }

    # 2. Preferential prefix — stops regex matching
    location ^~ /images/ {
        root /var/www/static;
        expires 30d;
    }

    # 3. Case-sensitive regex match
    location ~ \.php$ {
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        include fastcgi_params;
    }

    # 3. Case-insensitive regex match
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        expires 7d;
        add_header Cache-Control "public, no-transform";
    }

    # 4. Prefix match — lowest priority
    location /api/ {
        proxy_pass http://backend_api;
    }

    # 4. Default — catch-all
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

### Key Directives Reference Table

| Directive | Context | Purpose | Example |
|-----------|---------|---------|---------|
| `listen` | server | Port to listen on | `listen 80;` |
| `server_name` | server | Domain matching | `server_name example.com;` |
| `root` | http/server/location | Document root | `root /var/www/html;` |
| `index` | http/server/location | Default index files | `index index.html;` |
| `try_files` | location | File serving logic | `try_files $uri $uri/ =404;` |
| `return` | server/location | Redirect/respond | `return 301 https://$host$request_uri;` |
| `rewrite` | server/location | URL rewriting | `rewrite ^/old(.*)$ /new$1 permanent;` |
| `include` | any | Include files | `include snippets/ssl.conf;` |
| `error_page` | server/location | Custom error pages | `error_page 404 /404.html;` |
| `access_log` | http/server/location | Access log path | `access_log /var/log/nginx/app.log;` |

### Practical Exercises

1. Create a minimal `nginx.conf` with a single server block serving `Hello, World!`
2. Add a location block that returns a 403 for any request to `/admin/`
3. Add custom error pages for 404 and 500 errors

```nginx
# Exercise 1 solution — minimal server
server {
    listen 80 default_server;
    server_name _;
    root /var/www/html;

    location / {
        return 200 "Hello, World!\n";
        add_header Content-Type text/plain;
    }
}

# Exercise 2 solution — block /admin
location /admin {
    return 403 "Forbidden\n";
}

# Exercise 3 solution — custom error pages
error_page 404 /errors/404.html;
error_page 500 502 503 504 /errors/50x.html;

location /errors/ {
    root /var/www;
    internal;  # Cannot be accessed directly by clients
}
```

### Debugging Section

```bash
# Test configuration
sudo nginx -t

# View full resolved configuration (all includes expanded)
sudo nginx -T | less

# Find which server block handles a request
# Look for: [debug] ... the "server_name" directive matches

# Check what NGINX is actually doing
sudo nginx -t 2>&1
# common errors:
# "unknown directive" — typo in directive name
# "host not found in upstream" — backend not reachable
# "no such file or directory" — wrong root path
```

---

## Chapter 5: Serving Static Files

### What You Will Learn
- Efficient static file serving configuration
- `root` vs `alias` directives
- Browser caching with `expires` and `Cache-Control`
- MIME types and the importance of correct content types
- Optimizing static file delivery with `sendfile`, `tcp_nopush`

### Why This Exists

Serving static files (HTML, CSS, JS, images, fonts, videos) is NGINX's most fundamental use case and the one it excels at. NGINX can serve hundreds of thousands of static file requests per second from a single server — far more than any application server. Offloading static files to NGINX dramatically reduces load on your application.

### How It Works Internally

When NGINX receives a request for a static file:

1. Parse the URI from the HTTP request
2. Map the URI to a filesystem path using `root` or `alias`
3. Call `stat()` to check if the file exists
4. Use `sendfile()` syscall to send file directly from kernel space to socket (zero-copy)
5. Client receives the file

The `sendfile()` syscall is the key optimization — it avoids copying data from kernel space to user space and back, making static file serving extremely fast.

### Architecture: Static File Serving

```
Client Request: GET /css/style.css

    Client
      │ HTTP GET /css/style.css
      ▼
  ┌──────────────────────────────────┐
  │           NGINX Worker           │
  │                                  │
  │  1. Parse URI: /css/style.css    │
  │  2. root = /var/www/html         │
  │  3. Full path: /var/www/html/    │
  │               css/style.css      │
  │  4. stat() — file exists?        │
  │  5. sendfile() → socket          │
  └──────────────────────────────────┘
      │
      │ HTTP/1.1 200 OK
      │ Content-Type: text/css
      │ [file contents]
      ▼
    Client
```

### Configuration: Complete Static File Server

```nginx
server {
    listen 80;
    server_name static.example.com;

    # Document root — all URIs are resolved relative to this path
    root /var/www/static;

    # Enable efficient file sending
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;

    # ─── HTML Files ────────────────────────────────
    location / {
        index index.html index.htm;
        try_files $uri $uri/ =404;

        # Short cache — HTML changes frequently
        expires 1h;
        add_header Cache-Control "public, must-revalidate";
    }

    # ─── CSS and JavaScript ─────────────────────────
    location ~* \.(css|js)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        add_header Vary "Accept-Encoding";

        # Enable gzip for text assets
        gzip on;
        gzip_types text/css application/javascript;
    }

    # ─── Images ─────────────────────────────────────
    location ~* \.(jpg|jpeg|png|gif|ico|svg|webp)$ {
        expires 30d;
        add_header Cache-Control "public, no-transform";

        # Open file descriptor cache (performance)
        open_file_cache max=1000 inactive=30s;
        open_file_cache_valid 60s;
        open_file_cache_min_uses 2;
        open_file_cache_errors on;
    }

    # ─── Fonts ───────────────────────────────────────
    location ~* \.(woff|woff2|eot|ttf|otf)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";

        # Allow cross-origin font requests (CDN use)
        add_header Access-Control-Allow-Origin "*";
    }

    # ─── Videos ──────────────────────────────────────
    location ~* \.(mp4|webm|ogg)$ {
        # Support range requests for video seeking
        mp4;
        mp4_buffer_size 1m;
        mp4_max_buffer_size 5m;

        expires 7d;
        add_header Cache-Control "public";
    }

    # ─── Deny hidden files (.htaccess, .git, etc.) ───
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }

    # ─── Favicon — silence 404 noise ─────────────────
    location = /favicon.ico {
        access_log off;
        log_not_found off;
    }

    # ─── Robots.txt ───────────────────────────────────
    location = /robots.txt {
        access_log off;
        log_not_found off;
    }

    # ─── Custom 404 page ─────────────────────────────
    error_page 404 /404.html;
    location = /404.html {
        internal;
    }

    access_log /var/log/nginx/static.example.com-access.log;
    error_log  /var/log/nginx/static.example.com-error.log;
}
```

### `root` vs `alias` — Critical Difference

This is one of the most confusing aspects for beginners:

```nginx
# root: appends the ENTIRE location URI to the root path
location /images/ {
    root /var/www;
    # Request: GET /images/photo.jpg
    # File served: /var/www/images/photo.jpg
    # URI is APPENDED to root
}

# alias: REPLACES the location prefix with the alias path
location /images/ {
    alias /var/www/photos/;
    # Request: GET /images/photo.jpg
    # File served: /var/www/photos/photo.jpg
    # Location prefix is REPLACED by alias
}
```

**Rule of thumb**: Use `root` when the directory name matches the URI. Use `alias` when they differ.

### Open File Cache

For high-traffic static file servers, NGINX can cache open file descriptors to avoid repeated `open()` and `stat()` syscalls:

```nginx
http {
    open_file_cache max=10000 inactive=30s;
    open_file_cache_valid 60s;
    open_file_cache_min_uses 2;
    open_file_cache_errors on;
}
```

| Directive | Purpose |
|-----------|---------|
| `max` | Maximum number of cached file descriptors |
| `inactive` | Remove if not accessed within this time |
| `valid` | How often to check if cached info is still valid |
| `min_uses` | Minimum uses before caching |
| `errors` | Cache file not found results (avoids repeated stat()) |

### Practical Exercises

1. Set up a virtual host serving a static website from `/var/www/mysite`
2. Add browser caching headers — 1 year for assets, 1 hour for HTML
3. Test cache headers: `curl -I http://localhost/style.css` and verify `Cache-Control` header
4. Test that hidden files are blocked: `curl http://localhost/.htaccess`

### Debugging Section

```bash
# Test a static file is being served correctly
curl -I http://localhost/index.html

# Check what file path NGINX maps to
# Enable debug logging temporarily
error_log /var/log/nginx/error.log debug;
# Then watch:
tail -f /var/log/nginx/error.log | grep "open()"

# Verify MIME type
curl -I http://localhost/script.js | grep Content-Type
# Should be: Content-Type: application/javascript

# Check open file cache stats (NGINX stub_status module required)
curl http://localhost/nginx_status
```

---

# Level 2 — Intermediate

---

## Chapter 6: Reverse Proxy

### What You Will Learn
- What a reverse proxy is and why every production system uses one
- Configuring NGINX as a reverse proxy for any backend
- Setting correct proxy headers (`X-Real-IP`, `X-Forwarded-For`, etc.)
- WebSocket proxying
- Proxying to Node.js, Python, Java, and Docker containers

### Why This Exists

A **reverse proxy** sits in front of your application servers, accepting client connections and forwarding them to the appropriate backend. Without a reverse proxy:
- Your application server is directly exposed to the internet
- You cannot easily scale horizontally
- SSL termination must be handled by each application
- You have no central point for rate limiting, auth, caching, or logging

NGINX as a reverse proxy gives you all these capabilities in one place.

### How It Works Internally

```
WITHOUT reverse proxy:
  Client ──────────────────────────────► Node.js:3000 (exposed directly)
  Client ──────────────────────────────► Python:8000 (exposed directly)

WITH reverse proxy:
  Client ──► NGINX:443 ──► Node.js:3000 (private network)
                      └──► Python:8000 (private network)
```

When NGINX proxies a request:
1. Receive HTTP request from client
2. Open a connection to the upstream server (or reuse from connection pool)
3. Forward the request, adding proxy headers
4. Buffer the upstream response
5. Return the response to the client

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│                  Reverse Proxy Architecture              │
│                                                         │
│  Client                                                 │
│    │ HTTPS:443                                          │
│    ▼                                                    │
│  ┌─────────────────────────────────┐                   │
│  │    NGINX (Reverse Proxy)        │                   │
│  │                                 │                   │
│  │  ┌─────────────────────────┐   │                   │
│  │  │ /api  → :8080 (API)     │   │  Private Network  │
│  │  │ /app  → :3000 (Node.js) │   │                   │
│  │  │ /     → :80   (Static)  │   │                   │
│  │  └─────────────────────────┘   │                   │
│  └──────────────┬──────────────────┘                   │
│                 │                                       │
│    ┌────────────┼──────────────┐                       │
│    ▼            ▼              ▼                       │
│  ┌──────┐   ┌──────┐   ┌──────────┐                   │
│  │ API  │   │ App  │   │  Static  │                   │
│  │:8080 │   │:3000 │   │  Files   │                   │
│  └──────┘   └──────┘   └──────────┘                   │
└─────────────────────────────────────────────────────────┘
```

### Basic Reverse Proxy Configuration

```nginx
# /etc/nginx/sites-available/nodeapp.conf

server {
    listen 80;
    server_name app.example.com;

    location / {
        # Forward all requests to Node.js running on port 3000
        proxy_pass http://127.0.0.1:3000;

        # Pass original headers to the backend
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Reusable Proxy Parameters Snippet

```nginx
# /etc/nginx/snippets/proxy-params.conf
proxy_set_header Host               $host;
proxy_set_header X-Real-IP          $remote_addr;
proxy_set_header X-Forwarded-For    $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto  $scheme;
proxy_set_header X-Forwarded-Host   $server_name;

proxy_http_version 1.1;
proxy_set_header Connection "";

proxy_connect_timeout    60s;
proxy_send_timeout       60s;
proxy_read_timeout       60s;

proxy_buffer_size        4k;
proxy_buffers            8 4k;
proxy_busy_buffers_size  8k;
```

### Production Reverse Proxy Configuration

```nginx
upstream node_backend {
    server 127.0.0.1:3000;
    keepalive 32;   # Keep 32 connections open to backend
}

server {
    listen 80;
    server_name app.example.com;

    # Redirect HTTP to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name app.example.com;

    # SSL configuration (see Chapter 10)
    ssl_certificate     /etc/nginx/ssl/example.com/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/example.com/privkey.pem;
    include snippets/ssl-params.conf;

    # Security headers
    include snippets/security-headers.conf;

    # Client body size limit
    client_max_body_size 50m;

    # Proxy timeout settings
    proxy_connect_timeout  60s;
    proxy_send_timeout    300s;
    proxy_read_timeout    300s;

    # ─── API routes ─────────────────────────────────────
    location /api/ {
        include snippets/proxy-params.conf;
        proxy_pass http://node_backend;

        # Rate limit API requests
        limit_req zone=api_limit burst=20 nodelay;
    }

    # ─── Static assets (served by NGINX, not the app) ───
    location /static/ {
        alias /var/www/app/public/;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # ─── WebSocket support ───────────────────────────────
    location /ws/ {
        include snippets/proxy-params.conf;
        proxy_pass http://node_backend;

        # WebSocket upgrade headers
        proxy_http_version 1.1;
        proxy_set_header Upgrade    $http_upgrade;
        proxy_set_header Connection "upgrade";

        # WebSocket timeout (stay alive)
        proxy_read_timeout 86400s;
    }

    # ─── Application ─────────────────────────────────────
    location / {
        include snippets/proxy-params.conf;
        proxy_pass http://node_backend;

        # Cache backend 404s briefly
        proxy_intercept_errors on;
        error_page 404 /404.html;
    }

    access_log /var/log/nginx/app.example.com-access.log;
    error_log  /var/log/nginx/app.example.com-error.log;
}
```

### Understanding Proxy Headers

| Header | Value | Purpose |
|--------|-------|---------|
| `Host` | `$host` | The original `Host` header from the client |
| `X-Real-IP` | `$remote_addr` | The direct client IP |
| `X-Forwarded-For` | `$proxy_add_x_forwarded_for` | Chain of IPs (multiple proxies) |
| `X-Forwarded-Proto` | `$scheme` | `http` or `https` |
| `X-Forwarded-Host` | `$server_name` | Original hostname |

Your application must use `X-Real-IP` or `X-Forwarded-For` to get the client's real IP address.

### WebSocket Proxying

WebSockets require an HTTP Upgrade. NGINX must be explicitly configured to pass the upgrade headers:

```nginx
location /socket.io/ {
    proxy_pass http://node_backend;
    proxy_http_version 1.1;
    proxy_set_header Upgrade    $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host       $host;
    proxy_cache_bypass $http_upgrade;
}
```

### Docker Container Proxy

When proxying to Docker containers, use the container name as the upstream (if on the same Docker network):

```nginx
upstream docker_app {
    server app:3000;  # Docker service name
}

server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://docker_app;
        include snippets/proxy-params.conf;
    }
}
```

### Common Mistakes

- Not setting `proxy_set_header Host $host` — the backend receives NGINX's internal hostname
- Forgetting `X-Forwarded-For` — application logs show NGINX's IP, not client's
- Using `proxy_pass http://backend/` (trailing slash) with `location /api/` — changes URI behavior
- Not setting `keepalive` in upstream — creates a new TCP connection per request

### Debugging Section

```bash
# Test proxy is working
curl -v http://localhost/api/health

# Check upstream connectivity
curl http://127.0.0.1:3000/health

# See what headers NGINX is sending to backend
# In your Node.js app, log: console.log(req.headers)

# Watch access log for proxy errors
tail -f /var/log/nginx/app.example.com-access.log
# 502 = backend unreachable
# 504 = backend timed out
```

---

## Chapter 7: Load Balancing

### What You Will Learn
- Load balancing algorithms available in NGINX
- Defining upstream groups
- Health checks
- Session persistence (sticky sessions)
- Active vs passive health monitoring

### Why This Exists

Running a single application server is a single point of failure. If it crashes, your service is down. Load balancing distributes traffic across multiple instances, providing:
- **Fault tolerance**: If one server fails, others continue serving traffic
- **Horizontal scaling**: Add more servers to handle increased load
- **Rolling deployments**: Update one server at a time without downtime

### Load Balancing Algorithms

```nginx
# ─── Round Robin (default) ────────────────────────────────
# Requests distributed sequentially across servers
upstream backend_rr {
    server 10.0.0.1:3000;
    server 10.0.0.2:3000;
    server 10.0.0.3:3000;
}

# ─── Weighted Round Robin ─────────────────────────────────
# Server with weight=3 gets 3x more requests
# Use when servers have different capacities
upstream backend_weighted {
    server 10.0.0.1:3000 weight=3;
    server 10.0.0.2:3000 weight=2;
    server 10.0.0.3:3000 weight=1;
}

# ─── Least Connections ────────────────────────────────────
# Request goes to the server with fewest active connections
# Best for requests with variable processing time
upstream backend_least_conn {
    least_conn;
    server 10.0.0.1:3000;
    server 10.0.0.2:3000;
    server 10.0.0.3:3000;
}

# ─── IP Hash ──────────────────────────────────────────────
# Client IP determines server — same client always goes to same server
# Provides basic session persistence (sticky sessions)
upstream backend_iphash {
    ip_hash;
    server 10.0.0.1:3000;
    server 10.0.0.2:3000;
    server 10.0.0.3:3000;
}

# ─── Random ───────────────────────────────────────────────
# Random server selection (useful with many servers)
upstream backend_random {
    random;
    server 10.0.0.1:3000;
    server 10.0.0.2:3000;
    server 10.0.0.3:3000;
    server 10.0.0.4:3000;
}
```

### Upstream Server Parameters

```nginx
upstream backend {
    server 10.0.0.1:3000;                              # Default
    server 10.0.0.2:3000 weight=2;                     # 2x traffic
    server 10.0.0.3:3000 max_fails=3 fail_timeout=30s; # Health check
    server 10.0.0.4:3000 backup;                       # Only used if others fail
    server 10.0.0.5:3000 down;                         # Marked as unavailable

    # Connection keepalive to upstream
    keepalive 32;
    keepalive_requests 1000;
    keepalive_timeout 60s;
}
```

| Parameter | Description |
|-----------|-------------|
| `weight=N` | Relative weight (default 1) |
| `max_fails=N` | Failed requests before server is marked down |
| `fail_timeout=Ns` | How long to mark server as down |
| `backup` | Only used when all primary servers are down |
| `down` | Permanently mark server as unavailable |
| `keepalive N` | Number of idle keepalive connections per worker |

### Passive Health Checks

NGINX open-source uses **passive** health checks — it marks a server as down after a certain number of failed requests:

```nginx
upstream backend {
    server 10.0.0.1:3000 max_fails=3 fail_timeout=30s;
    server 10.0.0.2:3000 max_fails=3 fail_timeout=30s;
    server 10.0.0.3:3000 max_fails=3 fail_timeout=30s;
}
```

Meaning: If server `10.0.0.1:3000` fails 3 times within 30 seconds, NGINX stops sending it traffic for 30 seconds.

### Active Health Checks (NGINX Plus)

NGINX Plus supports active health checks that proactively test upstream servers:

```nginx
# NGINX Plus only
upstream backend {
    zone backend 64k;
    server 10.0.0.1:3000;
    server 10.0.0.2:3000;
    server 10.0.0.3:3000;
}

server {
    location / {
        proxy_pass http://backend;
        health_check interval=5s fails=3 passes=2 uri=/health;
    }
}
```

### Complete Load Balancer Configuration

```nginx
# /etc/nginx/conf.d/upstream.conf

upstream api_servers {
    least_conn;

    server 10.0.1.10:8080 weight=3 max_fails=3 fail_timeout=30s;
    server 10.0.1.11:8080 weight=3 max_fails=3 fail_timeout=30s;
    server 10.0.1.12:8080 weight=2 max_fails=3 fail_timeout=30s;
    server 10.0.1.13:8080 backup;

    keepalive 64;
    keepalive_requests 1000;
    keepalive_timeout 75s;
}

upstream web_servers {
    server 10.0.2.10:3000;
    server 10.0.2.11:3000;
    server 10.0.2.12:3000;

    keepalive 32;
}


# /etc/nginx/sites-available/lb.conf

server {
    listen 443 ssl http2;
    server_name example.com;

    ssl_certificate     /etc/nginx/ssl/example.com/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/example.com/privkey.pem;

    # API load balancing
    location /api/ {
        proxy_pass http://api_servers;
        include snippets/proxy-params.conf;

        # For long-running requests, increase timeout
        proxy_read_timeout 120s;
    }

    # Web app load balancing
    location / {
        proxy_pass http://web_servers;
        include snippets/proxy-params.conf;
    }

    access_log /var/log/nginx/lb-access.log;
    error_log  /var/log/nginx/lb-error.log;
}
```

### Load Balancing Diagram

```
         Clients (10,000 concurrent)
              │
              ▼
   ┌──────────────────────┐
   │   NGINX Load Balancer │
   │                      │
   │   Algorithm: least_conn
   └──────┬───────┬───────┘
          │       │       │
    weight=3  weight=3  weight=2
          │       │       │
          ▼       ▼       ▼
       ┌──────┐ ┌──────┐ ┌──────┐
       │ App1 │ │ App2 │ │ App3 │
       │:8080 │ │:8080 │ │:8080 │
       └──────┘ └──────┘ └──────┘
                               │
                      ┌────────┘ (backup — only if App1-3 fail)
                      ▼
                   ┌──────┐
                   │ App4 │
                   │:8080 │
                   └──────┘
```

### Debugging Section

```bash
# Check upstream health in real time (look for connection errors)
tail -f /var/log/nginx/error.log | grep upstream

# Common upstream errors:
# "connect() failed (111: Connection refused)" — backend down
# "upstream timed out (110: Connection timed out)" — backend too slow
# "no live upstreams while connecting to upstream" — all servers failed

# Test individual backends
curl http://10.0.0.1:3000/health
curl http://10.0.0.2:3000/health

# NGINX status page (shows active connections)
curl http://localhost/nginx_status
```

---

## Chapter 8: Caching

### What You Will Learn
- How NGINX proxy caching works
- Cache zones, keys, and TTLs
- Cache bypass and purging
- Microcaching (caching for 1 second) for high-traffic scenarios
- `Cache-Control` and `Expires` header interaction

### Why This Exists

Without caching, every client request hits your backend application server, which queries the database, processes data, and returns a response. This chain is slow and resource-intensive. NGINX caching stores backend responses on disk and serves subsequent identical requests directly — orders of magnitude faster, with zero backend load.

### How It Works Internally

```
First request (cache MISS):
  Client → NGINX → Backend → Response stored in cache → Client

Subsequent requests (cache HIT):
  Client → NGINX → Serve from cache (no backend contact)
```

NGINX uses a two-level directory structure for cache storage, with an in-memory hash table mapping cache keys to file paths.

### Cache Configuration

```nginx
# ── Step 1: Define cache zone in http {} context ─────────────

http {
    # proxy_cache_path: where to store cached files
    # levels: directory depth (reduces files per directory)
    # keys_zone: shared memory zone (name:size)
    # max_size: maximum total cache size on disk
    # inactive: remove if not accessed within this time
    # use_temp_path: write directly to cache dir (performance)

    proxy_cache_path /var/cache/nginx/api
        levels=1:2
        keys_zone=api_cache:10m
        max_size=1g
        inactive=60m
        use_temp_path=off;

    proxy_cache_path /var/cache/nginx/static
        levels=1:2
        keys_zone=static_cache:10m
        max_size=10g
        inactive=24h
        use_temp_path=off;
}
```

```nginx
# ── Step 2: Use cache in server/location blocks ────────────────

server {
    listen 80;
    server_name api.example.com;

    location /api/ {
        proxy_pass http://api_backend;
        include snippets/proxy-params.conf;

        # Enable caching with our defined zone
        proxy_cache api_cache;

        # Cache key — what makes a request unique
        proxy_cache_key "$scheme$request_method$host$request_uri";

        # Cache these HTTP methods
        proxy_cache_methods GET HEAD;

        # Cache TTL per status code
        proxy_cache_valid 200 302  10m;   # Cache 200/302 for 10 minutes
        proxy_cache_valid 404      1m;    # Cache 404 for 1 minute
        proxy_cache_valid any      0;     # Don't cache anything else

        # Serve stale cache if backend is down or slow
        proxy_cache_use_stale error timeout http_500 http_502 http_503 http_504;

        # How long to lock the cache key while fetching from backend
        proxy_cache_lock on;
        proxy_cache_lock_timeout 5s;

        # Add header to show cache status (HIT, MISS, BYPASS, EXPIRED)
        add_header X-Cache-Status $upstream_cache_status;

        # Bypass cache if client sends Cache-Control: no-cache
        proxy_cache_bypass $http_cache_control;
    }
}
```

### Cache Key Design

The cache key determines what makes two requests "the same" for caching purposes:

```nginx
# Default key — method + scheme + host + URI (no query params)
proxy_cache_key "$request_method$scheme$host$request_uri";

# Include query string (useful for APIs with filtering)
proxy_cache_key "$scheme$request_method$host$uri$args";

# Include Authorization header (user-specific caches)
proxy_cache_key "$scheme$request_method$host$request_uri$http_authorization";

# For logged-in vs anonymous users
map $http_cookie $cache_vary {
    "~*session_id" "1";  # Has session cookie
    default          "0"; # Anonymous user
}
proxy_cache_key "$scheme$request_method$host$request_uri$cache_vary";
```

### Microcaching

Microcaching stores responses for just 1 second. For high-traffic APIs receiving 10,000 req/s, even 1 second of caching reduces backend load by 99.99%:

```nginx
proxy_cache_valid 200 1s;   # Cache for 1 second only

# Combined with serve-stale for thundering herd protection:
proxy_cache_use_stale updating;    # Serve old cache while refreshing
proxy_cache_background_update on;  # Refresh in background
```

### Cache Bypass Conditions

```nginx
# Create a variable that controls cache bypass
# Bypass if: client sends no-cache, or request has auth header, or specific cookie
map $http_cache_control $bypass_cache {
    "no-cache"      1;
    default         0;
}

location /api/ {
    proxy_pass http://backend;
    proxy_cache api_cache;
    proxy_cache_valid 200 5m;

    # Bypass cache based on conditions
    proxy_cache_bypass $bypass_cache $http_authorization $cookie_session;
    proxy_no_cache     $bypass_cache $http_authorization $cookie_session;
}
```

### Cache Purging (NGINX Plus or with module)

```nginx
# NGINX Plus: purge by cache key
location ~ /purge(/.*) {
    # Only allow purge from internal IPs
    allow 10.0.0.0/8;
    deny  all;

    proxy_cache_purge api_cache "$scheme$request_method$host$1";
}
```

### Creating Cache Directories

```bash
# Create cache directories
sudo mkdir -p /var/cache/nginx/api
sudo mkdir -p /var/cache/nginx/static

# Set correct ownership
sudo chown nginx:nginx /var/cache/nginx -R

# Monitor cache size
du -sh /var/cache/nginx/
```

### Production Use Case

A news website with 500,000 daily visitors and 10 API servers saved 70% of API calls by implementing NGINX caching with 5-minute TTLs for article content. The backend load dropped from 120,000 req/min to 36,000 req/min overnight.

### Debugging Section

```bash
# Check cache hit rate
grep "X-Cache-Status" /var/log/nginx/access.log | awk '{print $NF}' | sort | uniq -c

# Test cache behavior
curl -I http://localhost/api/articles
# First request: X-Cache-Status: MISS
# Second request: X-Cache-Status: HIT

# Find cached files
ls -la /var/cache/nginx/api/

# Check cache zone usage (NGINX stub_status)
curl http://localhost/nginx_status
```

---

## Chapter 9: Gzip Compression

### What You Will Learn
- How gzip compression reduces bandwidth and improves performance
- Configuring gzip in NGINX
- Which content types to compress (and which not to)
- Brotli compression (modern alternative to gzip)
- Testing compression with `curl`

### Why This Exists

Uncompressed HTTP responses waste bandwidth. A 500KB JSON API response becomes ~50KB after gzip compression — a 10x reduction. For users on slow connections or mobile networks, this directly improves load time. Server CPU cost of compression is minimal compared to the bandwidth and latency savings.

### Configuration

```nginx
http {
    # Enable gzip compression
    gzip on;

    # Minimum response size to compress (don't compress tiny responses)
    gzip_min_length 256;

    # Compression level 1-9 (6 is a good balance of speed vs ratio)
    gzip_comp_level 6;

    # Compress responses for proxied requests too
    gzip_proxied any;

    # Add Vary: Accept-Encoding header (required for CDN compatibility)
    gzip_vary on;

    # Compress these MIME types
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/javascript
        application/x-javascript
        application/json
        application/xml
        application/xml+rss
        application/atom+xml
        application/rss+xml
        application/vnd.ms-fontobject
        application/x-font-ttf
        font/opentype
        image/svg+xml
        image/x-icon;

    # Disable gzip for IE6 (ancient browser with gzip bugs)
    gzip_disable "msie6";

    # Buffer size for gzip
    gzip_buffers 32 4k;

    # HTTP version for gzip (1.1 required for proxies)
    gzip_http_version 1.1;
}
```

### Static Gzip (Pre-compressed Files)

For maximum performance, pre-compress static files and have NGINX serve the `.gz` version:

```bash
# Pre-compress files
gzip -k -9 /var/www/html/app.js
# Creates app.js.gz alongside app.js
```

```nginx
# Serve pre-compressed files if available
location ~* \.(js|css|html|xml|json)$ {
    gzip_static on;   # Serve .gz file if it exists
    expires 1y;
    add_header Cache-Control "public, immutable";
}
```

### Testing Compression

```bash
# Test if response is compressed
curl -H "Accept-Encoding: gzip" -I http://localhost/app.js
# Look for: Content-Encoding: gzip

# Check actual compression ratio
curl -H "Accept-Encoding: gzip" -o /dev/null -s -w \
    "Size: %{size_download} bytes, Time: %{time_total}s\n" \
    http://localhost/app.js

# Decompress and view content
curl -H "Accept-Encoding: gzip" http://localhost/data.json | gunzip | jq
```

### What NOT to Compress

| Type | Reason |
|------|--------|
| `image/jpeg`, `image/png`, `image/webp` | Already compressed — gzip adds overhead |
| `video/mp4`, `audio/mpeg` | Already compressed media formats |
| `application/zip`, `application/gzip` | Already compressed archives |
| Small responses (<256 bytes) | Overhead exceeds savings |

---

## Chapter 10: SSL / HTTPS Setup

### What You Will Learn
- Obtaining SSL certificates with Let's Encrypt / Certbot
- Manual SSL configuration
- Modern TLS settings (TLS 1.2, TLS 1.3)
- HSTS, OCSP stapling
- Perfect Forward Secrecy
- HTTP/2 configuration

### Why This Exists

HTTPS is not optional. Without SSL/TLS:
- Passwords, session cookies, and sensitive data are transmitted in plaintext
- Attackers can intercept and modify traffic (man-in-the-middle)
- Modern browsers mark HTTP sites as "Not Secure"
- Search engines penalize unencrypted sites

NGINX handles SSL termination — it decrypts HTTPS traffic and forwards plain HTTP to your backends, so your applications don't need to handle SSL complexity.

### Let's Encrypt with Certbot (Recommended)

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx -y

# Obtain certificate (Certbot modifies nginx config automatically)
sudo certbot --nginx -d example.com -d www.example.com

# Obtain certificate without modifying nginx (manual setup)
sudo certbot certonly --nginx -d example.com -d www.example.com

# Test auto-renewal
sudo certbot renew --dry-run

# Cron job for auto-renewal (already added by certbot, verify it)
sudo crontab -l | grep certbot
# 0 12 * * * /usr/bin/certbot renew --quiet
```

### Complete SSL Server Configuration

```nginx
# /etc/nginx/snippets/ssl-params.conf
# Modern TLS configuration — secure and compatible

# TLS protocols: disable old/broken TLSv1.0, TLSv1.1
ssl_protocols TLSv1.2 TLSv1.3;

# Cipher suites — prioritize ECDHE for Perfect Forward Secrecy
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256;

# Server cipher preference
ssl_prefer_server_ciphers off;

# SSL session cache (reduces TLS handshake cost for repeat visitors)
ssl_session_cache   shared:SSL:10m;
ssl_session_timeout 1d;
ssl_session_tickets off;

# Diffie-Hellman parameters (pre-generated, prevents weak DH)
ssl_dhparam /etc/nginx/ssl/dhparam.pem;

# OCSP Stapling — embed certificate status in handshake
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4 1.1.1.1 valid=300s;
resolver_timeout 5s;

# HSTS — tell browsers to always use HTTPS (6 months)
add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload" always;
```

```nginx
# Generate DH parameters (do this once, takes a minute)
# sudo openssl dhparam -out /etc/nginx/ssl/dhparam.pem 2048
```

```nginx
# /etc/nginx/sites-available/ssl-example.conf

# Redirect HTTP → HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name example.com www.example.com;

    # Allow Let's Encrypt ACME challenges
    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    # Redirect everything else to HTTPS
    location / {
        return 301 https://$host$request_uri;
    }
}

# HTTPS Server
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name example.com www.example.com;

    # Certificate files
    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # Trusted certificate chain for OCSP stapling
    ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;

    # Include our TLS parameters
    include snippets/ssl-params.conf;

    # Include security headers
    include snippets/security-headers.conf;

    root /var/www/example.com;

    location / {
        try_files $uri $uri/ /index.html;
    }

    access_log /var/log/nginx/ssl-access.log;
    error_log  /var/log/nginx/ssl-error.log;
}
```

### HTTP/2

HTTP/2 is enabled by adding `http2` to the `listen` directive:

```nginx
listen 443 ssl http2;
```

HTTP/2 benefits:
- **Multiplexing**: Multiple requests over a single connection
- **Header compression**: HPACK reduces overhead
- **Server push**: Send resources before the client requests them
- **Binary protocol**: More efficient than HTTP/1.1 text

### Testing SSL Configuration

```bash
# Test SSL configuration
curl -I https://example.com

# Test with SSL Labs (online tool)
# https://www.ssllabs.com/ssltest/analyze.html?d=example.com

# Check supported protocols
openssl s_client -connect example.com:443 -tls1_2
openssl s_client -connect example.com:443 -tls1_3

# Check certificate expiry
echo | openssl s_client -connect example.com:443 2>/dev/null | \
    openssl x509 -noout -dates

# Check HSTS header
curl -I https://example.com | grep Strict-Transport-Security
```

---

# Level 3 — Advanced

---

## Chapter 11: Performance Tuning

### What You Will Learn
- OS-level optimizations for NGINX
- `worker_processes`, `worker_connections`, `worker_rlimit_nofile`
- TCP optimizations (`sendfile`, `tcp_nopush`, `tcp_nodelay`)
- Connection keepalive tuning
- Buffer sizes and timeout tuning
- Benchmarking with `wrk` and `ab`

### Why This Exists

A default NGINX installation is conservative. Production workloads require careful tuning of both NGINX and the underlying OS. Proper tuning can increase throughput by 5-10x on the same hardware.

### OS-Level Tuning

Before tuning NGINX, the OS must be optimized:

```bash
# /etc/sysctl.conf — Kernel network parameters

# Maximum number of open file descriptors
fs.file-max = 200000

# TCP connection settings
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.tcp_max_syn_backlog = 4096
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_tw_reuse = 1

# Increase TCP receive/send buffers
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216

# Increase listen backlog
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535

# Apply changes
sudo sysctl -p
```

```bash
# Increase file descriptor limits
# /etc/security/limits.conf
nginx soft nofile 65535
nginx hard nofile 65535
```

### Optimized nginx.conf

```nginx
# ────────────────────────────────────────────────────────────────
# PRODUCTION-TUNED nginx.conf
# ────────────────────────────────────────────────────────────────

# Number of worker processes = number of CPU cores
# 'auto' is almost always correct
worker_processes auto;

# Worker CPU affinity — bind workers to specific CPUs
# (manual, alternative to auto)
# worker_cpu_affinity auto;

# Maximum open files per worker process
# Should be >= worker_connections * 2
worker_rlimit_nofile 65535;

error_log /var/log/nginx/error.log warn;
pid       /run/nginx.pid;


events {
    # Max connections per worker
    # Total connections = worker_processes × worker_connections
    worker_connections 4096;

    # Accept multiple connections at once
    multi_accept on;

    # Best connection processing method for Linux
    use epoll;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    # ── Log Format ────────────────────────────────────────────
    log_format main '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent" '
                    'rt=$request_time uct=$upstream_connect_time '
                    'uht=$upstream_header_time urt=$upstream_response_time';

    access_log /var/log/nginx/access.log main buffer=32k flush=5s;
    error_log  /var/log/nginx/error.log warn;

    # ── File Handling ─────────────────────────────────────────
    sendfile           on;   # Kernel-level file sending
    tcp_nopush         on;   # Send full packets (batch headers + first chunk)
    tcp_nodelay        on;   # Send data immediately (for keepalive connections)
    aio                on;   # Asynchronous file I/O
    directio           4m;   # Use direct I/O for files > 4MB

    # ── Open File Cache ───────────────────────────────────────
    open_file_cache max=20000 inactive=60s;
    open_file_cache_valid 30s;
    open_file_cache_min_uses 2;
    open_file_cache_errors on;

    # ── Keepalive ─────────────────────────────────────────────
    keepalive_timeout  65;           # How long to keep connection open
    keepalive_requests 1000;         # Max requests per keepalive connection
    reset_timedout_connection on;    # Reset timed out connections

    # ── Request Timeouts ─────────────────────────────────────
    client_body_timeout   12s;       # Time to receive request body
    client_header_timeout 12s;       # Time to receive request headers
    send_timeout          10s;       # Time between send operations

    # ── Buffer Sizes ─────────────────────────────────────────
    client_body_buffer_size    128k;
    client_header_buffer_size  1k;
    client_max_body_size       50m;
    large_client_header_buffers 4 16k;

    # ── Proxy Buffers ─────────────────────────────────────────
    proxy_buffer_size          4k;
    proxy_buffers              8 4k;
    proxy_busy_buffers_size    8k;
    proxy_temp_file_write_size 8k;

    # ── Hash Table Sizes ──────────────────────────────────────
    types_hash_max_size        2048;
    server_names_hash_max_size 512;
    server_names_hash_bucket_size 64;
    variables_hash_max_size    1024;

    # ── Hide Version ─────────────────────────────────────────
    server_tokens off;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

### Benchmarking

```bash
# Install wrk (HTTP benchmarking tool)
sudo apt install wrk -y

# Basic benchmark: 100 concurrent connections, 30 seconds, 4 threads
wrk -t4 -c100 -d30s http://localhost/

# Benchmark with Lua script (custom headers, POST body)
wrk -t4 -c100 -d30s -s post.lua http://localhost/api/endpoint

# Apache Bench (simpler alternative)
ab -n 10000 -c 100 http://localhost/

# Interpreting results:
# Requests/sec — throughput
# Latency (avg/p99) — response time percentiles
# Transfer/sec — bandwidth
```

### Performance Metrics to Target

| Metric | Baseline NGINX | Tuned NGINX |
|--------|---------------|-------------|
| Static file req/s | 10,000 | 80,000+ |
| Concurrent connections | 1,024 | 65,535+ |
| Memory per 10K connections | 150MB | 25MB |
| Proxy req/s | 5,000 | 40,000+ |

---

## Chapter 12: Rate Limiting

### What You Will Learn
- The `limit_req` module for request rate limiting
- The `limit_conn` module for connection limiting
- Burst handling and `nodelay`
- Per-IP, per-user, per-endpoint rate limits
- Rate limit logging and monitoring

### Why This Exists

Without rate limiting:
- A single malicious client can exhaust your server with millions of requests per second
- Scrapers can hammer your API and download your entire database
- Brute force attacks on login endpoints go unchecked
- DDoS attacks are amplified

Rate limiting protects your application by allowing only a defined number of requests per time window.

### How It Works Internally

NGINX implements rate limiting using a **leaky bucket** algorithm:
- Requests arrive at variable rates
- They're processed at a fixed rate
- Excess requests that don't fit in the burst queue are rejected (429)

### Configuration

```nginx
http {
    # ── Define Rate Limit Zones ───────────────────────────────
    # Format: limit_req_zone KEY zone=NAME:SIZE rate=RATE;

    # Limit by client IP — 10 requests per second per IP
    limit_req_zone $binary_remote_addr zone=per_ip:10m rate=10r/s;

    # Limit login attempts — 5 per minute per IP
    limit_req_zone $binary_remote_addr zone=login_limit:10m rate=5r/m;

    # Limit by API key — 100 requests per second per key
    limit_req_zone $http_x_api_key zone=per_api_key:10m rate=100r/s;

    # ── Define Connection Limit Zones ─────────────────────────
    # Limit simultaneous connections per IP
    limit_conn_zone $binary_remote_addr zone=conn_per_ip:10m;

    # Global status code for rate-limited requests
    limit_req_status  429;
    limit_conn_status 429;

    # Log rate limiting events
    limit_req_log_level warn;
    limit_conn_log_level warn;
}
```

```nginx
server {
    listen 80;
    server_name api.example.com;

    # ── Global connection limit ──────────────────────────────
    # Max 50 simultaneous connections per IP
    limit_conn conn_per_ip 50;

    # ── Login endpoint — strict limiting ─────────────────────
    location /auth/login {
        proxy_pass http://backend;
        include snippets/proxy-params.conf;

        # 5 requests per minute, no burst allowed
        limit_req zone=login_limit nodelay;

        # Return custom JSON error for rate limited requests
        limit_req_status 429;
    }

    # ── API endpoints — moderate limiting ────────────────────
    location /api/ {
        proxy_pass http://backend;
        include snippets/proxy-params.conf;

        # 10 req/s with burst of 20
        # burst=20: allow up to 20 extra requests in queue
        # nodelay: process burst immediately (don't add delay)
        limit_req zone=per_ip burst=20 nodelay;
    }

    # ── Search endpoint — expensive, strict ─────────────────
    location /api/search {
        proxy_pass http://backend;
        include snippets/proxy-params.conf;

        # 2 req/s, burst of 5
        limit_req zone=per_ip burst=5 nodelay;
    }

    # ── Custom 429 response page ─────────────────────────────
    error_page 429 /rate_limit.json;
    location = /rate_limit.json {
        internal;
        return 429 '{"error": "Too Many Requests", "message": "Slow down, please."}';
        add_header Content-Type application/json;
        add_header Retry-After 60;
    }
}
```

### Rate Limiting with Whitelisting

```nginx
# Whitelist trusted IPs from rate limiting
geo $limit_key {
    default         $binary_remote_addr;
    10.0.0.0/8      "";    # Internal network — no limit
    192.168.0.0/16  "";    # Private network — no limit
    1.2.3.4         "";    # Trusted partner IP — no limit
}

limit_req_zone $limit_key zone=api_limit:10m rate=10r/s;

location /api/ {
    # $limit_key is "" for whitelisted IPs
    # NGINX skips rate limiting when the key is empty
    limit_req zone=api_limit burst=20 nodelay;
}
```

### Real-World Rate Limit Strategy

```nginx
# Different limits for different tiers
map $http_x_api_tier $api_rate {
    "free"       "5r/s";
    "pro"        "50r/s";
    "enterprise" "500r/s";
    default      "1r/s";
}
```

---

## Chapter 13: Security Hardening

### What You Will Learn
- Security headers (CSP, X-Frame-Options, HSTS)
- Hiding NGINX version and server information
- Blocking common attack patterns
- Protection against Clickjacking, XSS, MIME sniffing
- IP-based access control
- Request filtering

### Why This Exists

An NGINX server exposed to the internet is constantly probed by automated scanners, botnets, and attackers. Security hardening reduces attack surface and protects against common web vulnerabilities.

### Security Headers

```nginx
# /etc/nginx/snippets/security-headers.conf

# Remove NGINX version from headers
server_tokens off;

# Prevent clickjacking — don't allow site to be embedded in iframes
add_header X-Frame-Options "SAMEORIGIN" always;

# XSS Protection (legacy browsers)
add_header X-XSS-Protection "1; mode=block" always;

# Prevent MIME type sniffing
add_header X-Content-Type-Options "nosniff" always;

# Control referrer information
add_header Referrer-Policy "strict-origin-when-cross-origin" always;

# HSTS — force HTTPS for 6 months
add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload" always;

# Content Security Policy (adjust for your app)
add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' data:; connect-src 'self'; frame-ancestors 'none';" always;

# Permissions Policy (restrict browser features)
add_header Permissions-Policy "geolocation=(), microphone=(), camera=(), payment=(), usb=(), accelerometer=(), gyroscope=()" always;

# Remove X-Powered-By if NGINX is adding it
more_clear_headers X-Powered-By;
```

### Request Filtering

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;

    # ── Block Common Attack Patterns ─────────────────────────

    # Block requests with no User-Agent
    if ($http_user_agent = "") {
        return 403;
    }

    # Block known bad bots (extend this list)
    if ($http_user_agent ~* "(masscan|nikto|sqlmap|nmap|acunetix|openvas|burpsuite)") {
        return 403;
    }

    # Block requests with invalid HTTP methods
    if ($request_method !~ ^(GET|HEAD|POST|PUT|DELETE|OPTIONS|PATCH)$) {
        return 405;
    }

    # Block SQL injection patterns in query strings
    if ($query_string ~* "union.*select|insert.*into|drop.*table|update.*set") {
        return 403;
    }

    # Block path traversal
    if ($request_uri ~* "\.\./") {
        return 403;
    }

    # ── IP Access Control ────────────────────────────────────

    # Admin panel — only allow from specific IPs
    location /admin/ {
        allow 10.0.0.0/8;
        allow 192.168.0.0/16;
        allow 1.2.3.4;      # Your office IP
        deny  all;

        proxy_pass http://admin_backend;
        include snippets/proxy-params.conf;
    }

    # ── File Upload Security ─────────────────────────────────
    location /upload {
        client_max_body_size 10m;          # Limit upload size

        # Only allow specific MIME types (application must also validate)
        if ($content_type !~ "multipart/form-data") {
            return 415;
        }

        proxy_pass http://upload_backend;
        include snippets/proxy-params.conf;
    }

    # ── Block Sensitive Files ────────────────────────────────
    location ~* \.(env|git|svn|htaccess|htpasswd|log|conf|bak|sql|sh)$ {
        deny all;
        return 404;
    }

    # Block .git directory
    location ~ /\.git {
        deny all;
        return 404;
    }

    # ── Limit Request Body Size Globally ────────────────────
    client_max_body_size 10m;

    # Limit request header size
    large_client_header_buffers 4 16k;
    client_header_buffer_size 1k;
}
```

### DDoS Protection Basics

```nginx
http {
    # ── Slow loris protection ─────────────────────────────────
    # Timeout slow connections that don't send data
    client_body_timeout   10s;
    client_header_timeout 10s;
    send_timeout          10s;

    # ── Connection limit per IP ───────────────────────────────
    limit_conn_zone $binary_remote_addr zone=conn_limit:10m;

    # ── Request rate limit ────────────────────────────────────
    limit_req_zone $binary_remote_addr zone=req_limit:10m rate=30r/s;

    server {
        limit_conn conn_limit 20;           # Max 20 simultaneous connections/IP
        limit_req zone=req_limit burst=50 nodelay;

        # ── Block countries (requires ngx_http_geoip_module) ──
        # geoip_country /usr/share/GeoIP/GeoIP.dat;
        # if ($geoip_country_code = "CN") { return 403; }
    }
}
```

---

## Chapter 14: Logging Deep Dive

### What You Will Learn
- `access.log` vs `error.log` — structure and purpose
- Custom log formats
- Conditional logging
- Log rotation
- Structured JSON logging for log aggregators (ELK, Splunk)
- Log analysis with `awk` and `goaccess`

### Log Types

```nginx
# Error log — records NGINX errors, including upstream failures
error_log /var/log/nginx/error.log warn;
# Levels: debug, info, notice, warn, error, crit, alert, emerg

# Access log — records every HTTP request
access_log /var/log/nginx/access.log main;
```

### Custom Log Format

```nginx
http {
    # ── Standard Format ──────────────────────────────────────
    log_format main
        '$remote_addr - $remote_user [$time_local] '
        '"$request" $status $body_bytes_sent '
        '"$http_referer" "$http_user_agent"';

    # ── Extended Format with Timing ──────────────────────────
    log_format extended
        '$remote_addr - $remote_user [$time_local] '
        '"$request" $status $body_bytes_sent '
        '"$http_referer" "$http_user_agent" '
        'rt=$request_time '                     # Total request time
        'uct=$upstream_connect_time '           # Time to connect to upstream
        'uht=$upstream_header_time '            # Time to receive upstream headers
        'urt=$upstream_response_time '          # Time to receive upstream body
        'cs=$upstream_cache_status';            # Cache: HIT/MISS/BYPASS

    # ── JSON Format (for ELK/Datadog/Splunk) ─────────────────
    log_format json_combined escape=json
        '{'
            '"time":"$time_iso8601",'
            '"remote_addr":"$remote_addr",'
            '"request_id":"$request_id",'
            '"remote_user":"$remote_user",'
            '"request":"$request",'
            '"status":$status,'
            '"body_bytes_sent":$body_bytes_sent,'
            '"request_time":$request_time,'
            '"http_referrer":"$http_referer",'
            '"http_user_agent":"$http_user_agent",'
            '"upstream_addr":"$upstream_addr",'
            '"upstream_status":"$upstream_status",'
            '"upstream_response_time":"$upstream_response_time",'
            '"cache_status":"$upstream_cache_status",'
            '"ssl_protocol":"$ssl_protocol",'
            '"ssl_cipher":"$ssl_cipher"'
        '}';

    # Use different formats per server
    server {
        access_log /var/log/nginx/app-access.log json_combined;
        access_log /var/log/nginx/app-timing.log extended;
    }
}
```

### Conditional Logging

```nginx
# Skip logging for health checks (reduces log noise)
map $request_uri $loggable {
    ~^/health  0;   # Don't log /health
    ~^/ping    0;   # Don't log /ping
    default    1;   # Log everything else
}

server {
    access_log /var/log/nginx/access.log main if=$loggable;
}
```

### Log Rotation

```bash
# /etc/logrotate.d/nginx
/var/log/nginx/*.log {
    daily                       # Rotate daily
    missingok                   # Don't error if log doesn't exist
    rotate 52                   # Keep 52 rotated files (1 year)
    compress                    # Compress old logs with gzip
    delaycompress               # Don't compress last rotation
    notifempty                  # Don't rotate empty files
    create 640 nginx adm        # Create new log with these permissions
    sharedscripts
    postrotate
        # Signal NGINX to reopen log files (zero-downtime)
        if [ -f /run/nginx.pid ]; then
            kill -USR1 `cat /run/nginx.pid`
        fi
    endscript
}
```

### Log Analysis

```bash
# Most frequent IP addresses
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head 20

# Top requested URLs
awk '{print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head 20

# Response status code distribution
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn

# Average response time
awk -F'rt=' '{sum+=$2; n++} END {print "Avg:", sum/n, "s"}' /var/log/nginx/access.log

# Requests per hour
awk '{print $4}' /var/log/nginx/access.log | cut -d: -f1,2 | sort | uniq -c

# Real-time dashboard with GoAccess
sudo apt install goaccess -y
goaccess /var/log/nginx/access.log --log-format=COMBINED --real-time-html -o /var/www/html/report.html
```

---

# Level 4 — Production

---

## Chapter 15: High Availability Setup

### What You Will Learn
- Single point of failure elimination
- Active-Passive and Active-Active HA patterns
- Keepalived + NGINX for automatic failover
- Virtual IP (VIP) failover
- Zero-downtime deployments

### Architecture

```
Active-Passive HA:

    ┌─────────────────────────────────────────────┐
    │       Virtual IP: 10.0.0.100 (floating)     │
    │         (moves to healthy server)            │
    └───────────────────┬─────────────────────────┘
                        │
         ┌──────────────┴──────────────┐
         │                             │
         ▼                             ▼
 ┌───────────────┐             ┌───────────────┐
 │   NGINX-1     │             │   NGINX-2     │
 │   (MASTER)    │◄──VRRP──────│   (BACKUP)    │
 │   10.0.0.101  │   heartbeat │   10.0.0.102  │
 └───────┬───────┘             └───────────────┘
         │                     (takes over VIP if
         │                      NGINX-1 fails)
         ▼
 ┌───────────────────────────────────┐
 │          Backend Servers          │
 │  App1:3000  App2:3000  App3:3000  │
 └───────────────────────────────────┘
```

### Keepalived Configuration

```bash
# Install Keepalived on both servers
sudo apt install keepalived -y
```

```bash
# /etc/keepalived/keepalived.conf — MASTER server

global_defs {
    router_id NGINX_MASTER
    script_user root
    enable_script_security
}

# Check if NGINX is running
vrrp_script check_nginx {
    script "/usr/bin/systemctl is-active nginx"
    interval 2      # Check every 2 seconds
    weight  -20     # Subtract 20 from priority if check fails
    fall    2       # 2 failures = fail
    rise    2       # 2 successes = recover
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100        # Higher = preferred

    advert_int 1        # VRRP advertisement interval (seconds)

    authentication {
        auth_type PASS
        auth_pass secret123
    }

    virtual_ipaddress {
        10.0.0.100/24   # The floating Virtual IP
    }

    track_script {
        check_nginx
    }
}
```

```bash
# /etc/keepalived/keepalived.conf — BACKUP server
# (identical, except state and priority)

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 90         # Lower = secondary

    # ... rest same as MASTER
}
```

### Zero-Downtime NGINX Reload

```bash
# NGINX reload — sends SIGHUP to master process
# Master reads new config, starts new workers, old workers finish existing requests
sudo systemctl reload nginx

# Or directly:
sudo kill -HUP $(cat /run/nginx.pid)

# Graceful shutdown — finish current requests then stop
sudo kill -QUIT $(cat /run/nginx.pid)

# Upgrade NGINX binary without downtime (advanced):
# 1. Install new nginx binary
# 2. Send USR2 to master (starts new master with new binary)
# 3. Send WINCH to old master (gracefully stop old workers)
# 4. Send QUIT to old master (stop old master)
```

---

## Chapter 16: Load Balancing Strategies

### What You Will Learn
- Choosing the right algorithm for your workload
- Session persistence strategies
- Blue-Green deployments with NGINX
- Canary releases
- A/B testing with NGINX

### Blue-Green Deployment

```nginx
# Switch all traffic between blue and green with a config change + reload

upstream blue {
    server 10.0.1.10:3000;
    server 10.0.1.11:3000;
}

upstream green {
    server 10.0.2.10:3000;
    server 10.0.2.11:3000;
}

server {
    listen 80;
    server_name example.com;

    # To switch from blue to green:
    # Change 'blue' to 'green' and reload
    location / {
        proxy_pass http://blue;
        include snippets/proxy-params.conf;
    }
}
```

### Canary Release (Gradual Rollout)

```nginx
# Send 10% of traffic to canary (new version)

upstream stable {
    server 10.0.1.10:3000 weight=9;  # 90%
    server 10.0.1.11:3000 weight=9;  # 90%
}

upstream canary {
    server 10.0.2.10:3000 weight=1;  # 10%
}

# Use split_clients for precise percentage splits
split_clients "$remote_addr$http_user_agent" $app_upstream {
    10%     "canary";        # 10% → canary
    *       "stable";        # 90% → stable
}

server {
    location / {
        proxy_pass http://$app_upstream;
        include snippets/proxy-params.conf;
    }
}
```

### A/B Testing

```nginx
# A/B test based on a cookie
map $cookie_ab_test $upstream_backend {
    "group_b"  "backend_b";
    default    "backend_a";
}

server {
    location / {
        proxy_pass http://$upstream_backend;
        include snippets/proxy-params.conf;

        # Set A/B test cookie for new visitors (50/50 split)
        add_header Set-Cookie "ab_test=$ab_group; Path=/; Max-Age=86400";
    }
}

# Randomly assign group
split_clients "$remote_addr" $ab_group {
    50%  "group_b";
    *    "group_a";
}
```

---

## Chapter 17: NGINX with Docker

### What You Will Learn
- Running NGINX in Docker
- NGINX as a Docker reverse proxy
- Docker Compose with NGINX
- NGINX with Docker Swarm
- Custom NGINX Docker images
- Automatic SSL with NGINX + Let's Encrypt in Docker

### Basic Docker Setup

```dockerfile
# Dockerfile — Custom NGINX image
FROM nginx:1.24-alpine

# Remove default config
RUN rm /etc/nginx/conf.d/default.conf

# Copy custom configuration
COPY nginx.conf /etc/nginx/nginx.conf
COPY sites/ /etc/nginx/sites-available/
COPY snippets/ /etc/nginx/snippets/

# Copy SSL certificates (better: mount at runtime)
# COPY ssl/ /etc/nginx/ssl/

# Create cache directories
RUN mkdir -p /var/cache/nginx/api /var/cache/nginx/static \
    && chown -R nginx:nginx /var/cache/nginx

# Test configuration
RUN nginx -t

EXPOSE 80 443

CMD ["nginx", "-g", "daemon off;"]
```

### Docker Compose with NGINX

```yaml
# docker-compose.yml — Full application stack

version: '3.8'

services:
  # ── NGINX ─────────────────────────────────────────────────
  nginx:
    image: nginx:1.24-alpine
    container_name: nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/sites:/etc/nginx/sites-enabled:ro
      - ./nginx/snippets:/etc/nginx/snippets:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
      - ./nginx/logs:/var/log/nginx
      - static_files:/var/www/static:ro
    depends_on:
      - api
      - web
    networks:
      - frontend
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "nginx", "-t"]
      interval: 30s
      timeout: 10s
      retries: 3

  # ── Node.js API ────────────────────────────────────────────
  api:
    build: ./api
    container_name: api
    expose:
      - "3000"   # Expose to Docker network only (not host)
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://postgres:secret@db:5432/app
    depends_on:
      - db
      - redis
    networks:
      - backend
    restart: unless-stopped
    deploy:
      replicas: 3    # Run 3 instances

  # ── React Web App ─────────────────────────────────────────
  web:
    build:
      context: ./web
      target: production
    container_name: web
    expose:
      - "8080"
    networks:
      - backend
    restart: unless-stopped

  # ── PostgreSQL ─────────────────────────────────────────────
  db:
    image: postgres:15-alpine
    container_name: db
    environment:
      - POSTGRES_DB=app
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=secret
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - backend
    restart: unless-stopped

  # ── Redis ─────────────────────────────────────────────────
  redis:
    image: redis:7-alpine
    container_name: redis
    command: redis-server --appendonly yes
    volumes:
      - redisdata:/data
    networks:
      - backend
    restart: unless-stopped

volumes:
  pgdata:
  redisdata:
  static_files:

networks:
  frontend:
  backend:
    internal: true   # Backend network not accessible from outside Docker
```

```nginx
# nginx/sites/app.conf — NGINX config for Docker Compose

# Reference Docker services by their service name
upstream api_backend {
    server api:3000;
    keepalive 32;
}

upstream web_frontend {
    server web:8080;
    keepalive 16;
}

server {
    listen 80;
    server_name example.com www.example.com;

    # Let's Encrypt ACME challenge
    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com www.example.com;

    ssl_certificate     /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;
    include snippets/ssl-params.conf;

    # ── API ────────────────────────────────────────────────
    location /api/ {
        proxy_pass http://api_backend;
        include snippets/proxy-params.conf;
        limit_req zone=api_limit burst=50 nodelay;
    }

    # ── WebSocket ──────────────────────────────────────────
    location /ws {
        proxy_pass http://api_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade    $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host       $host;
    }

    # ── Web App ────────────────────────────────────────────
    location / {
        proxy_pass http://web_frontend;
        include snippets/proxy-params.conf;
    }

    access_log /var/log/nginx/app-access.log;
    error_log  /var/log/nginx/app-error.log;
}
```

### Auto-SSL with Certbot in Docker

```yaml
# Add to docker-compose.yml for automatic Let's Encrypt

services:
  certbot:
    image: certbot/certbot
    volumes:
      - ./certbot/conf:/etc/letsencrypt
      - ./certbot/www:/var/www/certbot
    entrypoint: >
      /bin/sh -c 'trap exit TERM;
      while :; do
        certbot renew;
        sleep 12h & wait $${!};
      done;'

  nginx:
    volumes:
      - ./certbot/conf:/etc/letsencrypt:ro
      - ./certbot/www:/var/www/certbot:ro
```

```bash
# Initial certificate generation
docker compose run --rm certbot certonly --webroot \
    -w /var/www/certbot \
    -d example.com \
    -d www.example.com \
    --email admin@example.com \
    --agree-tos \
    --no-eff-email
```

---

## Chapter 18: CDN Integration

### What You Will Learn
- How CDNs work with NGINX
- Setting correct cache headers for CDN caching
- Handling CDN-specific headers (`CF-Connecting-IP`, `X-Forwarded-For`)
- Cache purging through CDN APIs
- Multi-CDN failover

### Architecture

```
User (Tokyo)                   User (London)
     │                              │
     ▼                              ▼
 CDN PoP (Tokyo)               CDN PoP (London)
 (serves cached content)       (serves cached content)
     │                              │
     └─────────────────┬────────────┘
                       │ Cache MISS only
                       ▼
             ┌─────────────────┐
             │  Origin NGINX   │
             │  (your server)  │
             └────────┬────────┘
                      │
                      ▼
              ┌──────────────┐
              │   Backend    │
              └──────────────┘
```

### Cache Headers for CDN

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;

    # ── Static Assets — Cache at CDN for 1 year ─────────────
    location ~* \.(js|css|woff2|png|jpg|webp|svg)$ {
        root /var/www/html;

        # Tell CDN to cache for 1 year
        expires 1y;
        add_header Cache-Control "public, max-age=31536000, immutable";

        # CDN uses Vary to cache different versions for different encodings
        add_header Vary "Accept-Encoding";

        # Allow CDN to cache this
        add_header CDN-Cache-Control "max-age=31536000";
    }

    # ── API Responses — Don't cache at CDN ──────────────────
    location /api/ {
        proxy_pass http://backend;
        include snippets/proxy-params.conf;

        # Don't cache API responses at CDN
        add_header Cache-Control "no-store, no-cache, must-revalidate";
        add_header CDN-Cache-Control "no-store";
    }

    # ── HTML Pages — Short CDN cache ─────────────────────────
    location / {
        proxy_pass http://backend;
        include snippets/proxy-params.conf;

        # Cache HTML at CDN for 5 minutes
        add_header Cache-Control "public, max-age=300, s-maxage=300";
    }
}
```

### Cloudflare Integration

```nginx
# When behind Cloudflare, the real client IP comes from CF-Connecting-IP
# Tell NGINX to trust Cloudflare's IP ranges

set_real_ip_from 103.21.244.0/22;
set_real_ip_from 103.22.200.0/22;
set_real_ip_from 103.31.4.0/22;
set_real_ip_from 104.16.0.0/13;
set_real_ip_from 104.24.0.0/14;
set_real_ip_from 108.162.192.0/18;
set_real_ip_from 131.0.72.0/22;
set_real_ip_from 141.101.64.0/18;
set_real_ip_from 162.158.0.0/15;
set_real_ip_from 172.64.0.0/13;
set_real_ip_from 173.245.48.0/20;
set_real_ip_from 188.114.96.0/20;
set_real_ip_from 190.93.240.0/20;
set_real_ip_from 197.234.240.0/22;
set_real_ip_from 198.41.128.0/17;
set_real_ip_from 2400:cb00::/32;

# Use CF-Connecting-IP header for real client IP
real_ip_header CF-Connecting-IP;
```

---

## Chapter 19: Monitoring

### What You Will Learn
- NGINX stub_status module
- Integration with Prometheus (nginx-prometheus-exporter)
- Grafana dashboards for NGINX
- Alerting on key metrics
- Real-time monitoring with GoAccess

### NGINX Stub Status

```nginx
# Enable the built-in status endpoint
server {
    listen 127.0.0.1:8080;  # Bind to localhost only

    location /nginx_status {
        stub_status on;
        allow 127.0.0.1;
        allow 10.0.0.0/8;
        deny all;
    }
}
```

```bash
# Sample output:
curl http://127.0.0.1:8080/nginx_status

Active connections: 291
server accepts handled requests
 16630948 16630948 31070465
Reading: 6 Writing: 179 Waiting: 106

# Active connections: Current open connections
# accepts: Total accepted connections
# handled: Successfully handled connections (should equal accepts)
# requests: Total HTTP requests
# Reading: Connections reading request headers
# Writing: Connections writing response
# Waiting: Idle keepalive connections
```

### Prometheus Integration

```bash
# Install nginx-prometheus-exporter
docker run -d \
    --name nginx-exporter \
    -p 9113:9113 \
    nginx/nginx-prometheus-exporter:latest \
    -nginx.scrape-uri=http://nginx:8080/nginx_status
```

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'nginx'
    static_configs:
      - targets: ['nginx-exporter:9113']
    scrape_interval: 15s
```

### Key Metrics to Alert On

| Metric | Alert Threshold | Meaning |
|--------|----------------|---------|
| `nginx_connections_active` | > 80% of max | Connection saturation |
| `nginx_http_requests_total` | Sudden drop | Possible outage |
| HTTP 5xx rate | > 1% | Backend errors |
| `upstream_response_time` | > 2s avg | Slow backends |
| `nginx_connections_waiting` | > 75% of active | Keepalive saturation |

---

# Level 5 — Expert

---

## Chapter 20: NGINX Internals

### What You Will Learn
- NGINX architecture deep dive
- How configuration is parsed and stored
- Module system (static and dynamic)
- Request lifecycle phases
- NGINX as a state machine

### The 11 Request Processing Phases

NGINX processes every HTTP request through exactly 11 ordered phases. Modules register handlers for specific phases:

| Phase | Purpose | Example Handlers |
|-------|---------|-----------------|
| `NGX_HTTP_POST_READ_PHASE` | Read request | `realip` module |
| `NGX_HTTP_SERVER_REWRITE_PHASE` | Server-level rewrites | `rewrite` module |
| `NGX_HTTP_FIND_CONFIG_PHASE` | Find matching `location` | Core |
| `NGX_HTTP_REWRITE_PHASE` | Location-level rewrites | `rewrite` module |
| `NGX_HTTP_POST_REWRITE_PHASE` | Post-rewrite processing | Core |
| `NGX_HTTP_PREACCESS_PHASE` | Pre-access (rate limiting) | `limit_req`, `limit_conn` |
| `NGX_HTTP_ACCESS_PHASE` | Access control | `access`, `auth_basic` |
| `NGX_HTTP_POST_ACCESS_PHASE` | Post-access | Core |
| `NGX_HTTP_TRY_FILES_PHASE` | `try_files` handling | Core |
| `NGX_HTTP_CONTENT_PHASE` | Generate response | `proxy`, `fastcgi`, `static` |
| `NGX_HTTP_LOG_PHASE` | Logging | `log` module |

### NGINX Memory Architecture

```
┌─────────────────────────────────────────────────────┐
│                 NGINX Memory Model                  │
│                                                     │
│  ┌──────────────────────────────────────────────┐   │
│  │            Master Process Memory             │   │
│  │  - Config (shared with workers)              │   │
│  │  - Shared memory zones (cache, rate limits)  │   │
│  └──────────────────────────────────────────────┘   │
│                                                     │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐        │
│  │  Worker 1 │  │  Worker 2 │  │  Worker 3 │        │
│  │           │  │           │  │           │        │
│  │ ┌───────┐ │  │ ┌───────┐ │  │ ┌───────┐ │        │
│  │ │Conn   │ │  │ │Conn   │ │  │ │Conn   │ │        │
│  │ │Pool   │ │  │ │Pool   │ │  │ │Pool   │ │        │
│  │ └───────┘ │  │ └───────┘ │  │ └───────┘ │        │
│  │           │  │           │  │           │        │
│  │ ┌───────┐ │  │ ┌───────┐ │  │ ┌───────┐ │        │
│  │ │Request│ │  │ │Request│ │  │ │Request│ │        │
│  │ │Pool   │ │  │ │Pool   │ │  │ │Pool   │ │        │
│  │ └───────┘ │  │ └───────┘ │  │ └───────┘ │        │
│  └───────────┘  └───────────┘  └───────────┘        │
│                                                     │
│  ┌──────────────────────────────────────────────┐   │
│  │          Shared Memory Zones (all workers)   │   │
│  │  - proxy_cache (disk-backed)                 │   │
│  │  - limit_req zones                           │   │
│  │  - limit_conn zones                          │   │
│  │  - upstream keepalive                        │   │
│  └──────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

### Dynamic Modules

```bash
# List loaded modules
nginx -V 2>&1 | grep -o -- '--with-[^ ]*' | sort

# Load a dynamic module
cat /etc/nginx/modules-enabled/50-mod-http-geoip2.conf
# load_module modules/ngx_http_geoip2_module.so;

# In nginx.conf (main context):
load_module modules/ngx_http_geoip2_module.so;
```

---

## Chapter 21: Worker Processes

### What You Will Learn
- Worker process lifecycle
- CPU affinity optimization
- Worker process signals
- Connection handling internals

### Worker Process Configuration

```nginx
# Optimal worker settings for production

# Match CPU count (auto is almost always best)
worker_processes auto;

# Bind workers to specific CPUs (manual)
# 4 CPUs:
# worker_cpu_affinity 0001 0010 0100 1000;

# Or auto-assign:
worker_cpu_affinity auto;

# Maximum open file descriptors per worker
worker_rlimit_nofile 65535;

# Priority of worker processes (-20 to 20, -20 = highest)
worker_priority -1;

events {
    worker_connections 8192;
    multi_accept on;
    use epoll;
}
```

### Process Signals

```bash
# NGINX master process accepts these signals:
# TERM, INT — fast shutdown (drop active connections)
# QUIT      — graceful shutdown (finish active connections)
# HUP       — reload configuration
# USR1      — reopen log files
# USR2      — upgrade executable file
# WINCH     — gracefully stop workers (used during upgrade)

# Send signals:
sudo kill -HUP  $(cat /run/nginx.pid)   # Reload config
sudo kill -USR1 $(cat /run/nginx.pid)   # Reopen logs
sudo kill -QUIT $(cat /run/nginx.pid)   # Graceful stop
```

---

## Chapter 22: Event Loop

### What You Will Learn
- How NGINX's event loop works
- `epoll` vs `select` vs `kqueue`
- Non-blocking I/O internals
- Timer management

### How the Event Loop Works

```
NGINX Worker Event Loop:

1. Initialize epoll file descriptor
2. Register listening socket for READ events
3. Loop:
   a. epoll_wait() — block until any event occurs
   b. For each event:
      - If listen socket: accept() new connection
        - Add connection socket to epoll
      - If client data: read() request
        - Process through phases
        - If proxying: connect to upstream, add to epoll
        - Write() response when ready
      - If timeout: close stale connection
4. Never block — everything is asynchronous
```

```nginx
events {
    # epoll — Linux 2.6+ (best for Linux production)
    use epoll;

    # kqueue — BSD/macOS
    # use kqueue;

    # select — fallback (least efficient)
    # use select;

    worker_connections 4096;
    multi_accept on;    # Accept all pending connections at once
}
```

---

## Chapter 23: Memory Management

### What You Will Learn
- NGINX memory pools
- Request-scoped vs connection-scoped allocations
- Shared memory zones for inter-process communication
- Memory tuning

### NGINX Memory Pools

NGINX uses memory pools to avoid fragmentation and enable fast allocation/deallocation:

- **Connection pool**: Lives for the duration of a connection. Contains connection state, SSL context, etc.
- **Request pool**: Lives for the duration of a single HTTP request. Contains headers, variables, buffers.
- When the request ends, the entire pool is freed in O(1) — no per-object deallocation.

### Shared Memory Configuration

```nginx
http {
    # Rate limiting zones (shared between workers)
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    # 10m = 10 megabytes = ~80,000 IP entries

    # Proxy cache (disk-backed, in-memory index)
    proxy_cache_path /var/cache/nginx/api
        levels=1:2
        keys_zone=api_cache:10m    # 10MB shared memory for cache index
        max_size=1g;               # 1GB disk space

    # Upstream keepalive (per-worker, not shared)
    upstream backend {
        server 10.0.0.1:3000;
        keepalive 32;
    }
}
```

---

# Level 6 — Master

---

## Chapter 24: Large-Scale Architecture

### What You Will Learn
- Designing NGINX for millions of concurrent users
- Multi-layer proxy architectures
- Geographic routing
- Anycast and BGP
- Global traffic management

### Multi-Layer Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                  Global Load Balancing Layer                    │
│                    (DNS + Anycast/GeoDNS)                       │
└────────────────┬────────────────────────┬───────────────────────┘
                 │                        │
                 ▼                        ▼
      ┌──────────────────┐      ┌──────────────────┐
      │   Edge Layer     │      │   Edge Layer     │
      │   US-EAST        │      │   EU-WEST        │
      │                  │      │                  │
      │  NGINX (x2, HA)  │      │  NGINX (x2, HA)  │
      │  - SSL terminate │      │  - SSL terminate │
      │  - DDoS protect  │      │  - DDoS protect  │
      │  - Rate limit    │      │  - Rate limit    │
      └────────┬─────────┘      └────────┬─────────┘
               │                         │
               ▼                         ▼
      ┌──────────────────┐      ┌──────────────────┐
      │  API Gateway     │      │  API Gateway     │
      │  Layer           │      │  Layer           │
      │                  │      │                  │
      │  NGINX (x3, LB)  │      │  NGINX (x3, LB)  │
      │  - Auth          │      │  - Auth          │
      │  - Routing       │      │  - Routing       │
      │  - Caching       │      │  - Caching       │
      └────────┬─────────┘      └────────┬─────────┘
               │                         │
               └──────────┬──────────────┘
                          │
               ┌──────────┼──────────┐
               ▼          ▼          ▼
          ┌────────┐ ┌────────┐ ┌────────┐
          │Service │ │Service │ │Service │
          │   A    │ │   B    │ │   C    │
          │(x10)   │ │(x5)    │ │(x8)    │
          └────────┘ └────────┘ └────────┘
```

### Geographic Routing with GeoIP2

```bash
# Install GeoIP2 module
sudo apt install libnginx-mod-http-geoip2 libmaxminddb-dev -y

# Download GeoLite2 database
sudo mkdir -p /usr/share/GeoIP
# Register at maxmind.com and download GeoLite2-Country.mmdb
```

```nginx
load_module modules/ngx_http_geoip2_module.so;

http {
    geoip2 /usr/share/GeoIP/GeoLite2-Country.mmdb {
        $geoip2_country_code country iso_code;
        $geoip2_country_name country names en;
    }

    # Route to region-specific backends
    map $geoip2_country_code $backend_pool {
        US      "us_backend";
        CA      "us_backend";
        GB      "eu_backend";
        DE      "eu_backend";
        FR      "eu_backend";
        JP      "asia_backend";
        CN      "asia_backend";
        default "us_backend";
    }

    upstream us_backend {
        server us-app-1.internal:3000;
        server us-app-2.internal:3000;
    }

    upstream eu_backend {
        server eu-app-1.internal:3000;
        server eu-app-2.internal:3000;
    }

    upstream asia_backend {
        server asia-app-1.internal:3000;
        server asia-app-2.internal:3000;
    }

    server {
        listen 443 ssl http2;
        server_name example.com;

        location /api/ {
            proxy_pass http://$backend_pool;
            include snippets/proxy-params.conf;

            # Pass country to backend for analytics
            proxy_set_header X-Country-Code $geoip2_country_code;
        }
    }
}
```

---

## Chapter 25: Multi-Region Deployment

### What You Will Learn
- Active-Active multi-region configuration
- Data replication strategies
- Read replicas and geographic routing
- Failover between regions
- Latency-based routing

### Architecture

```
                    ┌─────────────────┐
                    │   GeoDNS        │
                    │   Routes users  │
                    │   to nearest    │
                    │   region        │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
      ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
      │  US-EAST-1   │ │  EU-WEST-1   │ │  AP-EAST-1   │
      │              │ │              │ │              │
      │  NGINX(HA)   │ │  NGINX(HA)   │ │  NGINX(HA)   │
      │  Apps(x5)    │ │  Apps(x3)    │ │  Apps(x3)    │
      │  DB-Primary  │ │  DB-Replica  │ │  DB-Replica  │
      │  Redis       │ │  Redis       │ │  Redis       │
      └──────┬───────┘ └──────┬───────┘ └──────┬───────┘
             │                │                │
             └────────────────┴────────────────┘
                       Database Replication
                       (writes to primary,
                        reads from nearest)
```

### Cross-Region Failover

```nginx
# When primary upstream fails, try the backup region
upstream primary_region {
    server 10.1.0.10:3000 max_fails=2 fail_timeout=10s;
    server 10.1.0.11:3000 max_fails=2 fail_timeout=10s;
}

upstream failover_region {
    server 10.2.0.10:3000;
    server 10.2.0.11:3000;
}

server {
    location / {
        # Try primary first
        proxy_pass http://primary_region;
        include snippets/proxy-params.conf;

        # On error, use proxy_next_upstream to try next server
        proxy_next_upstream error timeout http_502 http_503 http_504;

        # Intercept backend errors
        proxy_intercept_errors on;
    }
}
```

---

## Chapter 26: Failover Strategies

### What You Will Learn
- Automated failover vs manual failover
- Circuit breaker pattern with NGINX
- Retry logic and idempotency
- Graceful degradation

### Circuit Breaker with NGINX

```nginx
# Mark a server as down after failures, try backup
upstream api {
    server primary-api:3000   max_fails=5 fail_timeout=60s;
    server secondary-api:3000 max_fails=5 fail_timeout=60s backup;
}

server {
    location /api/ {
        proxy_pass http://api;
        include snippets/proxy-params.conf;

        # Retry on specific errors
        proxy_next_upstream error timeout http_502 http_503 http_504;

        # Limit retries to prevent amplification
        proxy_next_upstream_tries 3;
        proxy_next_upstream_timeout 10s;
    }
}
```

### Graceful Degradation

```nginx
# Serve a cached response or static fallback if backend is down

proxy_cache_use_stale error timeout http_500 http_502 http_503 http_504 updating;

# If cache is empty and backend is down, serve a maintenance page
error_page 502 503 504 /maintenance.html;
location = /maintenance.html {
    root /var/www/static;
    internal;
}
```

---

## Chapter 27: Enterprise Design Patterns

### What You Will Learn
- API Gateway pattern with NGINX
- Service mesh integration
- JWT authentication at NGINX layer
- mTLS between NGINX and backends
- Zero-trust networking

### API Gateway Pattern

```nginx
# NGINX as full API Gateway

http {
    # ── JWT Validation (requires auth_jwt module or Lua) ─────

    # Shared secret or JWKS endpoint for JWT validation
    # Using auth_request to validate with an auth service:

    server {
        listen 443 ssl http2;
        server_name api.example.com;

        # ── Authentication ────────────────────────────────────
        location /auth/validate {
            internal;
            proxy_pass http://auth_service:8080/validate;
            proxy_pass_request_body off;
            proxy_set_header Content-Length "";
            proxy_set_header X-Original-URI $request_uri;
            proxy_set_header Authorization $http_authorization;
        }

        # ── Protected API — requires valid JWT ────────────────
        location /api/v1/ {
            # Validate JWT before proxying
            auth_request /auth/validate;
            auth_request_set $auth_status $upstream_status;
            auth_request_set $auth_user $upstream_http_x_auth_user;

            # Pass user info to backend
            proxy_set_header X-Auth-User $auth_user;

            proxy_pass http://api_backend;
            include snippets/proxy-params.conf;
        }

        # ── Public endpoints — no auth ────────────────────────
        location /api/v1/public/ {
            proxy_pass http://api_backend;
            include snippets/proxy-params.conf;
        }

        # ── Return 401 for auth failures ─────────────────────
        error_page 401 = @error401;
        location @error401 {
            return 401 '{"error":"Unauthorized","message":"Valid JWT required"}';
            add_header Content-Type application/json;
        }
    }
}
```

### mTLS (Mutual TLS) Between NGINX and Backends

```nginx
# Verify backend's certificate (TLS verification)
upstream secure_backend {
    server backend.internal:443;
}

server {
    location /api/ {
        proxy_pass https://secure_backend;

        # Verify backend certificate
        proxy_ssl_verify on;
        proxy_ssl_trusted_certificate /etc/nginx/ssl/ca.pem;
        proxy_ssl_verify_depth 2;

        # Send our client certificate
        proxy_ssl_certificate     /etc/nginx/ssl/client.pem;
        proxy_ssl_certificate_key /etc/nginx/ssl/client.key;

        # Reuse SSL sessions
        proxy_ssl_session_reuse on;
    }
}
```

---

# Real-World Projects

---

## Project 1: Static Website Hosting

**Scenario**: Host a React single-page application with asset caching, compression, and security headers.

### Complete Configuration

```bash
# Project setup
sudo mkdir -p /var/www/myapp
sudo chown $USER:$USER /var/www/myapp

# Build React app and deploy
npm run build
sudo cp -r build/* /var/www/myapp/
```

```nginx
# /etc/nginx/sites-available/myapp.conf

server {
    listen 80;
    listen [::]:80;
    server_name myapp.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name myapp.example.com;

    root /var/www/myapp;
    index index.html;

    ssl_certificate     /etc/letsencrypt/live/myapp.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/myapp.example.com/privkey.pem;
    include snippets/ssl-params.conf;
    include snippets/security-headers.conf;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_types text/plain text/css application/javascript application/json image/svg+xml;

    # React Router — serve index.html for all routes
    location / {
        try_files $uri $uri/ /index.html;
        expires 1h;
        add_header Cache-Control "public, must-revalidate";
    }

    # Cache JS/CSS bundles with content hash in filename (immutable)
    location ~* \.(js|css)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        gzip_static on;
    }

    # Cache images
    location ~* \.(jpg|jpeg|png|gif|ico|svg|webp)$ {
        expires 30d;
        add_header Cache-Control "public";
    }

    # Deny hidden files
    location ~ /\. {
        deny all;
    }

    access_log /var/log/nginx/myapp-access.log;
    error_log  /var/log/nginx/myapp-error.log;
}
```

```bash
# Enable and test
sudo ln -s /etc/nginx/sites-available/myapp.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## Project 2: Reverse Proxy for Node.js App

**Scenario**: Proxy a Node.js Express API running on port 3000 with rate limiting, SSL, and logging.

```nginx
# /etc/nginx/conf.d/upstream.conf
upstream nodejs {
    server 127.0.0.1:3000;
    keepalive 32;
}

# /etc/nginx/sites-available/nodeapi.conf

limit_req_zone $binary_remote_addr zone=node_limit:10m rate=30r/s;

server {
    listen 80;
    server_name api.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;

    ssl_certificate     /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;
    include snippets/ssl-params.conf;
    include snippets/security-headers.conf;

    client_max_body_size 10m;

    location / {
        proxy_pass http://nodejs;
        proxy_http_version 1.1;
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Connection        "";

        proxy_connect_timeout 60s;
        proxy_send_timeout    60s;
        proxy_read_timeout    60s;

        limit_req zone=node_limit burst=50 nodelay;
    }

    # Health check endpoint — no rate limiting
    location /health {
        proxy_pass http://nodejs;
        proxy_http_version 1.1;
        access_log off;
    }

    access_log /var/log/nginx/nodeapi-access.log;
    error_log  /var/log/nginx/nodeapi-error.log;
}
```

---

## Project 3: Load Balanced Application

**Scenario**: 3 app servers behind NGINX with least-connections balancing, health checks, and session persistence for specific routes.

```nginx
# /etc/nginx/conf.d/lb-upstream.conf

upstream app_cluster {
    least_conn;

    server 10.0.1.10:3000 weight=1 max_fails=3 fail_timeout=30s;
    server 10.0.1.11:3000 weight=1 max_fails=3 fail_timeout=30s;
    server 10.0.1.12:3000 weight=1 max_fails=3 fail_timeout=30s;
    server 10.0.1.13:3000 backup;

    keepalive 64;
}

upstream session_cluster {
    ip_hash;   # Session persistence by IP

    server 10.0.1.10:3000;
    server 10.0.1.11:3000;
    server 10.0.1.12:3000;
}

# /etc/nginx/sites-available/loadbalancer.conf

server {
    listen 443 ssl http2;
    server_name lb.example.com;

    ssl_certificate     /etc/letsencrypt/live/lb.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/lb.example.com/privkey.pem;
    include snippets/ssl-params.conf;

    # Stateless API — use least_conn
    location /api/ {
        proxy_pass http://app_cluster;
        include snippets/proxy-params.conf;
        add_header X-Upstream-Addr $upstream_addr;
    }

    # Session-based app — use ip_hash
    location /app/ {
        proxy_pass http://session_cluster;
        include snippets/proxy-params.conf;
    }

    # NGINX status for monitoring
    location /nginx_status {
        stub_status on;
        allow 10.0.0.0/8;
        deny all;
    }

    access_log /var/log/nginx/lb-access.log;
    error_log  /var/log/nginx/lb-error.log;
}
```

---

## Project 4: Production NGINX Setup

**Scenario**: Complete production-grade NGINX for a SaaS application — multi-service, SSL, caching, rate limiting, security headers, monitoring.

```nginx
# /etc/nginx/nginx.conf

worker_processes auto;
worker_rlimit_nofile 65535;
error_log /var/log/nginx/error.log warn;
pid /run/nginx.pid;

events {
    worker_connections 4096;
    multi_accept on;
    use epoll;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format json_logs escape=json
        '{'
            '"time":"$time_iso8601",'
            '"remote_addr":"$remote_addr",'
            '"request":"$request",'
            '"status":$status,'
            '"bytes_sent":$body_bytes_sent,'
            '"request_time":$request_time,'
            '"upstream_time":"$upstream_response_time",'
            '"cache_status":"$upstream_cache_status",'
            '"user_agent":"$http_user_agent"'
        '}';

    access_log /var/log/nginx/access.log json_logs buffer=32k flush=5s;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    keepalive_requests 1000;
    server_tokens off;
    reset_timedout_connection on;

    client_body_timeout    12s;
    client_header_timeout  12s;
    send_timeout           10s;
    client_max_body_size   50m;

    gzip on;
    gzip_vary on;
    gzip_comp_level 6;
    gzip_types text/plain text/css application/javascript application/json image/svg+xml;

    open_file_cache max=20000 inactive=60s;
    open_file_cache_valid 30s;
    open_file_cache_min_uses 2;
    open_file_cache_errors on;

    # Rate limiting zones
    limit_req_zone $binary_remote_addr zone=global:10m      rate=30r/s;
    limit_req_zone $binary_remote_addr zone=api:10m         rate=10r/s;
    limit_req_zone $binary_remote_addr zone=auth:10m        rate=5r/m;
    limit_conn_zone $binary_remote_addr zone=conn_limit:10m;

    # Cache zones
    proxy_cache_path /var/cache/nginx/api
        levels=1:2
        keys_zone=api_cache:20m
        max_size=2g
        inactive=1h
        use_temp_path=off;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

```nginx
# /etc/nginx/conf.d/upstreams.conf

upstream api {
    least_conn;
    server 10.0.1.10:3000 max_fails=3 fail_timeout=30s;
    server 10.0.1.11:3000 max_fails=3 fail_timeout=30s;
    server 10.0.1.12:3000 max_fails=3 fail_timeout=30s;
    server 10.0.1.13:3000 backup;
    keepalive 64;
}

upstream frontend {
    server 10.0.2.10:8080;
    server 10.0.2.11:8080;
    keepalive 32;
}

upstream admin {
    server 10.0.3.10:9000;
    keepalive 8;
}
```

```nginx
# /etc/nginx/sites-available/production.conf

# ── HTTP → HTTPS redirect ─────────────────────────────────────
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

# ── Main Production Server ────────────────────────────────────
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name example.com www.example.com;

    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;
    include snippets/ssl-params.conf;
    include snippets/security-headers.conf;

    limit_conn conn_limit 50;
    limit_req zone=global burst=100 nodelay;

    # ── Auth endpoints — strict rate limiting ─────────────────
    location ~ ^/api/(auth|login|register|password) {
        proxy_pass http://api;
        include snippets/proxy-params.conf;
        limit_req zone=auth nodelay;
        limit_req_status 429;
    }

    # ── API — cached for GET requests ────────────────────────
    location /api/ {
        proxy_pass http://api;
        include snippets/proxy-params.conf;

        limit_req zone=api burst=30 nodelay;

        # Only cache GET requests
        proxy_cache api_cache;
        proxy_cache_methods GET;
        proxy_cache_key "$scheme$request_method$host$request_uri";
        proxy_cache_valid 200 5m;
        proxy_cache_valid 404 1m;
        proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
        proxy_cache_background_update on;
        proxy_cache_lock on;

        # Don't cache if Authorization header present (user-specific data)
        proxy_cache_bypass $http_authorization;
        proxy_no_cache     $http_authorization;

        add_header X-Cache-Status $upstream_cache_status;
    }

    # ── WebSocket ─────────────────────────────────────────────
    location /ws/ {
        proxy_pass http://api;
        proxy_http_version 1.1;
        proxy_set_header Upgrade    $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host       $host;
        proxy_read_timeout 86400s;
    }

    # ── Static assets ─────────────────────────────────────────
    location /static/ {
        alias /var/www/static/;
        expires 1y;
        add_header Cache-Control "public, immutable";
        gzip_static on;
    }

    # ── Admin — restricted access ─────────────────────────────
    location /admin/ {
        allow 10.0.0.0/8;
        allow 192.168.0.0/16;
        deny all;

        proxy_pass http://admin;
        include snippets/proxy-params.conf;
    }

    # ── Frontend SPA ──────────────────────────────────────────
    location / {
        proxy_pass http://frontend;
        include snippets/proxy-params.conf;

        # Cache HTML briefly
        proxy_cache_valid 200 1m;
        proxy_cache api_cache;
    }

    # ── NGINX status (internal only) ─────────────────────────
    location /nginx_status {
        stub_status on;
        allow 127.0.0.1;
        allow 10.0.0.0/8;
        deny all;
    }

    # ── Error pages ───────────────────────────────────────────
    error_page 404 /errors/404.html;
    error_page 500 502 503 504 /errors/50x.html;

    location /errors/ {
        root /var/www;
        internal;
    }

    access_log /var/log/nginx/production-access.log json_logs;
    error_log  /var/log/nginx/production-error.log warn;
}
```

---

# Debugging Section

Understanding how to diagnose NGINX issues is as important as knowing how to configure it. Production incidents often happen at 3am — you need to move fast and accurately.

## access.log vs error.log

### `access.log` — What Happened

Every HTTP request is recorded here (unless you disable it). Format example:

```
10.0.0.1 - - [31/Mar/2026:12:34:56 +0000] "GET /api/users HTTP/1.1" 200 1234 "-" "curl/7.88.1"
│           │   │                          │                          │   │    │    └─ User-Agent
│           │   │                          │                          │   │    └────── Referer
│           │   │                          │                          │   └─────────── Response bytes
│           │   │                          │                          └──────────────── Status code
│           │   │                          └──────────────────────────────────────────── Request
│           │   └──────────────────────────────────────────────────────────────────────── Time
│           └──────────────────────────────────────────────────────────────────────────── User
└──────────────────────────────────────────────────────────────────────────────────────── Client IP
```

**HTTP Status Code Reference:**

| Code | Meaning | Common Causes |
|------|---------|--------------|
| 200 | OK | Success |
| 301/302 | Redirect | Working as expected |
| 400 | Bad Request | Malformed client request |
| 401 | Unauthorized | Auth required |
| 403 | Forbidden | Access denied |
| 404 | Not Found | Wrong path, missing file |
| 429 | Too Many Requests | Rate limit hit |
| 499 | Client Closed Request | Client disconnected |
| 502 | Bad Gateway | Backend unreachable |
| 503 | Service Unavailable | Backend down or overloaded |
| 504 | Gateway Timeout | Backend took too long |

### `error.log` — What Went Wrong

```bash
# Real-time error monitoring
sudo tail -f /var/log/nginx/error.log

# Filter by error level
sudo grep "\[error\]" /var/log/nginx/error.log
sudo grep "\[crit\]"  /var/log/nginx/error.log

# Common error messages:
# [error] connect() failed (111: Connection refused)
#         → Backend server is not running
# [error] upstream timed out (110: Connection timed out)
#         → Backend is too slow
# [error] no live upstreams while connecting to upstream
#         → All backend servers are down
# [warn]  limiting requests, excess: X by zone
#         → Rate limit triggered
# [error] open() "/var/www/html/missing.html" failed (2: No such file or directory)
#         → Root path wrong or file missing
# [crit]  *1 SSL_do_handshake() failed
#         → SSL certificate issue
```

## `nginx -t` — Configuration Testing

```bash
# Test configuration before reload (ALWAYS do this first)
sudo nginx -t

# Success output:
# nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
# nginx: configuration file /etc/nginx/nginx.conf test is successful

# Failure output example:
# nginx: [emerg] unknown directive "proxu_pass" in /etc/nginx/sites-enabled/app.conf:15
#                                    ↑ typo!

# Test and show full configuration
sudo nginx -T | less

# Test a specific config file
sudo nginx -t -c /path/to/test.conf
```

## `systemctl status nginx` — Service Status

```bash
sudo systemctl status nginx

# Output interpretation:
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled)  ← Will start on boot
     Active: active (running)                                      ← Currently running
    Process: 1234 ExecStartPre=/usr/sbin/nginx -t                 ← Config was tested
   Main PID: 1235 (nginx)                                          ← Master process PID
      Tasks: 5 (limit: 4096)
     Memory: 12.3M
        CPU: 234ms
     CGroup: /system.slice/nginx.service
             ├─1235 nginx: master process /usr/sbin/nginx
             ├─1236 nginx: worker process
             ├─1237 nginx: worker process
             ├─1238 nginx: worker process
             └─1239 nginx: worker process

# If nginx is failed:
# Active: failed
# Look at the last few lines for the error message
```

## Troubleshooting Config Errors

```bash
# Step-by-step debugging workflow:

# 1. Test configuration
sudo nginx -t

# 2. If test fails, find the error
sudo nginx -T 2>&1 | grep -A5 "error"

# 3. Check recent changes
git diff /etc/nginx/   # If you track config in git

# 4. Reload safely (only if -t passes)
sudo systemctl reload nginx

# 5. If nginx crashed, check system logs
sudo journalctl -u nginx -n 50

# 6. Check if port 80/443 is in use by another process
sudo ss -tlnp | grep ':80\|:443'
sudo fuser 80/tcp

# 7. Test upstream connectivity
curl -v http://127.0.0.1:3000/health

# 8. Check file permissions
ls -la /var/www/html/
ls -la /etc/nginx/ssl/

# 9. Enable debug logging temporarily
error_log /var/log/nginx/debug.log debug;
# WARNING: debug generates enormous logs — disable in production

# 10. Monitor logs in real time
sudo tail -f /var/log/nginx/error.log /var/log/nginx/access.log
```

## Common Troubleshooting Scenarios

### 502 Bad Gateway

```bash
# Diagnosis:
# 1. Is the backend running?
sudo systemctl status node-app

# 2. Is it listening on the expected port?
sudo ss -tlnp | grep 3000

# 3. Can NGINX reach it?
curl http://127.0.0.1:3000/

# 4. Check upstream in NGINX error log
grep "connect() failed" /var/log/nginx/error.log
```

### 504 Gateway Timeout

```bash
# Backend is too slow. Options:
# 1. Increase NGINX timeouts
proxy_read_timeout 300s;
proxy_send_timeout 300s;

# 2. Optimize backend query
# 3. Add caching

# Diagnose slow requests:
grep "upstream_response_time" /var/log/nginx/access.log | \
    awk '{print $NF}' | sort -n | tail 20
```

### 403 Forbidden

```bash
# Possible causes:
# 1. Permission denied on file
ls -la /var/www/html/index.html
# Should be readable by nginx user: sudo chown -R nginx:nginx /var/www

# 2. SELinux blocking (RHEL/CentOS)
sudo ausearch -c nginx | tail 10
sudo setsebool -P httpd_can_network_connect 1

# 3. Explicit deny in config
grep -r "deny" /etc/nginx/
```

### SSL Certificate Errors

```bash
# Test certificate
openssl s_client -connect example.com:443 2>&1 | head 30

# Check expiry
echo | openssl s_client -connect example.com:443 2>/dev/null | \
    openssl x509 -noout -dates

# Verify cert chain
openssl verify -CAfile /etc/ssl/certs/ca-certificates.crt \
    /etc/letsencrypt/live/example.com/fullchain.pem

# Renew Let's Encrypt certificate
sudo certbot renew --force-renewal -d example.com
sudo systemctl reload nginx
```

---

# Production Folder Structure

```
/etc/nginx/
├── nginx.conf                          ← Root config (minimal)
│
├── conf.d/
│   ├── upstreams.conf                  ← All upstream blocks
│   ├── cache-zones.conf                ← proxy_cache_path definitions
│   ├── rate-limits.conf                ← limit_req_zone definitions
│   └── gzip.conf                       ← Compression settings
│
├── sites-available/
│   ├── example.com.conf                ← Production site
│   ├── api.example.com.conf            ← API service
│   ├── admin.example.com.conf          ← Admin panel
│   └── monitoring.example.com.conf     ← Monitoring endpoint
│
├── sites-enabled/
│   ├── example.com.conf -> ../sites-available/example.com.conf
│   ├── api.example.com.conf -> ../sites-available/api.example.com.conf
│   └── admin.example.com.conf -> ../sites-available/admin.example.com.conf
│
├── snippets/
│   ├── ssl-params.conf                 ← TLS/SSL settings
│   ├── security-headers.conf           ← HTTP security headers
│   └── proxy-params.conf               ← Shared proxy_set_header directives
│
└── ssl/
    ├── dhparam.pem                     ← DH parameters
    └── example.com/
        ├── fullchain.pem               ← Certificate + chain
        └── privkey.pem                 ← Private key

/var/log/nginx/
├── access.log                          ← Combined access log
├── error.log                           ← Global error log
├── example.com-access.log             ← Per-vhost access log
├── example.com-error.log              ← Per-vhost error log
├── api.example.com-access.log
└── api.example.com-error.log

/var/www/
├── html/                               ← Default web root
├── example.com/                        ← Site root
│   ├── public/                         ← Static files
│   └── uploads/                        ← User uploads
└── certbot/                            ← Let's Encrypt ACME challenge root

/var/cache/nginx/
├── api/                                ← API response cache
└── static/                             ← Static asset cache
```

---

# Security Best Practices Reference

## Complete Security Checklist

| Category | Action | Config / Command |
|----------|--------|-----------------|
| **TLS** | Use TLS 1.2+ only | `ssl_protocols TLSv1.2 TLSv1.3;` |
| **TLS** | Disable weak ciphers | `ssl_ciphers ECDHE-...` |
| **TLS** | Enable HSTS | `Strict-Transport-Security` header |
| **TLS** | Use DH parameters | `ssl_dhparam /etc/nginx/ssl/dhparam.pem;` |
| **Headers** | Hide server version | `server_tokens off;` |
| **Headers** | Prevent clickjacking | `X-Frame-Options: SAMEORIGIN` |
| **Headers** | Prevent MIME sniffing | `X-Content-Type-Options: nosniff` |
| **Headers** | Set CSP | `Content-Security-Policy: ...` |
| **Rate Limiting** | Limit requests per IP | `limit_req_zone` + `limit_req` |
| **Rate Limiting** | Limit connections per IP | `limit_conn_zone` + `limit_conn` |
| **Rate Limiting** | Strict auth endpoint limits | `rate=5r/m` on login routes |
| **Access Control** | Restrict admin by IP | `allow 10.0.0.0/8; deny all;` |
| **Request Filtering** | Block bad bots | User-Agent filtering |
| **Request Filtering** | Block invalid methods | `if ($request_method !~ ...)` |
| **Request Filtering** | Deny hidden files | `location ~ /\. { deny all; }` |
| **Request Filtering** | Limit body size | `client_max_body_size 10m;` |
| **Logging** | Enable access logs | `access_log /var/log/nginx/...;` |
| **Logging** | Monitor error logs | `error_log ... warn;` |
| **Updates** | Keep NGINX updated | `sudo apt upgrade nginx` regularly |
| **SSL Certs** | Auto-renew Let's Encrypt | Certbot cron job |
| **DDoS** | Connection timeouts | `client_body_timeout 10s;` |
| **DDoS** | Connection limits | `limit_conn` per IP |
| **DDoS** | Upstream failover | `max_fails` + `fail_timeout` |

## Rate Limiting Summary

```nginx
http {
    # Zones
    limit_req_zone $binary_remote_addr zone=global:10m  rate=30r/s;
    limit_req_zone $binary_remote_addr zone=api:10m     rate=10r/s;
    limit_req_zone $binary_remote_addr zone=login:10m   rate=5r/m;
    limit_conn_zone $binary_remote_addr zone=conns:10m;

    server {
        # Global connection limit
        limit_conn conns 20;

        # Per-endpoint rate limits
        location /           { limit_req zone=global burst=50 nodelay; }
        location /api/       { limit_req zone=api    burst=20 nodelay; }
        location /auth/login { limit_req zone=login             nodelay; }
    }
}
```

## HTTPS Enforcement

```nginx
# Redirect all HTTP → HTTPS
server {
    listen 80 default_server;
    server_name _;
    return 301 https://$host$request_uri;
}

# HSTS in HTTPS server
add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload" always;
```

## DDoS Protection Basics

```nginx
http {
    # Slow loris protection
    client_body_timeout   10s;
    client_header_timeout 10s;
    send_timeout          10s;

    # Limit connection and request rates
    limit_conn_zone $binary_remote_addr zone=ddos_conn:10m;
    limit_req_zone  $binary_remote_addr zone=ddos_req:10m rate=30r/s;

    server {
        limit_conn ddos_conn 20;
        limit_req  zone=ddos_req burst=50 nodelay;

        # Drop connections immediately after timeout
        reset_timedout_connection on;
    }
}
```

---

## Final Word

You have now covered NGINX from first installation to globally distributed, enterprise-grade architecture. The key to mastery is not memorizing configuration syntax — it is understanding *why* each directive exists, what problem it solves, and when to apply it.

The best NGINX engineers think in terms of:
- **Traffic flow**: Where does a request come from? Where does it go? What happens at each hop?
- **Failure modes**: What happens if the backend is down? If the network is slow? If the client disconnects?
- **Observability**: Can I see exactly what NGINX is doing? Can I diagnose a production incident in minutes?
- **Iteration**: Start simple, measure, then tune. Never over-engineer before you have data.

Every production system is different. Use this handbook as a reference and a foundation — adapt every configuration to your specific requirements, traffic patterns, and security needs.

> "NGINX is a precision tool. Master the fundamentals, and everything else is configuration."

---

*NGINX Mastery Guide — Production Reference Edition*
*Covers NGINX 1.24+ | Docker 24+ | Ubuntu 22.04+ | CentOS 9+*
