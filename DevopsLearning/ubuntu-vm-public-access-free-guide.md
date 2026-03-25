# 🌐 Make Ubuntu VM Public Using Cloudflare Tunnel (Free Guide)

> **A complete beginner-friendly guide to exposing your local Ubuntu VM to the internet — for free, without port forwarding or a static IP.**

---

## 📋 Table of Contents

1. [Requirements](#requirements)
2. [Step 1 — Check Local Service](#step-1--check-local-service)
3. [Step 2 — Install Cloudflare Tunnel](#step-2--install-cloudflare-tunnel)
4. [Step 3 — Start the Tunnel](#step-3--start-the-tunnel)
5. [Step 4 — Keep Tunnel Running](#step-4--keep-tunnel-running)
6. [Step 5 — Optional: Custom Domain (Free)](#step-5--optional-custom-domain-free)
7. [Step 6 — Security Best Practices](#step-6--security-best-practices)
8. [Troubleshooting](#troubleshooting)
9. [Architecture Diagram](#architecture-diagram)
10. [Advantages](#advantages)
11. [Conclusion](#conclusion)
12. [🆓 All Other Free Methods to Make Your VM Public](#-all-other-free-methods-to-make-your-vm-public)

---

## ✅ Requirements

Before you begin, make sure you have the following:

| Requirement | Details |
|---|---|
| 🖥️ Ubuntu VM | VirtualBox / VMware / KVM — any Ubuntu 20.04+ |
| 🌐 Internet Connection | Active and stable connection inside the VM |
| 💻 Basic Terminal Knowledge | Ability to run `sudo` commands |
| ⚙️ A Running Service | Node.js app, Nginx, Apache, Python server, etc. |
| 📧 Cloudflare Account | Free at [cloudflare.com](https://cloudflare.com) (only needed for custom domain) |

> 💡 **No static IP, no router access, and no port forwarding required!**

---

## Step 1 — Check Local Service

Before exposing your VM to the internet, confirm your local application is actually running and accessible inside the VM.

### 🔍 Find Your VM's Local IP Address

```bash
ip a
```

**Example output:**
```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP>
    inet 192.168.1.105/24 brd 192.168.1.255 scope global dynamic enp0s3
```

📌 Note down your IP — in this example it's `192.168.1.105`.

---

### 🚀 Start a Quick Test Server (if you don't have one)

**Using Python (built-in):**
```bash
# Serve current directory on port 3000
python3 -m http.server 3000
```

**Using Node.js:**
```bash
# Simple one-liner HTTP server
npx serve . -p 3000
```

**Using Nginx:**
```bash
sudo systemctl start nginx
# Nginx runs on port 80 by default
```

---

### 🧪 Verify the Service is Running

```bash
# Test via localhost
curl http://localhost:3000

# Test via your VM's IP address
curl http://192.168.1.105:3000
```

✅ **Expected output:** HTML content, JSON response, or a success message from your app.

❌ **If you get `Connection refused`:** Your service is not running. Start it first before proceeding.

```bash
# Check what's listening on port 3000
sudo ss -tlnp | grep 3000

# Or check all listening ports
sudo ss -tlnp
```

---

## Step 2 — Install Cloudflare Tunnel

Cloudflare Tunnel (`cloudflared`) creates a secure outbound connection from your VM to Cloudflare's edge network — **no inbound firewall rules needed**.

### 📥 Download and Install

```bash
# Download the latest cloudflared .deb package
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb

# Install the package
sudo dpkg -i cloudflared-linux-amd64.deb
```

> 💡 **What this does:** `wget` downloads the installer. `dpkg -i` installs it as a system package.

---

### ✅ Verify Installation

```bash
cloudflared --version
```

**Expected output:**
```
cloudflared version 2024.x.x (built ...)
```

If the command is not found, try:
```bash
# Add to PATH manually if needed
export PATH=$PATH:/usr/local/bin
```

---

## Step 3 — Start the Tunnel

This is the magic step! One command creates a public URL that routes directly to your local service.

### 🚀 Launch the Tunnel

```bash
cloudflared tunnel --url http://localhost:3000
```

> 🔄 Replace `3000` with your actual port number (e.g., `80` for Nginx, `8080`, `5000`, etc.)

---

### 📡 What You'll See in the Output

```
2024-01-15T10:23:01Z INF Thank you for trying Cloudflare Tunnel.
2024-01-15T10:23:01Z INF Registered tunnel connection connIndex=0
2024-01-15T10:23:02Z INF +-------------------------------------------------------+
2024-01-15T10:23:02Z INF |  Your quick Tunnel has been created! Visit it at     |
2024-01-15T10:23:02Z INF |  https://random-name.trycloudflare.com                |
2024-01-15T10:23:02Z INF +-------------------------------------------------------+
```

🌍 **Your public URL:** `https://random-name.trycloudflare.com`

This URL is:
- ✅ Accessible from **anywhere in the world**
- ✅ Works on **mobile phones, tablets, other computers**
- ✅ Uses **HTTPS by default** (secure!)
- ⚠️ **Temporary** — a new URL is generated each time you restart (use Step 5 for a permanent URL)

---

### 🧪 Test It

Open the URL in your browser, or test with curl from another machine:

```bash
curl https://random-name.trycloudflare.com
```

---

## Step 4 — Keep Tunnel Running

By default, if you close your terminal, the tunnel stops. Here are three ways to keep it running permanently.

---

### Option A: Using `screen` (Simple)

```bash
# Install screen
sudo apt install screen -y

# Start a new screen session named "tunnel"
screen -S tunnel

# Inside the screen session, start the tunnel
cloudflared tunnel --url http://localhost:3000

# Detach from screen without stopping it
# Press: Ctrl + A, then D

# Later, reattach to the session
screen -r tunnel
```

---

### Option B: Using `tmux` (Better)

```bash
# Install tmux
sudo apt install tmux -y

# Create a new tmux session
tmux new -s tunnel

# Start the tunnel
cloudflared tunnel --url http://localhost:3000

# Detach from tmux
# Press: Ctrl + B, then D

# Reattach later
tmux attach -t tunnel
```

---

### Option C: Using `systemd` Service (Best — Auto-starts on Boot) ⭐

This is the **recommended production approach** — the tunnel automatically restarts if it crashes or if the VM reboots.

**Step 1:** Create the service file:
```bash
sudo nano /etc/systemd/system/cloudflared-tunnel.service
```

**Step 2:** Paste this content:
```ini
[Unit]
Description=Cloudflare Tunnel
After=network.target
Wants=network-online.target

[Service]
Type=simple
User=ubuntu
ExecStart=/usr/local/bin/cloudflared tunnel --url http://localhost:3000
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

> 📝 Replace `ubuntu` with your actual username (check with `whoami`).
> 📝 Replace `3000` with your actual port.

**Step 3:** Enable and start the service:
```bash
# Reload systemd to pick up the new service
sudo systemctl daemon-reload

# Enable auto-start on boot
sudo systemctl enable cloudflared-tunnel

# Start the service now
sudo systemctl start cloudflared-tunnel

# Check its status
sudo systemctl status cloudflared-tunnel
```

**Useful management commands:**
```bash
# View live logs
sudo journalctl -u cloudflared-tunnel -f

# Stop the tunnel
sudo systemctl stop cloudflared-tunnel

# Restart after config changes
sudo systemctl restart cloudflared-tunnel
```

---

## Step 5 — Optional: Custom Domain (Free)

Want a permanent URL like `app.yourdomain.com` instead of a random `trycloudflare.com` URL?

### 🌐 Prerequisites

- A domain name (can be free from [Freenom](https://freenom.com) or very cheap from Cloudflare)
- A **free Cloudflare account** at [cloudflare.com](https://cloudflare.com)
- Your domain's DNS managed by Cloudflare

---

### 🔧 Setup Steps

**Step 1:** Log in to Cloudflare and authenticate `cloudflared`:
```bash
cloudflared tunnel login
```

This opens a browser. Select your domain and authorize it.

**Step 2:** Create a named tunnel:
```bash
cloudflared tunnel create my-vm-tunnel
```

This creates a tunnel and stores credentials in `~/.cloudflared/`.

**Step 3:** Create a config file:
```bash
nano ~/.cloudflared/config.yml
```

Paste:
```yaml
tunnel: my-vm-tunnel
credentials-file: /home/ubuntu/.cloudflared/<YOUR-TUNNEL-ID>.json

ingress:
  - hostname: app.example.com
    service: http://localhost:3000
  - service: http_status:404
```

> 📝 Replace `app.example.com` with your actual subdomain.
> 📝 Replace `<YOUR-TUNNEL-ID>` with the UUID shown after `cloudflared tunnel create`.

**Step 4:** Create the DNS record:
```bash
cloudflared tunnel route dns my-vm-tunnel app.example.com
```

**Step 5:** Start the tunnel using the config:
```bash
cloudflared tunnel run my-vm-tunnel
```

🎉 Your app is now available at `https://app.example.com` — **permanently and for free!**

---

## Step 6 — Security Best Practices

Exposing your VM to the internet requires caution. Follow these practices:

### 🔒 1. Always Use HTTPS

Cloudflare Tunnel provides HTTPS by default. Never use plain `http://` tunnels in production.

```bash
# ✅ Good
cloudflared tunnel --url http://localhost:3000
# (Cloudflare handles HTTPS on the public side automatically)
```

---

### 🛡️ 2. Configure UFW Firewall

Only allow traffic you need:

```bash
# Enable firewall
sudo ufw enable

# Allow SSH (so you don't lock yourself out!)
sudo ufw allow ssh

# Block direct external access to your app port
# (Cloudflare handles routing — no need to expose it directly)
sudo ufw deny 3000

# Check status
sudo ufw status verbose
```

---

### 🔑 3. Add Authentication with Cloudflare Access (Free)

Protect your app with email-based login — no passwords needed:

1. Go to **Cloudflare Zero Trust Dashboard** → [one.dash.cloudflare.com](https://one.dash.cloudflare.com)
2. Navigate to **Access → Applications → Add an Application**
3. Choose **Self-hosted**
4. Set your domain (`app.example.com`) and configure allowed email addresses

Now only approved emails can access your app — entirely free for up to 50 users.

---

### ⏱️ 4. Rate Limiting

In your config file, you can add rate limiting rules via Cloudflare WAF (free tier includes basic rules):

```yaml
# In config.yml — add headers to help identify real traffic
ingress:
  - hostname: app.example.com
    service: http://localhost:3000
    originRequest:
      connectTimeout: 30s
      noTLSVerify: false
```

For advanced rate limiting, use Cloudflare's **WAF Rules** in the dashboard under **Security → WAF**.

---

### 🔍 5. Monitor Access Logs

```bash
# Watch cloudflared logs in real time
sudo journalctl -u cloudflared-tunnel -f

# View recent logs
sudo journalctl -u cloudflared-tunnel --since "1 hour ago"
```

---

## 🔧 Troubleshooting

### ❌ Issue: "Connection refused" when running curl

**Cause:** Your local service is not running.

```bash
# Check if service is running on port 3000
sudo ss -tlnp | grep 3000

# Start your service first
node app.js &          # For Node.js
python3 -m http.server 3000 &   # For Python
sudo systemctl start nginx      # For Nginx
```

---

### ❌ Issue: Port is blocked

**Cause:** UFW or iptables is blocking the port.

```bash
# Check firewall status
sudo ufw status

# Temporarily allow the port for testing
sudo ufw allow 3000

# Check iptables rules
sudo iptables -L -n
```

---

### ❌ Issue: Tunnel keeps disconnecting

**Cause:** Network instability or resource limits.

```bash
# Use systemd with auto-restart (see Step 4, Option C)
# Or increase keepalive settings:
cloudflared tunnel --url http://localhost:3000 \
  --heartbeat-interval 5s \
  --retries 10
```

---

### ❌ Issue: "Failed to fetch" in cloudflared

**Cause:** DNS resolution issues or outbound firewall.

```bash
# Test outbound connectivity
curl -v https://cloudflare.com

# Check DNS resolution
nslookup cloudflare.com

# If using a corporate network, try a different DNS
sudo nano /etc/resolv.conf
# Add: nameserver 1.1.1.1
```

---

### ❌ Issue: "cloudflared: command not found"

```bash
# Verify installation path
which cloudflared
ls /usr/local/bin/cloudflared

# Reinstall if missing
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb
```

---

### ❌ Issue: systemd service fails to start

```bash
# Check detailed error
sudo systemctl status cloudflared-tunnel
sudo journalctl -xe -u cloudflared-tunnel

# Common fix: wrong username in service file
whoami   # Get your actual username
sudo nano /etc/systemd/system/cloudflared-tunnel.service
# Update User= to match your actual username
sudo systemctl daemon-reload
sudo systemctl restart cloudflared-tunnel
```

---

## 🗺️ Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                    HOW CLOUDFLARE TUNNEL WORKS                      │
└─────────────────────────────────────────────────────────────────────┘

  👤 User (anywhere)
       │
       │  HTTPS Request to:
       │  https://random-name.trycloudflare.com
       │
       ▼
  ┌─────────────────────────────────────┐
  │        Cloudflare Edge Network      │
  │   (Global CDN + DDoS Protection)   │
  └─────────────────────────────────────┘
       │
       │  Encrypted outbound tunnel
       │  (initiated FROM your VM, not TO it)
       │
       ▼
  ┌───────────────────────────────────────────────┐
  │              Your Ubuntu VM                   │
  │  ┌─────────────────┐   ┌───────────────────┐  │
  │  │  cloudflared    │──▶│  Your Application │  │
  │  │  (tunnel agent) │   │  localhost:3000   │  │
  │  └─────────────────┘   └───────────────────┘  │
  └───────────────────────────────────────────────┘

  ✅ No port forwarding needed
  ✅ No inbound firewall rules
  ✅ Works behind NAT / CGNAT
  ✅ Works on mobile hotspot
  ✅ Encrypted end-to-end
```

---

## 🌟 Advantages

| Feature | Cloudflare Tunnel | Traditional Port Forwarding |
|---|---|---|
| 💰 Cost | **Free** | Free (but needs router access) |
| 🔒 HTTPS | **Automatic** | Manual SSL setup |
| 🛡️ DDoS Protection | **Yes (Cloudflare)** | No |
| 🌍 Works behind NAT | **Yes** | Depends on ISP |
| 📱 Works on mobile hotspot | **Yes** | No |
| 🔧 Router access needed | **No** | Yes |
| 📍 Static IP needed | **No** | Recommended |
| ⚡ Setup time | **~5 minutes** | 15–30 minutes |

---

## 🏁 Conclusion

Cloudflare Tunnel is one of the most powerful free tools available for developers who need to expose local services. Here's when to use it:

| Use Case | Recommendation |
|---|---|
| 🧪 **Development** | ✅ Perfect — test webhooks, share work-in-progress |
| 🔬 **Testing** | ✅ Perfect — show clients, QA teams, stakeholders |
| 🎯 **Demo** | ✅ Perfect — live demos without complex setup |
| 🏭 **Small Production** | ✅ Good — personal projects, internal tools |
| 🏢 **Large Production** | ⚠️ Consider paid plans for SLAs and advanced features |

> 💡 **Pro tip:** For anything beyond personal/small projects, consider upgrading to Cloudflare's paid plans or deploying to a proper cloud provider for better reliability guarantees.

---
---

# 🆓 All Other Free Methods to Make Your Ubuntu VM Publicly Accessible

Here is a complete reference of **every free method** available — how they work, their pros/cons, and exact setup commands.

---

## 📊 Quick Comparison Table

| Method | Difficulty | Speed | Permanent URL | Self-hosted | Best For |
|---|---|---|---|---|---|
| Cloudflare Tunnel | ⭐ Easy | ⚡ Fast | ✅ (with domain) | No | Production/Dev |
| ngrok | ⭐ Easy | ⚡ Fast | ❌ (paid) | No | Quick testing |
| Serveo | ⭐ Easy | 🔄 Medium | ✅ (custom subdomain) | No | SSH/HTTP |
| LocalTunnel | ⭐ Easy | 🔄 Medium | ✅ (custom subdomain) | No | Node.js devs |
| Bore | ⭐ Easy | ⚡ Fast | ❌ | Can self-host | Privacy-focused |
| FRP | ⭐⭐ Medium | ⚡ Fast | ✅ | Yes (needs VPS) | Power users |
| WireGuard VPN | ⭐⭐⭐ Hard | ⚡ Fast | ✅ | Yes (needs VPS) | Privacy + full control |
| Tailscale | ⭐ Easy | ⚡ Fast | ✅ (internal) | No | Team access |
| ZeroTier | ⭐ Easy | ⚡ Fast | ✅ (internal) | No | Team access |
| PlayIt.gg | ⭐ Easy | ⚡ Fast | ✅ | No | Game servers |
| SSH Reverse Tunnel | ⭐⭐ Medium | ⚡ Fast | ✅ | Yes (needs VPS) | Advanced users |
| Oracle Free VPS | ⭐⭐⭐ Hard | ⚡ Fast | ✅ (public IP) | Yes | Full control |

---

## 🔵 Method 1: ngrok (Most Popular for Development)

**What it is:** The original tunneling tool. Creates instant public URLs for any local port.

**Free tier limits:** 1 tunnel, 1 domain (random), rate limited to ~40 req/min.

### Setup

```bash
# Step 1: Create free account at https://ngrok.com
# Then get your authtoken from the dashboard

# Step 2: Download ngrok
wget https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-linux-amd64.tgz
tar xvzf ngrok-v3-stable-linux-amd64.tgz
sudo mv ngrok /usr/local/bin/

# Step 3: Authenticate
ngrok config add-authtoken YOUR_AUTHTOKEN_HERE

# Step 4: Start tunnel
ngrok http 3000
```

**Output:**
```
Forwarding    https://abc123.ngrok-free.app -> http://localhost:3000
```

### Keep Running with systemd

```bash
sudo nano /etc/systemd/system/ngrok.service
```

```ini
[Unit]
Description=ngrok tunnel
After=network.target

[Service]
ExecStart=/usr/local/bin/ngrok http 3000 --log=stdout
Restart=always
User=ubuntu

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable --now ngrok
```

> ✅ **Best for:** Quick demos, webhook testing, sharing work with teammates.

---

## 🟢 Method 2: Serveo (Zero Install)

**What it is:** A completely free SSH-based tunneling service. No software to install — just SSH!

**Free tier:** Unlimited usage, custom subdomains, no account needed.

### Setup

```bash
# Expose local port 3000 with a random subdomain
ssh -R 80:localhost:3000 serveo.net

# Request a specific subdomain
ssh -R myapp:80:localhost:3000 serveo.net
# Your app is now at: https://myapp.serveo.net
```

### Keep Running (Persistent SSH Tunnel)

```bash
# Install autossh for automatic reconnection
sudo apt install autossh -y

# Run with autossh
autossh -M 0 -o "ServerAliveInterval 30" -o "ServerAliveCountMax 3" \
  -R myapp:80:localhost:3000 serveo.net
```

### systemd Service for Serveo

```bash
sudo nano /etc/systemd/system/serveo.service
```

```ini
[Unit]
Description=Serveo SSH Tunnel
After=network.target

[Service]
ExecStart=/usr/bin/autossh -M 0 -o "StrictHostKeyChecking=no" \
  -o "ServerAliveInterval 30" -R myapp:80:localhost:3000 serveo.net
Restart=always
User=ubuntu

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable --now serveo
```

> ✅ **Best for:** Quick sharing, no-install environments, SSH tunnels.
> ⚠️ **Note:** Serveo can be unreliable sometimes. Have a backup plan.

---

## 🟡 Method 3: LocalTunnel (`lt`)

**What it is:** Open-source tunnel tool, runs via Node.js/npm. Free and easy.

**Free tier:** Unlimited, custom subdomains.

### Setup

```bash
# Install Node.js if not already installed
sudo apt install nodejs npm -y

# Install localtunnel globally
sudo npm install -g localtunnel

# Start the tunnel
lt --port 3000

# With a custom subdomain
lt --port 3000 --subdomain myapp
# Your app: https://myapp.loca.lt
```

### Self-host LocalTunnel (Optional)

If you have a VPS, you can run your own LocalTunnel server:

```bash
git clone https://github.com/localtunnel/server.git
cd server
npm install
DOMAIN=tunnel.yourdomain.com node index.js
```

> ✅ **Best for:** Node.js developers, quick sharing.
> ⚠️ **Note:** Visitors see a landing page and must click a button first (can be bypassed with auth headers).

---

## 🔴 Method 4: Bore (Lightweight & Open Source)

**What it is:** A minimal Rust-based tunneling tool. Fast, lightweight, and can be self-hosted.

**Free tier:** Use the public `bore.pub` server free.

### Setup

```bash
# Install Rust first (if not installed)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source ~/.cargo/env

# Install bore
cargo install bore-cli

# Start tunnel (exposes localhost:3000 to bore.pub)
bore local 3000 --to bore.pub
```

**Output:**
```
2024-01-15T10:00:00Z  INFO  listening at bore.pub:PORT_NUMBER
```

Access via: `http://bore.pub:PORT_NUMBER`

### Self-host Bore on a Free Oracle VPS

```bash
# On your VPS (Oracle Free Tier):
cargo install bore-cli
bore server --min-port 1024

# On your local VM:
bore local 3000 --to your-vps-ip.compute.oracle.com
```

> ✅ **Best for:** Privacy-focused users, self-hosting enthusiasts.

---

## 🟣 Method 5: FRP (Fast Reverse Proxy) — Needs a Free VPS

**What it is:** A powerful reverse proxy tool. Requires a VPS (use Oracle Free Tier — free forever).

**Why it's "free":** Oracle Cloud gives you 2 free VMs forever with public IPs.

### Part A: Get a Free Oracle Cloud VPS

1. Sign up at [cloud.oracle.com](https://cloud.oracle.com) — Free Tier (always free, no credit card charges after trial)
2. Create a **Compute Instance**: AMD or Ampere (ARM), Ubuntu 22.04
3. Note the **public IP address**

### Part B: Install FRP on the VPS (Server)

```bash
# On your Oracle VPS:
wget https://github.com/fatedier/frp/releases/download/v0.54.0/frp_0.54.0_linux_amd64.tar.gz
tar xvf frp_0.54.0_linux_amd64.tar.gz
cd frp_0.54.0_linux_amd64

# Configure the server
cat > frps.ini << 'EOF'
[common]
bind_port = 7000
vhost_http_port = 8080
EOF

# Start FRP server
./frps -c frps.ini
```

### Part C: Install FRP on Your Ubuntu VM (Client)

```bash
# On your local Ubuntu VM:
wget https://github.com/fatedier/frp/releases/download/v0.54.0/frp_0.54.0_linux_amd64.tar.gz
tar xvf frp_0.54.0_linux_amd64.tar.gz
cd frp_0.54.0_linux_amd64

# Configure the client
cat > frpc.ini << 'EOF'
[common]
server_addr = YOUR_ORACLE_VPS_IP
server_port = 7000

[myapp]
type = http
local_port = 3000
custom_domains = YOUR_ORACLE_VPS_IP
EOF

# Start FRP client
./frpc -c frpc.ini
```

Access your app at: `http://YOUR_ORACLE_VPS_IP:8080`

> ✅ **Best for:** Maximum control, high performance, permanent setup.

---

## 🔵 Method 6: SSH Reverse Tunnel (Manual — Needs a Free VPS)

**What it is:** Pure SSH-based tunneling. No additional software needed.

### Setup

```bash
# On your Ubuntu VM, create a reverse tunnel to your VPS
ssh -N -R 0.0.0.0:8080:localhost:3000 user@YOUR_VPS_IP

# On the VPS, enable GatewayPorts in SSH config
sudo nano /etc/ssh/sshd_config
# Add/change: GatewayPorts yes
sudo systemctl restart sshd
```

Now anyone accessing `http://YOUR_VPS_IP:8080` reaches your local VM.

### Make It Permanent with autossh

```bash
sudo apt install autossh -y

# Create systemd service
sudo nano /etc/systemd/system/ssh-tunnel.service
```

```ini
[Unit]
Description=SSH Reverse Tunnel
After=network.target

[Service]
ExecStart=/usr/bin/autossh -M 0 -N \
  -o "ServerAliveInterval=30" \
  -o "ServerAliveCountMax=3" \
  -o "StrictHostKeyChecking=no" \
  -i /home/ubuntu/.ssh/id_rsa \
  -R 0.0.0.0:8080:localhost:3000 \
  user@YOUR_VPS_IP
Restart=always
User=ubuntu

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable --now ssh-tunnel
```

> ✅ **Best for:** Advanced users who want full control with no third-party services.

---

## 🟢 Method 7: Tailscale (Free for Personal Use)

**What it is:** A zero-config VPN mesh network. Creates a secure private network between all your devices — including your VM.

**Free tier:** Up to 100 devices, 3 users — completely free forever.

### Setup

```bash
# Install Tailscale on your Ubuntu VM
curl -fsSL https://tailscale.com/install.sh | sh

# Start and authenticate
sudo tailscale up
# Opens a browser — log in with Google/GitHub/Microsoft

# Get your VM's Tailscale IP
tailscale ip -4
# Example: 100.64.0.5
```

Now anyone on your Tailscale network (including your phone) can access:
`http://100.64.0.5:3000`

### Share with External Users (Tailscale Share)

```bash
# Share a specific node with an external user
# Go to: https://login.tailscale.com/admin/machines
# Click your machine → Share → Enter their email
```

> ✅ **Best for:** Team access, secure internal tools, accessing VM from your phone.
> ⚠️ **Note:** External users also need a Tailscale account.

---

## 🟡 Method 8: ZeroTier (Free Mesh VPN)

**What it is:** Similar to Tailscale — creates a virtual LAN across the internet.

**Free tier:** Up to 25 devices, unlimited networks.

### Setup

```bash
# Install ZeroTier
curl -s https://install.zerotier.com | sudo bash

# Join a network (create one at https://my.zerotier.com)
sudo zerotier-cli join YOUR_NETWORK_ID

# Check your ZeroTier IP
sudo zerotier-cli listnetworks
```

Approve your device in the ZeroTier web dashboard, then access via the assigned IP.

> ✅ **Best for:** Private team networks, game servers, file sharing.

---

## 🎮 Method 9: PlayIt.gg (Great for Game Servers & Web Apps)

**What it is:** Free tunnel service originally for Minecraft/game servers but works for any TCP/UDP/HTTP service.

**Free tier:** 1 tunnel, no rate limits, permanent subdomain.

### Setup

```bash
# Download PlayIt agent
wget https://github.com/playit-cloud/playit-agent/releases/latest/download/playit-linux-amd64
chmod +x playit-linux-amd64
sudo mv playit-linux-amd64 /usr/local/bin/playit

# Run and follow setup instructions
playit
# Opens a URL — create account and claim tunnel
```

> ✅ **Best for:** Game servers, persistent tunnels, TCP/UDP protocols.

---

## ☁️ Method 10: Oracle Cloud Free Tier (Full Public IP — Best Long-term Solution)

**What it is:** Oracle gives you **2 free VMs forever** (always free, not a trial) with real public IP addresses. Deploy your app there directly instead of tunneling.

### Free Resources You Get

| Resource | Amount |
|---|---|
| AMD VM instances | 2 (1/8 OCPU, 1 GB RAM each) |
| ARM Ampere instances | Up to 4 OCPUs, 24 GB RAM |
| Storage | 200 GB block storage |
| Data transfer | 10 TB/month outbound |
| Public IPs | Included |

### Quick Setup

1. Sign up at [cloud.oracle.com](https://cloud.oracle.com) (requires credit card but **will NOT be charged** for Always Free resources)
2. Create Instance → Choose **Ubuntu 22.04** → Pick **Always Free eligible** shape
3. Download SSH key pair
4. SSH into your new server with a real public IP:

```bash
ssh -i ~/Downloads/ssh-key.key ubuntu@YOUR_PUBLIC_IP

# Now deploy your app directly — no tunneling needed!
```

> ✅ **Best for:** Permanent hosting, production apps, when you want a real server with a real IP.

---

## 🔥 Summary: Which Method Should You Use?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DECISION GUIDE                                    │
└─────────────────────────────────────────────────────────────────────┘

 Need it in < 1 minute with no account?
 └─▶ Serveo (ssh -R 80:localhost:3000 serveo.net)

 Need it quickly and reliably with free account?
 └─▶ Cloudflare Tunnel (best choice for most users)
 └─▶ ngrok (great for dev/testing)

 Node.js developer?
 └─▶ LocalTunnel (npm install -g localtunnel)

 Need a permanent URL for free?
 └─▶ Cloudflare Tunnel + custom domain
 └─▶ PlayIt.gg

 Need team/private access only?
 └─▶ Tailscale (easiest)
 └─▶ ZeroTier (more control)

 Want maximum control & performance?
 └─▶ FRP + Oracle Free VPS
 └─▶ SSH Reverse Tunnel + Oracle Free VPS

 Want to skip tunneling entirely?
 └─▶ Oracle Cloud Free Tier (deploy app directly)

 Game server or TCP/UDP?
 └─▶ PlayIt.gg
 └─▶ FRP
```

---

## 🔐 Universal Security Reminder

Regardless of which method you use:

```bash
# 1. Always keep your system updated
sudo apt update && sudo apt upgrade -y

# 2. Disable root SSH login
sudo sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sudo systemctl restart sshd

# 3. Enable firewall
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https

# 4. Monitor who's accessing your service
sudo tail -f /var/log/auth.log      # SSH login attempts
sudo tail -f /var/log/nginx/access.log  # Web requests (if using Nginx)

# 5. Set up fail2ban to block brute-force attacks
sudo apt install fail2ban -y
sudo systemctl enable --now fail2ban
```

---

*📅 Guide last updated: 2025 | All methods verified on Ubuntu 20.04+ | All methods free as of writing*
