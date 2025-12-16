# The Backend Engineer's Masterclass: Docker, CI/CD, Nginx, & PM2

## 📚 Introduction
This guide takes you from **Zero** (writing code on your laptop) to **Hero** (deploying a secure, scalable, automated production system). It explains *how* things work, *why* we use them, and *exact steps* to implement them.

---

# Phase 1: The Foundations (Beginner)

## 1. The Components Explained (Simpler Terms)

Imagine running a Restaurant.

*   **EC2 Server:** The physical building.
*   **Docker:** The lunchbox. It keeps the food (code) exactly the same from the kitchen (laptop) to the customer (server).
*   **Nginx:** The Receptionist. It greets customers (users), decides where they should sit, and stops bad people from entering.
*   **PM2:** The Kitchen Manager. It makes sure the cooks (Node.js) are working. If a cook faints (crashes), PM2 wakes them up instantly.
*   **CI/CD:** The Delivery Truck. It automatically drives your prepared lunchboxes from the kitchen to the building.

## 2. Why "Works on my machine" is a lie
Without Docker, you install Node v18 on your laptop, but the server has Node v14. Your app crashes.
**Docker Solution:** You package Node v18 INSIDE the lunchbox. The server just holds the lunchbox. It doesn't care what's inside.

---

# Phase 2: Implementation (Intermediate)

## 3. Dockerizing React (The "Multi-Stage" Standard)
We don't just "copy files". We define a streamlined build process.

**File:** `Dockerfile` (Frontend)
```dockerfile
# --- Stage 1: The Kitchen (Build) ---
FROM node:18-alpine AS builder
# "Alpine" means a tiny, lightweight version of Linux (5MB vs 100MB)

WORKDIR /app
COPY package.json ./
RUN npm install
COPY . .
# We accept arguments from the Delivery Truck (CI/CD)
ARG VITE_API_BASE_URL
ENV VITE_API_BASE_URL=$VITE_API_BASE_URL
# Cook the meal
RUN npm run build 

# --- Stage 2: The Service (Production) ---
FROM nginx:alpine
# Throw away the kitchen tools (Node.js). Keep ONLY the food (dist folder).
COPY --from=builder /app/dist /usr/share/nginx/html
# Copy our specialized receptionist instructions
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

## 4. Configuring Nginx (The Gatekeeper)
Nginx is fast. Very fast. It handles thousands of connections while Node.js handles the logic.

**File:** `nginx.conf`
```nginx
server {
    listen 80;

    # 1. Performance: Compress files before sending
    gzip on;
    gzip_types text/plain application/javascript text/css;

    # 2. Security: Hide backend details
    server_tokens off;

    # 3. Route: API Requests -> Send to PM2 Backend
    location /api {
        # 'host.docker.internal' or '172.17.0.1' lets Docker talk to the Host machine
        proxy_pass http://172.17.0.1:8081; 
        
        # Pass the real user's IP, otherwise Backend sees '127.0.0.1'
        proxy_set_header X-Real-IP $remote_addr;
    }

    # 4. Route: React App -> Serve static files
    location / {
        root /usr/share/nginx/html;
        try_files $uri /index.html; # Vital for React Router!
    }
}
```

## 5. Setting up the Backend with PM2
Do not run `node index.js`. It uses 1 CPU core. If you have 4 cores, 3 are doing nothing.

**File:** `ecosystem.config.js` (Root of backend)
```javascript
module.exports = {
  apps: [{
    name: "backend-api",
    script: "./dist/index.js",
    instances: "max",    // USE ALL CORES
    exec_mode: "cluster", // Load balance between cores
    env: {
      NODE_ENV: "production",
      PORT: 8081
    }
  }]
}
```
**Command:** `pm2 start ecosystem.config.js`

---

# Phase 3: Automation & CI/CD (Advanced)

## 6. The "Runner" Architecture
Use a **Self-Hosted Runner**.
*   **Cost:** Free.
*   **Security:** High.
*   **Speed:** Blazing fast (deploying to itself).

**How to Install (On EC2):**
1. Go to GitHub Repo -> Settings -> Actions -> Runners -> New Self-Hosted Runner.
2. Run the provided commands on your EC2.
3. Run `./svc.sh install` and `./svc.sh start` to make it run in the background 24/7.

## 7. The CI/CD Pipeline (`cicd.yml`)
This is the instruction manual for the automation.

```yaml
name: Deploy Production
on: 
  push:
    branches: ["main"] # Only run when code hits main

jobs:
  # Job 1: Build the 'Lunchbox' (Docker Image)
  # Runs on GitHub's Cloud (Cleaner, faster internet for downloads)
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build & Push
        run: |
          docker login -u ${{ secrets.DOCKER_USER }} -p ${{ secrets.DOCKER_PASS }}
          docker build -t my-app:latest .
          docker push my-app:latest

  # Job 2: Serve the Food (Deploy)
  # Runs on YOUR EC2 (The Self-Hosted Runner)
  deploy:
    needs: build # Wait for build to finish
    runs-on: self-hosted
    steps:
      - name: Deploy
        run: |
          # 1. Get latest code
          docker pull my-app:latest
          
          # 2. Kill the old version (|| true prevents error if not running)
          docker rm -f my-frontend-container || true
          
          # 3. Start the new version
          docker run -d -p 4001:80 --name my-frontend-container my-app:latest
```

---

# Phase 4: Production Readiness (Industry Standard)

## 8. Security Checklist 🔒
1.  **Strict Firewall (Security Groups):** 
    *   Allow Port 80/443 (HTTP/S) from `0.0.0.0/0`.
    *   BLOCK Port 8081 (Backend) from public. Only allow `localhost`.
    *   BLOCK Port 5432 (Database) from public.
2.  **HTTPS (SSL):** 
    *   Never run HTTP in production.
    *   Use **Certbot**: `sudo apt install certbot` (Free SSL).
3.  **Secrets Management:**
    *   Never commit `.env`. Inject them via GitHub Secrets.

## 9. Performance Tuning 🚀
1.  **Database Connection Pooling:** Ensure Prisma/Mongoose reuses connections, or your DB will crash under load.
2.  **CDN:** Serve images from AWS S3 + CloudFront, not your server.
3.  **Rate Limiting:** In Nginx, limit users to 10 requests/second to prevent DDoS.

## 10. Monitoring 📊
1.  **Uptime:** Use UptimeRobot (Free) to ping your site every 5 mins.
2.  **Logs:** PM2 logs are great locally. For pro setups, pipe them to Datadog or AWS CloudWatch.

## Summary: The Workflow
1.  Code on Laptop -> 2. Git Push -> 3. GitHub Actions Builds Docker Image -> 4. Self-Hosted Runner on EC2 Pulls Image -> 5. App Updates Automatically.

This is the standard, professional way to build modern web infrastructure.
