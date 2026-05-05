# PM2 — Complete Documentation for Node.js Process Manager

> **Version:** Covers PM2 v5.x | **Audience:** Beginners to Intermediate | **Platform:** Linux / macOS / Windows

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Installation](#2-installation)
3. [Basic Commands](#3-basic-commands)
4. [Process Management](#4-process-management)
5. [Logs Management](#5-logs-management)
6. [Monitoring](#6-monitoring)
7. [Environment Variables](#7-environment-variables)
8. [Ecosystem File](#8-ecosystem-file)
9. [Auto Start & Persistence](#9-auto-start--persistence)
10. [Advanced Commands](#10-advanced-commands)
11. [Log Rotation](#11-log-rotation)
12. [Debugging](#12-debugging)
13. [Best Practices](#13-best-practices)
14. [Cheat Sheet](#14-cheat-sheet)

---

## 1. Introduction

### What is PM2?

PM2 (Process Manager 2) is a **production-grade process manager for Node.js** applications. It keeps your apps alive forever, reloads them without downtime, and helps you manage application logs and monitoring — all from the command line.

### Why Use PM2?

- ✅ Automatically **restarts** your app if it crashes
- ✅ Manages **multiple apps** from one place
- ✅ Built-in **log management**
- ✅ Supports **cluster mode** for multi-core CPU usage
- ✅ Handles **environment variables** per environment
- ✅ Survives **server reboots** (via startup scripts)
- ✅ Works great with **VPS, EC2, DigitalOcean**, and similar hosting

---

## 2. Installation

### Install PM2 Globally via npm

```bash
npm install -g pm2
```

### Install Using yarn

```bash
yarn global add pm2
```

### Verify Installation

```bash
pm2 --version
```

Expected output example:

```
5.3.1
```

### Update PM2 to Latest Version

```bash
npm install -g pm2@latest
pm2 update
```

> ⚠️ Always run `pm2 update` after upgrading PM2 to reload the in-memory daemon with the new version.

---

## 3. Basic Commands

### Start an Application

```bash
pm2 start app.js
```

### Start with a Custom Name

```bash
pm2 start app.js --name "my-api"
```

> ✅ Always name your processes — it makes management much easier.

### Start with a Specific Interpreter

```bash
pm2 start app.py --interpreter python3
pm2 start server.sh --interpreter bash
```

### Stop a Process

```bash
pm2 stop my-api
# or by process ID
pm2 stop 0
```

### Restart a Process

```bash
pm2 restart my-api
```

### Delete a Process (Removes from PM2 List)

```bash
pm2 delete my-api
```

### Start Multiple Apps at Once

```bash
pm2 start app1.js --name "frontend"
pm2 start app2.js --name "backend"
pm2 start worker.js --name "queue-worker"
```

### Pass Arguments to Your App

```bash
pm2 start app.js --name "api" -- --port 3000 --env production
```

> ⚠️ The `--` separator is required before passing arguments to your application.

---

## 4. Process Management

### List All Running Processes

```bash
pm2 list
# or
pm2 ls
# or
pm2 status
```

Sample output:

```
┌─────┬──────────────┬─────────────┬─────────┬─────────┬──────────┬────────┬──────┬───────────┐
│ id  │ name         │ namespace   │ version │ mode    │ pid      │ uptime │ ↺    │ status    │
├─────┼──────────────┼─────────────┼─────────┼─────────┼──────────┼────────┼──────┼───────────┤
│ 0   │ my-api       │ default     │ 1.0.0   │ fork    │ 12345    │ 2h     │ 0    │ online    │
└─────┴──────────────┴─────────────┴─────────┴─────────┴──────────┴────────┴──────┴───────────┘
```

### Show Detailed Info for a Process

```bash
pm2 show my-api
# or
pm2 describe my-api
```

This displays:
- Script path and arguments
- Error and output log file paths
- Restart count and uptime
- CPU and memory usage
- Environment variables

### Sort or Filter Process List

```bash
# Sort by name
pm2 list --sort name:asc

# Sort by CPU usage
pm2 list --sort cpu:desc
```

---

## 5. Logs Management

> ⚠️ This section is critical for debugging and production monitoring.

### View Logs for All Apps

```bash
pm2 logs
```

### View Logs for a Specific App

```bash
pm2 logs my-api
```

### View Last N Lines of Logs

```bash
# View last 200 lines
pm2 logs my-api --lines 200

# View last 50 lines for all apps
pm2 logs --lines 50
```

### Log File Location

By default, PM2 stores logs here:

```
~/.pm2/logs/
```

Inside that folder:

```
~/.pm2/logs/my-api-out.log     # Standard output (console.log)
~/.pm2/logs/my-api-error.log   # Error output (console.error, uncaught exceptions)
```

### Difference: Out Log vs Error Log

| Log Type | File | Content |
|---|---|---|
| **Out log** | `<name>-out.log` | `console.log()`, `process.stdout` |
| **Error log** | `<name>-error.log` | `console.error()`, uncaught exceptions, crashes |

### View Error Logs Only

```bash
tail -f ~/.pm2/logs/my-api-error.log
```

### View Output Logs Only

```bash
tail -f ~/.pm2/logs/my-api-out.log
```

### Clear / Flush All Logs

```bash
pm2 flush
```

### Clear Logs for a Specific App

```bash
pm2 flush my-api
```

> ✅ `pm2 flush` empties the log files but does **not** delete them. Log rotation is handled separately (see Section 11).

### Custom Log File Paths

```bash
pm2 start app.js --name "my-api" \
  --output ~/.pm2/logs/my-api-custom-out.log \
  --error ~/.pm2/logs/my-api-custom-err.log
```

### Disable Logging for an App

```bash
pm2 start app.js --name "my-api" --log /dev/null
```

---

## 6. Monitoring

### Real-Time Dashboard

```bash
pm2 monit
```

This opens an **interactive terminal dashboard** showing:

- CPU and memory usage per process
- Log output streams (real-time)
- Restart count and uptime
- Event loop latency

> ✅ `pm2 monit` is your best friend during debugging and performance checks. Press `q` or `Ctrl+C` to exit.

### Web-Based Dashboard (PM2 Plus)

PM2 offers a cloud-based monitoring dashboard at [pm2.io](https://pm2.io). Link your server with:

```bash
pm2 link <secret_key> <public_key>
```

> ⚠️ PM2 Plus is a paid service after the free tier. For most VPS setups, `pm2 monit` is sufficient.

---

## 7. Environment Variables

### How PM2 Handles Environment Variables

PM2 **captures the environment at startup** — it does not reload `.env` files automatically on restart unless you tell it to.

### Method 1: Pass Inline

```bash
pm2 start app.js --name "my-api" --env production
```

### Method 2: Use `--` to Pass Directly

```bash
NODE_ENV=production PORT=3000 pm2 start app.js --name "my-api"
```

### Method 3: Use dotenv in Your App

Install dotenv in your project:

```bash
npm install dotenv
```

Add at the very top of your `app.js`:

```javascript
require('dotenv').config();
```

Then start normally:

```bash
pm2 start app.js --name "my-api"
```

> ✅ Using `dotenv` inside your app is the most reliable method for `.env` files in PM2.

### Update Environment Variables Without Full Restart

```bash
pm2 restart my-api --update-env
```

> ⚠️ Without `--update-env`, restarting PM2 will **not** pick up new `.env` changes.

### Common Mistakes with Environment Variables

| Mistake | Fix |
|---|---|
| Edited `.env` but PM2 didn't pick it up | Run `pm2 restart my-api --update-env` |
| `process.env.PORT` is undefined | Make sure `dotenv.config()` runs before anything else |
| Different env in terminal vs PM2 | Use ecosystem file with explicit `env` block |
| Missing env after server reboot | Define all vars in `ecosystem.config.js` |

---

## 8. Ecosystem File

The ecosystem file is a **configuration file** for defining and managing multiple PM2 processes declaratively.

### Create an Ecosystem File

```bash
pm2 ecosystem
```

This generates `ecosystem.config.js` in the current directory.

### Example: `ecosystem.config.js`

```javascript
module.exports = {
  apps: [
    {
      name: "my-api",
      script: "./src/server.js",
      instances: 2,                    // Number of instances (use "max" for all CPU cores)
      exec_mode: "cluster",            // "fork" (default) or "cluster"
      watch: false,                    // Set to true to auto-restart on file changes
      max_memory_restart: "500M",      // Auto-restart if memory exceeds this
      autorestart: true,               // Restart on crash

      // Default environment
      env: {
        NODE_ENV: "development",
        PORT: 3000,
      },

      // Production environment
      env_production: {
        NODE_ENV: "production",
        PORT: 8080,
      },

      // Log settings
      output: "./logs/out.log",
      error: "./logs/error.log",
      log_date_format: "YYYY-MM-DD HH:mm:ss",

      // Restart delay in ms
      restart_delay: 3000,

      // Ignore watch for these folders
      ignore_watch: ["node_modules", "logs"],
    },

    {
      name: "queue-worker",
      script: "./workers/queue.js",
      instances: 1,
      exec_mode: "fork",
      env: {
        NODE_ENV: "development",
      },
      env_production: {
        NODE_ENV: "production",
      },
    },
  ],
};
```

### Start Using the Ecosystem File

```bash
# Start with default env
pm2 start ecosystem.config.js

# Start with production env
pm2 start ecosystem.config.js --env production
```

### Restart / Reload All Apps in Ecosystem

```bash
pm2 reload ecosystem.config.js --env production
```

> ✅ Use `reload` instead of `restart` in production for **zero-downtime** updates.

---

## 9. Auto Start & Persistence

This is **critical for VPS deployments** — without this, PM2 processes won't survive a server reboot.

### Step 1: Save the Current Process List

```bash
pm2 save
```

This saves your currently running processes to `~/.pm2/dump.pm2`.

### Step 2: Generate and Register the Startup Script

```bash
pm2 startup
```

PM2 will output a command — **copy and run it**. It looks like:

```bash
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu
```

> ⚠️ You must **run the exact command PM2 gives you**. It is user/OS specific.

### Step 3: Verify It Works

```bash
# Check PM2 service is enabled
systemctl status pm2-<your-username>
```

### Full Persistence Workflow

```bash
# 1. Start your apps
pm2 start ecosystem.config.js --env production

# 2. Save the process list
pm2 save

# 3. Register startup (run the output command)
pm2 startup

# 4. Reboot and verify
sudo reboot
# After reboot:
pm2 list
```

> ✅ Every time you **add or remove** a process, run `pm2 save` again to update the saved state.

---

## 10. Advanced Commands

### Zero-Downtime Reload (Cluster Mode Only)

```bash
pm2 reload my-api
```

> ✅ `reload` gracefully restarts workers one by one — no dropped connections. Only works in `cluster` mode.

### Restart All Processes

```bash
pm2 restart all
```

### Delete All Processes

```bash
pm2 delete all
```

### Scale Processes Up or Down (Cluster Mode)

```bash
# Scale to 4 instances
pm2 scale my-api 4

# Scale to max available CPU cores
pm2 scale my-api max

# Scale down to 1 instance
pm2 scale my-api 1
```

### Send Signal to a Process

```bash
pm2 sendSignal SIGUSR2 my-api
```

### Trigger Process-Specific Actions

```bash
# Trigger graceful shutdown
pm2 stop my-api --kill-timeout 5000
```

### Execute a Command in All Processes

```bash
pm2 trigger my-api customEvent
```

### Pull Latest Code and Restart (Deploy)

```bash
pm2 deploy ecosystem.config.js production update
```

> ✅ PM2 deploy is useful for simple CI/CD pipelines on a VPS without full DevOps tooling.

---

## 11. Log Rotation

> ⚠️ Without log rotation, log files grow indefinitely and can fill your disk — especially in production.

### Install pm2-logrotate

```bash
pm2 install pm2-logrotate
```

### Configure Log Rotation

```bash
# Set maximum log file size before rotation (default: 10MB)
pm2 set pm2-logrotate:max_size 50M

# Number of rotated files to retain
pm2 set pm2-logrotate:retain 7

# Enable compression of old log files
pm2 set pm2-logrotate:compress true

# Rotation interval using cron syntax (daily at midnight)
pm2 set pm2-logrotate:rotateInterval '0 0 * * *'

# Date format appended to rotated file names
pm2 set pm2-logrotate:dateFormat YYYY-MM-DD_HH-mm-ss

# Rotate on PM2 restart
pm2 set pm2-logrotate:rotateModule true
```

### View Current pm2-logrotate Config

```bash
pm2 conf pm2-logrotate
```

### Recommended Production Configuration

```bash
pm2 set pm2-logrotate:max_size 100M
pm2 set pm2-logrotate:retain 14
pm2 set pm2-logrotate:compress true
pm2 set pm2-logrotate:rotateInterval '0 0 * * *'
```

> ✅ This keeps 14 days of compressed logs, rotating at midnight daily — solid for most production use cases.

---

## 12. Debugging

### Common Issue 1: Environment Variables Not Updating

**Symptom:** Changed `.env` but app still uses old values after `pm2 restart`.

**Fix:**

```bash
pm2 restart my-api --update-env
```

Or define all env vars in `ecosystem.config.js` and reload:

```bash
pm2 reload ecosystem.config.js --env production
```

---

### Common Issue 2: App Keeps Crashing / Restarting in a Loop

**Symptom:** Restart count is increasing rapidly in `pm2 list`.

**Fix:**

```bash
# Check error logs immediately
pm2 logs my-api --err --lines 100

# Or tail the error file
tail -100 ~/.pm2/logs/my-api-error.log
```

Common causes:
- Missing `node_modules` — run `npm install`
- Wrong `NODE_ENV` — check your env config
- Port already in use — check with `lsof -i :3000`
- Syntax error in code — check the error log

---

### Common Issue 3: Logs Not Showing

**Symptom:** `pm2 logs` shows nothing or old data.

**Fix:**

```bash
# Flush logs and check again
pm2 flush

# Make sure app is actually running
pm2 list

# Confirm log file path
pm2 describe my-api | grep -i log
```

---

### Common Issue 4: PM2 Not Starting on Boot

**Symptom:** After server reboot, PM2 processes are gone.

**Fix:**

```bash
# Check if startup was configured
systemctl status pm2-$USER

# Re-run startup setup
pm2 startup
# Run the command it outputs

# Save current processes
pm2 save
```

---

### Common Issue 5: `pm2 save` Not Working After Reboot

**Symptom:** PM2 starts but no apps are loaded.

**Fix:**

```bash
# Restore from saved dump
pm2 resurrect

# Re-save current state
pm2 save
```

---

### Useful Debugging Commands

```bash
# Full PM2 daemon logs
pm2 logs pm2 --lines 50

# Check PM2 daemon status
pm2 ping

# Kill and restart PM2 daemon
pm2 kill
pm2 resurrect
```

---

## 13. Best Practices

### Naming Conventions

- ✅ Use descriptive names: `user-service`, `api-gateway`, `email-worker`
- ✅ Use hyphens, not spaces or underscores
- ⚠️ Avoid generic names like `app` or `server` — confusing in multi-app setups

### Log Management

- ✅ Always install `pm2-logrotate` before going to production
- ✅ Store logs in a dedicated `/logs` folder in your project, not just `~/.pm2/logs`
- ✅ Set `log_date_format` in your ecosystem file for easier debugging
- ⚠️ Don't use `console.log` excessively — it fills logs fast in high-traffic apps

### Production Tips

- ✅ Always use `ecosystem.config.js` — never raw CLI commands in production
- ✅ Use `exec_mode: "cluster"` with `instances: "max"` for CPU-bound Node.js apps
- ✅ Set `max_memory_restart` to prevent memory leaks from killing your server
- ✅ Use `pm2 reload` instead of `pm2 restart` in production to avoid downtime
- ✅ Run `pm2 save` after every process change
- ⚠️ Don't use `watch: true` in production — it causes unnecessary restarts
- ⚠️ Never hardcode secrets in `ecosystem.config.js` — use a `.env` file or secrets manager

### VPS-Specific Tips

- ✅ Always run `pm2 startup` after initial deployment
- ✅ Verify persistence by running `sudo reboot` and checking `pm2 list`
- ✅ Set up log rotation before deploying — not after logs are already huge

---

## 14. Cheat Sheet

| Command | Description |
|---|---|
| `pm2 start app.js --name "api"` | Start app with a name |
| `pm2 stop api` | Stop process by name |
| `pm2 restart api` | Restart process |
| `pm2 restart api --update-env` | Restart and reload env vars |
| `pm2 reload api` | Zero-downtime reload (cluster only) |
| `pm2 delete api` | Remove process from PM2 |
| `pm2 list` | Show all processes |
| `pm2 show api` | Show detailed info |
| `pm2 monit` | Real-time dashboard |
| `pm2 logs` | Stream all logs |
| `pm2 logs api` | Stream logs for one app |
| `pm2 logs api --lines 100` | View last 100 lines |
| `pm2 flush` | Clear all log files |
| `pm2 flush api` | Clear logs for one app |
| `pm2 start ecosystem.config.js` | Start from ecosystem file |
| `pm2 start ecosystem.config.js --env production` | Start with production env |
| `pm2 scale api 4` | Scale to 4 instances |
| `pm2 scale api max` | Scale to max CPU cores |
| `pm2 save` | Save current process list |
| `pm2 startup` | Generate startup script |
| `pm2 resurrect` | Restore saved processes |
| `pm2 kill` | Kill PM2 daemon |
| `pm2 update` | Update PM2 in-memory daemon |
| `pm2 install pm2-logrotate` | Install log rotation module |
| `pm2 set pm2-logrotate:max_size 50M` | Set max log file size |
| `pm2 conf pm2-logrotate` | View logrotate config |

---

> 📝 **Tip:** Bookmark `~/.pm2/logs/` and check it first whenever something goes wrong in production.

> 📖 **Official Docs:** [https://pm2.keymetrics.io/docs](https://pm2.keymetrics.io/docs)
