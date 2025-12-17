# Nginx Masterclass: The High-Performance Gateway 🚦

## Table of Contents
1.  [Introduction & Architecture](#1-introduction--architecture)
2.  [The Configuration Anatomy](#2-the-configuration-anatomy)
3.  [Serving Static Content (The Web Server)](#3-serving-static-content-the-web-server)
4.  [Reverse Proxy & Load Balancing](#4-reverse-proxy--load-balancing)
5.  [Security & SSL (HTTPS)](#5-security--ssl-https)
6.  [Performance Optimization (Caching & Gzip)](#6-performance-optimization-caching--gzip)
7.  [Essential Best Practices](#7-essential-best-practices)

---

## 1. Introduction & Architecture

### What is Nginx?
Nginx (pronounced "Engine X") is a web server, reverse proxy, load balancer, and HTTP cache. It powers the world's busiest sites (Netflix, Airbnb, GitHub).

### Why use it with Node/Python/Go?
Application servers (like Node.js) are great at logic but bad at handling 10,000 concurrent network connections. Nginx is designed to handle **tens of thousands of simultaneous connections** with low memory usage.

### How it works (Event-Driven)
*   **Apache (Old School):** Creates a new Process/Thread for every user. Heavy. Eating up RAM.
*   **Nginx (Modern):** Asynchronous and Event-driven.
    *   **Master Process:** Reads config and manages workers.
    *   **Worker Processes:** Handle thousands of requests each in a non-blocking loop.

---

## 2. The Configuration Anatomy

Located usually at `/etc/nginx/nginx.conf`.

```nginx
user www-data;
worker_processes auto; # Spawns 1 worker per CPU core

events {
    worker_connections 1024; # max connections per worker
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    # The actual site configurations usually live here:
    include /etc/nginx/conf.d/*.conf; 
    
    server {
        listen 80;
        server_name example.com;
        root /var/www/html;
    }
}
```

### Contexts
*   `main`: Global settings.
*   `http`: HTTP protocol specific.
*   `server`: A Virtual Host (e.g., website A vs website B).
*   `location`: Routing rules based on URL path (e.g., `/api` vs `/images`).

---

## 3. Serving Static Content (The Web Server)

Nginx serves files (React builds, Images, CSS) 10x-50x faster than Node.js.

```nginx
server {
    listen 80;
    server_name my-site.com;
    
    # 1. Root Directory
    root /var/www/my-react-app/build;
    index index.html;

    # 2. SPA Config (The "History API" Fix)
    # If file exists, serve it. If not, fallback to index.html (for React Router)
    location / {
        try_files $uri $uri/ /index.html;
    }
    
    # 3. Cache Assets aggressively
    location /static/ {
        expires 1y;
        add_header Cache-Control "public, no-transform";
    }
}
```

---

## 4. Reverse Proxy & Load Balancing

This is the most common use case in Modern DevOps.

### The Basic Proxy (Pass to Node.js)
```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://localhost:3000;
        
        # ESSENTIAL: Forward headers so Node knows the real user details
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Load Balancing (Upstreams)
Distribute traffic among multiple servers.

```nginx
upstream backend_cluster {
    # Algorithms:
    # (default) - Round Robin
    # least_conn; - Send to server with fewest active connections
    # ip_hash; - Sticky session (User A always lands on Server 1)
    
    server 10.0.0.1:3000 weight=3; # 3x more traffic
    server 10.0.0.2:3000;
    server 10.0.0.3:3000 down; # Mark as offline maintenance
}

server {
    location / {
        proxy_pass http://backend_cluster;
    }
}
```

---

## 5. Security & SSL (HTTPS) 🔒

### Redirect HTTP to HTTPS
```nginx
server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}
```

### SSL Configuration
Advanced hardening for "A+" rating on SSL Labs.
```nginx
server {
    listen 443 ssl http2;
    server_name example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # Modern Ciphers only
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    
    # HSTS (Strict Transport Security)
    # Tells browsers: "NEVER try to load this site on HTTP again"
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
}
```

### Rate Limiting (DDoS Protection)
```nginx
# Define Zone: 10MB memory, max 10 requests per second per IP
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;

server {
    location /api/login {
        # Burst allows a small spike (5 reqs), but strictly enforces averge
        limit_req zone=api_limit burst=5 nodelay; 
    }
}
```

---

## 6. Performance Optimization (Caching & Gzip)

### Gzip Compression
Compress valid text files to save bandwidth (70% smaller).
```nginx
gzip on;
gzip_types text/plain text/css application/json application/javascript text/xml application/xml+rss text/javascript;
gzip_min_length 1000; # Don't compress tiny files
```

### Micro-Caching
Cache dynamic API responses for 1 second. Can handle massive traffic spikes.
```nginx
proxy_cache_path /tmp/nginx_cache levels=1:2 keys_zone=my_cache:10m max_size=1g inactive=60m;

server {
    location /api/news-feed {
        proxy_cache my_cache;
        proxy_cache_valid 200 1s; # Cache success response for 1s
        proxy_path http://backend;
    }
}
```

---

## 7. Essential Best Practices
1.  **Test Config:** Always run `nginx -t` before reloading.
2.  **Graceful Reload:** Use `nginx -s reload` (Zero downtime) instead of `restart`.
3.  **Hide Version:** `server_tokens off;` (Prevents hackers seeing you are on Nginx 1.18).
4.  **Client Body Size:** Increase if expecting file uploads. `client_max_body_size 10M;`
