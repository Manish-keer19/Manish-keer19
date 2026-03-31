# 🐳 Docker Mastery Guide
## Beginner → Intermediate → Advanced → Production → Expert → Master

> **A Complete Industry-Level Docker Handbook**
> *Written from 15+ years of real-world production experience*

---

```
╔══════════════════════════════════════════════════════════════════╗
║          DOCKER MASTERY GUIDE — COMPLETE EDITION               ║
║   From Zero to Production-Grade Container Architecture          ║
╚══════════════════════════════════════════════════════════════════╝
```

---

## 📑 TABLE OF CONTENTS

### LEVEL 1 — BEGINNER
- [Chapter 1: What is Docker?](#chapter-1-what-is-docker)
- [Chapter 2: Installation & Setup](#chapter-2-installation--setup)
- [Chapter 3: Images vs Containers](#chapter-3-images-vs-containers)
- [Chapter 4: Docker CLI Basics](#chapter-4-docker-cli-basics)
- [Chapter 5: Running Your First Container](#chapter-5-running-your-first-container)

### LEVEL 2 — INTERMEDIATE
- [Chapter 6: Dockerfile Deep Dive](#chapter-6-dockerfile-deep-dive)
- [Chapter 7: Volumes & Persistent Storage](#chapter-7-volumes--persistent-storage)
- [Chapter 8: Docker Networking](#chapter-8-docker-networking)
- [Chapter 9: Environment Variables & Configuration](#chapter-9-environment-variables--configuration)

### LEVEL 3 — ADVANCED
- [Chapter 10: Multi-Stage Builds](#chapter-10-multi-stage-builds)
- [Chapter 11: Docker Compose](#chapter-11-docker-compose)
- [Chapter 12: Image Optimization](#chapter-12-image-optimization)
- [Chapter 13: Layer Caching & Build Performance](#chapter-13-layer-caching--build-performance)

### LEVEL 4 — PRODUCTION
- [Chapter 14: CI/CD with Docker](#chapter-14-cicd-with-docker)
- [Chapter 15: Docker Security](#chapter-15-docker-security)
- [Chapter 16: Private Registry](#chapter-16-private-registry)
- [Chapter 17: Scaling Containers](#chapter-17-scaling-containers)

### LEVEL 5 — EXPERT
- [Chapter 18: Docker Internals](#chapter-18-docker-internals)
- [Chapter 19: Namespaces & cgroups](#chapter-19-namespaces--cgroups)
- [Chapter 20: Storage Drivers](#chapter-20-storage-drivers)
- [Chapter 21: Networking Internals](#chapter-21-networking-internals)

### LEVEL 6 — MASTER
- [Chapter 22: Kubernetes Integration](#chapter-22-kubernetes-integration)
- [Chapter 23: Microservices Architecture](#chapter-23-microservices-architecture)
- [Chapter 24: High Availability Design](#chapter-24-high-availability-design)
- [Chapter 25: Production Design Patterns](#chapter-25-production-design-patterns)

### REAL WORLD PROJECTS
- [Project 1: Node.js App in Docker](#project-1-nodejs-app-in-docker)
- [Project 2: Full Stack App with Docker Compose](#project-2-full-stack-app-with-docker-compose)
- [Project 3: Microservices Architecture](#project-3-microservices-architecture)
- [Project 4: Production Deployment Pipeline](#project-4-production-deployment-pipeline)

### APPENDICES
- [Debugging Section](#debugging-section)
- [Production Folder Structure](#production-folder-structure)
- [Command Reference Cheat Sheet](#command-reference-cheat-sheet)

---

# LEVEL 1 — BEGINNER

---

## Chapter 1: What is Docker?

### 📚 What You Will Learn
- The problem Docker solves
- How Docker differs from virtual machines
- Core Docker concepts: daemon, client, image, container, registry
- Docker's place in the modern software ecosystem

### 🔍 Why This Exists

Before Docker, developers faced the infamous **"it works on my machine"** problem. Applications behaved differently across development, staging, and production environments because:
- OS versions differed
- Library/dependency versions conflicted
- System configurations varied
- Manual server setups were inconsistent and error-prone

Docker was created to **package an application with everything it needs** — code, runtime, libraries, system tools — into a standardized, portable unit called a **container**.

### ⚙️ How It Works Internally

Docker uses Linux kernel features to create isolated environments:

```
┌─────────────────────────────────────────────────────────┐
│                    HOST MACHINE                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │              DOCKER ENGINE                       │   │
│  │  ┌──────────────┐  ┌──────────────┐             │   │
│  │  │  Container A  │  │  Container B  │             │   │
│  │  │  ┌─────────┐ │  │  ┌─────────┐ │             │   │
│  │  │  │  App    │ │  │  │  App    │ │             │   │
│  │  │  │  Libs   │ │  │  │  Libs   │ │             │   │
│  │  │  └─────────┘ │  │  └─────────┘ │             │   │
│  │  └──────────────┘  └──────────────┘             │   │
│  │         Shared Linux Kernel                      │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### 🏗️ Architecture Explanation

Docker follows a **Client-Server architecture**:

```
┌─────────────┐         ┌──────────────────────────────────┐
│ Docker CLI  │ ──REST──▶│         Docker Daemon            │
│  (client)   │         │  ┌────────────┐ ┌─────────────┐  │
└─────────────┘         │  │  Images    │ │ Containers  │  │
                        │  └────────────┘ └─────────────┘  │
                        │  ┌────────────┐ ┌─────────────┐  │
                        │  │  Volumes   │ │  Networks   │  │
                        │  └────────────┘ └─────────────┘  │
                        └──────────────────────────────────┘
                                         │
                                         ▼
                              ┌────────────────────┐
                              │   Docker Registry  │
                              │   (Docker Hub /    │
                              │   Private Registry)│
                              └────────────────────┘
```

**Core Components:**

| Component | Role |
|-----------|------|
| Docker Daemon (`dockerd`) | Background service managing containers, images, volumes |
| Docker CLI | Command-line tool to interact with daemon |
| Docker Image | Read-only template to create containers |
| Docker Container | Running instance of an image |
| Docker Registry | Storage for Docker images (Docker Hub, ECR, GCR) |
| Dockerfile | Blueprint for building images |

### 🆚 Docker vs Virtual Machines

```
VIRTUAL MACHINE                      DOCKER CONTAINER
┌────────────────────────┐           ┌────────────────────────┐
│ App A    │ App B       │           │ App A    │ App B       │
├──────────┼─────────────┤           ├──────────┼─────────────┤
│ Guest OS │ Guest OS    │           │ Libs A   │ Libs B      │
├──────────┴─────────────┤           ├──────────┴─────────────┤
│      Hypervisor        │           │      Docker Engine      │
├────────────────────────┤           ├────────────────────────┤
│       Host OS          │           │       Host OS           │
├────────────────────────┤           ├────────────────────────┤
│      Hardware          │           │       Hardware          │
└────────────────────────┘           └────────────────────────┘
Size: GBs | Boot: Minutes            Size: MBs | Boot: Seconds
```

### 🌍 Real World Example

A Python Flask application needs Python 3.11, specific pip packages, and environment variables. Instead of asking every developer to manually install these, you write a `Dockerfile` once. Anyone — dev, QA, DevOps — runs the same container with identical behavior.

### 🔧 Common Mistakes
- Confusing containers with VMs — containers share the host kernel
- Thinking Docker is only for Linux — Docker Desktop runs on Mac/Windows too
- Using containers as persistent storage without volumes
- Running everything as root inside containers

### ✅ Best Practices
- Think of containers as **immutable**, **ephemeral** units
- Build **one process per container**
- Use Docker for both dev and prod environments
- Never store application state inside the container filesystem

### 🧑‍💻 Hands-On Tasks
1. Research Docker's history — when was it released and why?
2. Draw your own diagram of Docker architecture
3. List 5 problems Docker solves in your team's workflow

---

## Chapter 2: Installation & Setup

### 📚 What You Will Learn
- Installing Docker on Linux, macOS, and Windows
- Understanding Docker Desktop
- Post-install configuration
- Verifying your installation

### 🔍 Why This Exists

Docker needs to be properly installed and configured before use. Installation methods vary by OS, and post-install steps (like adding your user to the docker group) are often missed by beginners, leading to permission errors.

### ⚙️ How It Works Internally

On **Linux**, Docker installs directly as a system service (`dockerd`) that communicates via a Unix socket at `/var/run/docker.sock`.

On **macOS/Windows**, Docker Desktop creates a lightweight Linux VM (using HyperKit on Mac, WSL2 on Windows) because Docker containers require a Linux kernel.

```
macOS / Windows
┌──────────────────────────────────┐
│         Docker Desktop           │
│  ┌────────────────────────────┐  │
│  │   Linux VM (HyperKit/WSL2) │  │
│  │   ┌──────────────────────┐ │  │
│  │   │    Docker Daemon      │ │  │
│  │   │    Containers         │ │  │
│  │   └──────────────────────┘ │  │
│  └────────────────────────────┘  │
└──────────────────────────────────┘
```

### 💻 Installation — Linux (Ubuntu/Debian)

```bash
# Step 1: Update package index
sudo apt-get update

# Step 2: Install prerequisites
sudo apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Step 3: Add Docker's official GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Step 4: Set up the repository
echo \
  "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Step 5: Install Docker Engine
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io \
    docker-buildx-plugin docker-compose-plugin

# Step 6: Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker

# Step 7: Add user to docker group (avoid sudo)
sudo usermod -aG docker $USER
newgrp docker
```

### 💻 Installation — macOS

```bash
# Option 1: Download Docker Desktop
# https://www.docker.com/products/docker-desktop/

# Option 2: Homebrew
brew install --cask docker

# Launch Docker Desktop from Applications
# Wait for the whale icon to appear in the menu bar
```

### 💻 Installation — Windows

```powershell
# Requirements: Windows 10/11 with WSL2 enabled
# Step 1: Enable WSL2
wsl --install

# Step 2: Download Docker Desktop for Windows
# https://www.docker.com/products/docker-desktop/

# Step 3: During install, ensure "Use WSL2" is checked
```

### ✅ Verify Installation

```bash
# Check Docker version
docker --version
# Output: Docker version 26.x.x, build xxxxxx

# Check Docker info
docker info

# Run hello-world
docker run hello-world

# Check Docker Compose
docker compose version
```

### 🔧 Post-Install Configuration

```bash
# Configure Docker daemon settings
sudo nano /etc/docker/daemon.json
```

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 64000,
      "Soft": 64000
    }
  }
}
```

```bash
# Restart Docker after config change
sudo systemctl restart docker
```

### 🔧 Common Mistakes
- Forgetting to add user to docker group → always needing `sudo`
- Not enabling WSL2 on Windows before Docker Desktop
- Skipping `systemctl enable docker` → Docker doesn't start on reboot
- Using outdated Docker versions from default apt repos

### ✅ Best Practices
- Always install from Docker's official repository, not distro repos
- Use Docker Desktop for local development on Mac/Windows
- Configure log rotation to prevent disk fill
- Use `docker context` to manage multiple Docker endpoints

### 🧑‍💻 Hands-On Tasks
1. Install Docker on your machine
2. Run `docker run hello-world` and understand the output
3. Explore `docker info` and identify 5 key fields
4. Set up log rotation in `daemon.json`

### 🐞 Debugging

```bash
# Check if Docker daemon is running
sudo systemctl status docker

# View Docker daemon logs
sudo journalctl -u docker.service -f

# Test socket permissions
ls -la /var/run/docker.sock

# Fix permission denied error
sudo chmod 666 /var/run/docker.sock  # ⚠️ dev only, not production
```

---

## Chapter 3: Images vs Containers

### 📚 What You Will Learn
- What a Docker image is
- What a Docker container is
- The layered filesystem
- Image lifecycle and container lifecycle
- How images become containers

### 🔍 Why This Exists

This is the most fundamental concept in Docker. Confusing images and containers causes most beginner mistakes. Understanding the relationship between them is essential for every Docker operation.

### ⚙️ How It Works Internally

A Docker **image** is built from **layers**. Each instruction in a Dockerfile creates a new layer. Layers are **read-only** and **cached**.

A Docker **container** is a running image with an additional **writable layer** on top.

```
IMAGE (Read-Only Layers)          CONTAINER
┌─────────────────────┐           ┌─────────────────────┐
│   Layer 4: App      │           │   Writable Layer     │ ← container
├─────────────────────┤           ├─────────────────────┤
│   Layer 3: npm pkgs │           │   Layer 4: App      │ ← image
├─────────────────────┤           ├─────────────────────┤
│   Layer 2: Node.js  │           │   Layer 3: npm pkgs │ ← image
├─────────────────────┤           ├─────────────────────┤
│   Layer 1: Ubuntu   │           │   Layer 2: Node.js  │ ← image
└─────────────────────┘           ├─────────────────────┤
                                  │   Layer 1: Ubuntu   │ ← image
                                  └─────────────────────┘
```

### 🏗️ Image Anatomy

```bash
# Pull an image
docker pull node:18-alpine

# Inspect image layers
docker history node:18-alpine

# Inspect image metadata
docker inspect node:18-alpine
```

```
node:18-alpine
├── sha256:a3e... (base alpine layer)
├── sha256:b4f... (add node binaries)
├── sha256:c5a... (npm config)
└── Total: ~180MB compressed
```

### 🔧 Commands

```bash
# IMAGE COMMANDS
docker images                          # List all local images
docker images -a                       # Include intermediate images
docker pull nginx:1.25                 # Pull specific version
docker pull nginx:latest               # Pull latest
docker image inspect nginx             # Detailed image info
docker image history nginx             # Show image layers
docker image ls --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
docker image prune                     # Remove dangling images
docker image prune -a                  # Remove all unused images
docker rmi nginx                       # Remove an image
docker tag nginx:latest myapp:v1.0     # Tag an image

# CONTAINER COMMANDS
docker ps                              # List running containers
docker ps -a                           # List all containers
docker run nginx                       # Run a container
docker run -d nginx                    # Run detached (background)
docker run -it ubuntu bash             # Interactive terminal
docker start container_name            # Start stopped container
docker stop container_name             # Gracefully stop
docker kill container_name             # Force stop
docker rm container_name               # Remove container
docker rm -f container_name            # Force remove (running)
docker inspect container_name          # Detailed container info
```

### 🌍 Real World Example

```bash
# Pull the official Python image
docker pull python:3.11-slim

# See it in your local image store
docker images python

# Create a container from it (just create, don't run)
docker create --name myapp python:3.11-slim

# Start the created container
docker start myapp

# Now it's a running container!
docker ps

# Stop it
docker stop myapp

# Remove the container (image still exists)
docker rm myapp

# Verify image still exists
docker images python
```

### 📊 Image vs Container Comparison

| Property | Image | Container |
|----------|-------|-----------|
| State | Static, read-only | Dynamic, running process |
| Storage | Disk (layers) | Memory + thin writable layer |
| Persistence | Permanent until deleted | Temporary by default |
| Creation | `docker build` or `docker pull` | `docker run` |
| Sharing | Via registry | Cannot be shared directly |
| Count | One image | Many containers from one image |

### 🔧 Common Mistakes
- Thinking container changes persist to the image — they don't (without `docker commit`)
- Deleting images while containers still reference them
- Accumulating stopped containers and unused images
- Not tagging images with meaningful versions

### ✅ Best Practices
- Always use specific image tags, never rely on `latest` in production
- Clean up stopped containers regularly
- Use `docker system prune` in CI/CD pipelines
- Store images in a private registry for production

### 🧑‍💻 Hands-On Tasks
1. Pull 3 different images and compare their sizes
2. Run a container, modify a file inside, stop and restart — observe changes are preserved within the same container
3. Remove the container and run again — observe changes are gone
4. Run `docker history ubuntu` and understand each layer

---

## Chapter 4: Docker CLI Basics

### 📚 What You Will Learn
- The most important Docker commands
- How to read Docker command structure
- Container lifecycle management
- Output formatting and filtering

### 🔍 Why This Exists

The Docker CLI is your primary interface to Docker. Mastering it allows you to manage containers efficiently, debug problems quickly, and automate operations in scripts.

### ⚙️ How It Works Internally

The Docker CLI sends API requests to the Docker daemon via a Unix socket (`/var/run/docker.sock`) or TCP. Every `docker` command translates to an HTTP call to the Docker Engine REST API.

```
docker ps
    │
    ▼
POST /v1.43/containers/json HTTP/1.1
Host: unix:///var/run/docker.sock
    │
    ▼
Docker Daemon processes request
    │
    ▼
Returns JSON → CLI formats output
```

### 🔧 Essential Commands Reference

```bash
# ═══════════════════════════════════════
# CONTAINER LIFECYCLE
# ═══════════════════════════════════════

# Run container (pulls if not local)
docker run <image>

# Run with options
docker run \
  -d \                          # detached mode
  --name my-nginx \             # name the container
  -p 8080:80 \                  # port mapping host:container
  -e MY_VAR=value \             # environment variable
  -v /host/path:/container/path \ # volume mount
  --memory="256m" \             # memory limit
  --cpus="0.5" \                # CPU limit
  --restart unless-stopped \    # restart policy
  nginx:1.25

# List containers
docker ps                      # running
docker ps -a                   # all
docker ps -q                   # only IDs
docker ps --filter status=exited

# Stop / Start / Restart
docker stop <name|id>
docker start <name|id>
docker restart <name|id>

# Remove
docker rm <name|id>
docker rm $(docker ps -aq)     # remove all stopped
docker rm -f <name|id>         # force remove running

# ═══════════════════════════════════════
# INTERACTION
# ═══════════════════════════════════════

# Execute command in running container
docker exec -it <name> bash
docker exec -it <name> sh       # for alpine
docker exec <name> ls /app

# Attach to running container's stdin/stdout
docker attach <name>

# Copy files
docker cp <name>:/app/file.txt ./file.txt    # container → host
docker cp ./file.txt <name>:/app/file.txt    # host → container

# ═══════════════════════════════════════
# LOGS
# ═══════════════════════════════════════

docker logs <name>              # view logs
docker logs -f <name>           # follow/tail logs
docker logs --tail 50 <name>    # last 50 lines
docker logs --since 1h <name>   # last 1 hour

# ═══════════════════════════════════════
# INSPECTION
# ═══════════════════════════════════════

docker inspect <name>           # full JSON info
docker inspect -f '{{.NetworkSettings.IPAddress}}' <name>
docker stats                    # live resource usage
docker stats --no-stream        # one-time snapshot
docker top <name>               # processes in container
docker port <name>              # port mappings

# ═══════════════════════════════════════
# SYSTEM
# ═══════════════════════════════════════

docker system df                # disk usage
docker system prune             # remove unused resources
docker system prune -a          # remove everything unused
docker system info              # system-wide info

docker events                   # real-time Docker events
docker events --filter type=container
```

### 🌍 Real World Example

```bash
# Production-style container run
docker run \
  -d \
  --name webapp \
  --restart unless-stopped \
  -p 80:3000 \
  -e NODE_ENV=production \
  -e DATABASE_URL=postgresql://user:pass@db:5432/mydb \
  --memory="512m" \
  --cpus="1.0" \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  myorg/webapp:v2.1.0

# Verify it's running
docker ps --filter name=webapp

# Follow logs
docker logs -f webapp

# Check resource usage
docker stats webapp --no-stream
```

### 📊 Format Output with Go Templates

```bash
# Custom format
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# JSON output
docker inspect <name> --format '{{json .Config}}'

# Get specific field
docker inspect <name> --format '{{.State.Pid}}'

# Get mount points
docker inspect <name> --format '{{range .Mounts}}{{.Source}} → {{.Destination}}{{"\n"}}{{end}}'
```

### 🔧 Common Mistakes
- Using `docker kill` when `docker stop` is appropriate (kill sends SIGKILL immediately)
- Forgetting `-d` flag and losing the terminal
- Not naming containers with `--name` → hard to manage
- Exposing ports without binding to specific interface (`-p 0.0.0.0:80:80`)

### ✅ Best Practices
- Always name containers with `--name`
- Use `--restart unless-stopped` for production services
- Set memory and CPU limits on every production container
- Use `docker stats` to monitor resource consumption
- Log with max-size limits to prevent disk issues

### 🧑‍💻 Hands-On Tasks
1. Run an nginx container with name, port mapping, and restart policy
2. Use `docker exec` to explore the container filesystem
3. Use `docker stats` to watch resource usage
4. Practice `docker inspect` and extract the container IP address

---

## Chapter 5: Running Your First Container

### 📚 What You Will Learn
- The complete `docker run` lifecycle
- Port mapping explained
- Interactive vs detached mode
- Running real applications in containers

### 🔍 Why This Exists

Running your first container is where abstract concepts become real. This chapter bridges theory and practice, building the foundation for everything that follows.

### ⚙️ How It Works Internally

When you run `docker run nginx`:

```
docker run nginx
    │
    ├─ 1. Docker checks local image cache
    │       └─ Not found → pulls from Docker Hub
    │
    ├─ 2. Docker creates container namespace
    │       ├─ PID namespace (isolated process tree)
    │       ├─ NET namespace (isolated network stack)
    │       ├─ MNT namespace (isolated filesystem)
    │       └─ UTS namespace (isolated hostname)
    │
    ├─ 3. Docker sets up networking
    │       └─ Assigns IP from docker0 bridge
    │
    ├─ 4. Docker mounts layered filesystem
    │       └─ image layers (read-only) + writable layer
    │
    └─ 5. Docker starts entrypoint process
            └─ nginx: master process
```

### 🔧 Step-by-Step First Containers

```bash
# ── EXAMPLE 1: Hello World ──────────────────────────────
docker run hello-world

# ── EXAMPLE 2: Interactive Ubuntu Shell ─────────────────
docker run -it ubuntu:22.04 bash
# Now you're inside the container!
whoami      # root
hostname    # container ID
ls /        # Ubuntu filesystem
exit        # leave container

# ── EXAMPLE 3: Web Server ───────────────────────────────
docker run -d -p 8080:80 --name webserver nginx:alpine
# Open http://localhost:8080 in browser

# ── EXAMPLE 4: Database ─────────────────────────────────
docker run -d \
  --name postgres-dev \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=myapp \
  -p 5432:5432 \
  postgres:15-alpine

# Connect to it
docker exec -it postgres-dev psql -U admin -d myapp

# ── EXAMPLE 5: Run a command and exit ───────────────────
docker run --rm ubuntu:22.04 echo "Hello from container"
# --rm removes container after it exits

# ── EXAMPLE 6: Custom entrypoint ────────────────────────
docker run --rm -it --entrypoint sh nginx:alpine
```

### 🏗️ Port Mapping Deep Dive

```
Host Machine          Docker Container
─────────────────     ─────────────────
Port 8080         ──▶ Port 80 (nginx)
Port 5432         ──▶ Port 5432 (postgres)
Port 3000         ──▶ Port 3000 (node)

Command: -p <host_port>:<container_port>
Command: -p 127.0.0.1:8080:80   (bind to localhost only)
Command: -p 8080:80/udp         (UDP port)
```

```bash
# View port mappings
docker port webserver

# Bind to specific interface (security best practice)
docker run -d -p 127.0.0.1:8080:80 nginx

# Let Docker choose the host port
docker run -d -p 80 nginx   # docker assigns random host port
docker port <name>           # see what port was assigned
```

### 🌍 Real World Example: Python App

```bash
# Run a Python script in a container without installing Python
docker run --rm -v $(pwd):/app -w /app python:3.11-slim python script.py
```

### 🔧 Common Mistakes
- Forgetting `-p` → container runs but app not accessible from host
- Forgetting `-d` → container runs in foreground, blocks terminal
- Running database containers without volumes → data lost on restart
- Not using `--rm` for one-off tasks → accumulating stopped containers

### ✅ Best Practices
- Use `--rm` for temporary or one-off containers
- Bind to `127.0.0.1` for services that shouldn't be externally accessible
- Always check `docker ps` after running a container
- Use named containers in development for easy management

### 🧑‍💻 Hands-On Tasks
1. Run nginx on port 8080 and verify in browser
2. Run PostgreSQL, connect to it with `psql`, create a table
3. Run an Ubuntu container interactively and explore the filesystem
4. Run Python in a container to execute a local script
5. Use `docker stats` to monitor all running containers

---

# LEVEL 2 — INTERMEDIATE

---

## Chapter 6: Dockerfile Deep Dive

### 📚 What You Will Learn
- Every Dockerfile instruction in depth
- Build context and `.dockerignore`
- Best practices for production Dockerfiles
- ARG vs ENV, COPY vs ADD, CMD vs ENTRYPOINT

### 🔍 Why This Exists

The Dockerfile is the blueprint for your image. Poorly written Dockerfiles produce large, insecure, slow-to-build images. A well-crafted Dockerfile produces lean, fast, reproducible images.

### ⚙️ How It Works Internally

```
Dockerfile instructions
        │
        ▼
docker build (sends build context to daemon)
        │
        ▼
Docker Daemon builds each instruction as a layer
        │
        ├─ Layer 1: FROM ubuntu (pull base)
        ├─ Layer 2: RUN apt-get install (executes, creates layer)
        ├─ Layer 3: COPY . /app (copies files)
        └─ Layer 4: CMD (metadata, no new layer)
        │
        ▼
Resulting image (stack of layers)
```

### 📜 Complete Dockerfile Instructions Reference

```dockerfile
# ══════════════════════════════════════════════════════════
# DOCKERFILE COMPLETE REFERENCE
# ══════════════════════════════════════════════════════════

# FROM — Base image (REQUIRED, must be first)
FROM ubuntu:22.04
FROM node:18-alpine AS builder          # Named stage for multi-stage
FROM scratch                            # Minimal empty base

# ARG — Build-time variables (before FROM or after)
ARG NODE_VERSION=18
ARG BUILD_DATE
FROM node:${NODE_VERSION}-alpine

# ENV — Runtime environment variables
ENV NODE_ENV=production
ENV APP_HOME=/app \
    PORT=3000

# LABEL — Image metadata
LABEL maintainer="devops@company.com" \
      version="1.0" \
      description="My Application"

# WORKDIR — Set working directory (creates if not exists)
WORKDIR /app

# COPY — Copy files from build context to image
COPY package.json package-lock.json ./
COPY . .
COPY --from=builder /app/dist ./dist   # From another build stage

# ADD — Like COPY but can extract archives and fetch URLs
ADD archive.tar.gz /app/               # Auto-extracts
ADD https://example.com/file.txt /app/ # ⚠️ Avoid URL form; use curl

# RUN — Execute commands during build (creates layer)
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        ca-certificates && \
    rm -rf /var/lib/apt/lists/*         # Clean in same layer!

# USER — Set the user for subsequent instructions
RUN groupadd -r appuser && useradd -r -g appuser appuser
USER appuser

# EXPOSE — Document which ports the app uses (informational)
EXPOSE 3000
EXPOSE 3000/tcp
EXPOSE 53/udp

# VOLUME — Mark directory as volume mount point
VOLUME ["/app/data"]
VOLUME /app/logs

# CMD — Default command (can be overridden at runtime)
CMD ["node", "server.js"]              # Exec form (PREFERRED)
CMD node server.js                     # Shell form

# ENTRYPOINT — Main executable (cannot be easily overridden)
ENTRYPOINT ["docker-entrypoint.sh"]   # Exec form
ENTRYPOINT ["/usr/bin/nginx", "-g", "daemon off;"]

# HEALTHCHECK — How to check if container is healthy
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1

# ONBUILD — Trigger instruction for child images
ONBUILD COPY . /app
ONBUILD RUN npm install

# STOPSIGNAL — Signal to stop the container
STOPSIGNAL SIGTERM

# SHELL — Change default shell for RUN commands
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
```

### 🏗️ CMD vs ENTRYPOINT

```
┌─────────────────────────────────────────────────────────┐
│                 CMD vs ENTRYPOINT                        │
├────────────────┬───────────────────┬────────────────────┤
│ Dockerfile     │ docker run args   │ What runs          │
├────────────────┼───────────────────┼────────────────────┤
│ CMD ["nginx"]  │ (none)            │ nginx              │
│ CMD ["nginx"]  │ bash              │ bash               │
├────────────────┼───────────────────┼────────────────────┤
│ ENTRYPOINT     │ (none)            │ nginx              │
│ ["nginx"]      │                   │                    │
│ ENTRYPOINT     │ -g daemon off;    │ nginx -g daemon off│
│ ["nginx"]      │                   │ (args appended)    │
├────────────────┼───────────────────┼────────────────────┤
│ ENTRYPOINT     │ (none)            │ nginx default      │
│ ["nginx"]      │                   │                    │
│ CMD ["default"]│                   │                    │
│ ENTRYPOINT     │ -v                │ nginx -v           │
│ ["nginx"]      │                   │ (CMD overridden)   │
│ CMD ["default"]│                   │                    │
└────────────────┴───────────────────┴────────────────────┘
```

### 🌍 Real World Dockerfiles

#### Node.js Production Dockerfile

```dockerfile
# ── Stage 1: Dependencies ──────────────────────────────
FROM node:18-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --only=production

# ── Stage 2: Build ─────────────────────────────────────
FROM node:18-alpine AS builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build

# ── Stage 3: Production Runtime ────────────────────────
FROM node:18-alpine AS runner
WORKDIR /app

# Security: non-root user
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nextjs

# Copy only what's needed
COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY package.json ./

# Health check
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
    CMD wget -qO- http://localhost:3000/health || exit 1

USER nextjs
EXPOSE 3000
ENV NODE_ENV=production
CMD ["node", "dist/server.js"]
```

#### Python Production Dockerfile

```dockerfile
FROM python:3.11-slim AS base

# Prevent Python from buffering stdout/stderr
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

WORKDIR /app

# Install system dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        build-essential \
        libpq-dev && \
    rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Non-root user
RUN useradd -m -u 1000 appuser

# Copy application
COPY --chown=appuser:appuser . .

USER appuser

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=5s \
    CMD python -c "import requests; requests.get('http://localhost:8000/health')"

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "app:app"]
```

### 📄 .dockerignore File

```
# .dockerignore — Prevents unnecessary files from entering build context

# Version control
.git
.gitignore

# Dependencies (will be installed in container)
node_modules
vendor
__pycache__
*.pyc

# Build artifacts
dist
build
.next
target

# IDE files
.idea
.vscode
*.swp

# OS files
.DS_Store
Thumbs.db

# Environment files (sensitive!)
.env
.env.local
.env.production

# Test files
test
tests
spec
*.test.js
*.spec.js
coverage

# Documentation
README.md
docs

# Docker files themselves
Dockerfile*
docker-compose*
```

### 🔧 Build Commands

```bash
# Basic build
docker build -t myapp:1.0 .

# Build with different Dockerfile name
docker build -f Dockerfile.prod -t myapp:prod .

# Build with build arguments
docker build \
  --build-arg NODE_ENV=production \
  --build-arg APP_VERSION=1.2.3 \
  -t myapp:1.2.3 .

# Build with no cache
docker build --no-cache -t myapp:latest .

# Build and push in one command
docker build -t myregistry.com/myapp:v1.0 . && \
  docker push myregistry.com/myapp:v1.0

# Build for multiple platforms (buildx)
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t myapp:latest \
  --push .
```

### 🔧 Common Mistakes
- Installing packages and not cleaning apt cache in the same `RUN` layer
- Using `ADD` instead of `COPY` for local files (use COPY always)
- Copying entire context with `COPY . .` without `.dockerignore`
- Running as root in production containers
- Using `CMD` with shell form — can't receive signals properly
- Not pinning base image versions → non-deterministic builds

### ✅ Best Practices
- Always use exec form (`["cmd", "arg"]`) for CMD and ENTRYPOINT
- Clean up in the same RUN layer as installation
- Use non-root users in production
- Pin base image versions (e.g., `node:18.19.0-alpine3.18`)
- Use `.dockerignore` aggressively
- Separate dependency install from code copy for better caching

### 🧑‍💻 Hands-On Tasks
1. Write a Dockerfile for a Node.js Express app from scratch
2. Add a non-root user and a healthcheck
3. Test CMD override at `docker run`
4. Write a `.dockerignore` and measure build context size difference

---

## Chapter 7: Volumes & Persistent Storage

### 📚 What You Will Learn
- Types of Docker storage
- Named volumes, bind mounts, tmpfs
- Volume lifecycle management
- Production storage patterns

### 🔍 Why This Exists

Containers are ephemeral — when a container is removed, its writable layer is gone. Volumes provide persistent storage that exists independently of container lifecycle.

### ⚙️ How It Works Internally

```
Docker Storage Types
┌───────────────────────────────────────────────────────┐
│                                                       │
│  ┌─────────────────┐  ┌──────────────┐  ┌─────────┐  │
│  │  Named Volume   │  │ Bind Mount   │  │  tmpfs  │  │
│  │                 │  │              │  │         │  │
│  │ /var/lib/docker │  │ Any host dir │  │ Memory  │  │
│  │ /volumes/mydata │  │ /home/user/  │  │ only    │  │
│  │                 │  │ myproject    │  │         │  │
│  │ Managed by      │  │ Managed by   │  │ Lost on │  │
│  │ Docker          │  │ you          │  │ restart │  │
│  └─────────────────┘  └──────────────┘  └─────────┘  │
│           │                   │               │       │
│           └───────────────────┴───────────────┘       │
│                               │                       │
│                    ┌──────────▼──────────┐            │
│                    │     Container       │            │
│                    │   /app/data  → Vol  │            │
│                    └─────────────────────┘            │
└───────────────────────────────────────────────────────┘
```

### 🔧 Volume Commands

```bash
# ── NAMED VOLUMES ─────────────────────────────────────
docker volume create mydata
docker volume ls
docker volume inspect mydata
docker volume rm mydata
docker volume prune                     # remove all unused

# Run with named volume
docker run -d \
  --name postgres \
  -v pgdata:/var/lib/postgresql/data \
  postgres:15-alpine

# ── BIND MOUNTS ───────────────────────────────────────
# Mount host directory into container
docker run -d \
  --name webapp \
  -v $(pwd)/src:/app/src \             # development hot reload
  node:18-alpine

# Read-only bind mount
docker run -d \
  -v $(pwd)/config:/app/config:ro \
  myapp:latest

# ── TMPFS MOUNTS ──────────────────────────────────────
# In-memory storage (no disk I/O, lost on restart)
docker run -d \
  --tmpfs /app/cache:size=100m \
  myapp:latest

# ── MULTIPLE VOLUMES ──────────────────────────────────
docker run -d \
  --name myapp \
  -v app-data:/app/data \
  -v app-logs:/app/logs \
  -v $(pwd)/config:/app/config:ro \
  myapp:latest
```

### 🌍 Real World Example: Database with Volume

```bash
# Create a dedicated volume for database
docker volume create postgres-prod-data

# Run PostgreSQL with persistent storage
docker run -d \
  --name postgres-prod \
  --restart unless-stopped \
  -e POSTGRES_USER=appuser \
  -e POSTGRES_PASSWORD=strongpassword \
  -e POSTGRES_DB=production \
  -v postgres-prod-data:/var/lib/postgresql/data \
  -p 127.0.0.1:5432:5432 \
  postgres:15.4-alpine

# Create a database backup
docker exec postgres-prod pg_dump -U appuser production > backup.sql

# Restore from backup
docker exec -i postgres-prod psql -U appuser production < backup.sql

# Inspect where volume data is stored on host
docker volume inspect postgres-prod-data
# Location: /var/lib/docker/volumes/postgres-prod-data/_data
```

### 📊 Storage Comparison

| Feature | Named Volume | Bind Mount | tmpfs |
|---------|-------------|------------|-------|
| Managed by | Docker | User | Docker |
| Portability | High | Low (host paths) | N/A |
| Performance | Good | OS-dependent | Excellent |
| Persistence | Yes | Yes | No |
| Backup | `docker volume` | `cp` from host | N/A |
| Use case | DB, app data | Dev hot-reload | Secrets, cache |

### 📁 Volume in Compose (Preview)

```yaml
services:
  db:
    image: postgres:15-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro

  app:
    image: myapp:latest
    volumes:
      - app-uploads:/app/uploads
      - ./config/app.yml:/app/config.yml:ro

volumes:
  pgdata:
    driver: local
  app-uploads:
    driver: local
```

### 🔧 Common Mistakes
- Using bind mounts in production (host path dependency)
- Not using volumes for database containers → data loss
- Mounting too much of the host filesystem → security risk
- Forgetting `:ro` flag for config files that shouldn't be modified
- Not backing up volumes before container operations

### ✅ Best Practices
- Use named volumes for databases and persistent app data
- Use bind mounts only in development for hot-reload
- Use tmpfs for sensitive temporary data (secrets, tokens)
- Label volumes for identification
- Implement regular volume backup procedures
- Use volume drivers for cloud storage (AWS EBS, Azure Disk)

### 🧑‍💻 Hands-On Tasks
1. Run MySQL with a named volume, add data, remove container, recreate — verify data persists
2. Use bind mount to develop a Node.js app with hot reload
3. Backup and restore a PostgreSQL volume
4. Use `docker volume inspect` to find where data lives on disk

---

## Chapter 8: Docker Networking

### 📚 What You Will Learn
- Docker network types (bridge, host, none, overlay)
- Container-to-container communication
- DNS resolution in Docker
- Custom network creation and management

### 🔍 Why This Exists

Docker networking determines how containers communicate with each other and with the outside world. Understanding it is essential for multi-container applications and microservices.

### ⚙️ How It Works Internally

```
Docker Networking Architecture

Internet
    │
    │ (port 80 → -p 80:80)
    ▼
Host eth0 (192.168.1.10)
    │
    ▼
docker0 bridge (172.17.0.1)
    │
    ├──── Container A (172.17.0.2)
    ├──── Container B (172.17.0.3)
    └──── Container C (172.17.0.4)

Custom Bridge Network
mynet (172.18.0.0/16)
    │
    ├──── frontend (172.18.0.2) ← can reach backend by name!
    └──── backend  (172.18.0.3)

DNS: Docker provides built-in DNS
     frontend → resolves "backend" → 172.18.0.3
```

### 🔧 Network Commands

```bash
# ── LIST AND INSPECT ──────────────────────────────────
docker network ls
docker network inspect bridge
docker network inspect mynet

# ── CREATE ────────────────────────────────────────────
# Custom bridge network
docker network create myapp-network

# With subnet
docker network create \
  --driver bridge \
  --subnet 172.20.0.0/16 \
  --ip-range 172.20.240.0/20 \
  --gateway 172.20.0.1 \
  myapp-network

# ── CONNECT / DISCONNECT ──────────────────────────────
docker network connect myapp-network container_name
docker network disconnect myapp-network container_name

# ── REMOVE ────────────────────────────────────────────
docker network rm myapp-network
docker network prune

# ── RUN WITH NETWORK ──────────────────────────────────
docker run -d \
  --name backend \
  --network myapp-network \
  mybackend:latest

docker run -d \
  --name frontend \
  --network myapp-network \
  -p 80:3000 \
  myfrontend:latest
```

### 🌍 Real World Example: Multi-Container App

```bash
# Create isolated network
docker network create webapp-net

# Start database (no port exposure — internal only)
docker run -d \
  --name postgres \
  --network webapp-net \
  -e POSTGRES_PASSWORD=secret \
  postgres:15-alpine

# Start API (knows about postgres by name)
docker run -d \
  --name api \
  --network webapp-net \
  -e DATABASE_URL=postgresql://postgres:secret@postgres:5432/mydb \
  -p 127.0.0.1:3001:3001 \
  myapi:latest

# Start frontend (knows about api by name)
docker run -d \
  --name frontend \
  --network webapp-net \
  -e API_URL=http://api:3001 \
  -p 80:80 \
  myfrontend:latest

# Verify connectivity
docker exec frontend ping api
docker exec api ping postgres
```

### 📊 Network Types

| Network | Description | Use Case |
|---------|-------------|----------|
| `bridge` | Default; isolated network per container | Development, single-host |
| `host` | Container shares host network stack | Performance-critical apps |
| `none` | No networking | Isolated batch jobs |
| `overlay` | Multi-host networking | Docker Swarm |
| `macvlan` | Container has own MAC address | Legacy app network access |

### 🔧 Common Mistakes
- Using default bridge network — containers can't find each other by name
- Exposing database ports to the public — massive security risk
- Not isolating services into separate networks
- Using host networking in production without understanding security implications

### ✅ Best Practices
- Always create custom bridge networks for multi-container apps
- Use container names for service discovery, not IPs
- Keep databases on internal networks, never expose to internet
- Use separate networks to isolate different application tiers

### 🧑‍💻 Hands-On Tasks
1. Create a custom bridge network and connect two containers
2. Verify name-based DNS resolution between containers
3. Try connecting containers on default bridge by name (it fails)
4. Build frontend + backend + DB on isolated network

---

## Chapter 9: Environment Variables & Configuration

### 📚 What You Will Learn
- ENV in Dockerfile vs runtime `-e` flag
- `.env` files with Docker and Docker Compose
- Secrets management approaches
- Configuration best practices for 12-factor apps

### 🔍 Why This Exists

Hardcoding configuration is an anti-pattern. Docker provides multiple mechanisms to inject configuration at runtime, keeping images portable across environments.

### ⚙️ How It Works Internally

```
Configuration Injection Pipeline

Build time:           ARG → only during build
Runtime (env):        ENV in Dockerfile → default
Runtime (run flag):   -e KEY=VALUE → override
Runtime (.env file):  --env-file .env → batch override
Runtime (secrets):    /run/secrets/ → mounted file
```

### 🔧 Methods of Passing Configuration

```bash
# ── INLINE ────────────────────────────────────────────
docker run -e NODE_ENV=production -e PORT=3000 myapp

# ── FROM HOST ENVIRONMENT ─────────────────────────────
export DB_PASSWORD=supersecret
docker run -e DB_PASSWORD myapp    # inherits from host

# ── FROM FILE ─────────────────────────────────────────
# .env file
NODE_ENV=production
PORT=3000
DB_HOST=postgres
DB_PASSWORD=supersecret
API_KEY=abc123xyz

docker run --env-file .env myapp

# ── IN DOCKERFILE ─────────────────────────────────────
FROM node:18-alpine
ENV NODE_ENV=production \
    PORT=3000 \
    LOG_LEVEL=info
```

### 🌍 Real World: 12-Factor App Configuration

```dockerfile
# Dockerfile — no hardcoded secrets
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
# Defaults with ENV
ENV PORT=8000 \
    LOG_LEVEL=INFO \
    WORKERS=4
CMD ["sh", "-c", "gunicorn --bind 0.0.0.0:${PORT} --workers ${WORKERS} app:app"]
```

```bash
# Production run with all config injected
docker run -d \
  --name myapi \
  --env-file /etc/myapp/prod.env \
  -e SECRET_KEY="$(cat /run/secrets/app_secret)" \
  myapi:v1.0
```

### 🔐 Secrets — Secure Approach

```bash
# ❌ BAD — secret in history, image layers
docker run -e DB_PASSWORD=mysecret myapp

# ✅ BETTER — read from file
docker run \
  -v /path/to/secrets/db_password.txt:/run/secrets/db_password:ro \
  -e DB_PASSWORD_FILE=/run/secrets/db_password \
  myapp

# ✅ BEST — Docker Swarm secrets (production)
echo "mysecretpassword" | docker secret create db_password -
docker service create \
  --secret db_password \
  myapp
# Secret available at /run/secrets/db_password inside container
```

### 🔧 Common Mistakes
- Putting secrets in Dockerfile ENV — visible in image layers
- Committing `.env` files to version control
- Using same `.env` for dev and prod
- Logging environment variables in application startup

### ✅ Best Practices
- Use secrets management (Vault, AWS Secrets Manager, Docker Secrets) for sensitive data
- Never put secrets in Docker images
- Use separate `.env` files per environment
- Validate required env vars at application startup

### 🧑‍💻 Hands-On Tasks
1. Run an app with env vars from inline flags, then from a file
2. Read a "secret" from a mounted file in the container
3. Write an entrypoint script that validates required env vars
4. Examine image layers to confirm secrets don't appear

---

# LEVEL 3 — ADVANCED

---

## Chapter 10: Multi-Stage Builds

### 📚 What You Will Learn
- What multi-stage builds are and why they matter
- Separating build environment from runtime
- Copying artifacts between stages
- Size reduction techniques

### 🔍 Why This Exists

Build environments need compilers, SDKs, development tools. Runtime environments don't. Before multi-stage builds, developers either shipped enormous images (with all build tools) or maintained complex CI scripts to separate build and runtime.

### ⚙️ How It Works Internally

```
Multi-Stage Build Flow

Stage 1: builder          Stage 2: runner
┌─────────────────┐      ┌──────────────────┐
│ node:18         │      │ node:18-alpine   │
│ npm install     │      │ (tiny runtime)   │
│ npm run build   │ ───▶ │ COPY dist ./dist │
│ (1.2GB with     │      │ (only 180MB!)    │
│  dev deps)      │      └──────────────────┘
└─────────────────┘
    Discarded!
```

### 🌍 Real World Multi-Stage Dockerfiles

#### Go Application (10MB final image!)

```dockerfile
# Stage 1: Build
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

# Stage 2: Minimal runtime
FROM alpine:3.18 AS runner
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/app .
EXPOSE 8080
CMD ["./app"]

# Result: ~10MB image vs 300MB+ without multi-stage
```

#### React Frontend (nginx server)

```dockerfile
# Stage 1: Build React app
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Serve with nginx
FROM nginx:1.25-alpine AS runner
# Remove default nginx config
RUN rm /etc/nginx/conf.d/default.conf
# Add custom nginx config
COPY nginx.conf /etc/nginx/conf.d/app.conf
# Copy built assets from builder
COPY --from=builder /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

```nginx
# nginx.conf
server {
    listen 80;
    server_name _;
    root /usr/share/nginx/html;
    index index.html;

    # Handle React Router (SPA)
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache static assets
    location ~* \.(js|css|png|jpg|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    gzip on;
    gzip_types text/plain text/css application/json application/javascript;
}
```

#### Java Spring Boot

```dockerfile
# Stage 1: Build with Maven
FROM maven:3.9-eclipse-temurin-17 AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

# Stage 2: Extract layers for better caching
FROM eclipse-temurin:17-jre-alpine AS layers
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
RUN java -Djarmode=layertools -jar app.jar extract

# Stage 3: Minimal runtime
FROM eclipse-temurin:17-jre-alpine AS runner
WORKDIR /app
RUN addgroup -S spring && adduser -S spring -G spring
COPY --from=layers /app/dependencies/ ./
COPY --from=layers /app/spring-boot-loader/ ./
COPY --from=layers /app/snapshot-dependencies/ ./
COPY --from=layers /app/application/ ./
USER spring
EXPOSE 8080
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```

### 🔧 Build Specific Stages

```bash
# Build only a specific stage (for debugging)
docker build --target builder -t myapp:builder .
docker build --target runner -t myapp:latest .

# Build with BuildKit (faster, better caching)
DOCKER_BUILDKIT=1 docker build -t myapp:latest .

# Or enable BuildKit by default in daemon.json
# { "features": { "buildkit": true } }
```

### 📊 Image Size Comparison

| Application | Without Multi-Stage | With Multi-Stage | Reduction |
|-------------|---------------------|------------------|-----------|
| Node.js app | 1.2 GB | 180 MB | 85% |
| Go app | 800 MB | 12 MB | 98% |
| React (nginx) | 1.0 GB | 25 MB | 97% |
| Java Spring | 500 MB | 280 MB | 44% |

### 🔧 Common Mistakes
- Not using `--no-install-recommends` in intermediate stages
- Copying entire build context instead of specific artifacts
- Not using `.dockerignore` — huge build contexts slow things down
- Keeping test frameworks and dev tools in final stage

### ✅ Best Practices
- Always use multi-stage builds for compiled languages
- Use Alpine or distroless images for final stage
- Copy only the exact artifacts needed at runtime
- Use `BuildKit` for parallel stage building

### 🧑‍💻 Hands-On Tasks
1. Build a Node.js app without multi-stage, measure image size
2. Convert to multi-stage, measure again
3. Build a Go binary with scratch as final stage
4. Build a React app served by nginx

---

## Chapter 11: Docker Compose

### 📚 What You Will Learn
- Docker Compose file structure
- Service, network, and volume definitions
- Compose commands and lifecycle
- Override files and environment-based configs

### 🔍 Why This Exists

Real applications have multiple services — frontend, backend, database, cache, message queue. Starting each with `docker run` and manual network setup is tedious and error-prone. Docker Compose defines the entire application stack in a single YAML file.

### ⚙️ How It Works Internally

```
docker-compose.yml defines:
┌─────────────────────────────────────────────────┐
│  services:          networks:      volumes:      │
│    frontend    ──▶  app-net   ──▶  db-data       │
│    backend          admin-net      uploads       │
│    postgres                                     │
│    redis                                        │
│    nginx                                        │
└─────────────────────────────────────────────────┘
            │
            ▼ docker compose up
┌─────────────────────────────────────────────────┐
│  Creates networks → Creates volumes →           │
│  Starts containers in dependency order          │
└─────────────────────────────────────────────────┘
```

### 📄 Complete docker-compose.yml Reference

```yaml
# docker-compose.yml — Full stack web application
version: '3.9'

# ── SERVICES ──────────────────────────────────────────────
services:

  # Nginx reverse proxy
  nginx:
    image: nginx:1.25-alpine
    container_name: nginx-proxy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/certs:/etc/nginx/certs:ro
      - static-files:/usr/share/nginx/html/static:ro
    depends_on:
      - frontend
      - backend
    networks:
      - frontend-net
    healthcheck:
      test: ["CMD", "nginx", "-t"]
      interval: 30s
      timeout: 5s
      retries: 3

  # Frontend (React/Next.js)
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
      target: runner
      args:
        - NODE_ENV=production
    container_name: app-frontend
    restart: unless-stopped
    environment:
      - NODE_ENV=production
      - NEXT_PUBLIC_API_URL=http://backend:3001
    env_file:
      - ./frontend/.env.production
    networks:
      - frontend-net
    depends_on:
      - backend
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:3000/api/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 10s

  # Backend (Node.js API)
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: app-backend
    restart: unless-stopped
    environment:
      - NODE_ENV=production
      - PORT=3001
      - DATABASE_URL=postgresql://${DB_USER}:${DB_PASSWORD}@postgres:5432/${DB_NAME}
      - REDIS_URL=redis://redis:6379
    env_file:
      - .env
    volumes:
      - uploads:/app/uploads
      - ./backend/config:/app/config:ro
    networks:
      - frontend-net
      - backend-net
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3001/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 15s
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M

  # PostgreSQL Database
  postgres:
    image: postgres:15.4-alpine
    container_name: app-postgres
    restart: unless-stopped
    environment:
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_DB=${DB_NAME}
      - PGDATA=/var/lib/postgresql/data/pgdata
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./postgres/init:/docker-entrypoint-initdb.d:ro
    networks:
      - backend-net
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER} -d ${DB_NAME}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
    deploy:
      resources:
        limits:
          memory: 1G

  # Redis Cache
  redis:
    image: redis:7-alpine
    container_name: app-redis
    restart: unless-stopped
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    volumes:
      - redisdata:/data
    networks:
      - backend-net
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

  # Database admin (dev only — remove in prod)
  adminer:
    image: adminer:latest
    container_name: db-adminer
    restart: unless-stopped
    ports:
      - "127.0.0.1:8080:8080"
    networks:
      - backend-net
    profiles:
      - debug   # Only starts with: docker compose --profile debug up

# ── NETWORKS ──────────────────────────────────────────────
networks:
  frontend-net:
    driver: bridge
    name: app_frontend
  backend-net:
    driver: bridge
    name: app_backend
    internal: true    # No external internet access

# ── VOLUMES ───────────────────────────────────────────────
volumes:
  pgdata:
    driver: local
    name: app_postgres_data
  redisdata:
    driver: local
    name: app_redis_data
  uploads:
    driver: local
    name: app_uploads
  static-files:
    driver: local
    name: app_static
```

### 🔧 Compose Commands

```bash
# ── LIFECYCLE ─────────────────────────────────────────────
docker compose up                        # start all services
docker compose up -d                     # detached
docker compose up -d --build             # rebuild and start
docker compose up -d backend             # start specific service
docker compose down                      # stop and remove containers
docker compose down -v                   # also remove volumes
docker compose restart backend           # restart a service
docker compose stop                      # stop without removing
docker compose start                     # start stopped services

# ── STATUS ────────────────────────────────────────────────
docker compose ps
docker compose ps --services
docker compose top

# ── LOGS ──────────────────────────────────────────────────
docker compose logs
docker compose logs -f                   # follow
docker compose logs -f backend           # specific service
docker compose logs --tail 100 postgres

# ── EXEC ──────────────────────────────────────────────────
docker compose exec backend bash
docker compose exec postgres psql -U admin

# ── BUILD ─────────────────────────────────────────────────
docker compose build
docker compose build --no-cache backend
docker compose pull                      # pull latest images

# ── SCALE ─────────────────────────────────────────────────
docker compose up -d --scale backend=3

# ── CONFIG ────────────────────────────────────────────────
docker compose config                    # validate and view merged config
```

### 📄 Override Files Pattern

```
docker-compose.yml          → base config (shared)
docker-compose.override.yml → local dev overrides (auto-loaded)
docker-compose.prod.yml     → production overrides
docker-compose.test.yml     → CI/test overrides
```

```yaml
# docker-compose.override.yml (development)
services:
  backend:
    build:
      context: ./backend
    volumes:
      - ./backend/src:/app/src   # hot reload
    environment:
      - NODE_ENV=development
      - LOG_LEVEL=debug
    ports:
      - "3001:3001"              # expose for debugging

  postgres:
    ports:
      - "5432:5432"              # expose locally for dev tools
```

```bash
# Use specific override for production
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# CI/Test environment
docker compose -f docker-compose.yml -f docker-compose.test.yml up --abort-on-container-exit
```

### 🔧 Common Mistakes
- Using `links` instead of networks and service names
- Not setting `depends_on` with health conditions
- Exposing database ports in production compose files
- Not using `.env` files for secrets
- Using `version: '2'` — upgrade to version 3

### ✅ Best Practices
- Always use healthchecks with `depends_on` conditions
- Keep production and development compose configs separate
- Use Compose profiles for optional services
- Set resource limits on every service
- Use internal networks to isolate tiers

### 🧑‍💻 Hands-On Tasks
1. Convert a 3-container setup from `docker run` to Compose
2. Add healthchecks and `depends_on` conditions
3. Implement dev override with hot reload
4. Scale the backend service to 3 instances

---

## Chapter 12: Image Optimization

### 📚 What You Will Learn
- Reducing image size dramatically
- Choosing the right base images
- Combining RUN instructions
- Using distroless images

### 🔍 Why This Exists

Smaller images mean faster pulls, faster deployments, smaller attack surface, and lower storage costs. A 2GB image that takes 3 minutes to pull is a CI/CD bottleneck. A 50MB image pulls in seconds.

### 🔧 Optimization Techniques

```dockerfile
# ❌ UNOPTIMIZED Dockerfile
FROM ubuntu:22.04
RUN apt-get update
RUN apt-get install -y curl wget git
RUN apt-get install -y nodejs npm
COPY . .
RUN npm install
RUN npm run build
# Size: ~1.4 GB

# ✅ OPTIMIZED Dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && \
    npm cache clean --force
COPY . .
RUN npm run build

FROM node:18-alpine AS runner
WORKDIR /app
RUN addgroup -S nodejs && adduser -S nextjs -G nodejs
COPY --from=builder --chown=nextjs:nodejs /app/dist ./dist
COPY --from=builder --chown=nextjs:nodejs /app/node_modules ./node_modules
USER nextjs
EXPOSE 3000
CMD ["node", "dist/server.js"]
# Size: ~85 MB
```

### 📊 Base Image Sizes

| Base Image | Size | Use Case |
|------------|------|----------|
| `ubuntu:22.04` | 77 MB | General purpose |
| `debian:slim` | 74 MB | When apt is needed |
| `alpine:3.18` | 5 MB | Minimal, musl libc |
| `distroless/base` | 2 MB | No shell, maximum security |
| `scratch` | 0 MB | Static binaries only |
| `node:18` | 991 MB | Node.js (avoid!) |
| `node:18-slim` | 243 MB | Better |
| `node:18-alpine` | 176 MB | Preferred |

### 🔧 Key Optimization Techniques

```dockerfile
# 1. Combine RUN commands
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        ca-certificates \
        libpq5 && \
    rm -rf /var/lib/apt/lists/*

# 2. Copy package files first (cache layer)
COPY package.json package-lock.json ./
RUN npm ci
COPY . .

# 3. Use .dockerignore
# node_modules, .git, dist, tests, docs

# 4. Use --no-install-recommends
RUN apt-get install -y --no-install-recommends nginx

# 5. Remove temp files in same layer
RUN wget -O /tmp/app.tar.gz https://example.com/app.tar.gz && \
    tar -xzf /tmp/app.tar.gz -C /app && \
    rm /tmp/app.tar.gz

# 6. Use distroless for security
FROM gcr.io/distroless/nodejs18-debian11
COPY --from=builder /app/dist /app/dist
CMD ["/app/dist/server.js"]
```

### 🧑‍💻 Hands-On Tasks
1. Take an existing Dockerfile and reduce its size by 50%+
2. Compare `ubuntu`, `debian:slim`, and `alpine` base sizes
3. Build a Go app with scratch as final image
4. Analyze layers with `docker history` and `dive` tool

---

## Chapter 13: Layer Caching & Build Performance

### 📚 What You Will Learn
- How Docker layer caching works
- Cache invalidation rules
- Ordering instructions for maximum cache hits
- BuildKit features

### ⚙️ How It Works Internally

```
Layer Cache Logic

Each layer has a cache key based on:
  ├─ Parent layer hash
  ├─ Instruction type
  ├─ Instruction arguments
  └─ File checksums (for COPY/ADD)

If key matches existing layer → CACHE HIT (instant!)
If key doesn't match → CACHE MISS → rebuild this + all subsequent layers
```

### 🔧 Cache Optimization Strategy

```dockerfile
# ❌ BAD ORDER — code changes invalidate npm install layer
FROM node:18-alpine
WORKDIR /app
COPY . .                    # ← ANY file change invalidates cache
RUN npm install             # ← always rebuilds!
RUN npm run build

# ✅ GOOD ORDER — stable dependencies first
FROM node:18-alpine
WORKDIR /app
COPY package.json package-lock.json ./   # ← only changes when deps change
RUN npm ci                               # ← cached unless package files change
COPY . .                                 # ← code changes here
RUN npm run build                        # ← only rebuilds app, not deps
```

### 🔧 BuildKit Advanced Caching

```dockerfile
# syntax=docker/dockerfile:1.5

FROM node:18-alpine

# Mount npm cache (persists between builds!)
RUN --mount=type=cache,target=/root/.npm \
    npm ci

# Mount apt cache
RUN --mount=type=cache,target=/var/cache/apt \
    apt-get update && apt-get install -y curl

# Secret mounting (build-time secrets, never in layers)
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc \
    npm ci
```

```bash
# Build with BuildKit
DOCKER_BUILDKIT=1 docker build -t myapp .

# Specify secret at build time
docker build \
  --secret id=npmrc,src=/home/user/.npmrc \
  -t myapp .

# Export and import cache
docker build --cache-from myapp:latest -t myapp:latest .
```

---

# LEVEL 4 — PRODUCTION

---

## Chapter 14: CI/CD with Docker

### 📚 What You Will Learn
- Docker in CI/CD pipelines
- GitHub Actions, GitLab CI, Jenkins
- Image tagging strategies
- Automated testing with Docker

### 🔍 Why This Exists

CI/CD with Docker ensures every commit is tested and shipped as an immutable container image. This eliminates "works on my machine" problems in delivery pipelines.

### 🏗️ CI/CD Architecture

```
Developer pushes code
        │
        ▼
    GitHub / GitLab
        │
        ▼ trigger
    CI Pipeline
        │
        ├─ 1. Build Docker image
        ├─ 2. Run unit tests (in container)
        ├─ 3. Run integration tests (docker compose)
        ├─ 4. Security scan (Trivy/Snyk)
        ├─ 5. Push to registry
        └─ 6. Deploy to staging/prod
```

### 📄 GitHub Actions Pipeline

```yaml
# .github/workflows/docker.yml
name: Docker Build & Deploy

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Run tests
        run: |
          docker compose -f docker-compose.test.yml up \
            --abort-on-container-exit \
            --exit-code-from tests

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=sha,prefix=,suffix=,format=short
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=raw,value=latest,enable=${{ github.ref == 'refs/heads/main' }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          platforms: linux/amd64,linux/arm64

      - name: Security scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
```

### 📄 GitLab CI Pipeline

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - scan
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA

test:
  stage: test
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker compose -f docker-compose.test.yml up --abort-on-container-exit
  coverage: '/Lines\s*:\s*(\d+\.\d+)%/'

build:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker buildx build
        --cache-from $CI_REGISTRY_IMAGE:latest
        --cache-to type=inline
        --tag $IMAGE_TAG
        --tag $CI_REGISTRY_IMAGE:latest
        --push .
  only:
    - main
    - tags

scan:
  stage: scan
  image: aquasec/trivy:latest
  script:
    - trivy image --exit-code 1 --severity CRITICAL $IMAGE_TAG
  allow_failure: false

deploy-staging:
  stage: deploy
  image: alpine:latest
  environment:
    name: staging
    url: https://staging.example.com
  script:
    - apk add --no-cache openssh-client
    - ssh deployer@staging.example.com
        "docker pull $IMAGE_TAG &&
         docker stop app || true &&
         docker run -d --name app --restart unless-stopped $IMAGE_TAG"
  only:
    - main
```

### 📊 Tagging Strategy

| Tag Pattern | Example | Use Case |
|-------------|---------|----------|
| `latest` | `myapp:latest` | Most recent on main |
| Semantic version | `myapp:v2.1.3` | Release versioning |
| Git SHA | `myapp:a3f9b2c` | Pinned to exact commit |
| Branch name | `myapp:develop` | Branch builds |
| Date | `myapp:20240315` | Date-based releases |

### 🔧 Common Mistakes
- Building production images with dev dependencies
- Not scanning images for vulnerabilities before push
- Using `latest` as the only tag — no version traceability
- Not caching layers in CI — slow builds every time
- Storing secrets in CI environment variables that appear in logs

### ✅ Best Practices
- Always tag with git SHA for traceability
- Scan every image before pushing to registry
- Use multi-platform builds for ARM support
- Cache build layers in CI (GitHub Actions cache, registry cache)
- Use branch-based environments for feature testing

---

## Chapter 15: Docker Security

### 📚 What You Will Learn
- Container security fundamentals
- Running as non-root
- Capabilities and seccomp profiles
- Image scanning and vulnerability management
- Network security policies

### 🔍 Why This Exists

A container is not a security boundary by default. Without proper security configuration, a compromised container can impact the host and other containers. Security in Docker is a layered approach.

### 🏗️ Security Architecture

```
Security Layers

┌─────────────────────────────────────────────────────┐
│ 1. Image Security (no vulnerabilities, minimal)     │
├─────────────────────────────────────────────────────┤
│ 2. Non-root User (UID > 0)                          │
├─────────────────────────────────────────────────────┤
│ 3. Read-only Filesystem                             │
├─────────────────────────────────────────────────────┤
│ 4. Dropped Capabilities                             │
├─────────────────────────────────────────────────────┤
│ 5. Seccomp Profile (syscall filtering)              │
├─────────────────────────────────────────────────────┤
│ 6. AppArmor / SELinux Profile                       │
├─────────────────────────────────────────────────────┤
│ 7. Network Isolation (internal networks)            │
├─────────────────────────────────────────────────────┤
│ 8. Secrets Management (no secrets in images)        │
└─────────────────────────────────────────────────────┘
```

### 🔧 Security Hardening Commands

```bash
# ── RUN AS NON-ROOT ──────────────────────────────────
docker run --user 1000:1000 myapp

# ── READ-ONLY FILESYSTEM ─────────────────────────────
docker run --read-only myapp
# Add writable tmpfs for temp files
docker run --read-only \
  --tmpfs /tmp:size=50m \
  --tmpfs /app/cache:size=100m \
  myapp

# ── DROP CAPABILITIES ────────────────────────────────
docker run \
  --cap-drop ALL \
  --cap-add NET_BIND_SERVICE \   # only if binding port < 1024
  myapp

# ── NO NEW PRIVILEGES ────────────────────────────────
docker run --security-opt no-new-privileges myapp

# ── SECCOMP PROFILE ──────────────────────────────────
docker run --security-opt seccomp=/path/to/seccomp.json myapp

# ── LIMIT RESOURCES (prevent DoS) ────────────────────
docker run \
  --memory="256m" \
  --memory-swap="256m" \      # disable swap
  --cpus="0.5" \
  --pids-limit=100 \          # prevent fork bombs
  myapp

# ── SCAN IMAGE FOR VULNERABILITIES ───────────────────
# Using Trivy (recommended)
trivy image myapp:latest
trivy image --severity HIGH,CRITICAL myapp:latest

# Using Docker Scout
docker scout cves myapp:latest
docker scout recommendations myapp:latest
```

### 📄 Secure Dockerfile

```dockerfile
FROM node:18-alpine

# Install only runtime deps, clean up
RUN apk add --no-cache dumb-init && \
    rm -rf /var/cache/apk/*

WORKDIR /app

# Non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S -u 1001 -G nodejs nextjs

# Copy with correct ownership
COPY --chown=nextjs:nodejs package*.json ./
RUN npm ci --only=production && npm cache clean --force
COPY --chown=nextjs:nodejs . .

# Use non-root user
USER nextjs

# Use dumb-init for signal handling
ENTRYPOINT ["dumb-init", "--"]

# Expose non-privileged port
EXPOSE 3000

CMD ["node", "server.js"]
```

### 📄 Security Scanning in CI

```yaml
# Trivy security scan step
- name: Vulnerability scan
  run: |
    # Install Trivy
    wget -qO- https://aquasecurity.github.io/trivy-repo/deb/public.key | \
      gpg --dearmor > /usr/share/keyrings/trivy.gpg
    apt-get install trivy

    # Scan with exit code (fail CI on CRITICAL)
    trivy image \
      --exit-code 1 \
      --severity CRITICAL \
      --ignore-unfixed \
      myapp:latest
```

### 🔧 Common Mistakes
- Running containers as root (default!)
- Mounting Docker socket into containers (`/var/run/docker.sock`)
- Exposing all ports (`-p 0.0.0.0:port:port`)
- Using `:latest` without verification
- Not scanning images for CVEs
- Storing secrets in environment variables (visible via `docker inspect`)

### ✅ Best Practices
- Run every container as non-root
- Use `--cap-drop ALL` and add back only needed capabilities
- Enable `--read-only` filesystem where possible
- Scan images in CI/CD pipeline
- Use Docker Secrets or Vault for sensitive data
- Regularly update base images to patch CVEs

---

## Chapter 16: Private Registry

### 📚 What You Will Learn
- Why private registries are needed
- Running Docker Registry locally
- AWS ECR, Azure ACR, Google GCR
- Image pull secrets in Kubernetes

### 🔍 Why This Exists

Docker Hub has pull rate limits, is public by default, and not suitable for proprietary images. Private registries give you control over image distribution, access, and retention.

### 🔧 Local Registry

```bash
# Run your own registry
docker run -d \
  --name registry \
  --restart unless-stopped \
  -p 5000:5000 \
  -v registry-data:/var/lib/registry \
  -e REGISTRY_STORAGE_DELETE_ENABLED=true \
  registry:2

# Push to local registry
docker tag myapp:latest localhost:5000/myapp:latest
docker push localhost:5000/myapp:latest

# Pull from local registry
docker pull localhost:5000/myapp:latest

# List images in registry
curl http://localhost:5000/v2/_catalog
curl http://localhost:5000/v2/myapp/tags/list
```

### 🔧 AWS ECR

```bash
# Authenticate with ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS \
  --password-stdin 123456789.dkr.ecr.us-east-1.amazonaws.com

# Create repository
aws ecr create-repository --repository-name myapp --region us-east-1

# Tag and push
docker tag myapp:latest \
  123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

docker push \
  123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

# Add lifecycle policy (auto-delete old images)
aws ecr put-lifecycle-policy \
  --repository-name myapp \
  --lifecycle-policy-text file://ecr-lifecycle.json
```

### 🔧 Secured Private Registry with Auth

```yaml
# docker-compose.registry.yml
version: '3.9'
services:
  registry:
    image: registry:2
    restart: unless-stopped
    ports:
      - "5000:5000"
    environment:
      - REGISTRY_AUTH=htpasswd
      - REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm
      - REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd
      - REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt
      - REGISTRY_HTTP_TLS_KEY=/certs/domain.key
    volumes:
      - ./auth:/auth
      - ./certs:/certs
      - registry-data:/var/lib/registry
volumes:
  registry-data:
```

```bash
# Create auth file
mkdir auth
docker run --entrypoint htpasswd httpd:2 \
  -Bbn admin strongpassword > auth/htpasswd
```

---

## Chapter 17: Scaling Containers

### 📚 What You Will Learn
- Horizontal scaling with Docker Compose
- Docker Swarm for orchestration
- Load balancing containers
- Health-based routing

### 🔧 Scaling with Docker Compose

```bash
# Scale a service
docker compose up -d --scale backend=5

# View instances
docker compose ps backend
```

### 📄 Docker Swarm

```bash
# Initialize Swarm
docker swarm init --advertise-addr <manager-ip>

# Add worker nodes (run on workers)
docker swarm join --token <token> <manager-ip>:2377

# Deploy stack
docker stack deploy -c docker-compose.prod.yml myapp

# Scale service in Swarm
docker service scale myapp_backend=5

# View services
docker service ls
docker service ps myapp_backend

# Rolling update
docker service update \
  --image myapp:v2.0 \
  --update-parallelism 1 \
  --update-delay 10s \
  myapp_backend
```

### 📄 Swarm Stack File

```yaml
# docker-compose.prod.yml (Swarm compatible)
version: '3.9'
services:
  backend:
    image: myregistry.com/myapp:v1.0
    deploy:
      replicas: 5
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
      rollback_config:
        parallelism: 0
        order: stop-first
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      resources:
        limits:
          cpus: '0.5'
          memory: 256M
        reservations:
          cpus: '0.25'
          memory: 128M
    networks:
      - app-net

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    configs:
      - source: nginx_conf
        target: /etc/nginx/nginx.conf
    deploy:
      placement:
        constraints:
          - node.role == manager
    networks:
      - app-net

networks:
  app-net:
    driver: overlay

configs:
  nginx_conf:
    file: ./nginx/nginx.conf
```

---

# LEVEL 5 — EXPERT

---

## Chapter 18: Docker Internals

### 📚 What You Will Learn
- How Docker Engine works
- containerd and runc
- OCI standards
- The container creation process step by step

### ⚙️ Docker Engine Architecture

```
Docker Architecture — Deep Dive

┌─────────────────────────────────────────────────────────────┐
│                     docker CLI                              │
└──────────────────────────┬──────────────────────────────────┘
                           │ REST API (unix socket / TCP)
┌──────────────────────────▼──────────────────────────────────┐
│                    dockerd (Docker Daemon)                   │
│   Image management, Networks, Volumes, API server           │
└──────────────────────────┬──────────────────────────────────┘
                           │ gRPC
┌──────────────────────────▼──────────────────────────────────┐
│                   containerd                                 │
│   Container lifecycle, Image distribution, Snapshots        │
└──────────────────────────┬──────────────────────────────────┘
                           │ OCI runtime spec
┌──────────────────────────▼──────────────────────────────────┐
│                   runc                                       │
│   Actually creates containers using Linux kernel features   │
│   Namespaces + cgroups + seccomp + capabilities             │
└─────────────────────────────────────────────────────────────┘
```

### 🔍 Container Creation Process

```
docker run nginx

1. CLI → POST /containers/create to dockerd
2. dockerd → containerd: create container
3. containerd → pull image if not present
4. containerd → create bundle (OCI runtime spec)
5. containerd → invoke runc
6. runc → set up namespaces:
   ├─ clone() with CLONE_NEWPID (new PID namespace)
   ├─ clone() with CLONE_NEWNET (new network namespace)
   ├─ clone() with CLONE_NEWNS  (new mount namespace)
   ├─ clone() with CLONE_NEWUTS (new hostname namespace)
   └─ clone() with CLONE_NEWIPC (new IPC namespace)
7. runc → set up cgroups (resource limits)
8. runc → pivot_root to container filesystem
9. runc → exec entrypoint process
10. Container is running
```

### 🔧 Direct containerd Interaction

```bash
# Install ctr (containerd CLI)
# Work with containerd directly (bypassing dockerd)

# Pull image via containerd
ctr images pull docker.io/library/nginx:latest

# List images
ctr images ls

# Run container with containerd
ctr run docker.io/library/nginx:latest nginx-test

# Alternatively use nerdctl (Docker-compatible CLI for containerd)
nerdctl run -d -p 80:80 nginx
```

---

## Chapter 19: Namespaces & cgroups

### 📚 What You Will Learn
- Linux namespaces and which ones Docker uses
- cgroups v1 and v2
- Resource limits internals
- Inspecting container isolation

### ⚙️ Linux Namespaces

```
Linux Namespace Types Used by Docker

┌──────────────┬──────────────────────────────────────────┐
│ Namespace    │ What It Isolates                         │
├──────────────┼──────────────────────────────────────────┤
│ PID          │ Process IDs (container has its own PID 1)│
│ NET          │ Network interfaces, routing, iptables    │
│ MNT          │ Filesystem mounts                        │
│ UTS          │ Hostname and domain name                 │
│ IPC          │ Inter-process communication              │
│ USER         │ User and group IDs                       │
│ CGROUP       │ cgroup visibility (v2)                   │
└──────────────┴──────────────────────────────────────────┘
```

### 🔧 Inspecting Namespaces

```bash
# Find container PID on host
docker inspect --format '{{.State.Pid}}' mycontainer

# View container's namespaces
ls -la /proc/<pid>/ns/

# Enter a container's namespace (low-level debugging)
sudo nsenter -t <pid> -n ip addr show

# View all processes on host (including container processes)
ps aux | grep <container_process>

# See container from host perspective
cat /proc/<pid>/cgroup
```

### ⚙️ cgroups (Control Groups)

```bash
# cgroups v2 location
ls /sys/fs/cgroup/

# Find container cgroup
cat /proc/<container_pid>/cgroup

# View memory limit
cat /sys/fs/cgroup/system.slice/docker-<container_id>.scope/memory.max

# View CPU quota
cat /sys/fs/cgroup/system.slice/docker-<container_id>.scope/cpu.max

# Set limits programmatically
docker run \
  --memory="512m" \           # cgroup memory.max
  --memory-swap="512m" \      # cgroup memory.memsw.max
  --cpu-quota=50000 \         # cgroup cpu.cfs_quota_us
  --cpu-period=100000 \       # cgroup cpu.cfs_period_us
  myapp
```

---

## Chapter 20: Storage Drivers

### 📚 What You Will Learn
- How Docker manages image layers on disk
- overlay2, devicemapper, btrfs, zfs
- Storage driver selection for production
- Performance implications

### ⚙️ How overlay2 Works

```
overlay2 — Most Common Storage Driver

Container Layer (Read/Write)
/var/lib/docker/overlay2/<id>/merged/
    │
    ├── upper/   (container writable layer)
    └── lower/   (read-only image layers, stacked)
        ├── layer 4: /app code
        ├── layer 3: npm packages
        ├── layer 2: node binary
        └── layer 1: alpine base

When container reads a file:
  └─ Check upper/ first → if not found → check lower/

When container writes a file:
  └─ Copy-on-Write: file copied from lower to upper, then modified
  └─ Original in lower/ is never changed
```

### 🔧 Storage Driver Commands

```bash
# Check current storage driver
docker info | grep -i storage

# Inspect overlay2 layers on disk
ls /var/lib/docker/overlay2/

# See container's layer structure
docker inspect <container> | python3 -c "
import sys, json
data = json.load(sys.stdin)
for m in data[0]['GraphDriver']['Data'].items():
    print(f'{m[0]}: {m[1]}')
"

# Check storage driver performance
docker system df
docker system df -v  # verbose
```

### 📊 Storage Driver Comparison

| Driver | Performance | Stability | Use Case |
|--------|-------------|-----------|----------|
| overlay2 | Excellent | Excellent | Default, recommended |
| devicemapper | Good | Good | RHEL/CentOS legacy |
| btrfs | Very Good | Good | When btrfs filesystem available |
| zfs | Excellent | Excellent | When ZFS available |
| vfs | Poor | Excellent | Testing only, no CoW |

---

## Chapter 21: Networking Internals

### 📚 What You Will Learn
- How Docker bridge networking works at the kernel level
- iptables rules created by Docker
- Container DNS resolution
- Network troubleshooting tools

### ⚙️ Bridge Network Internals

```
Linux Bridge Networking

Host Kernel
┌──────────────────────────────────────────────────────┐
│                                                      │
│  eth0 (192.168.1.10)     docker0 bridge (172.17.0.1) │
│     │                            │                   │
│     │         iptables NAT       │                   │
│     └────────────────────────────┘                   │
│                                    │                 │
│                         veth pairs │                 │
│                    ┌───────────────┴───────────────┐ │
│                    │                               │ │
│              veth abc123                     veth def456│
│                    │                               │ │
│              Container A (172.17.0.2)     Container B  │
└──────────────────────────────────────────────────────┘
```

### 🔧 Inspect Docker's iptables Rules

```bash
# View NAT rules (port publishing)
sudo iptables -t nat -L -n --line-numbers

# View FORWARD rules (container routing)
sudo iptables -L FORWARD -n

# View Docker-specific chains
sudo iptables -L DOCKER -n
sudo iptables -L DOCKER-USER -n

# View virtual network interfaces
ip link show | grep -E "docker|veth|br-"

# Inspect bridge
ip addr show docker0
brctl show docker0

# View routing table
ip route show
```

### 🔧 Container DNS Resolution

```bash
# Inside container — check DNS config
cat /etc/resolv.conf
# nameserver 127.0.0.11 (Docker's embedded DNS)

# Custom DNS for container
docker run --dns 8.8.8.8 myapp
docker run --dns-search company.local myapp

# Extra hosts
docker run --add-host redis.local:172.17.0.5 myapp
```

---

# LEVEL 6 — MASTER

---

## Chapter 22: Kubernetes Integration

### 📚 What You Will Learn
- How Docker images run in Kubernetes
- Building images for Kubernetes consumption
- Pod, Deployment, Service YAML
- Image pull secrets and private registries

### 🏗️ Docker → Kubernetes Flow

```
Developer Workflow

Write Code
    │
    ▼
Build Docker Image
docker build -t myapp:v1.0 .
    │
    ▼
Push to Registry
docker push registry.io/myapp:v1.0
    │
    ▼
Kubernetes Deployment
kubectl apply -f deployment.yaml
    │
    ▼ Kubernetes pulls image, creates Pods
┌────────────────────────────────────────────┐
│              Kubernetes Cluster            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐ │
│  │  Pod 1   │  │  Pod 2   │  │  Pod 3   │ │
│  │ myapp:v1 │  │ myapp:v1 │  │ myapp:v1 │ │
│  └──────────┘  └──────────┘  └──────────┘ │
└────────────────────────────────────────────┘
```

### 📄 Kubernetes Manifests

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: myapp
        version: v1.0
    spec:
      # Image pull secret for private registry
      imagePullSecrets:
        - name: registry-credentials

      # Security context
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 2000

      containers:
        - name: myapp
          image: myregistry.com/myapp:v1.0
          imagePullPolicy: Always
          ports:
            - containerPort: 3000
          env:
            - name: NODE_ENV
              value: "production"
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: app-secrets
                  key: db-password
          resources:
            requests:
              memory: "128Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 15
            periodSeconds: 20
          readinessProbe:
            httpGet:
              path: /ready
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 10
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL
```

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  namespace: production
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: ClusterIP
---
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  namespace: production
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
    - hosts:
        - app.example.com
      secretName: myapp-tls
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: myapp-service
                port:
                  number: 80
```

```bash
# Create image pull secret for private registry
kubectl create secret docker-registry registry-credentials \
  --docker-server=myregistry.com \
  --docker-username=myuser \
  --docker-password=mypassword \
  --docker-email=admin@example.com \
  -n production

# Deploy
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Watch rollout
kubectl rollout status deployment/myapp -n production

# Update image (rolling update)
kubectl set image deployment/myapp myapp=myregistry.com/myapp:v1.1 -n production

# Rollback
kubectl rollout undo deployment/myapp -n production
```

---

## Chapter 23: Microservices Architecture

### 📚 What You Will Learn
- Docker in microservices patterns
- Service mesh concepts
- Inter-service communication
- Distributed tracing and observability

### 🏗️ Microservices with Docker

```
Microservices Architecture

         ┌─────────────────────────────────────┐
         │            API Gateway              │
         │         (nginx/Kong/Traefik)        │
         └───────────────┬─────────────────────┘
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
   ┌─────────────┐ ┌──────────────┐ ┌──────────────┐
   │ User Service│ │Order Service │ │Product Service│
   │   (Node.js) │ │   (Python)   │ │    (Go)      │
   └──────┬──────┘ └──────┬───────┘ └──────┬───────┘
          │               │                │
   ┌──────▼──────┐ ┌──────▼───────┐ ┌──────▼───────┐
   │  Users DB   │ │  Orders DB   │ │ Products DB  │
   │ (PostgreSQL)│ │   (MySQL)    │ │  (MongoDB)   │
   └─────────────┘ └──────────────┘ └──────────────┘
          │               │                │
          └───────────────┼────────────────┘
                          │
                   ┌──────▼──────┐
                   │Message Queue│
                   │  (RabbitMQ) │
                   └─────────────┘
```

### 📄 Microservices Compose

```yaml
version: '3.9'

services:
  gateway:
    image: traefik:v3.0
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
    ports:
      - "80:80"
      - "8080:8080"  # Traefik dashboard
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      - gateway-net

  user-service:
    build: ./services/users
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.users.rule=PathPrefix(`/api/users`)"
    environment:
      - DATABASE_URL=postgresql://users_user:pass@users-db:5432/users
      - RABBITMQ_URL=amqp://rabbitmq:5672
    depends_on:
      users-db:
        condition: service_healthy
    networks:
      - gateway-net
      - users-net
      - messaging-net

  order-service:
    build: ./services/orders
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.orders.rule=PathPrefix(`/api/orders`)"
    environment:
      - DATABASE_URL=mysql://orders_user:pass@orders-db:3306/orders
      - USER_SERVICE_URL=http://user-service:3000
      - RABBITMQ_URL=amqp://rabbitmq:5672
    networks:
      - gateway-net
      - orders-net
      - messaging-net

  users-db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=users_user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=users
    volumes:
      - users-db-data:/var/lib/postgresql/data
    networks:
      - users-net
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U users_user"]
      interval: 10s

  orders-db:
    image: mysql:8.0
    environment:
      - MYSQL_USER=orders_user
      - MYSQL_PASSWORD=pass
      - MYSQL_DATABASE=orders
      - MYSQL_ROOT_PASSWORD=rootpass
    volumes:
      - orders-db-data:/var/lib/mysql
    networks:
      - orders-net

  rabbitmq:
    image: rabbitmq:3-management-alpine
    ports:
      - "15672:15672"  # Management UI
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq
    networks:
      - messaging-net

  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus-data:/prometheus
    networks:
      - monitoring-net

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana
    networks:
      - monitoring-net

networks:
  gateway-net:
  users-net:
    internal: true
  orders-net:
    internal: true
  messaging-net:
    internal: true
  monitoring-net:

volumes:
  users-db-data:
  orders-db-data:
  rabbitmq-data:
  prometheus-data:
  grafana-data:
```

---

## Chapter 24: High Availability Design

### 📚 What You Will Learn
- HA patterns for containerized workloads
- Active-active vs active-passive
- Health checks and auto-recovery
- Zero-downtime deployments

### 🏗️ HA Architecture

```
High Availability Architecture

Load Balancer (HAProxy/Nginx)
        │
        │  Round-robin / Least connections
        ├──────────────────────────────────┐
        │                                  │
        ▼                                  ▼
 App Server 1                      App Server 2
 (Docker container)                (Docker container)
        │                                  │
        └──────────────┬───────────────────┘
                       │
              ┌────────▼────────┐
              │   Database      │
              │  Primary Node   │
              └────────┬────────┘
                       │ replication
              ┌────────▼────────┐
              │ Database        │
              │ Replica (RO)    │
              └─────────────────┘
```

### 📄 Zero-Downtime Deployment Script

```bash
#!/bin/bash
# zero-downtime-deploy.sh

IMAGE="myregistry.com/myapp:${VERSION}"
SERVICE="app_backend"
HEALTHCHECK_URL="http://localhost:${PORT}/health"

echo "Starting zero-downtime deployment of ${IMAGE}"

# Pull new image
docker pull $IMAGE

# Scale up new version
docker service update \
  --image $IMAGE \
  --update-parallelism 1 \
  --update-delay 30s \
  --update-failure-action rollback \
  --update-monitor 60s \
  --update-order start-first \
  $SERVICE

# Wait for rollout
docker service update --wait $SERVICE

# Verify health
if curl -f $HEALTHCHECK_URL; then
  echo "✅ Deployment successful!"
else
  echo "❌ Health check failed, rolling back..."
  docker service rollback $SERVICE
  exit 1
fi
```

---

## Chapter 25: Production Design Patterns

### 📚 What You Will Learn
- The Sidecar pattern
- Ambassador pattern
- Init containers
- Config injection patterns

### 🏗️ Sidecar Pattern

```yaml
# Logging sidecar
version: '3.9'
services:
  app:
    image: myapp:latest
    volumes:
      - logs:/app/logs

  log-shipper:
    image: fluent/fluent-bit:latest
    volumes:
      - logs:/var/log/app:ro
      - ./fluent-bit.conf:/fluent-bit/etc/fluent-bit.conf:ro
    depends_on:
      - app

volumes:
  logs:
```

### 🏗️ Init Container Pattern

```yaml
services:
  db-migrate:
    image: myapp:latest
    command: ["npm", "run", "db:migrate"]
    environment:
      - DATABASE_URL=${DATABASE_URL}
    depends_on:
      postgres:
        condition: service_healthy

  app:
    image: myapp:latest
    depends_on:
      db-migrate:
        condition: service_completed_successfully
```

---

# REAL WORLD PROJECTS

---

## Project 1: Node.js App in Docker

### 📁 Project Structure

```
my-node-app/
├── src/
│   ├── app.js
│   ├── routes/
│   └── middleware/
├── package.json
├── package-lock.json
├── Dockerfile
├── .dockerignore
└── docker-compose.yml
```

### 📄 Application Files

```javascript
// src/app.js
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.use(express.json());

app.get('/health', (req, res) => {
  res.json({ status: 'healthy', timestamp: new Date().toISOString() });
});

app.get('/', (req, res) => {
  res.json({ message: 'Hello from Docker!', env: process.env.NODE_ENV });
});

app.listen(PORT, '0.0.0.0', () => {
  console.log(`Server running on port ${PORT}`);
});

module.exports = app;
```

```json
// package.json
{
  "name": "my-docker-app",
  "version": "1.0.0",
  "scripts": {
    "start": "node src/app.js",
    "dev": "nodemon src/app.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "nodemon": "^3.0.1",
    "jest": "^29.0.0"
  }
}
```

```dockerfile
# Dockerfile
FROM node:18-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

FROM node:18-alpine AS runner
RUN addgroup -S nodejs && adduser -S nodeapp -G nodejs
WORKDIR /app
COPY --from=deps --chown=nodeapp:nodejs /app/node_modules ./node_modules
COPY --chown=nodeapp:nodejs src ./src
COPY --chown=nodeapp:nodejs package.json ./
USER nodeapp
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD wget -qO- http://localhost:3000/health || exit 1
CMD ["node", "src/app.js"]
```

```
# .dockerignore
node_modules
.git
.gitignore
README.md
*.test.js
coverage
.env
Dockerfile*
docker-compose*
```

```bash
# Build and run
docker build -t my-node-app:1.0 .
docker run -d \
  --name my-node-app \
  -p 3000:3000 \
  -e NODE_ENV=production \
  --restart unless-stopped \
  my-node-app:1.0

# Test
curl http://localhost:3000
curl http://localhost:3000/health
```

---

## Project 2: Full Stack App with Docker Compose

### 📁 Project Structure

```
fullstack-app/
├── frontend/               # React app
│   ├── src/
│   ├── package.json
│   └── Dockerfile
├── backend/                # Node.js API
│   ├── src/
│   ├── package.json
│   └── Dockerfile
├── nginx/
│   └── nginx.conf
├── postgres/
│   └── init.sql
├── docker-compose.yml
├── docker-compose.override.yml
└── .env
```

### 📄 Core Compose File

```yaml
# docker-compose.yml
version: '3.9'

services:
  nginx:
    image: nginx:1.25-alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - frontend
      - backend
    networks:
      - app-net

  frontend:
    build:
      context: ./frontend
      target: runner
    environment:
      - REACT_APP_API_URL=/api
    networks:
      - app-net
    depends_on:
      - backend

  backend:
    build: ./backend
    environment:
      - NODE_ENV=production
      - PORT=3001
      - DATABASE_URL=postgresql://${DB_USER}:${DB_PASSWORD}@postgres:5432/${DB_NAME}
    env_file: .env
    networks:
      - app-net
      - db-net
    depends_on:
      postgres:
        condition: service_healthy

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./postgres/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    networks:
      - db-net
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

networks:
  app-net:
  db-net:
    internal: true

volumes:
  pgdata:
```

```nginx
# nginx/nginx.conf
upstream frontend {
    server frontend:3000;
}
upstream backend {
    server backend:3001;
}

server {
    listen 80;

    location / {
        proxy_pass http://frontend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /api {
        rewrite ^/api/(.*) /$1 break;
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /health {
        return 200 'OK';
        add_header Content-Type text/plain;
    }
}
```

```bash
# Start the full stack
cp .env.example .env
# Edit .env with your values

docker compose up -d

# Watch logs
docker compose logs -f

# Check all services
docker compose ps

# Teardown
docker compose down -v
```

---

## Project 3: Microservices Architecture

### 📁 Structure

```
microservices/
├── services/
│   ├── users/          # User service (Node.js)
│   ├── products/       # Product service (Python)
│   └── orders/         # Order service (Go)
├── gateway/            # API Gateway (nginx)
├── monitoring/         # Prometheus + Grafana
├── docker-compose.yml
└── .env
```

*See Chapter 23 for full Compose configuration*

```bash
# Deploy all microservices
docker compose up -d

# Scale product service
docker compose up -d --scale products=3

# Health check all services
docker compose ps
for svc in users products orders; do
  echo "=== $svc ==="
  docker compose exec $svc wget -qO- http://localhost:3000/health
done
```

---

## Project 4: Production Deployment Pipeline

### 📄 Complete CI/CD + Production Setup

```bash
#!/bin/bash
# deploy.sh — Production deployment script

set -euo pipefail

VERSION=${1:-latest}
REGISTRY=${REGISTRY:-myregistry.com}
APP_NAME="myapp"
IMAGE="${REGISTRY}/${APP_NAME}:${VERSION}"
COMPOSE_FILE="docker-compose.prod.yml"

log() { echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*"; }

# 1. Pull new image
log "Pulling image: ${IMAGE}"
docker pull "${IMAGE}"

# 2. Backup database
log "Creating database backup..."
docker compose -f $COMPOSE_FILE exec -T postgres \
  pg_dump -U $DB_USER $DB_NAME > "backup-$(date +%Y%m%d-%H%M%S).sql"

# 3. Update docker-compose to use new image
export APP_IMAGE="${IMAGE}"

# 4. Rolling update (zero-downtime)
log "Deploying ${IMAGE}..."
docker compose -f $COMPOSE_FILE up -d --no-deps --build app

# 5. Health check
log "Waiting for health check..."
for i in {1..30}; do
  if curl -sf http://localhost/health > /dev/null 2>&1; then
    log "✅ Deployment successful!"
    exit 0
  fi
  sleep 2
done

# 6. Rollback on failure
log "❌ Health check failed! Rolling back..."
docker compose -f $COMPOSE_FILE up -d --no-deps app
log "Rollback complete."
exit 1
```

---

# DEBUGGING SECTION

---

## 🐞 Complete Debugging Guide

### docker logs

```bash
# Basic log viewing
docker logs <container>

# Follow logs in real-time
docker logs -f <container>

# Timestamps
docker logs -t <container>

# Last N lines
docker logs --tail 100 <container>

# Logs since timestamp or duration
docker logs --since 2024-01-15T00:00:00 <container>
docker logs --since 30m <container>

# Combine options
docker logs -f --tail 50 --since 1h <container>

# Docker Compose logs
docker compose logs -f --tail 50 backend

# Save logs to file
docker logs <container> > app.log 2>&1
```

### docker exec

```bash
# Open interactive shell
docker exec -it <container> bash     # or sh for alpine
docker exec -it <container> /bin/sh

# Run single command
docker exec <container> ps aux
docker exec <container> cat /etc/hosts
docker exec <container> env | sort

# Run as different user
docker exec -u root <container> bash

# Check process list
docker exec <container> ps aux

# Check network
docker exec <container> netstat -tlnp
docker exec <container> ss -tlnp
docker exec <container> ping google.com

# Check disk
docker exec <container> df -h
docker exec <container> du -sh /app/*

# Check memory
docker exec <container> cat /proc/meminfo
docker exec <container> free -h
```

### docker inspect

```bash
# Full container details
docker inspect <container>

# Specific fields with Go template
docker inspect <container> -f '{{.State.Status}}'
docker inspect <container> -f '{{.State.ExitCode}}'
docker inspect <container> -f '{{.NetworkSettings.IPAddress}}'
docker inspect <container> -f '{{.HostConfig.Memory}}'

# All environment variables
docker inspect <container> -f '{{range .Config.Env}}{{.}}{{"\n"}}{{end}}'

# Mount points
docker inspect <container> -f '{{json .Mounts}}' | python3 -m json.tool

# Port bindings
docker inspect <container> -f '{{json .NetworkSettings.Ports}}' | python3 -m json.tool

# Inspect image layers
docker inspect <image>
docker history <image>
```

### Advanced Debugging

```bash
# Real-time resource monitoring
docker stats                          # all containers
docker stats <container>              # specific container
docker stats --no-stream --format \
  "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# Check container events
docker events --filter container=<name>
docker events --since 1h

# Strace processes inside container (debug system calls)
docker exec <container> strace -p 1

# Network debugging from inside container
docker exec -it <container> sh -c "
  apk add --no-cache curl nmap netcat-openbsd && \
  nc -zv postgres 5432 && \
  curl -v http://redis:6379
"

# Copy files for inspection
docker cp <container>:/app/error.log ./container-error.log

# Check container's filesystem diff
docker diff <container>
# A = added, C = changed, D = deleted

# Export container filesystem
docker export <container> > container-fs.tar
tar -tv < container-fs.tar | head -50

# View detailed events
docker inspect <container> | jq '.[0].State'
```

### Common Debug Scenarios

```bash
# Container exits immediately — view logs
docker logs <container>
# Check exit code
docker inspect <container> -f '{{.State.ExitCode}}'
# Run with override to debug
docker run -it --entrypoint sh myapp

# Port not accessible — check bindings
docker port <container>
# Check iptables
sudo iptables -t nat -L -n | grep <port>

# Out of disk space
docker system df
docker system prune -a
docker volume prune

# Container can't reach database
docker exec <container> ping <db_container_name>
docker exec <container> nc -zv <db_host> 5432

# Performance issues
docker stats <container>
docker exec <container> top
# Check for OOM kills
docker inspect <container> -f '{{.State.OOMKilled}}'
```

---

# PRODUCTION FOLDER STRUCTURE

---

```
/project-root/
├── .github/
│   └── workflows/
│       ├── ci.yml              # CI pipeline
│       └── deploy.yml          # CD pipeline
│
├── docker/
│   ├── app/
│   │   ├── Dockerfile          # Production Dockerfile
│   │   ├── Dockerfile.dev      # Development Dockerfile
│   │   └── entrypoint.sh       # Container entrypoint script
│   │
│   ├── nginx/
│   │   ├── nginx.conf          # Main nginx config
│   │   ├── conf.d/
│   │   │   ├── app.conf        # App server block
│   │   │   └── ssl.conf        # SSL configuration
│   │   └── certs/              # TLS certificates
│   │
│   ├── postgres/
│   │   ├── init/
│   │   │   └── 01-schema.sql   # Database init scripts
│   │   └── postgresql.conf     # Postgres config
│   │
│   └── monitoring/
│       ├── prometheus.yml
│       ├── grafana/
│       │   └── dashboards/
│       └── alertmanager.yml
│
├── compose/
│   ├── docker-compose.yml          # Base configuration
│   ├── docker-compose.override.yml # Dev overrides (auto-loaded)
│   ├── docker-compose.prod.yml     # Production config
│   ├── docker-compose.test.yml     # CI/Test config
│   └── docker-compose.staging.yml  # Staging config
│
├── scripts/
│   ├── deploy.sh               # Deployment script
│   ├── rollback.sh             # Rollback script
│   ├── backup.sh               # Database backup
│   └── health-check.sh         # Post-deploy health check
│
├── app/                        # Application source code
│   ├── src/
│   ├── tests/
│   ├── package.json
│   └── ...
│
├── .dockerignore
├── .env.example                # Template (commit this)
├── .env                        # Actual values (never commit!)
├── .env.production             # Production env template
└── README.md
```

```
# Directory permissions in production
/docker/nginx/certs/    → 600 (root only)
/docker/postgres/       → 700
.env                    → 600 (owner read only)
scripts/*.sh            → 750 (owner rwx, group rx)
```

---

# COMMAND REFERENCE CHEAT SHEET

---

## 📋 Quick Reference

### Container Lifecycle
```bash
docker run -d --name <name> -p <h>:<c> <image>   # Run container
docker ps [-a]                                     # List containers
docker stop/start/restart <name>                   # Control lifecycle
docker rm [-f] <name>                              # Remove container
docker rm $(docker ps -aq)                         # Remove all stopped
```

### Images
```bash
docker build -t <name>:<tag> .                    # Build image
docker pull/push <image>:<tag>                    # Pull/push
docker images                                      # List images
docker rmi <image>                                 # Remove image
docker tag <src> <dst>                            # Tag image
docker image prune -a                             # Clean unused
```

### Volumes
```bash
docker volume create <name>                       # Create volume
docker volume ls                                  # List volumes
docker volume inspect <name>                      # Inspect volume
docker volume rm <name>                           # Remove volume
docker volume prune                               # Remove unused
```

### Networks
```bash
docker network create <name>                      # Create network
docker network ls                                 # List networks
docker network inspect <name>                     # Inspect network
docker network connect <net> <container>          # Connect
docker network rm <name>                          # Remove network
```

### Docker Compose
```bash
docker compose up -d [--build]                   # Start stack
docker compose down [-v]                          # Stop & remove
docker compose ps                                 # Status
docker compose logs -f [service]                  # Logs
docker compose exec <svc> bash                   # Shell access
docker compose build [--no-cache]                # Build
docker compose pull                              # Pull images
```

### System
```bash
docker system df                                  # Disk usage
docker system prune [-a]                          # Cleanup
docker info                                       # System info
docker events                                     # Event stream
docker stats [--no-stream]                        # Resource usage
```

### Debugging
```bash
docker logs [-f] [--tail N] <name>               # Container logs
docker exec -it <name> bash                      # Shell into container
docker inspect <name>                             # Full details
docker diff <name>                               # Filesystem changes
docker top <name>                                # Running processes
docker port <name>                               # Port mappings
```

---

## 🎓 Final Words from a Senior Engineer

After 15+ years building production systems, here's what truly matters:

**1. Immutability First** — Containers should be cattle, not pets. Never modify running containers. Rebuild and redeploy.

**2. Security is Not Optional** — Non-root users, minimal images, secret management, and vulnerability scanning are table stakes in production.

**3. Observability or It Doesn't Exist** — Every container must have health checks, structured logging, and metrics. If you can't observe it, you can't fix it.

**4. Layer Caching is Money** — Well-ordered Dockerfiles mean CI pipelines that take 1 minute instead of 10. This compounds over hundreds of PRs.

**5. Everything as Code** — Dockerfile, docker-compose.yml, CI pipelines. Never click-ops. Version controlled, reviewed, tested.

**6. The Goal is Kubernetes** — Docker is the building block. The destination for serious production workloads is Kubernetes. Learn Docker deeply, then migrate.

```
User → Application → Docker Container → Docker Engine
     → Registry → Kubernetes → Cloud Infrastructure
                → Observability → Business Value
```

*Build, ship, run — repeatably, securely, at scale.*

---

> **📘 Docker Mastery Guide** | Senior DevOps Engineering Edition
> Version 2.0 | Production-Tested | Industry-Level
> 
> *"The best infrastructure is the one developers don't think about."*

---
```

