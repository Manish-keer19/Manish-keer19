# 🚀 MERN Deployment Guide — Nginx + PM2 + HTTPS (Ubuntu VPS)

> **Stack:** MongoDB · Express · React · Node.js
> **Server:** Ubuntu 22.04 LTS | **Backend Port:** 5000 | **No Docker**

---

## 📁 Expected Folder Structure

```
/var/www/project/          ← React build files (frontend)
/home/ubuntu/app/
├── client/                ← React source
│   ├── build/             ← After npm run build
│   └── package.json
├── server/                ← Node/Express backend
│   ├── index.js
│   ├── .env
│   └── package.json
```

---

## 1. Server Setup

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Node.js LTS (v20)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Verify
node -v && npm -v

# Install Nginx
sudo apt install -y nginx

# Install PM2 globally
sudo npm install -g pm2

# Verify PM2
pm2 -v
```

---

## 2. Clone Project

```bash
# Install Git (if not present)
sudo apt install -y git

# Clone your repo
cd /home/ubuntu
git clone https://github.com/your-username/your-repo.git app
cd app

# Install backend dependencies
cd server
npm install

# Install frontend dependencies
cd ../client
npm install
```

---

## 3. Backend Setup (PM2)

### Create `.env` file

```bash
cd /home/ubuntu/app/server
nano .env
```

```env
# Example .env
NODE_ENV=production
PORT=5000
MONGO_URI=mongodb+srv://user:password@cluster.mongodb.net/mydb
JWT_SECRET=your_super_secret_key_here
CLIENT_URL=https://yourdomain.com
```

### Start Backend with PM2

```bash
cd /home/ubuntu/app/server

# Start with PM2
pm2 start index.js --name "mern-backend"

# Verify it's running
pm2 list
pm2 logs mern-backend --lines 30
```

### Save & Enable Auto-Start on Reboot

```bash
# Save process list
pm2 save

# Generate startup script
pm2 startup
# ⚠️ Copy and run the exact command PM2 outputs — it's user/OS specific
# Example output:
# sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu
```

---

## 4. Frontend Build

```bash
cd /home/ubuntu/app/client

# Set API base URL before building (if using .env in React)
echo "REACT_APP_API_URL=https://yourdomain.com/api" > .env.production

# Build React app
npm run build

# Create web root directory
sudo mkdir -p /var/www/project

# Copy build files to Nginx web root
sudo cp -r build/* /var/www/project/

# Set correct ownership
sudo chown -R www-data:www-data /var/www/project
sudo chmod -R 755 /var/www/project
```

---

## 5. Nginx Configuration

### Create Config File

```bash
sudo nano /etc/nginx/sites-available/project
```

### Full Working Nginx Config

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name yourdomain.com www.yourdomain.com;

    # ─── Frontend (React Build) ───────────────────────────────────────────────
    root /var/www/project;
    index index.html;

    # Handle React Router — always return index.html for unknown routes
    location / {
        try_files $uri $uri/ /index.html;
    }

    # ─── Backend API (Reverse Proxy) ─────────────────────────────────────────
    location /api/ {
        proxy_pass         http://localhost:5000;
        proxy_http_version 1.1;

        proxy_set_header   Upgrade           $http_upgrade;
        proxy_set_header   Connection        'upgrade';
        proxy_set_header   Host              $host;
        proxy_set_header   X-Real-IP         $remote_addr;
        proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;

        proxy_cache_bypass $http_upgrade;
        proxy_read_timeout 90;
    }

    # ─── Static Asset Caching ─────────────────────────────────────────────────
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        access_log off;
    }

    # ─── Security Headers ─────────────────────────────────────────────────────
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
    add_header X-XSS-Protection "1; mode=block";

    # ─── Logging ──────────────────────────────────────────────────────────────
    access_log /var/log/nginx/project-access.log;
    error_log  /var/log/nginx/project-error.log;
}
```

> ✅ Replace `yourdomain.com` with your actual domain throughout this file.

### Enable Site & Remove Default

```bash
# Enable your site
sudo ln -s /etc/nginx/sites-available/project /etc/nginx/sites-enabled/

# Remove default Nginx page
sudo rm -f /etc/nginx/sites-enabled/default
```

---

## 6. Test & Restart Nginx

```bash
# Test config syntax
sudo nginx -t

# Expected output:
# nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
# nginx: configuration file /etc/nginx/nginx.conf test is successful

# Reload Nginx (no downtime)
sudo systemctl reload nginx

# Or full restart
sudo systemctl restart nginx

# Verify Nginx is running
sudo systemctl status nginx
```

> ⚠️ Never restart Nginx without running `nginx -t` first — a bad config will take the server down.

---

## 7. Domain Setup

Point your domain to your VPS at your DNS registrar (Namecheap, GoDaddy, Cloudflare, etc.):

| Record Type | Host | Value | TTL |
|---|---|---|---|
| `A` | `@` | `YOUR_VPS_IP` | Auto |
| `A` | `www` | `YOUR_VPS_IP` | Auto |

```bash
# Verify DNS is propagated (run from your local machine)
nslookup yourdomain.com

# Or
dig yourdomain.com +short
```

> ⚠️ DNS propagation can take **up to 24–48 hours**, though usually much faster.

---

## 8. HTTPS Setup (Let's Encrypt)

```bash
# Install Certbot + Nginx plugin
sudo apt install -y certbot python3-certbot-nginx

# Generate SSL certificate and auto-configure Nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Follow the prompts:
# - Enter your email
# - Agree to terms
# - Choose redirect HTTP → HTTPS (select option 2)
```

### Verify Auto-Renewal

```bash
# Test renewal dry run
sudo certbot renew --dry-run

# Check renewal timer (auto-runs twice daily)
sudo systemctl status certbot.timer
```

### After Certbot Runs — Your Nginx Config Will Become:

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    return 301 https://$host$request_uri;   # ← Added by Certbot
}

server {
    listen 443 ssl;
    server_name yourdomain.com www.yourdomain.com;

    ssl_certificate     /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
    include             /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam         /etc/letsencrypt/ssl-dhparams.pem;

    root  /var/www/project;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass         http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade           $http_upgrade;
        proxy_set_header   Connection        'upgrade';
        proxy_set_header   Host              $host;
        proxy_set_header   X-Real-IP         $remote_addr;
        proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    access_log /var/log/nginx/project-access.log;
    error_log  /var/log/nginx/project-error.log;
}
```

---

## 9. Final Testing

```bash
# ── Test Frontend ─────────────────────────────────────────
curl -I https://yourdomain.com
# Expected: HTTP/2 200

# ── Test API via Nginx Reverse Proxy ──────────────────────
curl https://yourdomain.com/api/health
# Expected: {"status":"ok"} or your health endpoint response

# ── Test HTTP Redirect to HTTPS ───────────────────────────
curl -I http://yourdomain.com
# Expected: HTTP/1.1 301 Moved Permanently
# Location: https://yourdomain.com/

# ── Check Backend is Running ──────────────────────────────
pm2 list
pm2 logs mern-backend --lines 20

# ── Check Nginx ───────────────────────────────────────────
sudo systemctl status nginx
sudo tail -f /var/log/nginx/project-error.log
```

---

## 10. Useful Commands Cheat Sheet

### PM2 Commands

```bash
pm2 list                              # List all processes
pm2 start index.js --name "api"       # Start app
pm2 restart mern-backend              # Restart app
pm2 restart mern-backend --update-env # Restart + reload .env
pm2 reload mern-backend               # Zero-downtime reload
pm2 stop mern-backend                 # Stop app
pm2 delete mern-backend               # Remove from PM2
pm2 logs mern-backend                 # Stream logs
pm2 logs mern-backend --lines 100     # Last 100 log lines
pm2 flush                             # Clear all logs
pm2 monit                             # Real-time dashboard
pm2 save                              # Save process list
pm2 startup                           # Generate boot script
pm2 resurrect                         # Restore saved processes
```

### Nginx Commands

```bash
sudo nginx -t                         # Test config syntax
sudo systemctl reload nginx           # Reload (no downtime)
sudo systemctl restart nginx          # Full restart
sudo systemctl status nginx           # Check status
sudo systemctl enable nginx           # Enable on boot
sudo systemctl disable nginx          # Disable on boot
sudo nginx -s quit                    # Graceful shutdown
```

### Logs Debugging

```bash
# Nginx logs
sudo tail -f /var/log/nginx/project-error.log
sudo tail -f /var/log/nginx/project-access.log

# PM2 logs
pm2 logs mern-backend --err --lines 50    # Errors only
pm2 logs mern-backend --out --lines 50    # Output only
tail -f ~/.pm2/logs/mern-backend-error.log

# System journal (Nginx)
sudo journalctl -u nginx -f

# Check what's listening on port 5000
sudo lsof -i :5000
sudo ss -tlnp | grep 5000
```

### Re-deploy After Code Changes

```bash
# Pull latest code
cd /home/ubuntu/app
git pull origin main

# Backend update
cd server
npm install
pm2 restart mern-backend --update-env

# Frontend update
cd ../client
npm install
npm run build
sudo rm -rf /var/www/project/*
sudo cp -r build/* /var/www/project/
sudo chown -R www-data:www-data /var/www/project

# Reload Nginx
sudo systemctl reload nginx
```

---

> ✅ **SSL Certificate** auto-renews every 90 days via `certbot.timer` — no manual action needed.
> ⚠️ After every `pm2 start/delete`, run `pm2 save` to persist changes across reboots.
