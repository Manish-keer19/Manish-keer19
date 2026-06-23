# 🔐 Ubuntu VPS Security Guide for Production Web Applications
### Nginx · Node.js · React · MySQL/PostgreSQL · SSL

> **Applies to:** Ubuntu 22.04 LTS / 24.04 LTS  
> **Stack:** Nginx · Node.js · React · MySQL/PostgreSQL · SSL/TLS  
> **Level:** Beginner-friendly, Production-focused

---

## Table of Contents

1. [Why VPS Security Matters](#1-why-vps-security-matters)
2. [Initial Server Setup](#2-initial-server-setup)
3. [Secure SSH Configuration](#3-secure-ssh-configuration)
4. [UFW Firewall Setup](#4-ufw-firewall-setup)
5. [Fail2Ban – Brute Force Protection](#5-fail2ban--brute-force-protection)
6. [Securing Nginx](#6-securing-nginx)
7. [Reverse Proxy Architecture](#7-reverse-proxy-architecture)
8. [SSL with Certbot (Let's Encrypt)](#8-ssl-with-certbot-lets-encrypt)
9. [Node.js Application Security](#9-nodejs-application-security)
10. [PM2 Process Security](#10-pm2-process-security)
11. [Database Security – MySQL & PostgreSQL](#11-database-security--mysql--postgresql)
12. [File Permissions & .env Protection](#12-file-permissions--env-protection)
13. [Automatic Security Updates](#13-automatic-security-updates)
14. [DDoS Protection & Rate Limiting](#14-ddos-protection--rate-limiting)
15. [Cloudflare Integration](#15-cloudflare-integration)
16. [Docker Security (If Using Docker)](#16-docker-security-if-using-docker)
17. [Backup Strategy](#17-backup-strategy)
18. [Monitoring, Logs & Alerts](#18-monitoring-logs--alerts)
19. [Malware & Rootkit Scanning](#19-malware--rootkit-scanning)
20. [Checking Open Ports & Active Connections](#20-checking-open-ports--active-connections)
21. [Detecting Suspicious Activity](#21-detecting-suspicious-activity)
22. [Common Beginner Mistakes](#22-common-beginner-mistakes)
23. [VPS Security Checklist](#23-vps-security-checklist)
24. [Maintenance Schedule](#24-maintenance-schedule)
25. [Troubleshooting](#25-troubleshooting)

---

## 1. Why VPS Security Matters

When you deploy a production application on a public VPS, you are exposed to the open internet **immediately**. Automated bots scan for open ports, attempt brute-force SSH logins, probe for database exposure, and exploit unpatched software — within **minutes** of a new server going live.

A single misconfiguration can lead to:

- Complete server compromise and data theft
- Your server becoming part of a botnet or crypto-mining operation
- Database leaks exposing user data (GDPR/legal liability)
- Ransomware encrypting your files
- Downtime and reputational damage

> **This guide takes a defense-in-depth approach:** multiple overlapping layers of security, so a failure in one layer does not compromise the entire system.

---

## 2. Initial Server Setup

### 2.1 First Login & System Update

Always start by updating the system. Outdated packages are the #1 vector for server compromise.

```bash
# Login as root (only for initial setup)
ssh root@YOUR_SERVER_IP

# Update package lists and upgrade all packages
apt update && apt upgrade -y

# Install essential tools
apt install -y curl wget git unzip ufw fail2ban vim nano htop net-tools
```

> **Why:** Package updates patch known CVEs (Common Vulnerabilities and Exposures). An unpatched Ubuntu server is trivially exploitable using public exploit databases.

### 2.2 Create a Non-Root Sudo User

**Never run your applications or log in as root in production.**

```bash
# Create a new user (replace 'deploy' with your preferred username)
adduser deploy

# Add user to sudo group
usermod -aG sudo deploy

# Verify group membership
groups deploy
# Output: deploy : deploy sudo
```

> **Why:** The `root` account has unrestricted access to everything. If an attacker gains access to a root session (via stolen credentials or an exploit), the server is fully compromised. A sudo user requires a password to perform privileged operations, creating an extra hurdle.

### 2.3 Lock Root Login

After creating your sudo user and confirming SSH key access (see Section 3), disable root login entirely.

```bash
# Edit SSH daemon config
sudo nano /etc/ssh/sshd_config

# Find and set:
PermitRootLogin no
```

### 2.4 Set the Server Timezone

```bash
# Set timezone (adjust to your region)
timedatectl set-timezone Asia/Kolkata

# Verify
timedatectl status
```

> **Why:** Correct timestamps are critical for log analysis, SSL certificate renewal, and correlating events during incident response.

### 2.5 Set a Hostname

```bash
hostnamectl set-hostname production-server
```

---

## 3. Secure SSH Configuration

SSH is the most targeted service on any public server. A hardened SSH configuration dramatically reduces your attack surface.

### 3.1 Generate SSH Keys (On Your Local Machine)

```bash
# Run on YOUR LOCAL machine (not the server)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Follow prompts — set a strong passphrase
# Keys are saved to ~/.ssh/id_ed25519 (private) and ~/.ssh/id_ed25519.pub (public)
```

> **Why use ed25519?** It is faster, smaller, and more secure than the older RSA algorithm. Always prefer ed25519 or RSA-4096 over RSA-2048 for new keys.

### 3.2 Copy Public Key to Server

```bash
# Method 1: Automatic (preferred)
ssh-copy-id -i ~/.ssh/id_ed25519.pub deploy@YOUR_SERVER_IP

# Method 2: Manual (if ssh-copy-id is unavailable)
# On the server, logged in as deploy:
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
# Paste the contents of your id_ed25519.pub file here
chmod 600 ~/.ssh/authorized_keys
```

### 3.3 Harden sshd_config

```bash
sudo nano /etc/ssh/sshd_config
```

Apply the following settings:

```nginx
# ============================================
# Secure SSH Configuration
# /etc/ssh/sshd_config
# ============================================

# Change default port (optional but recommended — see note below)
Port 2222

# Only listen on IPv4 (remove if you need IPv6)
AddressFamily inet

# Disable root login — CRITICAL
PermitRootLogin no

# Only allow specific users
AllowUsers deploy

# Disable password authentication — force key-only login
PasswordAuthentication no
PermitEmptyPasswords no
ChallengeResponseAuthentication no

# Disable unused authentication methods
UsePAM yes
KerberosAuthentication no
GSSAPIAuthentication no

# Disconnect idle sessions after 10 minutes
ClientAliveInterval 300
ClientAliveCountMax 2

# Limit login attempts
MaxAuthTries 3
MaxSessions 5

# Disable X11 forwarding (not needed on servers)
X11Forwarding no

# Disable TCP port forwarding (unless you specifically need it)
AllowTcpForwarding no

# Show last login info
PrintLastLog yes

# Use strong ciphers and MACs
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com
MACs hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com
KexAlgorithms curve25519-sha256,diffie-hellman-group14-sha256

# Log level for audit trail
LogLevel VERBOSE
```

> ⚠️ **Warning:** Always test your SSH configuration in a **new terminal session** before closing the current one. If you lock yourself out of a misconfigured SSH, you will need console/VNC access through your hosting provider.

```bash
# Validate sshd_config syntax BEFORE restarting
sudo sshd -t

# If no errors, reload SSH
sudo systemctl reload sshd

# Test in a NEW terminal window (don't close the existing one yet)
ssh -p 2222 deploy@YOUR_SERVER_IP
```

### 3.4 Optional: Change SSH Port

Changing from port 22 to a non-standard port (e.g., 2222, 8822, 49152–65535) reduces automated scan noise by ~90%.

```bash
# If you changed the port, also allow it in UFW (before reloading SSH!)
sudo ufw allow 2222/tcp comment "Custom SSH port"

# Then update your SSH client config locally (~/.ssh/config)
Host myserver
    HostName YOUR_SERVER_IP
    User deploy
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
```

> **Note:** This is "security through obscurity" and is not a replacement for key authentication, but it does meaningfully reduce log noise from bots.

---

## 4. UFW Firewall Setup

UFW (Uncomplicated Firewall) is a frontend for `iptables` that makes firewall management simple and readable.

### 4.1 Default Deny Policy

```bash
# Start with a clean slate — deny ALL incoming traffic
sudo ufw default deny incoming

# Allow all outgoing traffic (your server can initiate connections)
sudo ufw default allow outgoing

# Verify defaults
sudo ufw status verbose
```

> **Why deny-by-default?** Any port not explicitly opened is automatically blocked. This means a misconfigured service binding to an unexpected port is automatically protected.

### 4.2 Allow Essential Ports

```bash
# Allow SSH (use your custom port if you changed it)
sudo ufw allow 22/tcp comment "SSH"
# OR if using custom port:
sudo ufw allow 2222/tcp comment "Custom SSH"

# Allow HTTP and HTTPS
sudo ufw allow 80/tcp comment "HTTP"
sudo ufw allow 443/tcp comment "HTTPS"

# Enable UFW
sudo ufw enable

# Confirm rules
sudo ufw status numbered
```

Expected output:

```
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere
[ 2] 80/tcp                     ALLOW IN    Anywhere
[ 3] 443/tcp                    ALLOW IN    Anywhere
```

### 4.3 Allow Database Access Only From Local Network (Optional)

If you need database access from another internal server, restrict it:

```bash
# Allow MySQL from a specific internal IP only
sudo ufw allow from 10.0.0.5 to any port 3306 comment "MySQL from app server"

# NEVER do this:
# sudo ufw allow 3306  ← exposes your database to the entire internet
```

### 4.4 Block and Remove Rules

```bash
# Delete a rule by number
sudo ufw status numbered
sudo ufw delete 3

# Block a specific IP (useful when you detect an attacker)
sudo ufw deny from 203.0.113.100 comment "Blocked attacker"

# Reset all UFW rules (nuclear option — will disconnect SSH if not careful)
sudo ufw reset
```

### 4.5 Verify Firewall

```bash
# View all rules
sudo ufw status verbose

# View active iptables rules (lower level)
sudo iptables -L -n -v

# Test from outside — scan your server from another machine
nmap -sV YOUR_SERVER_IP
# Should only see ports 22 (or custom), 80, 443
```

---

## 5. Fail2Ban – Brute Force Protection

Fail2Ban monitors log files and automatically bans IPs that show malicious behavior (e.g., too many failed SSH login attempts).

### 5.1 Install and Configure

```bash
sudo apt install -y fail2ban

# Create local config (never edit the .conf files — they get overwritten on updates)
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

### 5.2 Core Configuration

```ini
# /etc/fail2ban/jail.local

[DEFAULT]
# Ban duration (in seconds). 3600 = 1 hour, -1 = permanent
bantime  = 3600

# Time window to count failures
findtime = 600

# Number of failures before banning
maxretry = 5

# Your IP — NEVER ban yourself
ignoreip = 127.0.0.1/8 YOUR_HOME_IP/32

# Email alerts (optional — requires a mail server)
destemail = admin@yourdomain.com
sendername = Fail2Ban
mta = sendmail

# Ban action
banaction = ufw

# ============================================
# SSH Jail
# ============================================
[sshd]
enabled  = true
port     = ssh,2222      # Add your custom SSH port
filter   = sshd
logpath  = /var/log/auth.log
maxretry = 3
bantime  = 86400         # 24 hours for SSH brute force

# ============================================
# Nginx Jails
# ============================================
[nginx-http-auth]
enabled  = true
port     = http,https
filter   = nginx-http-auth
logpath  = /var/log/nginx/error.log
maxretry = 5

[nginx-limit-req]
enabled  = true
port     = http,https
filter   = nginx-limit-req
logpath  = /var/log/nginx/error.log
maxretry = 10

[nginx-botsearch]
enabled  = true
port     = http,https
filter   = nginx-botsearch
logpath  = /var/log/nginx/access.log
maxretry = 2
```

### 5.3 Start and Enable Fail2Ban

```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
sudo systemctl status fail2ban
```

### 5.4 Useful Fail2Ban Commands

```bash
# View all active bans
sudo fail2ban-client status

# View bans for a specific jail
sudo fail2ban-client status sshd

# Unban an IP address
sudo fail2ban-client set sshd unbanip 203.0.113.100

# Test a filter against a log file
sudo fail2ban-regex /var/log/auth.log /etc/fail2ban/filter.d/sshd.conf

# Reload configuration after changes
sudo fail2ban-client reload
```

---

## 6. Securing Nginx

Nginx is your front-facing web server. Misconfigured Nginx can expose your application infrastructure, leak version information, or be abused for DDoS amplification.

### 6.1 Install Nginx

```bash
sudo apt install -y nginx
sudo systemctl enable nginx
sudo systemctl start nginx
```

### 6.2 Harden nginx.conf

```bash
sudo nano /etc/nginx/nginx.conf
```

```nginx
# /etc/nginx/nginx.conf

user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
    worker_connections 1024;
    multi_accept on;
    use epoll;
}

http {
    # ============================================
    # Basic Settings
    # ============================================
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 30;
    types_hash_max_size 2048;

    # HIDE SERVER VERSION — prevents version-specific exploits
    server_tokens off;

    # ============================================
    # Security Headers (Global)
    # ============================================
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Permissions-Policy "camera=(), microphone=(), geolocation=()" always;
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:;" always;

    # ============================================
    # Buffer Overflow Protection
    # ============================================
    client_body_buffer_size 10K;
    client_header_buffer_size 1k;
    client_max_body_size 8m;
    large_client_header_buffers 2 1k;

    # ============================================
    # Timeout Settings (DDoS Mitigation)
    # ============================================
    client_body_timeout 12;
    client_header_timeout 12;
    send_timeout 10;

    # ============================================
    # Rate Limiting Zones (used in server blocks)
    # ============================================
    limit_req_zone $binary_remote_addr zone=general:10m rate=10r/s;
    limit_req_zone $binary_remote_addr zone=api:10m rate=5r/s;
    limit_req_zone $binary_remote_addr zone=login:10m rate=2r/m;
    limit_conn_zone $binary_remote_addr zone=conn_limit_per_ip:10m;

    # ============================================
    # MIME Types & Logging
    # ============================================
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log warn;

    # ============================================
    # Gzip Compression
    # ============================================
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml application/json application/javascript
               application/xml+rss application/atom+xml image/svg+xml;

    # Include site configurations
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

### 6.3 Prevent Direct IP Access & Block Unknown Domains

Create a default catch-all server block that returns 444 (no response) for unrecognized hosts:

```bash
sudo nano /etc/nginx/sites-available/default
```

```nginx
# /etc/nginx/sites-available/default
# This MUST be the first server block processed (use lowest listen directive)

# Block direct IP access and unknown hostnames
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    listen 443 ssl default_server;
    listen [::]:443 ssl default_server;

    # Dummy SSL (Nginx requires a cert even for default blocks with SSL)
    ssl_certificate /etc/nginx/ssl/dummy.crt;
    ssl_certificate_key /etc/nginx/ssl/dummy.key;

    # Return 444: Nginx closes the connection without sending a response
    # This prevents information leakage and wastes no resources on bots
    return 444;
}
```

Generate a dummy self-signed certificate for the default block:

```bash
sudo mkdir -p /etc/nginx/ssl
sudo openssl req -x509 -nodes -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/dummy.key \
  -out /etc/nginx/ssl/dummy.crt \
  -days 3650 \
  -subj "/CN=localhost"
```

> **Why 444?** HTTP 444 is an Nginx-specific non-standard status code that causes Nginx to immediately close the TCP connection without sending an HTTP response. Bots scanning IP ranges get nothing — no redirect, no error page, no information. This is far more secure than returning 403 or 301 redirects.

### 6.4 Hide PHP/Server Information in Headers

```bash
# In nginx.conf or your server block:
fastcgi_hide_header X-Powered-By;
proxy_hide_header X-Powered-By;
more_clear_headers Server;
```

### 6.5 Test and Reload Nginx

```bash
# Always test configuration before reloading
sudo nginx -t

# Reload without dropping connections
sudo systemctl reload nginx

# Verify server_tokens is off
curl -I http://YOUR_SERVER_IP
# "Server: nginx" — NO version number. Good.
```

---

## 7. Reverse Proxy Architecture

In a secure production setup, your Node.js application **never** listens on a public port. It listens only on `127.0.0.1` (localhost), and Nginx acts as the public-facing reverse proxy.

```
Internet → UFW Firewall (80/443 only)
         → Nginx (public-facing, handles SSL, rate limiting, headers)
         → Node.js on 127.0.0.1:3000 (internal only, never exposed)
         → Database on 127.0.0.1:5432 (internal only, never exposed)
```

### 7.1 Create a Production Site Configuration

```bash
sudo nano /etc/nginx/sites-available/myapp.conf
```

```nginx
# /etc/nginx/sites-available/myapp.conf

# Upstream definition — your Node.js app
upstream nodejs_app {
    server 127.0.0.1:3000;
    keepalive 64;
}

# Redirect HTTP → HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name yourdomain.com www.yourdomain.com;

    # Allow Let's Encrypt certificate renewal
    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

# Main HTTPS server block
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name yourdomain.com www.yourdomain.com;

    # SSL Configuration (managed by Certbot)
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    # HSTS — forces browsers to use HTTPS for 1 year
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

    # Rate limiting
    limit_req zone=general burst=20 nodelay;
    limit_conn conn_limit_per_ip 15;

    # Logging
    access_log /var/log/nginx/myapp.access.log;
    error_log /var/log/nginx/myapp.error.log warn;

    # Root for static React build files (optional)
    root /var/www/myapp/build;
    index index.html;

    # Serve React static assets with caching
    location /static/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # API requests → Node.js backend
    location /api/ {
        limit_req zone=api burst=10 nodelay;

        proxy_pass http://nodejs_app;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;

        # Security: hide backend headers
        proxy_hide_header X-Powered-By;

        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Stricter rate limiting on login/auth endpoints
    location /api/auth/login {
        limit_req zone=login burst=5;
        proxy_pass http://nodejs_app;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # SPA fallback — serve index.html for all non-API routes
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Block access to sensitive files
    location ~ /\. {
        deny all;
        return 404;
    }

    location ~ \.(env|log|bak|sql|config|yaml|yml)$ {
        deny all;
        return 404;
    }
}
```

```bash
# Enable the site
sudo ln -s /etc/nginx/sites-available/myapp.conf /etc/nginx/sites-enabled/

# Test configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

---

## 8. SSL with Certbot (Let's Encrypt)

SSL/TLS encrypts traffic between clients and your server, preventing eavesdropping and man-in-the-middle attacks. Let's Encrypt provides free, automatically renewable certificates.

### 8.1 Install Certbot

```bash
# Install Certbot with Nginx plugin
sudo apt install -y certbot python3-certbot-nginx

# Verify installation
certbot --version
```

### 8.2 Obtain a Certificate

```bash
# Make sure your domain's DNS A record points to your server IP first!
# Check: dig +short yourdomain.com

sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com \
  --email admin@yourdomain.com \
  --agree-tos \
  --no-eff-email \
  --redirect
```

> **What Certbot does:** Automatically verifies domain ownership via HTTP challenge, issues the certificate, modifies your Nginx config to use it, and sets up automatic renewal.

### 8.3 Test Automatic Renewal

```bash
# Test renewal process (dry run — doesn't actually renew)
sudo certbot renew --dry-run

# View existing certificates and expiry dates
sudo certbot certificates

# Force renewal (if needed)
sudo certbot renew --force-renewal
```

### 8.4 Verify Renewal Cron Job

```bash
# Certbot installs a systemd timer (Ubuntu 22.04+)
systemctl list-timers | grep certbot

# Alternatively, check cron
cat /etc/cron.d/certbot
```

### 8.5 Verify SSL Configuration

```bash
# Check SSL rating (aim for A+)
# Visit: https://www.ssllabs.com/ssltest/analyze.html?d=yourdomain.com

# Or test locally
curl -I https://yourdomain.com
openssl s_client -connect yourdomain.com:443 -servername yourdomain.com
```

### 8.6 Strengthen SSL Configuration

```bash
# Generate strong Diffie-Hellman parameters (takes ~5 minutes)
sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 4096

# Add to your Nginx server block
ssl_dhparam /etc/ssl/certs/dhparam.pem;

# Modern SSL protocols only
ssl_protocols TLSv1.2 TLSv1.3;

# Strong cipher suite
ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256';
ssl_prefer_server_ciphers off;

# Enable SSL session caching
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 1d;
ssl_session_tickets off;

# OCSP Stapling (speeds up SSL handshake, verifies cert validity)
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 1.1.1.1 valid=300s;
resolver_timeout 5s;
```

---

## 9. Node.js Application Security

### 9.1 Bind Only to Localhost

Your Node.js app must **never** listen on `0.0.0.0` in production.

```javascript
// ❌ DANGEROUS — exposes app directly to the internet
app.listen(3000, '0.0.0.0');

// ✅ CORRECT — only accessible from this server
app.listen(3000, '127.0.0.1');

// Or simply (defaults to 127.0.0.1):
app.listen(3000);
```

Verify no public ports are open:

```bash
# Should show ONLY 127.0.0.1:3000, not 0.0.0.0:3000
sudo ss -tlnp | grep node
# or
sudo netstat -tlnp | grep node
```

### 9.2 Use Helmet.js for Security Headers

```bash
npm install helmet
```

```javascript
const helmet = require('helmet');

app.use(helmet()); // Applies 14+ security headers automatically

// Or fine-tune:
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      scriptSrc: ["'self'"],
      imgSrc: ["'self'", "data:", "https:"],
    },
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true,
  },
}));
```

### 9.3 Use a Non-Privileged System User

```bash
# Create a dedicated user for your Node.js app
sudo adduser --system --group --no-create-home nodeapp

# Change ownership of your app directory
sudo chown -R nodeapp:nodeapp /var/www/myapp

# Run PM2 as this user (see PM2 section)
```

### 9.4 Input Validation and SQL Injection Prevention

```bash
npm install express-validator
npm install express-rate-limit
```

```javascript
const rateLimit = require('express-rate-limit');
const { body, validationResult } = require('express-validator');

// Rate limiting middleware
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,
  message: { error: 'Too many requests' },
  standardHeaders: true,
  legacyHeaders: false,
});

app.use('/api/', apiLimiter);

// Always validate and sanitize input
app.post('/api/user',
  body('email').isEmail().normalizeEmail(),
  body('name').isLength({ min: 1, max: 100 }).trim().escape(),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    // ...proceed with database operations using parameterized queries
  }
);
```

> ⚠️ **Never concatenate user input into SQL queries.** Always use parameterized queries or an ORM like Sequelize/Prisma.

### 9.5 Environment Variables

```bash
# Install dotenv
npm install dotenv

# Load at the very top of your entry file
require('dotenv').config();

# Access variables
const dbPassword = process.env.DB_PASSWORD; // Never hardcode passwords
```

---

## 10. PM2 Process Security

PM2 is a production process manager for Node.js. It keeps your application running and restarts it if it crashes.

### 10.1 Install and Configure PM2

```bash
# Install PM2 globally
sudo npm install -g pm2

# Start your application
pm2 start /var/www/myapp/server.js --name "myapp" --user nodeapp

# Save process list (survives reboots)
pm2 save

# Generate and enable startup script
pm2 startup systemd -u deploy --hp /home/deploy
# Run the command it outputs, then:
sudo systemctl enable pm2-deploy
```

### 10.2 PM2 Ecosystem Configuration

```javascript
// /var/www/myapp/ecosystem.config.js
module.exports = {
  apps: [{
    name: 'myapp',
    script: 'server.js',
    cwd: '/var/www/myapp',
    instances: 'max',          // Use all CPU cores
    exec_mode: 'cluster',      // Enable cluster mode
    user: 'nodeapp',           // Run as non-root user
    env_production: {
      NODE_ENV: 'production',
      PORT: 3000,
    },
    // Security: prevent memory leaks from crashing server
    max_memory_restart: '500M',
    // Logging
    log_file: '/var/log/pm2/myapp-combined.log',
    out_file: '/var/log/pm2/myapp-out.log',
    error_file: '/var/log/pm2/myapp-err.log',
    log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
    // Auto-restart on crash
    autorestart: true,
    watch: false,              // NEVER enable watch in production
    max_restarts: 10,
    min_uptime: '10s',
  }]
};
```

```bash
# Start with ecosystem config
pm2 start ecosystem.config.js --env production

# Monitor processes
pm2 monit

# View logs
pm2 logs myapp --lines 100

# View process status
pm2 status

# Graceful reload (zero downtime)
pm2 reload myapp
```

> ⚠️ **Never use `watch: true` in production.** It restarts the app on any file change, which can cause instability and is a security risk if an attacker can write files to your app directory.

---

## 11. Database Security – MySQL & PostgreSQL

### 11.1 MySQL Security

#### Installation

```bash
sudo apt install -y mysql-server

# Run the security script — answer YES to everything
sudo mysql_secure_installation
```

The `mysql_secure_installation` script will:
- Set a root password
- Remove anonymous users
- Disallow remote root login
- Remove test database
- Reload privilege tables

#### Verify MySQL is Bound to Localhost Only

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

# Ensure this line exists:
bind-address = 127.0.0.1
mysqlx-bind-address = 127.0.0.1
```

```bash
sudo systemctl restart mysql

# Verify MySQL is NOT listening on 0.0.0.0
sudo ss -tlnp | grep mysql
# Should show: 127.0.0.1:3306 only
```

#### Create a Least-Privilege Application User

```bash
sudo mysql -u root -p
```

```sql
-- Create a dedicated database and user for your app
CREATE DATABASE myapp_production CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Create user (never use root for app connections)
CREATE USER 'myapp_user'@'localhost' IDENTIFIED BY 'STRONG_RANDOM_PASSWORD_HERE';

-- Grant only necessary permissions (not SUPER, FILE, DROP, etc.)
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, INDEX, ALTER ON myapp_production.* TO 'myapp_user'@'localhost';

FLUSH PRIVILEGES;

-- Verify
SHOW GRANTS FOR 'myapp_user'@'localhost';
```

#### Disable MySQL Binary Logging (If Not Using Replication)

```ini
# In /etc/mysql/mysql.conf.d/mysqld.cnf
# Binary logs consume disk space; disable if no replication needed
# skip-log-bin
```

### 11.2 PostgreSQL Security

#### Installation

```bash
sudo apt install -y postgresql postgresql-contrib
```

#### Verify PostgreSQL is Bound to Localhost

```bash
sudo nano /etc/postgresql/14/main/postgresql.conf

# Set:
listen_addresses = 'localhost'
```

```bash
sudo nano /etc/postgresql/14/main/pg_hba.conf

# Ensure connections require password (md5 or scram-sha-256)
# TYPE  DATABASE  USER     ADDRESS    METHOD
local   all       all                 peer
host    all       all      127.0.0.1/32  scram-sha-256
host    all       all      ::1/128       scram-sha-256
```

#### Create a Least-Privilege Application User

```bash
sudo -u postgres psql
```

```sql
-- Create database and user
CREATE DATABASE myapp_production;
CREATE USER myapp_user WITH ENCRYPTED PASSWORD 'STRONG_RANDOM_PASSWORD_HERE';

-- Grant only necessary privileges
GRANT CONNECT ON DATABASE myapp_production TO myapp_user;
\c myapp_production
GRANT USAGE ON SCHEMA public TO myapp_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO myapp_user;
GRANT USAGE ON ALL SEQUENCES IN SCHEMA public TO myapp_user;

-- Default privileges for future tables
ALTER DEFAULT PRIVILEGES IN SCHEMA public
  GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO myapp_user;

\q
```

```bash
sudo systemctl restart postgresql

# Verify PostgreSQL is only on localhost
sudo ss -tlnp | grep postgres
# Should show: 127.0.0.1:5432 only
```

---

## 12. File Permissions & .env Protection

### 12.1 Secure Application Directory

```bash
# Recommended ownership and permissions structure
sudo chown -R deploy:www-data /var/www/myapp

# Directories: rwxr-x--- (750)
sudo find /var/www/myapp -type d -exec chmod 750 {} \;

# Files: rw-r----- (640)
sudo find /var/www/myapp -type f -exec chmod 640 {} \;

# Executable scripts: rwxr-x--- (750)
sudo chmod 750 /var/www/myapp/server.js
```

### 12.2 Lock Down .env Files

```bash
# Only the app user can read .env
sudo chown nodeapp:nodeapp /var/www/myapp/.env
sudo chmod 600 /var/www/myapp/.env

# Verify
ls -la /var/www/myapp/.env
# -rw------- 1 nodeapp nodeapp  .env
```

> ⚠️ **Critical:** Never commit `.env` files to git. Add `.env` to `.gitignore` immediately.

```bash
echo ".env" >> /var/www/myapp/.gitignore
echo ".env.*" >> /var/www/myapp/.gitignore
echo "*.env" >> /var/www/myapp/.gitignore

# Check if .env was ever committed to git history
git log --all --full-history -- "**/.env"
```

### 12.3 Block .env from Web Access

Add to your Nginx config:

```nginx
# Block access to .env and other sensitive files
location ~ /\.(env|git|svn|htaccess) {
    deny all;
    return 404;
}

location ~ \.(env|log|bak|sql|config|ini|conf)$ {
    deny all;
    return 404;
}
```

### 12.4 Nginx File Permissions

```bash
# Nginx user should NOT have write access to app files
sudo chown -R root:www-data /etc/nginx
sudo chmod -R 640 /etc/nginx/nginx.conf
sudo chmod -R 640 /etc/nginx/sites-available/*
```

### 12.5 Check for World-Writable Files

```bash
# Find world-writable files (potential security risk)
sudo find / -xdev -type f -perm -0002 -ls 2>/dev/null

# Find SUID/SGID files (programs that run with elevated permissions)
sudo find / -xdev \( -perm -4000 -o -perm -2000 \) -type f -ls 2>/dev/null
```

---

## 13. Automatic Security Updates

### 13.1 Configure Unattended Upgrades

```bash
sudo apt install -y unattended-upgrades apt-listchanges

# Configure
sudo dpkg-reconfigure -plow unattended-upgrades
# Select "Yes"
```

```bash
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

```
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
    "${distro_id}ESMApps:${distro_codename}-apps-security";
    "${distro_id}ESM:${distro_codename}-infra-security";
};

// Remove unused packages after upgrade
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Remove-New-Unused-Dependencies "true";

// Reboot automatically if needed (e.g., kernel update)
// Set to "true" only if downtime is acceptable
Unattended-Upgrade::Automatic-Reboot "false";
Unattended-Upgrade::Automatic-Reboot-Time "03:00";

// Email notifications
Unattended-Upgrade::Mail "admin@yourdomain.com";
Unattended-Upgrade::MailReport "on-change";
```

```bash
sudo nano /etc/apt/apt.conf.d/20auto-upgrades
```

```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";
```

```bash
# Test the configuration
sudo unattended-upgrade --dry-run --debug

# Enable and start
sudo systemctl enable unattended-upgrades
sudo systemctl start unattended-upgrades

# Check status
sudo systemctl status unattended-upgrades
```

---

## 14. DDoS Protection & Rate Limiting

### 14.1 Nginx Rate Limiting

Rate limiting zones are defined in `nginx.conf` (see Section 6.2). Apply them in your server blocks:

```nginx
# In your server block:

# General rate limiting: 10 requests/second, burst of 20
limit_req zone=general burst=20 nodelay;

# Connection limit per IP
limit_conn conn_limit_per_ip 15;

# Specific endpoint protection — login endpoint: 2 requests/minute
location /api/auth/login {
    limit_req zone=login burst=3 nodelay;
    limit_req_status 429;
    # ... proxy_pass ...
}

# API endpoint: 5 requests/second, burst of 10
location /api/ {
    limit_req zone=api burst=10 nodelay;
    # ... proxy_pass ...
}
```

### 14.2 Block Bad User Agents and Bots

```nginx
# Add to your Nginx server block
map $http_user_agent $bad_bot {
    default         0;
    ~*malicious     1;
    ~*nikto         1;
    ~*sqlmap        1;
    ~*masscan       1;
    ~*zgrab         1;
    ~*nmap          1;
    ~*wget          1;  # Remove if you need wget-based clients
    ""              1;  # Block empty user agents
}

server {
    # ...
    if ($bad_bot) {
        return 444;
    }
}
```

### 14.3 Limit HTTP Methods

```nginx
# Only allow GET, POST, HEAD
if ($request_method !~ ^(GET|POST|HEAD|PUT|DELETE|OPTIONS)$ ) {
    return 444;
}
```

### 14.4 Protect Against Slowloris Attacks

Already covered in `nginx.conf` timeouts (Section 6.2), but reinforce with:

```nginx
keepalive_timeout 30;
keepalive_requests 100;
client_body_timeout 12;
client_header_timeout 12;
```

### 14.5 SYN Flood Protection via Kernel Parameters

```bash
sudo nano /etc/sysctl.conf
```

```
# Syn flood protection
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 5

# Disable IP source routing
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0

# Disable ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0

# Enable bad error message protection
net.ipv4.icmp_ignore_bogus_error_responses = 1

# Disable IPv6 if not needed
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
```

```bash
# Apply changes immediately
sudo sysctl -p
```

---

## 15. Cloudflare Integration

Cloudflare acts as a CDN and reverse proxy, hiding your server's real IP, filtering malicious traffic, and providing DDoS protection at the network layer.

### 15.1 Basic Setup

1. Sign up at [cloudflare.com](https://cloudflare.com)
2. Add your domain and change your registrar's nameservers to Cloudflare's
3. Enable **Proxy** (orange cloud) for your DNS A records

### 15.2 Restrict Nginx to Accept Only Cloudflare IPs

If you use Cloudflare, **only Cloudflare's IP ranges should connect to your server on port 80/443**. This prevents attackers from bypassing Cloudflare by hitting your server IP directly.

```bash
# Get Cloudflare IP ranges
curl https://www.cloudflare.com/ips-v4
curl https://www.cloudflare.com/ips-v6
```

```bash
sudo nano /etc/nginx/cloudflare-ips.conf
```

```nginx
# /etc/nginx/cloudflare-ips.conf
# Cloudflare IPv4 ranges (update periodically)
allow 173.245.48.0/20;
allow 103.21.244.0/22;
allow 103.22.200.0/22;
allow 103.31.4.0/22;
allow 141.101.64.0/18;
allow 108.162.192.0/18;
allow 190.93.240.0/20;
allow 188.114.96.0/20;
allow 197.234.240.0/22;
allow 198.41.128.0/17;
allow 162.158.0.0/15;
allow 104.16.0.0/13;
allow 104.24.0.0/14;
allow 172.64.0.0/13;
allow 131.0.72.0/22;
# Deny all others on ports 80/443
deny all;
```

Include in your Nginx server block:

```nginx
server {
    listen 443 ssl;
    server_name yourdomain.com;

    # Only accept connections from Cloudflare
    include /etc/nginx/cloudflare-ips.conf;

    # Get the real visitor IP from Cloudflare header
    real_ip_header CF-Connecting-IP;

    # ...rest of config
}
```

### 15.3 Recommended Cloudflare Security Settings

| Setting | Recommended Value |
|---|---|
| SSL/TLS mode | Full (Strict) |
| Min TLS Version | TLS 1.2 |
| HTTP Strict Transport Security | Enabled |
| Bot Fight Mode | Enabled |
| Security Level | Medium or High |
| WAF (Web Application Firewall) | Enabled (Pro+) |
| Rate Limiting | Configure per your app needs |
| Always Use HTTPS | Enabled |

---

## 16. Docker Security (If Using Docker)

### 16.1 Run Containers as Non-Root

```dockerfile
# Dockerfile
FROM node:20-alpine

# Create app directory with non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN chown -R appuser:appgroup /app

# Switch to non-root user
USER appuser

EXPOSE 3000
CMD ["node", "server.js"]
```

### 16.2 Use Docker Networks (Isolate Services)

```yaml
# docker-compose.yml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    networks:
      - frontend
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro

  app:
    build: .
    # NO ports exposed — only accessible through internal network
    networks:
      - frontend
      - backend
    environment:
      - NODE_ENV=production
    env_file:
      - .env

  db:
    image: postgres:15-alpine
    # NO ports exposed — database inaccessible from outside
    networks:
      - backend
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password

networks:
  frontend:
  backend:
    internal: true  # No internet access for backend network

volumes:
  postgres_data:

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### 16.3 Docker Security Best Practices

```bash
# Never run Docker as root
# Add your user to the docker group
sudo usermod -aG docker deploy

# Keep Docker and images updated
sudo apt upgrade docker-ce docker-ce-cli containerd.io

# Scan images for vulnerabilities
docker scout cves myapp:latest

# Limit container resources
docker run --memory="512m" --cpus="1.0" myapp

# Read-only filesystem where possible
docker run --read-only myapp

# Audit running containers
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}"

# Inspect container network exposure
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container_name
```

---

## 17. Backup Strategy

**The #1 rule:** Your backup is only as good as your last successful **restore test**.

### 17.1 Database Backups

```bash
# MySQL backup script
cat > /usr/local/bin/backup-mysql.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/var/backups/mysql"
DATE=$(date +%Y%m%d_%H%M%S)
DB_NAME="myapp_production"
DB_USER="backup_user"
DB_PASS="YOUR_BACKUP_USER_PASSWORD"

mkdir -p "$BACKUP_DIR"

mysqldump -u "$DB_USER" -p"$DB_PASS" \
  --single-transaction \
  --routines \
  --triggers \
  "$DB_NAME" | gzip > "$BACKUP_DIR/${DB_NAME}_${DATE}.sql.gz"

# Keep only last 30 days
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +30 -delete

echo "Backup completed: ${DB_NAME}_${DATE}.sql.gz"
EOF

chmod +x /usr/local/bin/backup-mysql.sh
```

```bash
# PostgreSQL backup script
cat > /usr/local/bin/backup-postgres.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/var/backups/postgresql"
DATE=$(date +%Y%m%d_%H%M%S)
DB_NAME="myapp_production"

mkdir -p "$BACKUP_DIR"

sudo -u postgres pg_dump -Fc "$DB_NAME" \
  > "$BACKUP_DIR/${DB_NAME}_${DATE}.dump"

# Keep only last 30 days
find "$BACKUP_DIR" -name "*.dump" -mtime +30 -delete
EOF

chmod +x /usr/local/bin/backup-postgres.sh
```

### 17.2 Application File Backup

```bash
cat > /usr/local/bin/backup-app.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/var/backups/app"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

tar -czf "$BACKUP_DIR/app_${DATE}.tar.gz" \
  --exclude='/var/www/myapp/node_modules' \
  --exclude='/var/www/myapp/.git' \
  /var/www/myapp \
  /etc/nginx/sites-available \
  /etc/nginx/nginx.conf

find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete
EOF

chmod +x /usr/local/bin/backup-app.sh
```

### 17.3 Schedule Backups with Cron

```bash
sudo crontab -e
```

```
# Daily DB backup at 2 AM
0 2 * * * /usr/local/bin/backup-mysql.sh >> /var/log/backup.log 2>&1

# Daily app backup at 2:30 AM
30 2 * * * /usr/local/bin/backup-app.sh >> /var/log/backup.log 2>&1
```

### 17.4 Off-Site Backup with rclone

```bash
# Install rclone
curl https://rclone.org/install.sh | sudo bash

# Configure a remote (e.g., AWS S3, Google Drive, Backblaze B2)
rclone config

# Sync backups to remote
rclone sync /var/backups remote:my-server-backups

# Add to cron (after local backup):
0 4 * * * rclone sync /var/backups remote:my-server-backups >> /var/log/rclone.log 2>&1
```

### 17.5 Test Restores Regularly

```bash
# MySQL restore test
gunzip -c /var/backups/mysql/myapp_production_20240101.sql.gz | mysql -u root -p test_restore_db

# PostgreSQL restore test
pg_restore -U postgres -d test_restore_db /var/backups/postgresql/myapp_production_20240101.dump
```

---

## 18. Monitoring, Logs & Alerts

### 18.1 System Log Files to Know

| Log File | Purpose |
|---|---|
| `/var/log/auth.log` | SSH logins, sudo usage, authentication events |
| `/var/log/syslog` | General system messages |
| `/var/log/nginx/access.log` | All HTTP requests to Nginx |
| `/var/log/nginx/error.log` | Nginx errors, upstream failures |
| `/var/log/fail2ban.log` | Fail2Ban ban/unban events |
| `/var/log/dpkg.log` | Package install/update history |
| `/var/log/ufw.log` | UFW firewall events (if enabled) |
| `/var/log/mysql/error.log` | MySQL errors |
| `/var/log/postgresql/` | PostgreSQL logs |
| `~/.pm2/logs/` | PM2 application logs |

### 18.2 Log Monitoring Commands

```bash
# Watch real-time Nginx access log
sudo tail -f /var/log/nginx/access.log

# Watch for failed SSH attempts
sudo tail -f /var/log/auth.log | grep -i "failed\|invalid\|error"

# Watch all system logs
sudo journalctl -f

# View recent Fail2Ban activity
sudo fail2ban-client status sshd

# Count requests per IP in Nginx access log (detect floods)
sudo awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -20

# Find 404 errors (potential vulnerability scanning)
sudo grep " 404 " /var/log/nginx/access.log | awk '{print $7}' | sort | uniq -c | sort -rn | head -20
```

### 18.3 Install and Configure Logwatch

```bash
sudo apt install -y logwatch

# Run a report (outputs to terminal)
sudo logwatch --output stdout --format text --range today --detail high

# Schedule daily email report
echo "/usr/sbin/logwatch --output mail --mailto admin@yourdomain.com --detail high" \
  | sudo tee /etc/cron.daily/logwatch
```

### 18.4 Set Up Uptime Monitoring

Use a free external monitoring service:

- **UptimeRobot** (uptimerobot.com) — 5-minute checks, free
- **Freshping** (freshping.io) — 1-minute checks, free tier
- **BetterUptime** (betteruptime.com) — incident management

These services check your site from outside and alert you via email/SMS if it goes down.

### 18.5 Disk Space and Memory Monitoring

```bash
# Check disk usage
df -h

# Check memory usage
free -h

# Check top processes by CPU/memory
htop

# Set up automatic disk space alert
cat > /usr/local/bin/disk-alert.sh << 'EOF'
#!/bin/bash
THRESHOLD=85
USAGE=$(df / | tail -1 | awk '{print $5}' | tr -d '%')

if [ "$USAGE" -gt "$THRESHOLD" ]; then
    echo "ALERT: Disk usage is at ${USAGE}% on $(hostname)" \
      | mail -s "Disk Space Alert" admin@yourdomain.com
fi
EOF

chmod +x /usr/local/bin/disk-alert.sh

# Run daily
echo "0 8 * * * /usr/local/bin/disk-alert.sh" | sudo crontab -
```

---

## 19. Malware & Rootkit Scanning

### 19.1 Install Scanning Tools

```bash
# Rootkit Hunter
sudo apt install -y rkhunter

# ClamAV antivirus
sudo apt install -y clamav clamav-daemon

# Chkrootkit
sudo apt install -y chkrootkit

# Lynis — security auditing tool
sudo apt install -y lynis
```

### 19.2 Run Initial Scans

```bash
# Update rkhunter database
sudo rkhunter --update

# Run rkhunter scan
sudo rkhunter --check --skip-keypress

# Update ClamAV signatures
sudo freshclam

# Scan web root with ClamAV
sudo clamscan -r --remove /var/www/ 2>&1 | grep -E "FOUND|Infected"

# Run chkrootkit
sudo chkrootkit

# Run Lynis security audit
sudo lynis audit system
# Results in: /var/log/lynis.log and /var/log/lynis-report.dat
```

### 19.3 Schedule Regular Scans

```bash
# Weekly rkhunter scan
echo "0 3 * * 0 /usr/bin/rkhunter --check --skip-keypress --report-warnings-only | mail -s 'rkhunter report' admin@yourdomain.com" \
  | sudo crontab -

# Daily ClamAV update
echo "0 2 * * * /usr/bin/freshclam >> /var/log/freshclam.log 2>&1" | sudo crontab -
```

---

## 20. Checking Open Ports & Active Connections

### 20.1 View All Listening Ports

```bash
# Show all listening TCP and UDP ports with process names
sudo ss -tlunp

# Alternative (older systems)
sudo netstat -tlunp

# Show only listening TCP ports
sudo ss -tlnp

# Show which process owns a specific port
sudo ss -tlnp sport = :3000
sudo fuser 3000/tcp
```

### 20.2 Scan Your Own Server from Outside

```bash
# Scan from another machine or use online scanner
# Quick TCP scan
nmap -sT -p 1-65535 YOUR_SERVER_IP

# Service detection (aggressive — use carefully, may trigger IDS)
nmap -sV -p 22,80,443 YOUR_SERVER_IP

# Check if a specific port is reachable
nc -zv YOUR_SERVER_IP 3306
# If MySQL is properly secured, this should FAIL
```

### 20.3 View Active Connections

```bash
# All active connections
sudo ss -tunap

# Count connections per remote IP
sudo ss -tn | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn | head

# Active connections to Nginx
sudo ss -tn state established '( dport = :80 or dport = :443 )'

# Watch connections in real time
watch -n 1 'ss -tn | grep ESTABLISHED | wc -l'
```

---

## 21. Detecting Suspicious Activity

### 21.1 Check for Unauthorized Users and Processes

```bash
# Who is currently logged in?
who
w
last | head -20

# Check for unexpected user accounts
cat /etc/passwd | awk -F: '$3 >= 1000 && $1 != "nobody" {print $1}'

# Check for users with root-level privileges
grep -v "^#" /etc/sudoers
sudo cat /etc/sudoers.d/*

# Find processes running as root
ps aux | grep "^root" | grep -v "PID\|\[" | head -30

# List all running processes with user
ps auxf
```

### 21.2 Check for Suspicious Cron Jobs

```bash
# View all cron jobs for all users
for user in $(cut -f1 -d: /etc/passwd); do
  crontab -u "$user" -l 2>/dev/null | grep -v "^#" | sed "s/^/$user: /"
done

# System-wide cron jobs
ls -la /etc/cron.d/
ls -la /etc/cron.daily/
ls -la /etc/cron.weekly/
cat /etc/crontab
```

### 21.3 Check for Hidden Files and Unusual Activity

```bash
# Recently modified files in web root (last 24 hours)
sudo find /var/www -type f -mtime -1 -ls

# Files modified in /etc in the last 7 days
sudo find /etc -type f -mtime -7 -ls 2>/dev/null

# Check for recently installed packages
dpkg --get-selections | tail -20
cat /var/log/dpkg.log | grep "install " | tail -20

# Check startup scripts for unusual entries
ls -la /etc/init.d/
systemctl list-units --type=service --state=active

# Check /tmp for unusual files (malware often drops files here)
ls -la /tmp/
ls -la /var/tmp/
```

### 21.4 Audit SSH Login History

```bash
# Successful SSH logins
grep "Accepted" /var/log/auth.log | tail -50

# Failed SSH attempts with IP
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn | head -20

# Find logins from unusual IPs
last | grep -v "reboot\|wtmp" | head -30
```

---

## 22. Common Beginner Mistakes

> These are the most frequently exploited misconfigurations in beginner production deployments.

### ❌ Mistake 1: Running Node.js Directly on Port 80/443

```bash
# WRONG — requires root, and exposes Node.js directly to internet
sudo node server.js --port 80

# CORRECT — use Nginx as reverse proxy, Node.js on localhost
node server.js --port 3000  # Nginx handles 80/443
```

### ❌ Mistake 2: Leaving Database Ports Open to the Internet

```bash
# WRONG — exposes MySQL to brute force from anywhere
sudo ufw allow 3306

# CORRECT — database should only be accessible from localhost
# (No ufw rule needed for localhost connections)
```

### ❌ Mistake 3: Storing Credentials in Code

```javascript
// WRONG
const db = mysql.createConnection({ password: 'mysecretpassword' });

// CORRECT
const db = mysql.createConnection({ password: process.env.DB_PASSWORD });
```

### ❌ Mistake 4: Running Everything as Root

```bash
# WRONG
ssh root@server
pm2 start app.js  # Running as root

# CORRECT
ssh deploy@server
pm2 start app.js  # Running as deploy or nodeapp user
```

### ❌ Mistake 5: Forgetting to Test SSH Config Before Restarting sshd

Always open a **second terminal** and test connectivity before closing the first.

### ❌ Mistake 6: Ignoring `server_tokens`

Without `server_tokens off`, Nginx announces its version in every HTTP response header, making it easy for attackers to find version-specific exploits.

### ❌ Mistake 7: Using HTTP Instead of HTTPS for Internal APIs

Even internal services should use TLS if they handle sensitive data. At minimum, your public-facing app must force HTTPS.

### ❌ Mistake 8: Weak or Reused Passwords for Database Users

Use a password manager to generate and store 30+ character random passwords for database users. These passwords are only ever stored in `.env` files — you never type them manually.

### ❌ Mistake 9: Not Setting Up Automatic Backups Before Going Live

A production server with no backup strategy is not production-ready.

### ❌ Mistake 10: Disabling Fail2Ban Because "It Banned Me Once"

```bash
# Instead of disabling Fail2Ban, add your IP to the ignore list:
sudo nano /etc/fail2ban/jail.local
# Add: ignoreip = 127.0.0.1/8 YOUR_HOME_IP/32
```

---

## 23. VPS Security Checklist

Use this checklist when setting up a new production server or auditing an existing one.

### Initial Hardening

- [ ] System updated: `apt update && apt upgrade -y`
- [ ] Non-root sudo user created
- [ ] SSH key authentication configured
- [ ] Root SSH login disabled (`PermitRootLogin no`)
- [ ] Password authentication disabled (`PasswordAuthentication no`)
- [ ] SSH idle timeout configured (`ClientAliveInterval 300`)
- [ ] UFW enabled with deny-by-default policy
- [ ] Only ports 22 (or custom), 80, and 443 open in UFW
- [ ] Fail2Ban installed and configured for SSH and Nginx
- [ ] Automatic security updates configured

### Nginx

- [ ] `server_tokens off` in nginx.conf
- [ ] Default server block returns 444 for unknown hostnames
- [ ] Direct IP access blocked
- [ ] Security headers configured (X-Frame-Options, CSP, HSTS, etc.)
- [ ] Rate limiting zones defined and applied
- [ ] Sensitive file extensions blocked (.env, .log, .sql)
- [ ] Nginx config tested with `nginx -t`

### SSL

- [ ] Let's Encrypt certificate installed
- [ ] HTTP redirects to HTTPS
- [ ] HSTS header enabled
- [ ] SSL rating is A or A+ (ssllabs.com)
- [ ] Certbot auto-renewal tested with `--dry-run`
- [ ] TLS 1.0 and 1.1 disabled

### Node.js & Application

- [ ] Node.js app binds to `127.0.0.1` only, not `0.0.0.0`
- [ ] `.env` file has `600` permissions
- [ ] `.env` is in `.gitignore`
- [ ] No secrets hardcoded in source code
- [ ] Input validation and sanitization implemented
- [ ] PM2 running as non-root user
- [ ] Helmet.js security headers applied
- [ ] Rate limiting in application layer

### Database

- [ ] MySQL/PostgreSQL bound to `127.0.0.1` only
- [ ] Database port not open in UFW
- [ ] Root database login disabled for remote access
- [ ] Dedicated low-privilege database user for application
- [ ] `mysql_secure_installation` completed (MySQL)
- [ ] Strong database passwords stored in `.env`

### Backups & Monitoring

- [ ] Automated daily database backups configured
- [ ] Backups include application files and Nginx config
- [ ] Off-site backup solution configured
- [ ] Backup restore tested at least once
- [ ] External uptime monitoring configured
- [ ] Log monitoring configured

### Advanced

- [ ] rkhunter / chkrootkit installed and baseline taken
- [ ] Lynis audit run and score reviewed
- [ ] Cloudflare proxy enabled (recommended)
- [ ] Only Cloudflare IPs allowed on ports 80/443 (if using Cloudflare)
- [ ] Docker containers run as non-root (if using Docker)
- [ ] All containers on isolated internal networks (if using Docker)

---

## 24. Maintenance Schedule

### Daily (Automated or ~5 minutes manual)

```bash
# Check for failed services
systemctl --failed

# Review Fail2Ban bans
sudo fail2ban-client status sshd

# Check disk space
df -h

# Scan error logs for anomalies
sudo tail -50 /var/log/nginx/error.log
sudo tail -50 /var/log/auth.log | grep -i "fail\|invalid\|error"

# Check PM2 status
pm2 status
```

### Weekly (~15 minutes)

```bash
# Review all security logs
sudo logwatch --output stdout --range week --detail high

# Check for unusual open ports
sudo ss -tlunp

# Update system packages
sudo apt update && sudo apt list --upgradable

# Review Nginx access logs for scanning activity
sudo awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -20

# Run rkhunter
sudo rkhunter --check --skip-keypress

# Verify backups completed
ls -lh /var/backups/mysql/
ls -lh /var/backups/app/

# Review active user accounts
cat /etc/passwd | awk -F: '$3 >= 1000 {print $1}'
```

### Monthly (~30 minutes)

```bash
# Full system update
sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y

# Update Node.js dependencies and audit
cd /var/www/myapp && npm audit
npm audit fix

# Update PM2
sudo npm install -g pm2@latest
pm2 update

# Run Lynis full audit
sudo lynis audit system

# Run ClamAV full scan
sudo freshclam
sudo clamscan -r /var/www/

# Test backup restore (to a test database)
gunzip -c /var/backups/mysql/latest.sql.gz | mysql -u root -p test_restore

# Review and rotate log files
sudo logrotate -f /etc/logrotate.conf

# Check SSL certificate expiry
sudo certbot certificates

# Review UFW rules — remove any that are no longer needed
sudo ufw status numbered

# Check Fail2Ban configuration for newly needed jails
```

---

## 25. Troubleshooting

### SSH: "Connection Refused" After Changing Port

```bash
# Problem: Changed SSH port but forgot to add UFW rule first
# Solution: Access server via VPS console (provider's web interface)

# On console, add the UFW rule
sudo ufw allow 2222/tcp

# Reload SSH daemon
sudo systemctl reload sshd
```

### Nginx: "502 Bad Gateway"

```bash
# Check if Node.js app is running
pm2 status
pm2 logs myapp --lines 50

# Check what's listening on the app port
sudo ss -tlnp | grep 3000

# Check Nginx error log
sudo tail -20 /var/log/nginx/error.log

# Common causes:
# 1. Node.js app not running → pm2 restart myapp
# 2. Wrong proxy_pass address in nginx.conf → should be 127.0.0.1:PORT
# 3. App crashed → check PM2 logs for errors
```

### Nginx: "413 Request Entity Too Large"

```nginx
# Increase the client_max_body_size in nginx.conf or server block
client_max_body_size 50m;  # Adjust as needed
```

### Certbot: Certificate Renewal Fails

```bash
# Check Certbot logs
sudo cat /var/log/letsencrypt/letsencrypt.log | tail -50

# Verify port 80 is accessible (needed for HTTP challenge)
sudo ufw status | grep 80

# Make sure Nginx is running
sudo systemctl status nginx

# Try manual renewal
sudo certbot renew --force-renewal --nginx

# Check if domain DNS resolves to this server
dig +short yourdomain.com
```

### Fail2Ban: Accidentally Banned Your Own IP

```bash
# Unban your IP from all jails
sudo fail2ban-client unban YOUR_IP

# Or unban from specific jail
sudo fail2ban-client set sshd unbanip YOUR_IP

# Prevent future self-bans
sudo nano /etc/fail2ban/jail.local
# Add to [DEFAULT]:
# ignoreip = 127.0.0.1/8 YOUR_HOME_IP/32
sudo systemctl reload fail2ban
```

### MySQL: "Access Denied" from Application

```bash
# Verify user exists and has correct host
sudo mysql -u root -p
```

```sql
SELECT user, host FROM mysql.user WHERE user = 'myapp_user';
SHOW GRANTS FOR 'myapp_user'@'localhost';

-- If wrong host, recreate the user:
DROP USER 'myapp_user'@'localhost';
CREATE USER 'myapp_user'@'localhost' IDENTIFIED BY 'password';
GRANT SELECT, INSERT, UPDATE, DELETE ON myapp_production.* TO 'myapp_user'@'localhost';
FLUSH PRIVILEGES;
```

### Server Under Heavy Load / Suspected DDoS

```bash
# Find top attacking IPs
sudo awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -20

# Block an IP immediately
sudo ufw deny from ATTACKING_IP comment "DDoS block $(date)"

# Temporarily reduce Nginx rate limits
# Edit nginx.conf, lower burst values, reload

# Check if Fail2Ban is active
sudo fail2ban-client status nginx-limit-req

# Check system resources
top
htop
vmstat 1 5
```

---

> **Final Note:** Security is not a one-time task. The threat landscape evolves constantly. Stay subscribed to Ubuntu Security Notices (usn.ubuntu.com), follow your stack's security advisories (Node.js, Nginx, MySQL), and conduct regular audits using this guide's monthly maintenance checklist.

---

*Guide version: 1.0 | Compatible with: Ubuntu 22.04 LTS, Ubuntu 24.04 LTS | Last reviewed: 2025*
