# Docker Production Cheat Sheet

> **Master containerization for NestJS, PostgreSQL, Redis, and Production Deployments.**

---

## 1. Dockerfile Examples

### Production NestJS Dockerfile (Multi-Stage Build)
Use multi-stage builds to keep image size small (from ~1GB to ~150MB).

```dockerfile
# ----------------------------------------
# Stage 1: Build
# ----------------------------------------
FROM node:18-alpine AS builder

WORKDIR /app

# Install dependencies (cache mount speeds up repeated builds)
COPY package*.json ./
COPY prisma ./prisma/

RUN npm ci

COPY . .

# Generate Prisma Client & Build NestJS
RUN npx prisma generate
RUN npm run build

# ----------------------------------------
# Stage 2: Production Run
# ----------------------------------------
FROM node:18-alpine AS runner

WORKDIR /app
ENV NODE_ENV production

# Copy only necessary files from builder
COPY --from=builder /app/package*.json ./
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/prisma ./prisma

# Create non-root user for security
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nestjs -u 1001
USER nestjs

# Expose port and start
EXPOSE 3000
CMD ["node", "dist/main"]
```

---

## 2. Docker Compose (Dev Environment)

Orchestrate your app + db + redis + tooling easily.

`docker-compose.yml`:
```yaml
version: '3.8'

services:
  # ---------------------
  # NestJS API
  # ---------------------
  api:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    volumes:
      - ./src:/app/src  # Hot Reload mapping
    environment:
      DATABASE_URL: postgres://user:password@postgres:5432/mydb
      REDIS_URL: redis://redis:6379
    depends_on:
      postgres:
        condition: service_healthy

  # ---------------------
  # PostgreSQL
  # ---------------------
  postgres:
    image: postgres:15-alpine
    restart: always
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d mydb"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ---------------------
  # Redis
  # ---------------------
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"

  # ---------------------
  # PgAdmin (Optional UI)
  # ---------------------
  pgadmin:
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@admin.com
      PGADMIN_DEFAULT_PASSWORD: admin
    ports:
      - "8080:80"
    depends_on:
      - postgres

volumes:
  postgres_data:
```

---

## 3. Essential Commands

### Build & Run
```bash
# Start all containers in background
docker-compose up -d

# Rebuild containers (if Dockerfile changed)
docker-compose up -d --build

# View logs (real-time)
docker-compose logs -f api

# Stop down
docker-compose down
```

### Cleanup
```bash
# Remove unused containers, networks, images (dangling)
docker system prune -f

# Remove volumes (WARNING: Deletes DB data)
docker-compose down -v
```

### Debugging
```bash
# Enter inside a running container
docker exec -it <container_name_or_id> /bin/sh

# Check resource usage
docker stats
```

---

## 4. Best Practices

### Image Caching
- **Order matters**: Copy `package.json` and run `npm install` *before* copying source code. Docker caches layers. If you change code, you don't want to reinstall npm packages every time.

### Security
- **Non-root user**: Always switch to a non-root user (like `USER nestjs`) in production.
- **Secrets**: Never hardcode `.env` in the Dockerfile. Use `docker-compose` `env_file` or inject at runtime.
- **Alpine**: Use `alpine` images for smaller attack surface and size.

### Networking
- **Internal DNS**: In `docker-compose`, services communicate via service name. NestJS talks to `postgres` (service name), NOT `localhost`.

---

## 5. Production Deployment Patterns

### Option A: VPS (Simple)
1. SSH into VPS.
2. `git pull`.
3. `docker-compose -f docker-compose.prod.yml up -d --build`.

### Option B: CI/CD (GitHub Actions)
1. **GitHub Action**: Build image -> Push to Docker Hub / GHCR.
2. **VPS**: Pull new image -> Restart container.

`.github/workflows/deploy.yml`:
```yaml
name: Build & Push
on:
  push:
    branches: [main]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Docker Login
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.DOCKER_USER }}
        password: ${{ secrets.DOCKER_PASS }}
    - name: Build & Push
      run: |
        docker build -t myuser/myapp:latest .
        docker push myuser/myapp:latest
```

---

## 6. Multi-Container Architecture Diagram

```mermaid
graph TD
    Client[Client Browser] -->|HTTP:80| Nginx[Nginx Proxy]
    Nginx -->|HTTP:3000| API[NestJS API Container]
    API -->|TCP:5432| DB[Postgres Container]
    API -->|TCP:6379| Redis[Redis Container]
    
    subgraph Docker Network
    API
    DB
    Redis
    end
```

---

## 7. Final Checklist: Docker

### Development
- [ ] `.dockerignore` exists (excludes `node_modules`, `dist`, `.git`).
- [ ] Multi-stage build is used.
- [ ] `npm ci` is used instead of `npm install` for deterministic builds.

### Production
- [ ] Image size is optimized (< 250MB).
- [ ] Running as non-root user inside container.
- [ ] Restart policy is `always` or `unless-stopped`.
- [ ] Database credentials are NOT in `docker-compose.yml` (Use `.env` file only on server).
- [ ] Healthchecks configured for DB dependent services.