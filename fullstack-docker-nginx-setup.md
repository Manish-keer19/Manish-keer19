# Complete Guide: Dockerizing a Full-Stack Application

## Table of Contents
1. [Introduction](#introduction)
2. [Project Structure](#project-structure)
3. [Frontend Setup (React + Vite + Nginx)](#frontend-setup)
4. [Backend Setup (Node.js + Express)](#backend-setup)
5. [Docker Compose Configuration](#docker-compose-configuration)
6. [Environment Variables](#environment-variables)
7. [Building and Running](#building-and-running)
8. [Docker Commands Reference](#docker-commands-reference)
9. [Production Considerations](#production-considerations)

---

## Introduction

This guide walks you through containerizing a full-stack application using Docker. We'll create isolated, reproducible environments for both frontend and backend services that can be deployed anywhere Docker runs.

**What you'll learn:**
- Creating multi-stage Docker builds
- Using Nginx as a production web server
- Configuring Docker networks and volumes
- Managing environment variables across services
- Essential Docker commands for development and production

---

## Project Structure

Here's the recommended directory structure for your Dockerized full-stack project:

```
my-fullstack-app/
├── frontend/
│   ├── src/
│   │   ├── App.jsx
│   │   ├── main.jsx
│   │   └── ...
│   ├── public/
│   ├── index.html
│   ├── package.json
│   ├── vite.config.js
│   ├── .env.production
│   ├── Dockerfile
│   └── nginx.conf
├── backend/
│   ├── src/
│   │   ├── server.js
│   │   ├── routes/
│   │   └── ...
│   ├── package.json
│   ├── .env
│   └── Dockerfile
├── docker-compose.yml
└── .dockerignore
```

### File Purpose Breakdown

| File | Purpose |
|------|---------|
| `frontend/Dockerfile` | Instructions to build the React app and serve with Nginx |
| `frontend/nginx.conf` | Nginx configuration for serving React SPA |
| `backend/Dockerfile` | Instructions to build and run the Node.js API |
| `docker-compose.yml` | Orchestrates both services, networks, and volumes |
| `.dockerignore` | Specifies files to exclude from Docker context |
| `.env` files | Environment-specific configuration |

---

## Frontend Setup

### Frontend Dockerfile

Create `frontend/Dockerfile`:

```dockerfile
# Stage 1: Build the React application
FROM node:20-alpine AS build

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Build the application
RUN npm run build

# Stage 2: Serve with Nginx
FROM nginx:alpine

# Copy built assets from build stage
COPY --from=build /app/dist /usr/share/nginx/html

# Copy custom nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Expose port 80
EXPOSE 80

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
```

**Explanation:**
- **Multi-stage build**: First stage builds the React app, second stage serves it
- **node:20-alpine**: Lightweight Node.js image for building
- **nginx:alpine**: Lightweight Nginx image for serving static files
- **npm ci**: Installs exact versions from package-lock.json (faster, more reliable)
- **COPY --from=build**: Copies only the built files, not source code or node_modules
- **daemon off**: Keeps Nginx running in foreground (required for Docker)

### Nginx Configuration

Create `frontend/nginx.conf`:

```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    # Gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Main location - serve React app
    location / {
        try_files $uri $uri/ /index.html;
    }

    # API proxy (optional - if you want Nginx to proxy API calls)
    location /api {
        proxy_pass http://backend:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    # Cache static assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Don't cache index.html
    location = /index.html {
        add_header Cache-Control "no-cache, no-store, must-revalidate";
    }
}
```

**Key Features:**
- **try_files**: Enables client-side routing for React Router
- **proxy_pass**: Optional API proxying through Nginx
- **Gzip**: Compresses responses for faster loading
- **Security headers**: Basic security improvements
- **Caching**: Aggressive caching for static assets, no caching for index.html

### Frontend Environment Variables

Create `frontend/.env.production`:

```env
VITE_API_URL=http://localhost:5000/api
```

In your React code (`src/config.js`):

```javascript
export const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:5000/api';
```

Usage in components:

```javascript
import { API_URL } from './config';

const fetchData = async () => {
  const response = await fetch(`${API_URL}/users`);
  const data = await response.json();
  return data;
};
```

---

## Backend Setup

### Backend Dockerfile

Create `backend/Dockerfile`:

```dockerfile
# Use official Node.js runtime
FROM node:20-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Create non-root user for security
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001 && \
    chown -R nodejs:nodejs /app

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 5000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD node -e "require('http').get('http://localhost:5000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# Start the application
CMD ["node", "src/server.js"]
```

**Explanation:**
- **Alpine Linux**: Minimal base image (smaller, faster)
- **npm ci --only=production**: Installs production dependencies only
- **Non-root user**: Security best practice
- **HEALTHCHECK**: Docker monitors application health
- **Single CMD**: Runs the application

### Backend Server Example

Create `backend/src/server.js`:

```javascript
const express = require('express');
const cors = require('cors');
require('dotenv').config();

const app = express();
const PORT = process.env.PORT || 5000;

// Middleware
app.use(cors({
  origin: process.env.FRONTEND_URL || 'http://localhost:3000',
  credentials: true
}));
app.use(express.json());

// Health check endpoint
app.get('/health', (req, res) => {
  res.status(200).json({ status: 'ok' });
});

// Sample API route
app.get('/api/users', (req, res) => {
  res.json([
    { id: 1, name: 'John Doe' },
    { id: 2, name: 'Jane Smith' }
  ]);
});

// Error handling
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Something went wrong!' });
});

app.listen(PORT, '0.0.0.0', () => {
  console.log(`Server running on port ${PORT}`);
});
```

### Backend Environment Variables

Create `backend/.env`:

```env
PORT=5000
NODE_ENV=production
DATABASE_URL=postgresql://user:password@db:5432/myapp
FRONTEND_URL=http://localhost:3000
JWT_SECRET=your-secret-key-change-in-production
```

**Important**: Never commit `.env` files to version control. Add them to `.gitignore`.

---

## Docker Compose Configuration

Create `docker-compose.yml` in the root directory:

```yaml
version: '3.8'

services:
  # Frontend Service
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: fullstack-frontend
    ports:
      - "3000:80"
    environment:
      - VITE_API_URL=http://localhost:5000/api
    depends_on:
      - backend
    networks:
      - fullstack-network
    restart: unless-stopped

  # Backend Service
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: fullstack-backend
    ports:
      - "5000:5000"
    environment:
      - PORT=5000
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - FRONTEND_URL=http://localhost:3000
    env_file:
      - ./backend/.env
    volumes:
      - ./backend/logs:/app/logs
    depends_on:
      - db
    networks:
      - fullstack-network
    restart: unless-stopped

  # Database Service (Optional - PostgreSQL example)
  db:
    image: postgres:16-alpine
    container_name: fullstack-db
    environment:
      - POSTGRES_USER=myuser
      - POSTGRES_PASSWORD=mypassword
      - POSTGRES_DB=myapp
    volumes:
      - postgres-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - fullstack-network
    restart: unless-stopped

# Networks
networks:
  fullstack-network:
    driver: bridge

# Volumes
volumes:
  postgres-data:
    driver: local
```

### Docker Compose Breakdown

**Services Section:**
- **frontend**: Builds and runs React app with Nginx
- **backend**: Builds and runs Node.js API
- **db**: Uses official PostgreSQL image (optional)

**Key Configuration:**
- **build.context**: Directory containing Dockerfile
- **ports**: Maps host:container ports
- **environment**: Inline environment variables
- **env_file**: Loads variables from file
- **volumes**: Persists data and shares files
- **depends_on**: Ensures services start in order
- **networks**: Allows inter-service communication
- **restart**: Restart policy for containers

**Networks:**
- Custom bridge network allows services to communicate using service names as hostnames

**Volumes:**
- Named volumes persist data beyond container lifecycle

---

## Environment Variables

### Passing Variables to Frontend

Since React is built at compile time, environment variables must be available during the build process.

**Option 1: Build-time variables (Recommended)**

In `docker-compose.yml`:

```yaml
frontend:
  build:
    context: ./frontend
    dockerfile: Dockerfile
    args:
      - VITE_API_URL=http://localhost:5000/api
```

In `frontend/Dockerfile`:

```dockerfile
ARG VITE_API_URL
ENV VITE_API_URL=$VITE_API_URL
RUN npm run build
```

**Option 2: Runtime injection with script**

For true runtime configuration, create a shell script that replaces placeholders in the built files.

### Passing Variables to Backend

Backend can access environment variables at runtime:

```javascript
const dbUrl = process.env.DATABASE_URL;
const port = process.env.PORT || 5000;
```

In Docker Compose:

```yaml
backend:
  environment:
    - PORT=5000
    - DATABASE_URL=${DATABASE_URL}
  env_file:
    - ./backend/.env
```

---

## Building and Running

### Initial Build

Build all services defined in docker-compose.yml:

```bash
docker compose build
```

Build specific service:

```bash
docker compose build frontend
```

Build without cache (fresh build):

```bash
docker compose build --no-cache
```

### Starting Services

Start all services in detached mode (background):

```bash
docker compose up -d
```

Start and see logs in real-time:

```bash
docker compose up
```

Start specific service:

```bash
docker compose up -d backend
```

### Rebuild and Start

Rebuild images and start containers:

```bash
docker compose up -d --build
```

### Accessing the Application

- **Frontend**: http://localhost:3000
- **Backend API**: http://localhost:5000/api
- **Database**: localhost:5432 (if using PostgreSQL)

---

## Docker Commands Reference

### Container Management

**View running containers:**
```bash
docker compose ps
```

**View all containers (including stopped):**
```bash
docker compose ps -a
```

**Stop all services:**
```bash
docker compose stop
```

**Stop specific service:**
```bash
docker compose stop backend
```

**Start stopped services:**
```bash
docker compose start
```

**Restart services:**
```bash
docker compose restart
```

**Restart specific service:**
```bash
docker compose restart frontend
```

### Logs and Debugging

**View logs from all services:**
```bash
docker compose logs
```

**Follow logs in real-time:**
```bash
docker compose logs -f
```

**View logs from specific service:**
```bash
docker compose logs backend
```

**View last 100 lines:**
```bash
docker compose logs --tail=100 backend
```

**Execute command in running container:**
```bash
docker compose exec backend sh
```

**Execute command in frontend container:**
```bash
docker compose exec frontend sh
```

### Cleanup

**Stop and remove containers:**
```bash
docker compose down
```

**Remove containers and volumes:**
```bash
docker compose down -v
```

**Remove containers, networks, and images:**
```bash
docker compose down --rmi all
```

**Remove everything including volumes:**
```bash
docker compose down -v --rmi all
```

**Remove unused images:**
```bash
docker image prune -a
```

**Remove all stopped containers:**
```bash
docker container prune
```

**View disk usage:**
```bash
docker system df
```

**Clean up everything:**
```bash
docker system prune -a --volumes
```

### Image Management

**List images:**
```bash
docker images
```

**Remove specific image:**
```bash
docker rmi <image-id>
```

**Tag image:**
```bash
docker tag fullstack-frontend:latest myregistry/frontend:v1.0
```

**Push to registry:**
```bash
docker push myregistry/frontend:v1.0
```

### Volume Management

**List volumes:**
```bash
docker volume ls
```

**Inspect volume:**
```bash
docker volume inspect postgres-data
```

**Remove volume:**
```bash
docker volume rm postgres-data
```

### Network Management

**List networks:**
```bash
docker network ls
```

**Inspect network:**
```bash
docker network inspect fullstack-network
```

---

## Production Considerations

### 1. Environment Variables

**Never hardcode secrets in Dockerfiles or docker-compose.yml.**

Use environment variable files:

```bash
# Create .env file in root
DATABASE_URL=postgresql://user:pass@db:5432/prod
JWT_SECRET=very-secure-random-string
```

Reference in docker-compose.yml:

```yaml
backend:
  env_file:
    - .env
```

### 2. Security Best Practices

**Run as non-root user:**
```dockerfile
USER nodejs
```

**Scan images for vulnerabilities:**
```bash
docker scan fullstack-backend
```

**Use specific image versions:**
```dockerfile
FROM node:20.10.0-alpine
```

**Minimize image layers:**
- Combine RUN commands
- Remove unnecessary files
- Use multi-stage builds

### 3. Create .dockerignore

Create `.dockerignore` in frontend and backend directories:

```
node_modules
npm-debug.log
.env
.env.local
.git
.gitignore
README.md
.vscode
.idea
dist
build
coverage
.DS_Store
```

### 4. Health Checks

Add health checks to services:

```yaml
backend:
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
    interval: 30s
    timeout: 10s
    retries: 3
    start_period: 40s
```

### 5. Resource Limits

Prevent containers from consuming too many resources:

```yaml
backend:
  deploy:
    resources:
      limits:
        cpus: '0.5'
        memory: 512M
      reservations:
        cpus: '0.25'
        memory: 256M
```

### 6. Logging Configuration

Configure logging drivers:

```yaml
backend:
  logging:
    driver: "json-file"
    options:
      max-size: "10m"
      max-file: "3"
```

### 7. Production Docker Compose

Create `docker-compose.prod.yml`:

```yaml
version: '3.8'

services:
  frontend:
    image: myregistry/frontend:latest
    restart: always
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  backend:
    image: myregistry/backend:latest
    restart: always
    deploy:
      replicas: 2
      resources:
        limits:
          memory: 512M
```

Run with:
```bash
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

### 8. SSL/TLS Configuration

For production, add SSL certificates with Nginx:

```nginx
server {
    listen 443 ssl http2;
    server_name yourdomain.com;

    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;

    # ... rest of configuration
}
```

### 9. CI/CD Integration

Example GitHub Actions workflow:

```yaml
name: Build and Deploy

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Build images
        run: docker compose build
      
      - name: Push to registry
        run: |
          echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
          docker compose push
      
      - name: Deploy
        run: |
          ssh user@server 'cd /app && docker compose pull && docker compose up -d'
```

---

## Quick Reference Commands

```bash
# Build and start
docker compose up -d --build

# View logs
docker compose logs -f

# Stop services
docker compose stop

# Restart services
docker compose restart

# Remove everything
docker compose down -v --rmi all

# Execute shell in container
docker compose exec backend sh

# View running containers
docker compose ps

# View resource usage
docker stats
```

---

## Troubleshooting

### Container won't start
```bash
# Check logs
docker compose logs backend

# Inspect container
docker inspect fullstack-backend
```

### Cannot connect to database
- Ensure services are on the same network
- Use service name as hostname (e.g., `db` not `localhost`)
- Check environment variables

### Port already in use
```bash
# Find process using port
lsof -i :3000

# Stop conflicting container
docker compose stop
```

### Permission denied errors
- Ensure proper file permissions
- Check user/group in Dockerfile
- Verify volume mount paths

### Image build fails
```bash
# Build without cache
docker compose build --no-cache

# Check Dockerfile syntax
docker compose config
```

---

## Conclusion

You now have a complete, production-ready Docker setup for your full-stack application. This configuration provides:

- Isolated, reproducible environments
- Easy deployment across different platforms
- Scalable architecture with Docker Compose
- Security best practices
- Efficient resource usage with multi-stage builds

**Next Steps:**
1. Customize the configuration for your specific needs
2. Add additional services (Redis, Elasticsearch, etc.)
3. Set up CI/CD pipelines
4. Configure monitoring and logging
5. Implement orchestration with Kubernetes for large-scale deployments

Remember to regularly update your base images and dependencies for security patches!