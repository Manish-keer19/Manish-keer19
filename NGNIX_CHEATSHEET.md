# NGINX Beginner to Advanced Cheatsheet

Complete guide from basics to production deployment.

---

## 1. Basics (Beginner)

### What is NGINX?

NGINX (pronounced "engine-x") is a **web server** that can also act as a:
- **Reverse Proxy**: Routes requests to backend servers
- **Load Balancer**: Distributes traffic across multiple servers
- **HTTP Cache**: Stores responses to speed up delivery
- **Static File Server**: Serves HTML, CSS, JS, images directly

Think of NGINX as a **traffic controller** for your web applications.

---

### Why use NGINX?

1. **High Performance**: Handles 10,000+ concurrent connections efficiently
2. **Low Memory**: Uses event-driven architecture (not thread-based like Apache)
3. **Static Files**: Serves files faster than Node.js/Python/PHP
4. **SSL Termination**: Handles HTTPS so your backend doesn't have to
5. **Load Balancing**: Distribute traffic across multiple backend servers
6. **Caching**: Store responses to reduce backend load
7. **Security**: Add rate limiting, IP filtering, DDoS protection

---

### NGINX vs Node.js

```
┌─────────────────────────────────────────────────────────┐
│                     USER BROWSER                        │
└────────────────────┬────────────────────────────────────┘
                     │ HTTP Request
                     ▼
         ┌───────────────────────┐
         │   NGINX (Port 80/443) │  ← Handles SSL, static files,
         │   Web Server          │    routing, load balancing
         └───────────┬───────────┘
                     │ Proxy to backend
                     ▼
         ┌───────────────────────┐
         │  Node.js (Port 3000)  │  ← Handles business logic,
         │  Application Server   │    database, APIs
         └───────────────────────┘
```

**Key Differences**:
- **NGINX**: Fast at serving static files, SSL, routing
- **Node.js**: Good at business logic, APIs, database operations
- **Best Practice**: Use NGINX in front of Node.js

---

### What is Reverse Proxy?

A **reverse proxy** sits between clients and your backend server.

```
WITHOUT Reverse Proxy:
User → http://yoursite.com:3000 → Node.js App

WITH Reverse Proxy (NGINX):
User → http://yoursite.com → NGINX → Node.js App (port 3000)
```

**Benefits**:
- Hide backend server details (security)
- Handle SSL certificates (one place)
- Serve multiple apps from one domain
- Load balancing

**Example**:
```nginx
# NGINX receives request on port 80
# Forwards to Node.js on port 3000
location /api {
    proxy_pass http://localhost:3000;
}
```

---

### What is Load Balancer?

Distributes incoming traffic across **multiple backend servers**.

```
                    ┌─────────────┐
         ┌─────────►│  Server 1   │
         │          │  Port 3001  │
         │          └─────────────┘
┌────────┴─────┐    ┌─────────────┐
│    NGINX     ├───►│  Server 2   │
│ Load Balancer│    │  Port 3002  │
└────────┬─────┘    └─────────────┘
         │          ┌─────────────┐
         └─────────►│  Server 3   │
                    │  Port 3003  │
                    └─────────────┘
```

**Example**:
```nginx
upstream backend {
    server localhost:3001;
    server localhost:3002;
    server localhost:3003;
}

server {
    location / {
        proxy_pass http://backend;
    }
}
```

---

### What is Upstream?

An **upstream** block defines a group of backend servers for load balancing.

```nginx
upstream myapp {
    server 192.168.1.10:3000;  # Server 1
    server 192.168.1.11:3000;  # Server 2
    server 192.168.1.12:3000 backup;  # Backup server
}

server {
    location / {
        proxy_pass http://myapp;  # References the upstream
    }
}
```

**Load Balancing Methods**:
```nginx
upstream myapp {
    # Round Robin (default) - distributes evenly
    server srv1.example.com;
    server srv2.example.com;
    
    # Least Connections - sends to server with fewest connections
    least_conn;
    
    # IP Hash - same client always goes to same server
    ip_hash;
    
    # Weighted - srv1 gets 3x more traffic than srv2
    server srv1.example.com weight=3;
    server srv2.example.com weight=1;
}
```

---

### What is Location Block?

A **location block** defines how NGINX handles requests to specific URLs.

```nginx
server {
    listen 80;
    
    # Exact match: only /about
    location = /about {
        return 200 "About page";
    }
    
    # Prefix match: /api, /api/users, /api/posts
    location /api {
        proxy_pass http://localhost:3000;
    }
    
    # Regex match: any .jpg, .png, .gif
    location ~* \.(jpg|png|gif)$ {
        expires 30d;
    }
    
    # Default fallback
    location / {
        root /var/www/html;
        try_files $uri $uri/ /index.html;
    }
}
```

**Location Matching Priority**:
1. `=` Exact match (highest priority)
2. `^~` Prefix match (no regex check)
3. `~` Regex (case-sensitive)
4. `~*` Regex (case-insensitive)
5. `/` Prefix match (lowest priority)

---

### What is Server Block?

A **server block** defines configuration for a specific domain or port.

```nginx
# Server block for website.com
server {
    listen 80;
    server_name website.com www.website.com;
    root /var/www/website;
    
    location / {
        try_files $uri /index.html;
    }
}

# Server block for api.website.com
server {
    listen 80;
    server_name api.website.com;
    
    location / {
        proxy_pass http://localhost:3000;
    }
}

# Default server (catches all other domains)
server {
    listen 80 default_server;
    return 444;  # Drop connection
}
```

---

### Default NGINX Folder Structure

```
/etc/nginx/
├── nginx.conf                 # Main config file
├── sites-available/           # All site configs
│   ├── default
│   ├── myapp.conf
│   └── api.conf
├── sites-enabled/             # Enabled site configs (symlinks)
│   └── default -> ../sites-available/default
├── conf.d/                    # Additional configs
├── snippets/                  # Reusable config snippets
│   └── ssl-params.conf
└── modules-enabled/           # Enabled modules

/var/log/nginx/
├── access.log                 # Request logs
└── error.log                  # Error logs

/var/www/
└── html/                      # Default web root
    └── index.html
```

**Key Files**:
- `/etc/nginx/nginx.conf` - Main configuration
- `/etc/nginx/sites-available/` - Store configs here
- `/etc/nginx/sites-enabled/` - Symlink active configs here
- `/var/log/nginx/` - Log files

---

### Common Commands

```bash
# Check NGINX version
nginx -v

# Test configuration (ALWAYS do this before reload!)
sudo nginx -t

# Start NGINX
sudo systemctl start nginx

# Stop NGINX
sudo systemctl stop nginx

# Restart NGINX (stops then starts)
sudo systemctl restart nginx

# Reload config (no downtime!)
sudo systemctl reload nginx

# Enable NGINX on boot
sudo systemctl enable nginx

# Check status
sudo systemctl status nginx

# View error logs
sudo tail -f /var/log/nginx/error.log

# View access logs
sudo tail -f /var/log/nginx/access.log

# Check which process is using port 80
sudo lsof -i :80

# Kill NGINX process
sudo pkill nginx
```

**Pro Tip**: Always use `sudo nginx -t` before reloading to catch config errors!

---

## 2. NGINX Configuration Deep-Dive

### Server Block Full Explanation

```nginx
# Complete server block with explanations
server {
    # Port to listen on
    listen 80;                          # IPv4
    listen [::]:80;                     # IPv6
    
    # Domain names this server responds to
    server_name example.com www.example.com;
    
    # Document root (where files are served from)
    root /var/www/example;
    
    # Default file to serve
    index index.html index.htm;
    
    # Access log location
    access_log /var/log/nginx/example.access.log;
    error_log /var/log/nginx/example.error.log;
    
    # Charset
    charset utf-8;
    
    # Client body size limit (for file uploads)
    client_max_body_size 10M;
    
    # Location blocks
    location / {
        try_files $uri $uri/ =404;
    }
    
    # Custom error pages
    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
}
```

---

### Locations, Regex, Routing

#### Basic Location Types

```nginx
server {
    # 1. EXACT MATCH (=)
    location = /about {
        return 200 "Exact match for /about only";
    }
    
    # 2. PREFIX MATCH (^~)
    location ^~ /images/ {
        root /var/www;
        # Matches /images/logo.png but stops regex checking
    }
    
    # 3. REGEX CASE-SENSITIVE (~)
    location ~ \.php$ {
        # Matches file.php, index.PHP will NOT match
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
    }
    
    # 4. REGEX CASE-INSENSITIVE (~*)
    location ~* \.(jpg|jpeg|png|gif|ico)$ {
        # Matches .JPG, .jpg, .PNG, etc.
        expires 30d;
    }
    
    # 5. PREFIX MATCH (default)
    location /api/ {
        proxy_pass http://localhost:3000;
    }
    
    # 6. FALLBACK
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

#### Advanced Routing Examples

```nginx
# Route API to backend
location /api {
    # Remove /api prefix when proxying
    rewrite ^/api/(.*)$ /$1 break;
    proxy_pass http://localhost:3000;
}

# Route by file extension
location ~* \.(css|js)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}

# Block access to hidden files
location ~ /\. {
    deny all;
}

# SPA routing (React, Vue, Angular)
location / {
    try_files $uri $uri/ /index.html;
    # If file doesn't exist, serve index.html
}

# Redirect www to non-www
server {
    listen 80;
    server_name www.example.com;
    return 301 http://example.com$request_uri;
}
```

#### Named Locations

```nginx
location / {
    try_files $uri $uri/ @backend;
}

location @backend {
    proxy_pass http://localhost:3000;
}
```

---

### proxy_pass Explained with Diagrams

#### Basic proxy_pass

```nginx
location /api {
    proxy_pass http://localhost:3000;
}
```

**Request Flow**:
```
User → http://example.com/api/users
              ↓
NGINX receives: /api/users
              ↓
Proxies to: http://localhost:3000/api/users
```

#### proxy_pass with trailing slash

```nginx
# WITHOUT trailing slash
location /api {
    proxy_pass http://localhost:3000;
}
# Request: /api/users → Backend: /api/users

# WITH trailing slash
location /api/ {
    proxy_pass http://localhost:3000/;
}
# Request: /api/users → Backend: /users (prefix removed!)
```

#### Complete Proxy Configuration

```nginx
location /api {
    # Backend server
    proxy_pass http://localhost:3000;
    
    # Pass original host header
    proxy_set_header Host $host;
    
    # Pass real client IP
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    
    # Pass protocol (http or https)
    proxy_set_header X-Forwarded-Proto $scheme;
    
    # Timeouts
    proxy_connect_timeout 60s;
    proxy_send_timeout 60s;
    proxy_read_timeout 60s;
    
    # Buffering
    proxy_buffering on;
    proxy_buffer_size 4k;
    proxy_buffers 8 4k;
    
    # Redirect handling
    proxy_redirect off;
}
```

#### Diagram: How proxy_pass Works

```
┌──────────┐  1. Request      ┌────────────────┐
│  Client  ├─────────────────►│ NGINX (Port 80)│
│ Browser  │  GET /api/users  └────────┬───────┘
└──────────┘                           │
                                       │ 2. proxy_pass
                                       │    adds headers
                                       ▼
                            ┌──────────────────────┐
                            │  Backend Node.js     │
                            │  (Port 3000)         │
                            │  Receives:           │
                            │  GET /api/users      │
                            │  X-Real-IP: 1.2.3.4  │
                            │  Host: example.com   │
                            └──────────┬───────────┘
                                       │ 3. Response
                                       │    JSON data
                                       ▼
┌──────────┐  4. Send to    ┌────────────────┐
│  Client  │◄───────────────┤ NGINX          │
│ Browser  │  client        │ (may cache)    │
└──────────┘                └────────────────┘
```

---

### gzip, Caching, Security Headers

#### gzip Compression

```nginx
# In /etc/nginx/nginx.conf or server block
gzip on;
gzip_vary on;
gzip_proxied any;
gzip_comp_level 6;  # 1-9, higher = more compression
gzip_types
    text/plain
    text/css
    text/xml
    text/javascript
    application/json
    application/javascript
    application/xml+rss
    application/rss+xml
    font/truetype
    font/opentype
    application/vnd.ms-fontobject
    image/svg+xml;
gzip_disable "msie6";  # Disable for old IE

# Minimum file size to compress (bytes)
gzip_min_length 256;
```

**Result**: Reduces file sizes by 60-80%!

#### Caching Static Files

```nginx
# Cache images, CSS, JS for 1 year
location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
    access_log off;  # Don't log static file requests
}

# Cache HTML for 1 hour
location ~* \.html$ {
    expires 1h;
    add_header Cache-Control "public";
}

# Don't cache API responses
location /api {
    add_header Cache-Control "no-store, no-cache, must-revalidate";
    proxy_pass http://localhost:3000;
}
```

#### Security Headers

```nginx
# Add to server block or location
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "no-referrer-when-downgrade" always;
add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;

# HSTS (force HTTPS for 1 year)
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```

#### Complete Performance Config

```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/html;
    
    # gzip compression
    gzip on;
    gzip_comp_level 6;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;
    
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    
    # Cache static assets
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # Main content
    location / {
        try_files $uri /index.html;
    }
}
```

---

### Rate Limiting

Protects your server from abuse and DDoS attacks.

#### Basic Rate Limiting

```nginx
# Define rate limit zone (in http block)
http {
    # Allow 10 requests per second per IP
    limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;
    
    server {
        location /api {
            # Apply rate limit
            limit_req zone=mylimit burst=20 nodelay;
            
            # burst=20: Allow up to 20 extra requests in queue
            # nodelay: Process burst requests immediately
            
            proxy_pass http://localhost:3000;
        }
    }
}
```

#### Advanced Rate Limiting

```nginx
http {
    # Different limits for different endpoints
    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
    limit_req_zone $binary_remote_addr zone=login_limit:10m rate=5r/m;
    limit_req_zone $binary_remote_addr zone=general:10m rate=100r/s;
    
    # Connection limits
    limit_conn_zone $binary_remote_addr zone=conn_limit:10m;
    
    server {
        # General rate limit
        location / {
            limit_req zone=general burst=50;
            root /var/www/html;
        }
        
        # Strict limit for login
        location /api/login {
            limit_req zone=login_limit burst=2 nodelay;
            proxy_pass http://localhost:3000;
        }
        
        # API rate limit
        location /api {
            limit_req zone=api_limit burst=20 nodelay;
            limit_conn conn_limit 10;  # Max 10 concurrent connections
            proxy_pass http://localhost:3000;
        }
    }
}
```

#### Rate Limit Status Codes

```nginx
# Return custom error for rate limit
limit_req_status 429;
limit_conn_status 429;

# Custom error page
error_page 429 /rate_limit.html;
location = /rate_limit.html {
    return 429 "Too many requests. Please try again later.\n";
}
```

---

### SSL with Certbot (Step-by-Step)

#### Install Certbot

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install certbot python3-certbot-nginx

# CentOS/RHEL
sudo yum install certbot python3-certbot-nginx
```

#### Basic NGINX Config (HTTP only, before SSL)

```bash
# Create config file
sudo nano /etc/nginx/sites-available/example.com
```

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    
    root /var/www/example;
    index index.html;
    
    location / {
        try_files $uri $uri/ =404;
    }
}
```

```bash
# Enable site
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/

# Test config
sudo nginx -t

# Reload
sudo systemctl reload nginx
```

#### Get SSL Certificate

```bash
# Automatic configuration (recommended)
sudo certbot --nginx -d example.com -d www.example.com

# Follow prompts:
# 1. Enter email
# 2. Agree to terms
# 3. Choose whether to redirect HTTP to HTTPS (choose 2)
```

#### Result: Certbot Auto-Updates Config

```nginx
server {
    server_name example.com www.example.com;
    root /var/www/example;
    index index.html;
    
    location / {
        try_files $uri $uri/ =404;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

# Redirect HTTP to HTTPS
server {
    if ($host = www.example.com) {
        return 301 https://$host$request_uri;
    }
    
    if ($host = example.com) {
        return 301 https://$host$request_uri;
    }
    
    listen 80;
    server_name example.com www.example.com;
    return 404;
}
```

#### Manual SSL Configuration (if not using Certbot auto-config)

```nginx
server {
    listen 443 ssl http2;
    server_name example.com www.example.com;
    
    # SSL certificates
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    
    # SSL settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=31536000" always;
    
    root /var/www/example;
    index index.html;
    
    location / {
        try_files $uri $uri/ =404;
    }
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;
}
```

#### Auto-Renewal

Certbot automatically sets up a renewal cron job.

```bash
# Test renewal
sudo certbot renew --dry-run

# Check renewal timer
sudo systemctl status certbot.timer

# Manual renewal
sudo certbot renew
```

#### Troubleshooting SSL

```bash
# Check certificate expiry
sudo certbot certificates

# Delete certificate
sudo certbot delete --cert-name example.com

# Re-issue certificate
sudo certbot --nginx -d example.com
```

---

## 3. Reverse Proxy for Node.js

### Serve Node.js Backend on Port 3000

#### Simple Node.js App

```javascript
// app.js
const express = require('express');
const app = express();

app.get('/', (req, res) => {
    res.json({ message: 'Hello from Node.js!' });
});

app.get('/api/users', (req, res) => {
    res.json({ users: ['Alice', 'Bob', 'Charlie'] });
});

app.listen(3000, () => {
    console.log('Server running on port 3000');
});
```

```bash
# Run directly
node app.js

# Run with PM2 (production)
pm2 start app.js --name api
```

---

### Reverse Proxy using NGINX on Port 80

```nginx
# /etc/nginx/sites-available/api.example.com
server {
    listen 80;
    server_name api.example.com;
    
    location / {
        proxy_pass http://localhost:3000;
        
        # Essential proxy headers
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}
```

```bash
# Enable and test
sudo ln -s /etc/nginx/sites-available/api.example.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

**Flow**:
```
User → http://api.example.com
                ↓
NGINX (port 80) → http://localhost:3000
                ↓
Node.js Express App
```

---

### Handle CORS

#### Option 1: CORS in NGINX (Simple)

```nginx
server {
    listen 80;
    server_name api.example.com;
    
    location / {
        # CORS headers
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS";
        add_header Access-Control-Allow-Headers "Content-Type, Authorization";
        
        # Handle preflight requests
        if ($request_method = OPTIONS) {
            add_header Access-Control-Allow-Origin *;
            add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS";
            add_header Access-Control-Allow-Headers "Content-Type, Authorization";
            add_header Content-Length 0;
            add_header Content-Type text/plain;
            return 204;
        }
        
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

#### Option 2: CORS in Node.js (Recommended)

```javascript
// app.js
const express = require('express');
const cors = require('cors');
const app = express();

// Allow specific origin
app.use(cors({
    origin: 'https://example.com',
    credentials: true
}));

// Or allow all origins
app.use(cors());

app.get('/api/users', (req, res) => {
    res.json({ users: ['Alice', 'Bob'] });
});

app.listen(3000);
```

```nginx
# NGINX config (simpler, CORS handled by Node.js)
server {
    listen 80;
    server_name api.example.com;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

### Handle WebSockets Upgrade

WebSockets require special handling for the `Upgrade` header.

```nginx
server {
    listen 80;
    server_name api.example.com;
    
    location / {
        proxy_pass http://localhost:3000;
        
        # Standard headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
        # WebSocket support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        # Timeouts for WebSocket
        proxy_read_timeout 86400;  # 24 hours
    }
}
```

#### Node.js WebSocket Server

```javascript
// app.js
const express = require('express');
const http = require('http');
const WebSocket = require('ws');

const app = express();
const server = http.createServer(app);
const wss = new WebSocket.Server({ server });

wss.on('connection', (ws) => {
    console.log('Client connected');
    
    ws.on('message', (message) => {
        console.log('Received:', message);
        ws.send(`Echo: ${message}`);
    });
});

app.get('/', (req, res) => {
    res.send('WebSocket server running');
});

server.listen(3000, () => {
    console.log('Server on port 3000');
});
```

---

### PM2 Setup

PM2 is a process manager that keeps your Node.js app running.

```bash
# Install PM2
npm install -g pm2

# Start app
pm2 start app.js --name api

# Start with specific Node version
pm2 start app.js --name api --interpreter=node

# Start with environment variables
pm2 start app.js --name api --env production

# View running apps
pm2 list

# View logs
pm2 logs api

# Stop app
pm2 stop api

# Restart app
pm2 restart api

# Delete app
pm2 delete api

# Auto-start on server boot
pm2 startup
pm2 save
```

#### PM2 Ecosystem File

```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'api',
    script: './app.js',
    instances: 2,  // Number of instances (or 'max' for CPU cores)
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'development',
      PORT: 3000
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 3000
    }
  }]
};
```

```bash
# Start using ecosystem file
pm2 start ecosystem.config.js

# Start in production mode
pm2 start ecosystem.config.js --env production
```

---

### Folder Structure

```
/var/www/
└── myapp/
    ├── backend/
    │   ├── node_modules/
    │   ├── package.json
    │   ├── app.js
    │   └── .env 




    