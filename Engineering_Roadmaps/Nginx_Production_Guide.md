# NGINX Beginner to Advanced Cheatsheet

A complete guide to mastering NGINX for web development and deployment.

---

## Table of Contents

1. [Basics (Beginner)](#1-basics-beginner)
2. [NGINX Configuration Deep-Dive](#2-nginx-configuration-deep-dive)
3. [Reverse Proxy for Node.js](#3-reverse-proxy-for-nodejs)
4. [Serve React Frontend](#4-serve-react-frontend)
5. [Deployment Guides](#5-deployment-guides)
    - [A) Deploy Node.js + React without Docker](#a-deploy-nodejs--react-without-docker)
    - [B) Deploy Node.js + React with Docker](#b-deploy-nodejs--react-with-docker)
6. [Debugging & Monitoring](#6-debugging--monitoring)
7. [Best Practices](#7-best-practices)

---

## 1. Basics (Beginner)

### What is NGINX?
NGINX (pronounced "engine-x") is a high-performance web server, reverse proxy, load balancer, and HTTP cache. It is known for its stability, rich feature set, simple configuration, and low resource consumption.

### Why use NGINX?
- **Performance**: Handles thousands of concurrent connections efficiently.
- **Reverse Proxy**: Hides your backend servers (like Node.js) from the public internet.
- **Load Balancing**: Distributes traffic across multiple servers.
- **Security**: Handles SSL/TLS termination, protecting your backend.
- **Static Content**: Serves images, CSS, and HTML much faster than application servers.

### NGINX vs Node.js
| Feature | NGINX | Node.js |
| :--- | :--- | :--- |
| **Primary Role** | Web Server / Reverse Proxy | Application Runtime |
| **Handling Requests** | Asynchronous, Event-driven (C++) | Asynchronous, Event-driven (JS) |
| **Static Files** | Extremely Fast | Slower (not optimized for this) |
| **Logic** | Configuration-based routing | Complex Business Logic |
| **Best Use** | Front-facing server, SSL, Caching | Backend API, Database logic |

**Conclusion**: Use **both**. NGINX sits in front to handle traffic, SSL, and static files, while Node.js runs behind it to process API requests.

### What is a Reverse Proxy?
A reverse proxy sits between the client (browser) and your backend server.

```
+------------------+       Request        +-------------------------+       Forward Request       +--------------------------+
|                  | -------------------> |                         | --------------------------> |                          |
| Client (Browser) |                      | NGINX (Port 80/443)     |                             | Node.js App (Port 3000)  |
|                  | <------------------- |                         | <-------------------------- |                          |
+------------------+       Response       +-------------------------+          Response           +--------------------------+
```

**Benefits**:
- Security (hides backend IP/Port).
- SSL Termination (HTTPS happens at NGINX).
- Caching.

### What is a Load Balancer?
Distributes incoming traffic across multiple backend servers to ensure no single server is overwhelmed.

```
                                               +------------------+
                                          +--> |   App Server 1   |
                                          |    +------------------+
+--------+           +---------+          |
| Client | --------> |  NGINX  | ---------+--> +------------------+
+--------+           +---------+          |    |   App Server 2   |
                                          |    +------------------+
                                          |
                                          +--> +------------------+
                                               |   App Server 3   |
                                               +------------------+
```

### Key Concepts

- **Upstream**: Defines a group of backend servers that NGINX can proxy traffic to (used for load balancing).
- **Server Block**: Defines a virtual server (like a website). It listens on a specific port and domain.
- **Location Block**: Defines how to process specific URLs (routes) within a server block (e.g., `/api`, `/images`).

### Default NGINX Folder Structure (Linux/Ubuntu)
- `/etc/nginx/`: Main configuration directory.
- `/etc/nginx/nginx.conf`: Global configuration file.
- `/etc/nginx/sites-available/`: Where you create your config files.
- `/etc/nginx/sites-enabled/`: Where you link active configs (symlinks).
- `/var/log/nginx/`: Logs (access.log, error.log).
- `/var/www/html/`: Default folder for static files.

### Common Commands
```bash
# Start NGINX
sudo systemctl start nginx

# Stop NGINX
sudo systemctl stop nginx

# Restart NGINX (Stop + Start)
sudo systemctl restart nginx

# Reload Config (Zero-downtime, use this after config changes)
sudo systemctl reload nginx

# Test Configuration Syntax (ALWAYS run this before reloading)
sudo nginx -t
```

---

## 2. NGINX Configuration Deep-Dive

### Server Block Explained
A basic server block looks like this:

```nginx
server {
    listen 80;                  # Port to listen on
    server_name example.com;    # Domain name

    root /var/www/example;      # Root directory for static files
    index index.html;           # Default file to serve

    location / {
        try_files $uri $uri/ =404; # Fallback rule
    }
}
```

### Locations, Regex, and Routing
Nginx matches the most specific location block.

```nginx
# Exact match (Highest priority)
location = /login {
    # Only matches /login
}

# Prefix match (Standard)
location /api {
    # Matches /api, /api/v1, /api/users
}

# Regex match (Case sensitive)
location ~ \.php$ {
    # Matches any .php file
}

# Regex match (Case insensitive)
location ~* \.(jpg|jpeg|png|gif)$ {
    # Matches image files
}
```

### Proxy Pass (Connecting to Backend)
This is the core of Reverse Proxying.

```nginx
location /api {
    proxy_pass http://localhost:3000; # Forward traffic to Node.js
    
    # Essential Headers for the backend to know the real client
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

### Gzip Compression
Reduce file size sent to the browser. Add inside `http` or `server` block.

```nginx
gzip on;
gzip_types text/plain text/css application/json application/javascript text/xml;
gzip_min_length 1000; # Don't compress tiny files
```

### Security Headers
Add these to protect your site.

```nginx
add_header X-Frame-Options "SAMEORIGIN";        # Prevent Clickjacking
add_header X-XSS-Protection "1; mode=block";    # XSS Filter
add_header X-Content-Type-Options "nosniff";    # Prevent MIME sniffing
```

### Rate Limiting
Prevent abuse (DDoS protection).

1. **Define limit** (in `nginx.conf` http block):
   ```nginx
   limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;
   ```

2. **Apply limit** (in `server` or `location` block):
   ```nginx
   location /api {
       limit_req zone=mylimit burst=20 nodelay;
       proxy_pass http://localhost:3000;
   }
   ```

### SSL with Certbot (Step-by-Step)
1. **Install Certbot**:
   ```bash
   sudo apt install certbot python3-certbot-nginx
   ```
2. **Run Certbot**:
   ```bash
   sudo certbot --nginx -d example.com -d www.example.com
   ```
   *Certbot will automatically modify your NGINX config to enable HTTPS and redirect HTTP to HTTPS.*

---

## 3. Reverse Proxy for Node.js

**Goal**: Serve Node.js (running on port 3000) via NGINX (port 80).

### Configuration
File: `/etc/nginx/sites-available/my-node-app`

```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### Handling WebSockets
The lines `proxy_set_header Upgrade $http_upgrade;` and `proxy_set_header Connection 'upgrade';` are crucial for WebSockets (e.g., Socket.io) to work through NGINX.

### Handling CORS (If needed at NGINX level)
Usually handled in Node.js, but can be done in NGINX:

```nginx
location /api {
    add_header 'Access-Control-Allow-Origin' '*';
    proxy_pass http://localhost:3000;
}
```

### PM2 Setup (Keep Node.js alive)
```bash
npm install -g pm2
pm2 start server.js --name "my-api"
pm2 save
pm2 startup
```

---

## 4. Serve React Frontend

**Goal**: Serve a built React app (static files) efficiently.

### Build React
```bash
npm run build
# Output is usually in /build or /dist folder
```

### Configuration
File: `/etc/nginx/sites-available/my-react-app`

```nginx
server {
    listen 80;
    server_name example.com;

    root /var/www/my-react-app/build;
    index index.html;

    # Serve Static Files
    location / {
        try_files $uri $uri/ /index.html; 
        # ^ IMPORTANT: This routes all unknown requests to index.html
        # allowing React Router to handle client-side routing.
    }

    # Cache Static Assets (Images, JS, CSS)
    location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
        expires 1y;
        add_header Cache-Control "public, no-transform";
    }
}
```

---

## 5. Deployment Guides

### A) Deploy Node.js + React without Docker

**Scenario**: One Ubuntu server. React on root (`/`), Node API on `/api`.

1. **Setup Folders**:
   ```bash
   mkdir -p /var/www/myapp/client
   mkdir -p /var/www/myapp/server
   ```

2. **Clone & Install**:
   - Clone repo to `/var/www/myapp`.
   - **Client**: `cd client && npm install && npm run build`.
   - **Server**: `cd server && npm install`.
   - **Start Server**: `pm2 start index.js --name "api"`.

3. **NGINX Config**:
   `/etc/nginx/sites-available/myapp`

   ```nginx
   server {
       listen 80;
       server_name example.com;

       # Frontend (React)
       location / {
           root /var/www/myapp/client/build;
           index index.html;
           try_files $uri $uri/ /index.html;
       }

       # Backend (Node API)
       location /api {
           proxy_pass http://localhost:3000; # Assuming /api is handled in express routes
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
       }
   }
   ```

4. **Enable & Restart**:
   ```bash
   sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
   sudo nginx -t
   sudo systemctl restart nginx
   ```

5. **SSL**:
   ```bash
   sudo certbot --nginx -d example.com
   ```

### B) Deploy Node.js + React with Docker

**Structure**:
```
/project
  /client (React)
    Dockerfile
  /server (Node)
    Dockerfile
  docker-compose.yml
  /nginx
    default.conf
```

**1. React Dockerfile** (`client/Dockerfile`)
```dockerfile
# Stage 1: Build
FROM node:18-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Serve with NGINX (Internal)
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**2. Node Dockerfile** (`server/Dockerfile`)
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
```

**3. NGINX Config** (`nginx/default.conf`)
```nginx
upstream client_upstream {
    server client:80; # 'client' matches service name in docker-compose
}

upstream api_upstream {
    server api:3000; # 'api' matches service name in docker-compose
}

server {
    listen 80;

    location / {
        proxy_pass http://client_upstream;
    }

    location /api {
        proxy_pass http://api_upstream;
    }
}
```

**4. Docker Compose** (`docker-compose.yml`)
```yaml
version: '3.8'
services:
  client:
    build: ./client
    restart: always

  api:
    build: ./server
    restart: always
    environment:
      - PORT=3000

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - client
      - api
```

**5. Run**:
```bash
docker-compose up -d --build
```

**SSL in Docker**:
Usually, it's easier to run a separate NGINX container or use a tool like `Traefik` or `Nginx Proxy Manager` for SSL termination in Docker environments, or mount Certbot certificates into the NGINX container.

---

## 6. Debugging & Monitoring

### Logs
- **Access Log**: Who visited?
  `tail -f /var/log/nginx/access.log`
- **Error Log**: What broke?
  `tail -f /var/log/nginx/error.log`

### Common Errors
1. **502 Bad Gateway**: NGINX cannot connect to the backend.
   - *Fix*: Check if Node.js is running (`pm2 status`), check port matching in `proxy_pass`.
2. **404 Not Found (React)**: Refreshing a page gives 404.
   - *Fix*: Ensure `try_files $uri $uri/ /index.html;` is in the location block.
3. **413 Request Entity Too Large**: Uploading big files fails.
   - *Fix*: Add `client_max_body_size 10M;` to server block.

### Testing Config
Always run this before restarting!
```bash
sudo nginx -t
```

### Check Ports
See what's running on ports.
```bash
sudo lsof -i :80
sudo lsof -i :3000
```

---

## 7. Best Practices

1. **Security**:
   - Always use HTTPS (Certbot).
   - Hide NGINX version: `server_tokens off;` in `nginx.conf`.
   - Use security headers (X-Frame-Options, etc.).

2. **Performance**:
   - Enable Gzip.
   - Enable Caching for static files.
   - Use HTTP/2 (Certbot enables this by default).

3. **Maintenance**:
   - Keep config files organized (one file per site).
   - Use descriptive comments in configs.
   - Monitor logs regularly.

---

