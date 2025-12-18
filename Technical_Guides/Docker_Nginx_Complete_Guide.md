# Docker + Nginx Master Learning Guide
**From Beginner to Expert System Architect**

This comprehensive guide is designed to take you from your first container to managing scalable, production-grade microservices architectures.

---

## 📚 Table of Contents
1. [Part 1: Docker Complete Guide](#part-1-docker-complete-guide)
2. [Part 2: Nginx Complete Guide](#part-2-nginx-complete-guide)
3. [Part 3: Real-World Production Setup](#part-3-real-world-production-setup-docker--nginx)
4. [Part 4: Architecture Diagrams](#part-4-diagrams)
5. [Part 5: "Try It Yourself" Labs](#part-5-try-it-yourself-labs)
6. [Part 6: Ultimate Cheat Sheets](#part-6-final-cheat-sheets)

---

# PART 1: DOCKER COMPLETE GUIDE

## 🟢 A. BEGINNER — The Foundation

### 1. What is Docker?
Docker is an open-platform that allows you to package applications and their dependencies into a standardized unit called a **container**.
- **Think of it like:** Shipping containers on a cargo ship. Before containers, goods (code) were packed in sacks, barrels, and crates (different OSs/envs), making shipping (deployment) chaotic. Standard containers fit on any ship (server).

### 2. Why We Use Docker (Real World Reasons)
| Problem Without Docker | Solution With Docker |
|------------------------|----------------------|
| "It works on my machine" but fails in production. | **Consistency:** Exact same environment everywhere. |
| Setting up a new dev environment takes days. | **Speed:** `docker-compose up` sets up DB, cache, backend in minutes. |
| Conflicting dependencies (Node 14 vs Node 18). | **Isolation:** Each app lives in its own container with its own libs. |

### 3. Images vs Containers
- **Image:** Read-only template (The Blueprint / The Class).
- **Container:** A running instance of an image (The House / The Object).

### 4. Basic Commands
| Command | Description | Example |
|---------|-------------|---------|
| `docker pull` | Downloads an image from Docker Hub. | `docker pull nginx` |
| `docker run` | Creates and starts a container. | `docker run -d -p 80:80 nginx` |
| `docker ps` | Lists running containers. | `docker ps` |
| `docker ps -a` | Lists all containers (stopped too). | `docker ps -a` |
| `docker stop` | Stops a running container. | `docker stop <id>` |
| `docker rm` | Removes a stopped container. | `docker rm <id>` |
| `docker images` | Lists downloaded images. | `docker images` |
| `docker rmi` | Removes an image. | `docker rmi nginx` |
| `docker logs` | View container output logs. | `docker logs -f <id>` |
| `docker exec -it` | Enter a running container's shell. | `docker exec -it <id> bash` |
| `docker inspect` | View detailed JSON info (IP, config). | `docker inspect <id>` |

### 5. Running Simple Containers
**nginx** (Web Server):
```bash
docker run -d --name my-web -p 8080:80 nginx
# Access at http://localhost:8080
```
**postgres** (Database):
```bash
docker run -d --name my-db -e POSTGRES_PASSWORD=secret -p 5432:5432 postgres
```
**redis** (Cache):
```bash
docker run -d --name my-redis -p 6379:6379 redis
```

---

## 🟡 B. INTERMEDIATE — Building & Persisting

### 1. Dockerfile Guide
A `Dockerfile` is a text document containing all the commands to assemble an image.

**Example: Simple Node.js App**
```dockerfile
# Base Image
FROM node:18-alpine

# Working Directory
WORKDIR /app

# Copy Dependencies first (Caching layer)
COPY package*.json ./

# Install Dependencies
RUN npm install

# Copy Code
COPY . .

# Expose Port
EXPOSE 3000

# Start Command
CMD ["npm", "start"]
```

### 2. Building Images
```bash
# Build image from current directory (.) and tag it (-t)
docker build -t my-node-app .
```

### 3. Volumes (Persistent Storage)
Containers are ephemeral (data is lost when removed). Volumes persist data.
```bash
# Create a volume
docker volume create pgdata

# Run Postgres with volume attached
docker run -d -v pgdata:/var/lib/postgresql/data postgres
```

### 4. Networks
- **Bridge:** Default. Containers on same bridge can talk via IP/name.
- **Host:** Container uses host's network directly.
- **None:** No networking.
```bash
docker network create my-net
docker run --net my-net ...
```

### 5. Environment Variables
Inject config without changing code.
```bash
docker run -e NODE_ENV=production -e DB_HOST=postgres ...
```

### 6. Health Checks
Environment-native way to check if app is actually ready.
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:3000/health || exit 1
```

---

## 🟠 C. ADVANCED — Production Architecture

### 1. Multi-stage Builds (Optimization)
Crucial for reducing image size.
```dockerfile
# Stage 1: Build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Production Run
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/main"]
```

### 2. Docker Compose
Manage multi-container apps with YAML.

### 3. Production docker-compose.yml
```yaml
version: '3.8'

services:
  backend:
    build: .
    ports: ["3000:3000"]
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/myapp
    depends_on:
      - db
      - redis

  db:
    image: postgres:15
    volumes:
      - pg-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=pass

  redis:
    image: redis:alpine

volumes:
  pg-data:
```

### 4. Image Optimization
- Use `alpine` images (smaller OS footprint).
- Minimize layers (combine `RUN` commands).
- Use `.dockerignore` to exclude `node_modules`, `.git`.

### 5. Docker Security
- **Rootless:** Run containers as non-root user (`USER node` in Dockerfile).
- **Read-only:** `docker run --read-only ...`
- **Secrets:** Don't pass secrets as env vars in plain text (use Docker Swarm Secrets or standard Vaults).

### 6. Resource Limits
Prevent one container from killing the server.
```yaml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
```

---

## 🔴 D. EXPERT — Scaling & Orchestration

### 1. Scalable Architectures
Use Docker setup that decouples state (DB/Redis) from stateless apps (Node/NestJS) so you can replicate the app containers infinitely.

### 2. CI/CD Pipeline Integration
**GitHub Actions Example:**
1. Checkout Code.
2. Build Docker Image.
3. Push to Docker Hub/ECR.
4. SSH into Server -> `docker pull` -> `docker-compose up -d`.

### 3. Orchestration: Swarm & K8s
- **Docker Swarm:** Native clustering. Easy to setup. Good for small-medium clusters.
- **Kubernetes (K8s):** Industry standard. Pods, Deployments, Services. Steep learning curve but maximum control.

### 4. Logging Stack (Loki + Promtail + Grafana)
Instead of `docker logs`, stream logs to a central dashboard.
- **Promtail:** Agent that gathers logs from containers.
- **Loki:** Log aggregation system.
- **Grafana:** Visualization UI.

### 5. Production Troubleshooting Methods
- **`docker events`**: Watch real-time daemon events.
- **`docker stats`**: Live CPU/RAM usage stream.
- **`docker system prune -a`**: Emergency cleanup of all unused data.

---

# PART 2: NGINX COMPLETE GUIDE

## 🟢 A. BEGINNER — The Web Server

### 1. What is Nginx?
A high-performance web server, reverse proxy, and load balancer. It is known for its stability and low resource consumption.

### 3. Basic Commands
| Command | Description |
|---------|-------------|
| `nginx -t` | Test configuration syntax. |
| `nginx -s reload` | Reload config without stopping. |
| `systemctl status nginx` | Check service status. |

### 4. Folder Structure (Standard Linux)
- `/etc/nginx/nginx.conf`: Main config.
- `/etc/nginx/sites-available/`: Configs you created.
- `/etc/nginx/sites-enabled/`: Symlinks to active configs.
- `/etc/nginx/conf.d/`: Extra configuration snippets.

### 5. Basic Server Block (Static Site)
```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        root /var/www/html;
        index index.html;
    }
}
```

---

## 🟡 B. INTERMEDIATE — Reverse Proxy

### 1. Reverse Proxy Basics
Nginx sits in front of your App (Node/NestJS). User -> Nginx -> Node.
Benefits: Security, SSL termination, Caching, Compression.

### 2. Proxy Pass to Node/NestJS
```nginx
server {
    listen 80;
    server_name api.mysite.com;

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

### 3. Handling CORS
```nginx
location /api/ {
    add_header 'Access-Control-Allow-Origin' '*';
    # ... other headers
}
```

---

## 🟠 C. ADVANCED — Performance & Security

### 1. SSL Setup (Certbot)
Use `certbot` to automatically edit Nginx config for HTTPS.
```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d example.com
```

### 2. Load Balancing
Distribute traffic across multiple backend instances.
```nginx
upstream backend_servers {
    server backend1:3000;
    server backend2:3000;
}

server {
    location / {
        proxy_pass http://backend_servers;
    }
}
```

### 3. Rate Limiting (DDoS Protection)
```nginx
# In http block
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;

# In server block
location /login {
    limit_req zone=one burst=5;
}
```

---

## 🔴 D. EXPERT — Production Ecosystem

### 1. Full Production Config
Combines SSL, Gzip, Caching, Proxying, and Security Headers.

### 2. High Availability
Running Nginx configs that support zero-downtime reloads and failover handling using `backup` directives in upstream.

---

# PART 3: REAL-WORLD PRODUCTION SETUP (DOCKER + NGINX)

## Architecture: NestJS + Postgres + Redis + Nginx

### 1. Folder Structure
```text
/my-project
├── docker-compose.yml
├── nginx/
│   ├── Dockerfile
│   └── default.conf
├── backend/ (NestJS)
│   ├── Dockerfile
│   ├── src/
│   └── package.json
└── .env
```

### 2. Backend (NestJS) Dockerfile
*File: `backend/Dockerfile`*
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
CMD ["node", "dist/main"]
```

### 3. Nginx Configuration
*File: `nginx/default.conf`*
```nginx
upstream nest_backend {
    server backend:3000;
}

server {
    listen 80;
    
    location / {
        proxy_pass http://nest_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location /api {
        rewrite ^/api/(.*) /$1 break;
        proxy_pass http://nest_backend;
    }
}
```

### 4. Docker Compose (The Glue)
*File: `docker-compose.yml`*
```yaml
version: '3.8'

services:
  backend:
    build: ./backend
    restart: always
    environment:
      - DATABASE_URL=postgresql://user:pass@postgres:5432/mydb
      - REDIS_HOST=redis
      - REDIS_PORT=6379
    depends_on:
      - postgres
      - redis
    networks:
      - app-network

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - backend
    networks:
      - app-network

  postgres:
    image: postgres:15-alpine
    restart: always
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes:
      - pg_data:/var/lib/postgresql/data
    networks:
      - app-network

  redis:
    image: redis:alpine
    restart: always
    networks:
      - app-network

volumes:
  pg_data:

networks:
  app-network:
    driver: bridge
```

---

# PART 4: DIAGRAMS

## 1. Docker Architecture

```mermaid
graph LR
    User[Client] -- usage --> CLI[Docker CLI]
    CLI -- commands --> Daemon[Docker Daemon]
    Daemon -- manages --> Container[Containers]
    Daemon -- builds --> Image[Images]
    Image -- instantiated as --> Container
    Registry[Docker Hub] -- pulls --> Image
```

## 2. Production Request Flow (Nginx + Compose)

```mermaid
sequenceDiagram
    participant U as User
    participant N as Nginx (Reverse Proxy)
    participant B as NestJS (Backend)
    participant D as PostgreSQL
    participant R as Redis

    U->>N: HTTP Request (api.com/users)
    N->>N: SSL Termination & Rate Limit Check
    N->>B: Forward Request (internal:3000)
    B->>R: Check Cache?
    alt Cache Hit
        R-->>B: Return Data
    else Cache Miss
        B->>D: SQL Query
        D-->>B: Row Data
        B->>R: Set Cache
    end
    B-->>N: JSON Response
    N-->>U: HTTP 200 OK (Gzipped)
```

---

# PART 5: TRY IT YOURSELF LABS

**Beginner Labs**
1. [ ] Install Docker Desktop and run `docker run hello-world`.
2. [ ] Pull `nginx` image and run it on port 8080.
3. [ ] Edit an `index.html` locally and mount it to the nginx container using `-v`.
4. [ ] Run a `redis` container and use `docker exec` to set a key.

**Intermediate Labs**
5. [ ] Create a `Dockerfile` for a simple "Hello World" Node.js app.
6. [ ] Build the image and run it.
7. [ ] Create a `docker-compose.yml` that runs your Node app and a Redis DB.
8. [ ] Modify code to check if Redis is reachable.
9. [ ] Inspect the network to see IPs of both containers.

**Advanced Labs**
10. [ ] Convert your Dockerfile to a **Multi-Stage Build**.
11. [ ] Create an Nginx container that proxy_passes to your Node container.
12. [ ] Add a `HEALTHCHECK` to your Dockerfile.
13. [ ] Run Postgres in Compose with a persistent volume. Restart and verify data retention.
14. [ ] Scale the Node app to 3 replicas using `docker-compose up --scale backend=3`.

**Expert Labs**
15. [ ] Configure Nginx as a Load Balancer for the 3 Node replicas.
16. [ ] Set up **Certbot** locally (or on a cheap VPS) to get real SSL.
17. [ ] Implement rate limiting in Nginx (1 request per second).
18. [ ] Set up **Promtail + Loki + Grafana** stack via Compose and view logs.
19. [ ] Simulate a container crash and observe Docker's `restart: always` policy.
20. [ ] Perform a database backup (`pg_dump`) from the host targeting the container.

---

# PART 6: FINAL CHEAT SHEETS

## 1. Docker Commands Cheat Sheet

| Category | Command | Action |
| :--- | :--- | :--- |
| **Lifecycle** | `docker run <image>` | Create & Start container |
| | `docker stop <id>` | Stop container |
| | `docker start <id>` | Start stopped container |
| | `docker restart <id>` | Restart container |
| | `docker rm <id>` | Delete container |
| **Images** | `docker build -t <name> .` | Build image from Dockerfile |
| | `docker pull <image>` | Download image |
| | `docker rmi <image>` | Delete image |
| | `docker image prune` | Remove unused images |
| **Info** | `docker ps` | List running containers |
| | `docker logs <id>` | Show logs |
| | `docker exec -it <id> sh` | Shell into container |
| | `docker stats` | Live resource usage |
| **System** | `docker system prune` | Clean up unused data |

## 2. Docker Compose Cheat Sheet

| Command | Action |
| :--- | :--- |
| `docker-compose up` | Build & Start all services |
| `docker-compose up -d` | Start in background (detached) |
| `docker-compose down` | Stop & Remove containers/nets |
| `docker-compose logs -f` | Follow logs of all services |
| `docker-compose build` | Rebuild images |
| `docker-compose restart` | Restart services |
| `docker-compose exec <svc> sh` | Shell into specific service |

## 3. Nginx Server Block Cheat Sheet

```nginx
server {
    listen 80;
    server_name example.com;
    
    # Root Directory Configuration
    location / {
        root /var/www/html;
        index index.html;
        try_files $uri $uri/ /index.html; # SPA Support
    }

    # Reverse Proxy Configuration
    location /api {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
    }
}
```

## 4. SSL & Security Checklist (Nginx)
- [ ] Redirect HTTP to HTTPS.
- [ ] Use `HTTP/2`.
- [ ] Disable server tokens (`server_tokens off;`).
- [ ] Set `X-Frame-Options SAMEORIGIN`.
- [ ] Set `X-Content-Type-Options nosniff`.
- [ ] Configure `gzip` compression.
- [ ] Use Strong SSL Ciphers (Mozilla SSL Gen).

---
*Created for the Advanced DevOps Learning Path.*
