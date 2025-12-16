# 🐳 Complete Docker Build Guide: React, Node.js, NestJS & More

## 📚 Table of Contents
- [Core Concepts](#core-concepts)
- [Mental Framework](#mental-framework)
- [React Applications](#react-applications)
- [Node.js/Express Applications](#nodejs-express-applications)
- [NestJS Applications](#nestjs-applications)
- [Full-Stack Applications](#full-stack-applications)
- [Best Practices](#best-practices)
- [Common Patterns](#common-patterns)
- [Troubleshooting](#troubleshooting)

---

## 🧠 Core Concepts

### The 5 Questions Framework

Before writing ANY Dockerfile, ask yourself:

```
1. What do I need to BUILD my app?
   → Build tools, dependencies, source code

2. What do I need to RUN my app?
   → Runtime, compiled files, minimal dependencies

3. How can I make it SMALLER?
   → Multi-stage builds, Alpine images, remove build artifacts

4. How can I make it FASTER?
   → Layer caching, optimal ordering, build cache

5. How can I make it SECURE?
   → Non-root user, no secrets, minimal attack surface
```

### Docker Layer Caching Rules

```
Order by CHANGE FREQUENCY (least → most):

1. Base image (FROM)           ← Never changes
2. System packages (RUN apk)   ← Rarely changes
3. Dependencies (package.json) ← Changes occasionally
4. Source code (COPY . .)      ← Changes frequently
5. Build output (RUN build)    ← Generated from source
```

### Multi-Stage Build Pattern

```
┌─────────────────────────────────────┐
│ STAGE 1: Builder (Temporary)        │
│ - All build tools                   │
│ - All dependencies                  │
│ - Source code                       │
│ - Compile/Build                     │
│ Result: Large image (1GB+)          │
└─────────────────────────────────────┘
              ↓
    COPY only what's needed
              ↓
┌─────────────────────────────────────┐
│ STAGE 2: Production (Final)         │
│ - Minimal runtime                   │
│ - Built artifacts only              │
│ - No build tools                    │
│ Result: Small image (50MB)          │
└─────────────────────────────────────┘
```

---

## ⚛️ React Applications

### Production Dockerfile (Multi-Stage with Nginx)

```dockerfile
# ================================
# Stage 1: Build the React App
# ================================
FROM node:20-alpine AS builder

# Set working directory
WORKDIR /app

# Copy package files first (for caching)
COPY package*.json ./

# Install ALL dependencies (including devDependencies for build)
RUN npm ci --only=production=false

# Copy source code
COPY . .

# Build arguments for environment variables
ARG VITE_API_BASE_URL
ARG REACT_APP_API_URL
ENV VITE_API_BASE_URL=${VITE_API_BASE_URL}
ENV REACT_APP_API_URL=${REACT_APP_API_URL}

# Build the application
RUN npm run build

# ================================
# Stage 2: Serve with Nginx
# ================================
FROM nginx:1.27-alpine

# Install curl for healthchecks
RUN apk add --no-cache curl

# Remove default nginx config
RUN rm /etc/nginx/conf.d/default.conf

# Copy custom nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copy built assets from builder stage
# For Vite: /app/dist
# For Create React App: /app/build
COPY --from=builder /app/dist /usr/share/nginx/html

# Add healthcheck
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/ || exit 1

# Expose port 80
EXPOSE 80

# Start nginx in foreground
CMD ["nginx", "-g", "daemon off;"]
```

### Development Dockerfile

```dockerfile
FROM node:20-alpine

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm install

# Copy source code
COPY . .

# Expose Vite dev server port (or 3000 for CRA)
EXPOSE 5173

# Start dev server with hot reload
# For Vite:
CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]

# For Create React App:
# CMD ["npm", "start"]
```

### Required nginx.conf for React

```nginx
server {
    listen 80;
    server_name _;
    
    root /usr/share/nginx/html;
    index index.html;

    # Enable gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript 
               application/x-javascript application/xml+rss 
               application/javascript application/json 
               image/svg+xml;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Cache static assets
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Handle React Router - serve index.html for all routes
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Custom error pages
    error_page 404 /index.html;
}
```

### Build Commands

```bash
# Production build
docker build -t my-react-app:latest \
  --build-arg VITE_API_BASE_URL=http://api.example.com \
  .

# Run production
docker run -d -p 80:80 --name react-app my-react-app:latest

# Development build
docker build -f Dockerfile.dev -t my-react-app:dev .

# Run development with volume mounting
docker run -d -p 5173:5173 \
  -v $(pwd):/app \
  -v /app/node_modules \
  --name react-dev \
  my-react-app:dev
```

---

## 🟢 Node.js/Express Applications

### Production Dockerfile (Multi-Stage)

```dockerfile
# ================================
# Stage 1: Build (if using TypeScript)
# ================================
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install ALL dependencies
RUN npm ci

# Copy source code
COPY . .

# Build TypeScript (if applicable)
RUN npm run build

# ================================
# Stage 2: Production
# ================================
FROM node:20-alpine

# Create app user (security)
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install ONLY production dependencies
RUN npm ci --only=production && \
    npm cache clean --force

# Copy built files from builder
# For TypeScript: dist folder
# For JavaScript: src folder
COPY --from=builder /app/dist ./dist

# Change ownership to non-root user
RUN chown -R nodejs:nodejs /app

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# Start the application
CMD ["node", "dist/server.js"]
```

### Simple Node.js Dockerfile (JavaScript only)

```dockerfile
FROM node:20-alpine

# Create app user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install production dependencies
RUN npm ci --only=production && \
    npm cache clean --force

# Copy source code
COPY . .

# Change ownership
RUN chown -R nodejs:nodejs /app

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Start application
CMD ["node", "server.js"]
```

### Development Dockerfile

```dockerfile
FROM node:20-alpine

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm install

# Copy source code
COPY . .

# Expose port
EXPOSE 3000

# Start with nodemon for hot reload
CMD ["npm", "run", "dev"]
```

### Build Commands

```bash
# Production build
docker build -t my-node-app:latest .

# Run production
docker run -d -p 3000:3000 \
  -e NODE_ENV=production \
  -e DATABASE_URL=postgresql://... \
  --name node-app \
  my-node-app:latest

# Development with volume mounting
docker run -d -p 3000:3000 \
  -v $(pwd):/app \
  -v /app/node_modules \
  -e NODE_ENV=development \
  --name node-dev \
  my-node-app:dev
```

---

## 🔴 NestJS Applications

### Production Dockerfile (Multi-Stage)

```dockerfile
# ================================
# Stage 1: Build
# ================================
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci

# Copy source code
COPY . .

# Build NestJS application
RUN npm run build

# ================================
# Stage 2: Production
# ================================
FROM node:20-alpine

# Create app user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install ONLY production dependencies
RUN npm ci --only=production && \
    npm cache clean --force

# Copy built files from builder
COPY --from=builder /app/dist ./dist

# Copy any static assets if needed
# COPY --from=builder /app/public ./public

# Change ownership
RUN chown -R nodejs:nodejs /app

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# Start NestJS application
CMD ["node", "dist/main.js"]
```

### Development Dockerfile

```dockerfile
FROM node:20-alpine

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm install

# Copy source code
COPY . .

# Expose port
EXPOSE 3000

# Start with watch mode
CMD ["npm", "run", "start:dev"]
```

### Build Commands

```bash
# Production build
docker build -t my-nest-app:latest .

# Run production
docker run -d -p 3000:3000 \
  -e NODE_ENV=production \
  -e DATABASE_URL=postgresql://... \
  --name nest-app \
  my-nest-app:latest

# Development
docker run -d -p 3000:3000 \
  -v $(pwd):/app \
  -v /app/node_modules \
  --name nest-dev \
  my-nest-app:dev
```

---

## 🔄 Full-Stack Applications

### Docker Compose for Full Stack

```yaml
version: '3.8'

services:
  # PostgreSQL Database
  database:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
      POSTGRES_DB: mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myuser"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Backend (NestJS/Node.js)
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "8081:8081"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://myuser:mypassword@database:5432/mydb
      - JWT_SECRET=your-secret-key
    depends_on:
      database:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8081/health"]
      interval: 30s
      timeout: 3s
      retries: 3

  # Frontend (React)
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
      args:
        VITE_API_BASE_URL: http://localhost:8081
    ports:
      - "80:80"
    depends_on:
      - backend
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/"]
      interval: 30s
      timeout: 3s
      retries: 3

volumes:
  postgres_data:
```

### Development Docker Compose

```yaml
version: '3.8'

services:
  database:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
      POSTGRES_DB: mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    ports:
      - "8081:8081"
    volumes:
      - ./backend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://myuser:mypassword@database:5432/mydb
    depends_on:
      - database

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    ports:
      - "5173:5173"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    environment:
      - VITE_API_BASE_URL=http://localhost:8081

volumes:
  postgres_data:
```

---

## ✅ Best Practices

### 1. Always Use .dockerignore

```
# .dockerignore
node_modules
dist
build
.git
.env
.env.local
*.log
.DS_Store
coverage
.vscode
.idea
npm-debug.log*
yarn-debug.log*
yarn-error.log*
```

### 2. Security Checklist

```dockerfile
# ✅ Use specific versions (not 'latest')
FROM node:20-alpine

# ✅ Run as non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs

# ✅ Don't include secrets in image
# Use environment variables or secrets management

# ✅ Use multi-stage builds
# Reduces attack surface

# ✅ Scan for vulnerabilities
# docker scan my-image:latest
```

### 3. Optimization Checklist

```dockerfile
# ✅ Use Alpine images
FROM node:20-alpine  # 5MB vs 900MB

# ✅ Optimize layer caching
COPY package*.json ./  # Before source code
RUN npm ci
COPY . .

# ✅ Clean up in same layer
RUN npm ci && \
    npm cache clean --force

# ✅ Use .dockerignore
# Prevents copying unnecessary files

# ✅ Multi-stage builds
# Only copy what's needed to production
```

### 4. Health Checks

```dockerfile
# For web servers
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# For Node.js without curl
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# For nginx
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost/ || exit 1
```

---

## 🎯 Common Patterns

### Pattern 1: TypeScript Application

```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/main.js"]
```

### Pattern 2: Static Site (React/Vue/Angular)

```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
CMD ["nginx", "-g", "daemon off;"]
```

### Pattern 3: Monorepo

```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
COPY packages/shared/package*.json ./packages/shared/
COPY packages/backend/package*.json ./packages/backend/
RUN npm ci
COPY . .
RUN npm run build --workspace=backend

# Production stage
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/packages/backend/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/main.js"]
```

---

## 🐛 Troubleshooting

### Issue: Build is slow

```bash
# Solution 1: Check layer caching
# Make sure package.json is copied before source code

# Solution 2: Use BuildKit
export DOCKER_BUILDKIT=1
docker build .

# Solution 3: Use build cache
docker build --cache-from my-image:latest .
```

### Issue: Image is too large

```bash
# Check image size
docker images my-image

# Solution 1: Use Alpine
FROM node:20-alpine  # Instead of node:20

# Solution 2: Multi-stage build
# Only copy necessary files to final stage

# Solution 3: Clean npm cache
RUN npm ci && npm cache clean --force
```

### Issue: Container exits immediately

```bash
# Check logs
docker logs container-name

# Common causes:
# 1. Main process exits (use CMD correctly)
# 2. Error in application
# 3. Wrong working directory

# Solution: Run interactively
docker run -it my-image sh
```

### Issue: Can't connect to database

```bash
# Solution 1: Use service name in docker-compose
DATABASE_URL=postgresql://user:pass@database:5432/db
# NOT localhost!

# Solution 2: Check network
docker network ls
docker network inspect network-name

# Solution 3: Wait for database
# Use depends_on with healthcheck
```

---

## 📊 Quick Reference

### Image Size Comparison

| Base Image | Size | Use Case |
|------------|------|----------|
| node:20 | ~900MB | ❌ Avoid |
| node:20-slim | ~200MB | ⚠️ OK |
| node:20-alpine | ~5MB | ✅ Best |
| nginx:alpine | ~25MB | ✅ Best for static |

### Build Time Optimization

| Technique | Time Saved | Complexity |
|-----------|------------|------------|
| Layer caching | 50-90% | Easy |
| BuildKit | 10-30% | Easy |
| Multi-stage | 20-40% | Medium |
| Cache mounts | 30-50% | Medium |

### Commands Cheat Sheet

```bash
# Build
docker build -t name:tag .
docker build --no-cache -t name:tag .
docker build --build-arg VAR=value -t name:tag .

# Run
docker run -d -p 3000:3000 name:tag
docker run -it name:tag sh
docker run -v $(pwd):/app name:tag

# Debug
docker logs container-name
docker exec -it container-name sh
docker inspect container-name

# Clean up
docker system prune -a
docker volume prune
docker image prune
```

---

## 🎓 Learning Path

1. **Beginner**: Single-stage Dockerfile for Node.js
2. **Intermediate**: Multi-stage build for React
3. **Advanced**: Full-stack with docker-compose
4. **Expert**: Optimized builds with BuildKit, security hardening

---

**Remember**: The best Dockerfile is one that is:
- ✅ Small (Alpine, multi-stage)
- ✅ Fast (layer caching, BuildKit)
- ✅ Secure (non-root, no secrets)
- ✅ Maintainable (clear, documented)

Happy Dockerizing! 🐳
