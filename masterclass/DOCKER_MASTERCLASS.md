# Docker Masterclass: The Complete Guide (Zero to Advanced) 🐳

## Table of Contents
1.  [Introduction & Architecture](#1-introduction--architecture)
2.  [Installation & Setup](#2-installation--setup)
3.  [Core Concepts Deep Dive](#3-core-concepts-deep-dive)
4.  [Essential & Advanced Commands](#4-essential--advanced-commands)
5.  [Mastering the Dockerfile](#5-mastering-the-dockerfile)
6.  [Networking Under the Hood](#6-networking-under-the-hood)
7.  [Data Persistence (Volumes)](#7-data-persistence-volumes)
8.  [Docker Compose & Orchestration](#8-docker-compose--orchestration)
9.  [Security Best Practices](#9-security-best-practices)
10. [Production Optimization](#10-production-optimization)
11. [Troubleshooting & Debugging](#11-troubleshooting--debugging)

---

## 1. Introduction & Architecture

### The "Why"
Before Docker, efficient software delivery was a nightmare. "It works on my machine" was the standard excuse.
*   **Virtual Machines (VMs):** Heavy. Each app gets a full OS (Guest OS). Slow boot, high RAM usage.
*   **Containers (Docker):** Light. Share the Host OS kernel. Instant boot, minimal overhead.

### Docker Architecture
Docker is a Client-Server application.
1.  **Docker Client:** The CLI tool (`docker build`, `docker run`). It talks to the daemon via REST API.
2.  **Docker Daemon (Dockerd):** The background process that manages images, containers, networks, and volumes.
3.  **Docker Registry:** Where images are stored (Docker Hub, AWS ECR, Azure ACR).

### How it works (The Internals)
*   **Namespaces:** provide isolation. Your container has its own Process ID (PID), Network (NET), and Filesystem (MNT) views. It thinks it's the only thing running on the machine.
*   **Control Groups (cgroups):** provide resource limiting. They ensure a container can't use more than 512MB RAM or 50% CPU if you restrict it.
*   **Union File System:** The magic behind images. Layers are stacked on top of each other.

---

## 2. Installation & Setup
*   **Windows/Mac:** Install **Docker Desktop**. It creates a lightweight Linux VM to run the daemon.
*   **Linux:** Install the engine directly (`apt-get install docker-ce`). This is native and fastest.

**Post-Install Check:**
```bash
docker version
docker info
```

---

## 3. Core Concepts Deep Dive

### Images (The Blueprint)
*   **Read-only:** Once built, an image never changes.
*   **Layered:** Each instruction in a Dockerfile adds a layer. Docker caches these layers.
*   **Inheritance:** You almost always start `FROM` a base image (like `ubuntu`, `node`, `python`).

### Containers (The Running Instance)
*   A container is just a writable layer added on top of the read-only image layers.
*   **Ephemeral:** Containers are meant to be stopped and destroyed. Do not store important data *inside* the container's file system.

---

## 4. Essential & Advanced Commands

### Lifecycle
```bash
# Build an image tagged 'my-app:v1' from current directory
docker build -t my-app:v1 .

# Run in background (-d), map host port 8080 to container 80 (-p)
# --name gives it a memorable name
docker run -d -p 8080:80 --name web-server my-app:v1

# Stop and Remove (Gracefully)
docker stop web-server
docker rm web-server

# Force kill and remove (Nuclear option)
docker rm -f web-server
```

### Inspection & Management
```bash
# List running containers
docker ps

# List ALL containers (including stopped ones)
docker ps -a

# See resource usage (CPU/RAM) relative to limits
docker stats

# Inspect low-level details (IP address, volume mounts, env vars)
docker inspect web-server

# View logs (Follow mode like 'tail -f')
docker logs -f web-server
```

### Cleanup (Pruning)
WARNING: These commands delete data.
```bash
# Remove unused images
docker image prune

# Nuclear cleanup: Remove all stopped containers, unused networks, and dangling images
docker system prune

# The "Clean Slate": Remove EVERYTHING not currently running
docker system prune -a --volumes
```

---

## 5. Mastering the Dockerfile

Writing a customized Dockerfile is the most critical skill.

### The Directives
*   `FROM`: The base image. ALWAYS use specific tags (e.g., `node:18-alpine` instead of `node:latest`).
*   `WORKDIR`: Sets the directory for subsequent commands.
*   `COPY` vs `ADD`: Use `COPY` by default. `ADD` has extra features (unpacking tars, fetching URLs) that can be dangerous or unpredictable.
*   `RUN`: Executes a command *during the build*. (e.g., `npm install`). Creates a new layer.
*   `CMD`: The default command to run when the *container starts*. Can be overridden.
*   `ENTRYPOINT`: The main executable. Arguments passed to `docker run` are appended to this.

### Advanced Strategy: Multi-Stage Builds
Drastically reduce image size by separating the "Build" environment from the "Runtime" environment.

**Example: React App**
```dockerfile
# STAGE 1: Build Protocol
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci 
COPY . .
RUN npm run build
# Result: A /app/build folder with static files. 
# The rest of this layer (node_modules usually 500MB+) is discarded.

# STAGE 2: Production Protocol
FROM nginx:alpine-slim
# Copy ONLY the build artifacts from the previous stage
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
# Final Image Size: ~20MB (vs 600MB+)
```

### .dockerignore
Always include a `.dockerignore` file. It works like `.gitignore`.
```text
node_modules
.git
Dockerfile
.env
```
This prevents massive local folders from being sent to the Docker daemon, speeding up builds and security.

---

## 6. Networking Under the Hood

By default, Docker isolates containers.
1.  **Bridge Network (Default):** Containers on the same bridge can talk to each other by IP. If you create a custom bridge, they can talk by **Name** (DNS resolution).
2.  **Host Network:** The container shares the host's networking namespace completely. Performance is native, but security is lower (port conflicts possible).
3.  **None:** No networking. Sandbox.

**Connecting Containers (The Easy Way):**
```bash
# Create a network
docker network create my-net

# Connect containers
docker run -d --net my-net --name db postgres
docker run -d --net my-net --name api my-node-app

# Now 'api' can talk to 'db' using the hostname "db"
```

---

## 7. Data Persistence (Volumes)

Never save database files inside the container layer. If you delete the container, the data is gone.

### Types of Mounts
1.  **Named Volumes (Recommended):** Managed by Docker. Stored in `/var/lib/docker/volumes`. Best for databases.
    ```bash
    docker run -v my-db-data:/var/lib/postgresql/data postgres
    ```
2.  **Bind Mounts:** Maps a folder on your Host machine to the container. Great for development (live reloading).
    ```bash
    docker run -v ./src:/app/src node-app
    ```

---

## 8. Docker Compose & Orchestration

`docker-compose.yml` is your Infrastructure as Code (IaC) for local development.

```yaml
version: '3.8'
services:
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DB_HOST=postgres
      - NODE_ENV=development
    volumes:
      - ./src:/app/src # Enable code updates without restarting
    depends_on:
      - postgres
    networks:
      - app-network

  postgres:
    image: postgres:14-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - pg-data:/var/lib/postgresql/data # Persistence
    networks:
      - app-network

volumes:
  pg-data:

networks:
  app-network:
```

**Commands:**
*   `docker-compose up -d`: Start everything.
*   `docker-compose down`: Stop and remove containers/networks.
*   `docker-compose logs -f`: View logs for all services.

---

## 9. Security Best Practices �

1.  **Don't run as Root:** By default, containers run as root. This is dangerous. Create a user in your Dockerfile.
    ```dockerfile
    RUN addgroup -S appgroup && adduser -S appuser -G appgroup
    USER appuser
    ```
2.  **Scan Images:** Use tools like `trivy` or `docker scan` to find vulnerabilities in base images.
3.  **Minimal Base Images:** Use `alpine` or `Google Distroless` images to reduce the attack surface.
4.  **Secrets Management:** Never hardcode passwords in Dockerfiles or Encode them in ENV vars if possible. Use Docker Secrets (Swarm) or simply inject them at runtime via `.env` files not committed to git.
5.  **Read-Only Filesystems:** If your app doesn't need to write to disk, enforce it.
    ```bash
    docker run --read-only my-app
    ```

---

## 10. Production Optimization

1.  **Layer Caching:** Order matters! Put `COPY package.json` and `npm install` *before* `COPY .`. This ensures dependencies are cached unless `package.json` changes.
2.  **Health Checks:** Tell Docker how to know if your app is actually alive (not just the process running, but serving traffic).
    ```dockerfile
    HEALTHCHECK --interval=30s --timeout=3s \
      CMD curl -f http://localhost/ || exit 1
    ```
3.  **Resource Limits:** Prevent a single container from killing your server.
    ```bash
    docker run --memory="512m" --cpus="1.0" my-app
    ```

---

## 11. Troubleshooting & Debugging

**"My container exits immediately!"**
*   Check logs: `docker logs <id>`
*   Common cause: The command finished. A container only lives as long as its main PID. (e.g., using `nohup` or backgrounding a process usually kills the container).

**"I can't connect to localhost:8080"**
*   Did you map ports? `-p 8080:80`.
*   Is the app inside listening on `0.0.0.0`? If it listens on `127.0.0.1` inside the container, it will NOT be accessible from outside.

**"I need to see inside the container"**
*   `docker exec -it <container_id> /bin/sh` (or `/bin/bash`).
*   This gives you a terminal *inside* the running container.

---
**Next Steps:** Once you master this, you are ready for **Kubernetes**, which takes these concepts and scales them across hundreds of servers.
