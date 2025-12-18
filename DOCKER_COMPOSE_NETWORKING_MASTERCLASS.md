# 🌐 Docker Compose Networking Masterclass
## From Beginner to Advanced

> **Complete guide to understanding Docker Compose networking with real-world examples**

---

## 📚 Table of Contents

1. [Networking Basics](#1-networking-basics)
2. [Default Network Behavior](#2-default-network-behavior)
3. [Custom Networks](#3-custom-networks)
4. [Network Drivers](#4-network-drivers)
5. [Multi-Network Architecture](#5-multi-network-architecture)
6. [Advanced Configurations](#6-advanced-configurations)
7. [Real-World Examples](#7-real-world-examples)
8. [Best Practices](#8-best-practices)
9. [Troubleshooting](#9-troubleshooting)

---

## 1. Networking Basics

### What is Docker Networking?

Docker networking allows containers to communicate with:
- **Each other** (container-to-container)
- **The host machine** (container-to-host)
- **The outside world** (container-to-internet)

### Key Concepts

```
┌─────────────────────────────────────────────────────┐
│  Host Machine (Your Computer)                       │
│                                                      │
│  ┌──────────────────────────────────────────────┐  │
│  │  Docker Network: my-network                  │  │
│  │                                               │  │
│  │  ┌──────────┐      ┌──────────┐             │  │
│  │  │Container1│◄────►│Container2│             │  │
│  │  │  (web)   │      │  (api)   │             │  │
│  │  └──────────┘      └──────────┘             │  │
│  │       ▲                  ▲                   │  │
│  └───────┼──────────────────┼───────────────────┘  │
│          │                  │                       │
│          └──────────────────┘                       │
│                   │                                 │
│              Port Mapping                           │
│         (localhost:3000:80)                         │
└─────────────────────────────────────────────────────┘
```

### Network Terminology

| Term | Meaning |
|------|---------|
| **Bridge Network** | Default network type, containers on same host |
| **Host Network** | Container uses host's network directly |
| **Overlay Network** | Multi-host networking (Docker Swarm) |
| **DNS Resolution** | Containers find each other by name |
| **Port Mapping** | Expose container ports to host |
| **Network Isolation** | Separate networks can't communicate |

---

## 2. Default Network Behavior

### Level 1: No Network Definition (Beginner)

**Docker Compose automatically creates a default network!**

```yaml
# docker-compose.yml
version: '3.8'

services:
  frontend:
    image: nginx:alpine
    ports:
      - "3000:80"
  
  backend:
    image: node:22-alpine
    ports:
      - "3001:3000"
```

**What Happens:**
```bash
# Docker creates:
Network Name: myproject_default
Type: bridge
Containers: frontend, backend (both auto-join)
```

**How Containers Communicate:**
```javascript
// Inside frontend container
fetch('http://backend:3000/api/users')
// ✅ Works! Docker DNS resolves 'backend' to backend container
```

### Default Network Features

✅ **Automatic DNS Resolution**
- Use service names as hostnames
- Example: `http://backend:3000`

✅ **Automatic Container Discovery**
- All services in same compose file join the network

✅ **Isolation from Other Projects**
- Each docker-compose project gets its own network

### Example: Simple 2-Tier App

```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    # Can call: http://api:3000
  
  api:
    image: node:22-alpine
    ports:
      - "3000:3000"
    # Can call: http://web:80
```

**Network Diagram:**
```
┌─────────────────────────────────────┐
│  Network: myapp_default             │
│                                     │
│  ┌─────┐          ┌─────┐          │
│  │ web │◄────────►│ api │          │
│  └─────┘          └─────┘          │
│     │                │              │
└─────┼────────────────┼──────────────┘
      │                │
   Port 80          Port 3000
      │                │
   localhost:80    localhost:3000
```

---

## 3. Custom Networks

### Level 2: Single Custom Network (Intermediate)

**Why use custom networks?**
- Better organization
- Explicit configuration
- Custom IP ranges
- Clearer documentation

```yaml
version: '3.8'

services:
  frontend:
    image: nginx:alpine
    networks:
      - app-network
  
  backend:
    image: node:22-alpine
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

### Network Configuration Options

```yaml
networks:
  app-network:
    driver: bridge
    driver_opts:
      com.docker.network.bridge.name: my_custom_bridge
    ipam:
      driver: default
      config:
        - subnet: 172.28.0.0/16
          gateway: 172.28.0.1
    labels:
      environment: production
      team: backend
```

**Explanation:**
- `driver: bridge` - Network type
- `subnet` - IP address range
- `gateway` - Network gateway IP
- `labels` - Metadata for organization

---

## 4. Network Drivers

### Bridge Network (Default)

**Best for:** Single-host applications

```yaml
networks:
  my-bridge:
    driver: bridge
```

**Features:**
- ✅ Container-to-container communication
- ✅ DNS resolution by service name
- ✅ Isolated from host network
- ✅ Port mapping to host

**Use Case:**
```yaml
services:
  app:
    image: myapp:latest
    networks:
      - my-bridge
  
  database:
    image: postgres:15
    networks:
      - my-bridge
    # app can connect to: postgresql://database:5432
```

---

### Host Network

**Best for:** Maximum performance, no isolation needed

```yaml
services:
  app:
    image: myapp:latest
    network_mode: host
```

**Features:**
- ✅ No network isolation
- ✅ Container uses host's IP
- ✅ No port mapping needed
- ❌ Less secure
- ❌ Port conflicts possible

**Example:**
```yaml
services:
  monitoring:
    image: prometheus:latest
    network_mode: host
    # Directly accessible at host's IP
```

---

### None Network

**Best for:** Completely isolated containers

```yaml
services:
  secure-app:
    image: myapp:latest
    network_mode: none
```

**Features:**
- ✅ Complete network isolation
- ❌ No network access at all
- ❌ Can't communicate with anything

---

### Overlay Network (Docker Swarm)

**Best for:** Multi-host deployments

```yaml
networks:
  overlay-net:
    driver: overlay
    attachable: true
```

**Features:**
- ✅ Spans multiple Docker hosts
- ✅ Encrypted by default
- ✅ Service discovery across hosts
- ⚠️ Requires Docker Swarm mode

---

## 5. Multi-Network Architecture

### Level 3: Multiple Networks (Advanced)

**Security through network isolation!**

```yaml
version: '3.8'

services:
  # Public-facing service
  nginx:
    image: nginx:alpine
    networks:
      - frontend-network
    ports:
      - "80:80"
  
  # Application server
  api:
    image: node:22-alpine
    networks:
      - frontend-network  # Can talk to nginx
      - backend-network   # Can talk to database
  
  # Database (isolated)
  postgres:
    image: postgres:15
    networks:
      - backend-network  # ONLY api can access
  
  # Cache (isolated)
  redis:
    image: redis:alpine
    networks:
      - backend-network  # ONLY api can access

networks:
  frontend-network:
    driver: bridge
  backend-network:
    driver: bridge
```

**Network Diagram:**
```
┌─────────────────────────────────────────────────────┐
│  frontend-network                                   │
│  ┌───────┐         ┌─────┐                         │
│  │ nginx │◄───────►│ api │                         │
│  └───────┘         └─────┘                         │
│      ▲                │                             │
└──────┼────────────────┼─────────────────────────────┘
       │                │
    Port 80             │
       │                │
┌──────┼────────────────┼─────────────────────────────┐
│      │                │  backend-network            │
│      ✗                ▼                             │
│  (blocked)      ┌─────┐                             │
│                 │ api │                             │
│                 └─────┘                             │
│                    │  │                             │
│              ┌─────┘  └─────┐                       │
│              ▼              ▼                       │
│         ┌──────────┐   ┌───────┐                   │
│         │ postgres │   │ redis │                   │
│         └──────────┘   └───────┘                   │
└─────────────────────────────────────────────────────┘
```

**Security Benefits:**
- ✅ nginx **CANNOT** access database directly
- ✅ nginx **CANNOT** access redis directly
- ✅ Only api can access backend services
- ✅ Principle of least privilege

---

### Real-World E-Commerce Example

```yaml
version: '3.8'

services:
  # Load Balancer (Public)
  nginx:
    image: nginx:alpine
    networks:
      - public
    ports:
      - "80:80"
      - "443:443"
  
  # Frontend App
  frontend:
    build: ./frontend
    networks:
      - public
      - app-network
  
  # Backend API
  backend:
    build: ./backend
    networks:
      - app-network
      - database-network
      - cache-network
  
  # Database (Private)
  postgres:
    image: postgres:15
    networks:
      - database-network
    environment:
      POSTGRES_PASSWORD: secret
  
  # Cache (Private)
  redis:
    image: redis:alpine
    networks:
      - cache-network
  
  # Background Jobs (Private)
  worker:
    build: ./worker
    networks:
      - database-network
      - cache-network

networks:
  public:
    driver: bridge
  app-network:
    driver: bridge
  database-network:
    driver: bridge
    internal: true  # No external access!
  cache-network:
    driver: bridge
    internal: true  # No external access!
```

**Access Matrix:**

| Service | public | app-network | database-network | cache-network |
|---------|--------|-------------|------------------|---------------|
| nginx | ✅ | ❌ | ❌ | ❌ |
| frontend | ✅ | ✅ | ❌ | ❌ |
| backend | ❌ | ✅ | ✅ | ✅ |
| postgres | ❌ | ❌ | ✅ | ❌ |
| redis | ❌ | ❌ | ❌ | ✅ |
| worker | ❌ | ❌ | ✅ | ✅ |

---

## 6. Advanced Configurations

### Network Aliases

**Give containers multiple names on a network:**

```yaml
services:
  api:
    image: node:22-alpine
    networks:
      app-network:
        aliases:
          - api-server
          - backend-api
          - my-api

networks:
  app-network:
```

**Now you can access the API using:**
- `http://api:3000`
- `http://api-server:3000`
- `http://backend-api:3000`
- `http://my-api:3000`

---

### Static IP Addresses

```yaml
services:
  database:
    image: postgres:15
    networks:
      backend-network:
        ipv4_address: 172.28.0.10

networks:
  backend-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.28.0.0/16
```

**When to use:**
- Legacy applications expecting specific IPs
- Firewall rules based on IP
- **Generally avoid** - use DNS names instead!

---

### Priority Networks

```yaml
services:
  app:
    image: myapp:latest
    networks:
      - network1
      - network2
    # First network (network1) becomes default route
```

---

### External Networks

**Connect to networks created outside docker-compose:**

```yaml
# docker-compose.yml (Project A)
services:
  app-a:
    image: app-a:latest
    networks:
      - shared-network

networks:
  shared-network:
    external: true
    name: company-network
```

```yaml
# docker-compose.yml (Project B)
services:
  app-b:
    image: app-b:latest
    networks:
      - shared-network

networks:
  shared-network:
    external: true
    name: company-network
```

**Both projects share the same network!**

---

### Network-wide DNS Configuration

```yaml
networks:
  app-network:
    driver: bridge
    driver_opts:
      com.docker.network.bridge.name: custom_bridge
      com.docker.network.bridge.enable_ip_masquerade: "true"
      com.docker.network.bridge.enable_icc: "true"
      com.docker.network.driver.mtu: "1500"
```

---

## 7. Real-World Examples

### Example 1: Microservices Architecture

```yaml
version: '3.8'

services:
  # API Gateway
  gateway:
    image: kong:latest
    networks:
      - public
      - services
    ports:
      - "8000:8000"
  
  # User Service
  user-service:
    build: ./services/user
    networks:
      - services
      - user-db-network
  
  user-db:
    image: postgres:15
    networks:
      - user-db-network
  
  # Order Service
  order-service:
    build: ./services/order
    networks:
      - services
      - order-db-network
  
  order-db:
    image: postgres:15
    networks:
      - order-db-network
  
  # Product Service
  product-service:
    build: ./services/product
    networks:
      - services
      - product-db-network
  
  product-db:
    image: mongo:latest
    networks:
      - product-db-network
  
  # Shared Cache
  redis:
    image: redis:alpine
    networks:
      - services

networks:
  public:
    driver: bridge
  services:
    driver: bridge
  user-db-network:
    driver: bridge
    internal: true
  order-db-network:
    driver: bridge
    internal: true
  product-db-network:
    driver: bridge
    internal: true
```

**Architecture:**
```
Internet
   │
   ▼
┌──────────┐
│ Gateway  │ (public + services)
└──────────┘
   │
   ▼
┌─────────────────────────────────────┐
│  services network                   │
│  ┌──────┐  ┌──────┐  ┌──────┐      │
│  │User  │  │Order │  │Product│     │
│  │Svc   │  │Svc   │  │Svc   │      │
│  └──────┘  └──────┘  └──────┘      │
│     │         │         │           │
└─────┼─────────┼─────────┼───────────┘
      │         │         │
      ▼         ▼         ▼
  ┌────────┐┌────────┐┌────────┐
  │User DB ││Order DB││Product │
  │(PG)    ││(PG)    ││DB(Mongo│
  └────────┘└────────┘└────────┘
  (isolated) (isolated)(isolated)
```

---

### Example 2: Development vs Production Networks

**Development:**
```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  app:
    build: .
    networks:
      - dev-network
    ports:
      - "3000:3000"  # Expose for debugging
  
  db:
    image: postgres:15
    networks:
      - dev-network
    ports:
      - "5432:5432"  # Expose for DB tools

networks:
  dev-network:
    driver: bridge
```

**Production:**
```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  app:
    build: .
    networks:
      - frontend
      - backend
    # No ports exposed!
  
  db:
    image: postgres:15
    networks:
      - backend
    # No ports exposed!
  
  nginx:
    image: nginx:alpine
    networks:
      - frontend
    ports:
      - "80:80"
      - "443:443"

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # No internet access
```

---

### Example 3: Multi-Environment Setup

```yaml
version: '3.8'

services:
  app:
    image: myapp:latest
    networks:
      - ${ENVIRONMENT:-development}

networks:
  development:
    driver: bridge
  staging:
    driver: bridge
  production:
    driver: bridge
```

**Usage:**
```bash
# Development
ENVIRONMENT=development docker-compose up

# Staging
ENVIRONMENT=staging docker-compose up

# Production
ENVIRONMENT=production docker-compose up
```

---

## 8. Best Practices

### ✅ DO's

1. **Use Service Names for Communication**
   ```yaml
   # ✅ Good
   DATABASE_URL=postgresql://postgres:5432/mydb
   
   # ❌ Bad
   DATABASE_URL=postgresql://172.28.0.5:5432/mydb
   ```

2. **Isolate Sensitive Services**
   ```yaml
   # ✅ Good - Database on private network
   services:
     db:
       networks:
         - backend-network
   
   networks:
     backend-network:
       internal: true
   ```

3. **Use Descriptive Network Names**
   ```yaml
   # ✅ Good
   networks:
     frontend-public
     backend-private
     database-isolated
   
   # ❌ Bad
   networks:
     net1
     net2
     net3
   ```

4. **Document Network Architecture**
   ```yaml
   # Add comments explaining network purpose
   networks:
     public:
       # External-facing services (nginx, frontend)
     app:
       # Application layer (API, services)
     data:
       # Data layer (databases, cache)
       internal: true
   ```

---

### ❌ DON'Ts

1. **Don't Expose Database Ports in Production**
   ```yaml
   # ❌ Bad - Security risk!
   services:
     postgres:
       ports:
         - "5432:5432"
   
   # ✅ Good - No external access
   services:
     postgres:
       networks:
         - backend
   ```

2. **Don't Use Host Network Unless Necessary**
   ```yaml
   # ❌ Bad - No isolation
   services:
     app:
       network_mode: host
   
   # ✅ Good - Proper isolation
   services:
     app:
       networks:
         - app-network
   ```

3. **Don't Hardcode IP Addresses**
   ```yaml
   # ❌ Bad
   REDIS_HOST=172.28.0.15
   
   # ✅ Good
   REDIS_HOST=redis
   ```

---

### Security Best Practices

```yaml
version: '3.8'

services:
  # Public services
  nginx:
    image: nginx:alpine
    networks:
      - dmz  # Demilitarized zone
    ports:
      - "80:80"
      - "443:443"
  
  # Application services
  api:
    build: ./api
    networks:
      - dmz
      - application
    # No ports exposed!
  
  # Data services
  database:
    image: postgres:15
    networks:
      - data
    # No ports exposed!
  
  cache:
    image: redis:alpine
    networks:
      - data
    # No ports exposed!

networks:
  dmz:
    driver: bridge
  application:
    driver: bridge
  data:
    driver: bridge
    internal: true  # Completely isolated from internet
```

---

## 9. Troubleshooting

### Common Issues and Solutions

#### Issue 1: Containers Can't Communicate

**Problem:**
```bash
curl: (6) Could not resolve host: backend
```

**Solutions:**

1. **Check if containers are on same network:**
   ```bash
   docker network inspect myproject_default
   ```

2. **Verify service names:**
   ```yaml
   services:
     backend:  # ← This is the hostname
       image: node:22-alpine
   ```

3. **Check network configuration:**
   ```yaml
   services:
     frontend:
       networks:
         - app-network  # Must match!
     backend:
       networks:
         - app-network  # Must match!
   ```

---

#### Issue 2: Port Already in Use

**Problem:**
```bash
Error: bind: address already in use
```

**Solutions:**

1. **Change host port:**
   ```yaml
   ports:
     - "3001:3000"  # Use different host port
   ```

2. **Find and kill process:**
   ```bash
   # Windows
   netstat -ano | findstr :3000
   taskkill /PID <PID> /F
   
   # Linux/Mac
   lsof -i :3000
   kill -9 <PID>
   ```

---

#### Issue 3: DNS Resolution Not Working

**Problem:**
```bash
getaddrinfo ENOTFOUND backend
```

**Solutions:**

1. **Use correct service name:**
   ```javascript
   // ✅ Correct
   fetch('http://backend:3000')
   
   // ❌ Wrong
   fetch('http://backend-service:3000')  // Unless aliased
   ```

2. **Check network membership:**
   ```bash
   docker-compose ps
   docker network inspect <network-name>
   ```

---

#### Issue 4: Can't Access from Host

**Problem:**
Can't access `http://localhost:3000`

**Solutions:**

1. **Check port mapping:**
   ```yaml
   services:
     app:
       ports:
         - "3000:3000"  # host:container
   ```

2. **Verify container is running:**
   ```bash
   docker-compose ps
   ```

3. **Check firewall:**
   ```bash
   # Windows
   netsh advfirewall firewall add rule name="Docker" dir=in action=allow protocol=TCP localport=3000
   ```

---

### Debugging Commands

```bash
# List all networks
docker network ls

# Inspect a network
docker network inspect <network-name>

# See which containers are on a network
docker network inspect <network-name> | grep -A 10 "Containers"

# Test connectivity from inside container
docker-compose exec frontend ping backend
docker-compose exec frontend nslookup backend
docker-compose exec frontend wget -O- http://backend:3000

# View container logs
docker-compose logs frontend
docker-compose logs backend

# Check container IP address
docker inspect <container-name> | grep IPAddress

# Test from host
curl http://localhost:3000
telnet localhost 3000
```

---

## 🎯 Quick Reference

### Network Types Comparison

| Feature | Bridge | Host | None | Overlay |
|---------|--------|------|------|---------|
| **Isolation** | ✅ Yes | ❌ No | ✅ Complete | ✅ Yes |
| **DNS Resolution** | ✅ Yes | ❌ No | ❌ No | ✅ Yes |
| **Port Mapping** | ✅ Yes | ❌ N/A | ❌ N/A | ✅ Yes |
| **Multi-Host** | ❌ No | ❌ No | ❌ No | ✅ Yes |
| **Performance** | Good | Best | N/A | Good |
| **Use Case** | Default | Performance | Security | Swarm |

---

### Network Configuration Cheat Sheet

```yaml
# Minimal (auto network)
services:
  app:
    image: myapp

# Single custom network
services:
  app:
    networks:
      - my-network
networks:
  my-network:

# Multiple networks
services:
  app:
    networks:
      - frontend
      - backend
networks:
  frontend:
  backend:

# With aliases
services:
  app:
    networks:
      my-network:
        aliases:
          - api
          - backend

# Static IP
services:
  app:
    networks:
      my-network:
        ipv4_address: 172.28.0.10
networks:
  my-network:
    ipam:
      config:
        - subnet: 172.28.0.0/16

# External network
networks:
  existing-network:
    external: true

# Internal network (no internet)
networks:
  private:
    internal: true
```

---

## 📖 Summary

### Key Takeaways

1. **Default is Usually Fine**
   - Docker Compose auto-creates networks
   - Perfect for simple apps

2. **Use Custom Networks For:**
   - Security isolation
   - Multiple environments
   - Complex architectures

3. **Network Isolation = Security**
   - Keep databases on private networks
   - Use `internal: true` for sensitive services

4. **DNS Over IPs**
   - Always use service names
   - Let Docker handle IP addresses

5. **Document Your Architecture**
   - Comment your networks
   - Draw diagrams
   - Make it clear for your team

---

## 🚀 Next Steps

1. **Practice with Examples**
   - Start with default network
   - Add custom networks
   - Experiment with isolation

2. **Build Real Projects**
   - Multi-tier applications
   - Microservices
   - Production deployments

3. **Learn Advanced Topics**
   - Docker Swarm
   - Kubernetes networking
   - Service mesh (Istio, Linkerd)

---

## 📚 Additional Resources

- [Docker Networking Documentation](https://docs.docker.com/network/)
- [Docker Compose Networking](https://docs.docker.com/compose/networking/)
- [Docker Network Drivers](https://docs.docker.com/network/drivers/)
- [Best Practices](https://docs.docker.com/develop/dev-best-practices/)

---

**Made with ❤️ for Docker learners**

*From Beginner to Advanced - Master Docker Compose Networking!*
