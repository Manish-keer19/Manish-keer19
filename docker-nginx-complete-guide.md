# Docker + NGINX: The Complete Beginner to Advanced Guide

A complete course on Docker, NGINX, and Full-Stack Deployment (React + Node.js).

---

## Table of Contents

1. [Docker Basics (Beginner Friendly)](#1-docker-basics-beginner-friendly)
2. [Essential Docker Commands](#2-essential-docker-commands)
3. [Dockerfile Deep Dive](#3-dockerfile-deep-dive)
4. [Docker Compose](#4-docker-compose)
5. [NGINX + Docker](#5-nginx--docker)
6. [Deployment Models](#6-deployment-models)
7. [Production Deployment Guide](#7-production-deployment-guide)
8. [Practical Full-Stack Example](#8-practical-full-stack-example)

---

## 1. Docker Basics (Beginner Friendly)

### What is Docker?
Docker is a platform that allows you to package your application and all its dependencies (libraries, runtime, tools) into a single unit called a **Container**. This ensures that your app runs exactly the same way on your laptop, your friend's laptop, and the production server.

### Why use Docker?
- **"It works on my machine"**: Solved forever.
- **Isolation**: Apps don't conflict with each other (e.g., App A needs Node 14, App B needs Node 18).
- **Portability**: Run anywhere (AWS, Azure, DigitalOcean, Raspberry Pi).
- **Efficiency**: Uses less resources than Virtual Machines.

### Docker vs Virtual Machine (VM)

| Feature | Docker Container | Virtual Machine (VM) |
| :--- | :--- | :--- |
| **Architecture** | Shares Host OS Kernel | Has its own full OS |
| **Size** | Megabytes (MB) | Gigabytes (GB) |
| **Boot Time** | Seconds | Minutes |
| **Performance** | Native performance | Slower (Overhead) |

### Core Concepts (Explained Simply)

1. **Image**: The "Blueprint" or "Recipe". It's a read-only file that contains your code + libraries.
   - *Analogy*: A Class in programming, or a CD-ROM.
2. **Container**: The "Running Instance" of an image.
   - *Analogy*: An Object (instance of a Class), or the running program.
3. **Volume**: A shared folder between your computer and the container. Used to persist data (like databases) so it doesn't vanish when the container stops.
   - *Analogy*: A USB drive plugged into the container.
4. **Network**: A virtual cable connecting containers so they can talk to each other.

### Docker Architecture

```mermaid
graph LR
    Client[Docker Client (CLI)] -- Commands --> Daemon[Docker Daemon (Server)]
    Daemon -- Manages --> Containers
    Daemon -- Manages --> Images
    Daemon -- Pulls/Pushes --> Registry[Docker Hub]
```

- **Docker Client**: The command line tool (`docker run ...`).
- **Docker Daemon**: The background process doing the heavy lifting.
- **Docker Hub**: The "App Store" for Docker images (Node, Nginx, Python, etc.).

### How Docker Builds Images
Docker reads a file called `Dockerfile`. It executes instructions step-by-step. Each step creates a "layer". Layers are cached, making subsequent builds super fast.

---

## 2. Essential Docker Commands

### Basic Commands

| Command | Description | Example |
| :--- | :--- | :--- |
| `docker run` | Create and start a container | `docker run -d -p 80:80 nginx` |
| `docker ps` | List running containers | `docker ps` |
| `docker ps -a` | List ALL containers (including stopped) | `docker ps -a` |
| `docker stop` | Stop a running container | `docker stop <container_id>` |
| `docker rm` | Remove a stopped container | `docker rm <container_id>` |
| `docker images` | List downloaded images | `docker images` |
| `docker rmi` | Remove an image | `docker rmi <image_id>` |
| `docker pull` | Download image from Hub | `docker pull node:18` |
| `docker exec` | Run a command INSIDE a container | `docker exec -it <id> sh` |
| `docker logs` | View container logs | `docker logs <id>` |

### Volume Commands
```bash
docker volume create my_data      # Create volume
docker volume ls                  # List volumes
docker volume rm my_data          # Remove volume
```

### Network Commands
```bash
docker network create my_net      # Create network
docker network ls                 # List networks
```

### Cleanup Commands (Save Space)
```bash
docker system prune -a            # Delete ALL stopped containers, unused networks, and images
```

### Container Lifecycle Example
```bash
# 1. Run NGINX in background (-d) mapping port 8080 on host to 80 in container (-p)
docker run -d -p 8080:80 --name my-web-server nginx

# 2. Check it's running
docker ps

# 3. Visit http://localhost:8080 in browser

# 4. Stop it
docker stop my-web-server

# 5. Remove it
docker rm my-web-server
```

---

## 3. Dockerfile Deep Dive

A `Dockerfile` is a text document that contains all the commands a user could call on the command line to assemble an image.

### Common Instructions

| Instruction | Description | Example |
| :--- | :--- | :--- |
| `FROM` | Base image to start from | `FROM node:18-alpine` |
| `WORKDIR` | Set working directory inside container | `WORKDIR /app` |
| `COPY` | Copy files from Host -> Container | `COPY . .` |
| `RUN` | Run command during BUILD time | `RUN npm install` |
| `CMD` | Run command during START time | `CMD ["node", "index.js"]` |
| `EXPOSE` | Document which port is intended to be used | `EXPOSE 3000` |
| `ENTRYPOINT` | Main executable of the container | `ENTRYPOINT ["nginx"]` |

### Base Images: Alpine vs Debian/Ubuntu
- **Alpine**: Extremely small (~5MB), secure. **Recommended for production**.
  - *Tag*: `node:18-alpine`
- **Debian/Ubuntu**: Larger, includes more tools. Good for debugging or if you need specific libraries not in Alpine.
  - *Tag*: `node:18` (usually Debian based)

### Multi-Stage Builds (The Pro Move)
Drastically reduce image size by separating "build" tools from "runtime" files.

**Example: React App**
```dockerfile
# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Serve
FROM nginx:alpine
# Copy ONLY the build output from Stage 1
COPY --from=builder /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Build Caching
Docker caches layers. Order matters!
**Bad:**
```dockerfile
COPY . .
RUN npm install
# If you change ANY code, npm install runs again!
```

**Good:**
```dockerfile
COPY package.json .
RUN npm install
COPY . .
# npm install only runs if package.json changes!
```

---

## 4. Docker Compose

`docker-compose.yml` allows you to define and run multi-container Docker applications.

### Structure Explained
```yaml
version: '3.8'  # Docker Compose version

services:       # List of containers
  backend:
    build: ./server       # Build from this folder
    ports:
      - "3000:3000"       # Host:Container
    environment:
      - DB_HOST=database  # 'database' is the service name below
    depends_on:
      - database          # Start database first
    networks:
      - app-network

  database:
    image: postgres:15-alpine
    volumes:
      - db-data:/var/lib/postgresql/data # Persist data
    networks:
      - app-network

networks:       # Define networks
  app-network:
    driver: bridge

volumes:        # Define volumes
  db-data:
```

### Commands
```bash
docker-compose up -d        # Start all services in background
docker-compose down         # Stop and remove containers/networks
docker-compose logs -f      # View logs of all services
docker-compose build        # Rebuild images
```

---

## 5. NGINX + Docker

### NGINX inside Docker
Running NGINX in Docker is cleaner than installing it on the host.

### Reverse Proxy Configuration
**Goal**: NGINX (Port 80) -> Node.js (Port 3000)

`nginx.conf`:
```nginx
server {
    listen 80;
    
    location / {
        # 'backend' is the service name in docker-compose
        proxy_pass http://backend:3000; 
        proxy_set_header Host $host;
    }
}
```

### Serving React Build
Mount the build folder into the NGINX container.

```yaml
services:
  nginx:
    image: nginx:alpine
    volumes:
      - ./build:/usr/share/nginx/html
```

---

## 6. Deployment Models

### A) Single Dockerfile Deployment
**Concept**: One container runs EVERYTHING (Node + React static files).
**Pros**: Simple.
**Cons**: Violates "One process per container" rule. Harder to scale.

**Structure**:
1. Build React.
2. Copy React build to Node `public` folder.
3. Node serves API AND static files.

### B) Multi-Dockerfile Deployment (Recommended)
**Concept**: Separate containers for Frontend (NGINX), Backend (Node), Database.
**Pros**: Scalable, clean separation, easy to update independently.
**Structure**:
- `client/Dockerfile` (Multi-stage: Build -> NGINX)
- `server/Dockerfile` (Node)
- `docker-compose.yml` (Orchestrates them)

### C) Kubernetes (Brief Summary)
**Concept**: Orchestration for hundreds of containers.
- **Pod**: Smallest unit (usually 1 container).
- **Deployment**: Manages replicas of Pods (e.g., "Run 3 copies of Backend").
- **Service**: Stable IP/Load Balancer for Pods.
**Usage**: You push your Docker images to a registry, and K8s pulls and runs them.

---

## 7. Production Deployment Guide

1. **Use Alpine Images**: Smaller attack surface, faster downloads.
2. **Remove Dev Dependencies**: `npm install --production`.
3. **Environment Variables**: NEVER hardcode secrets. Use `.env` files and inject them.
4. **Process Manager**: Use `dumb-init` or `tini` as entrypoint, or let Docker handle it. For Node, `node index.js` is fine in Docker, but `pm2-runtime` offers more control.
5. **Reverse Proxy**: Always put NGINX in front of Node.js for security, SSL, and compression.
6. **Caching**: Configure NGINX to cache static assets (`expires 1y;`).
7. **Security**:
   - Run as non-root user (create a user in Dockerfile).
   - Scan images for vulnerabilities (`docker scan`).
8. **Health Checks**: Add `HEALTHCHECK` instruction in Dockerfile.

---

## 8. Practical Full-Stack Example

**Goal**: Deploy React (Frontend) + Node (Backend) + NGINX (Reverse Proxy & SSL)

### Folder Structure
```
/my-app
  /client          # React App
    Dockerfile
    ...
  /server          # Node App
    Dockerfile
    ...
  /nginx
    default.conf
  docker-compose.yml
```

### 1. Client Dockerfile (`client/Dockerfile`)
```dockerfile
# Build Stage
FROM node:18-alpine as build
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

# Production Stage
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
# We don't need a custom nginx config HERE if we use the main proxy, 
# BUT usually we put a simple config here to handle SPA routing if standalone.
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 2. Server Dockerfile (`server/Dockerfile`)
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install --production
COPY . .
USER node  # Security: Run as non-root
EXPOSE 5000
CMD ["node", "index.js"]
```

### 3. NGINX Config (`nginx/default.conf`)
This NGINX sits in front of BOTH containers.

```nginx
upstream client_upstream {
    server client:80;
}

upstream api_upstream {
    server api:5000;
}

server {
    listen 80;
    server_name example.com;

    # Frontend
    location / {
        proxy_pass http://client_upstream;
        proxy_set_header Host $host;
    }

    # Backend API
    location /api {
        proxy_pass http://api_upstream;
        proxy_set_header Host $host;
    }
}
```

### 4. Docker Compose (`docker-compose.yml`)
```yaml
version: '3.8'

services:
  client:
    build: ./client
    restart: always
    networks:
      - app-net

  api:
    build: ./server
    restart: always
    environment:
      - PORT=5000
      - DB_URL=postgres://...
    networks:
      - app-net

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - client
      - api
    networks:
      - app-net

networks:
  app-net:
    driver: bridge
```

### 5. Production Commands
```bash
# Build and Run in background
docker-compose up -d --build

# Check logs
docker-compose logs -f

# Update application
git pull
docker-compose up -d --build
# Docker detects changes and rebuilds only what's needed
```

### 6. Handling HTTPS (Certbot)
To add SSL, the easiest way in Docker is to use a pre-configured image like `nginx-proxy` + `acme-companion` OR run Certbot on the host and mount certificates.

**Simple Method (Certbot on Host):**
1. Run Certbot on host machine to generate certs.
2. Mount `/etc/letsencrypt` into NGINX container.
3. Update `nginx/default.conf` to listen on 443 and point to certs.

```yaml
  nginx:
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
      - /etc/letsencrypt:/etc/letsencrypt
    ports:
      - "80:80"
      - "443:443"
```

---
*Generated by Antigravity*
