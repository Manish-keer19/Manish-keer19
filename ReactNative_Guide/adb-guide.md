# 📱 ADB (Android Debug Bridge) — Complete Developer Guide

> A beginner-to-advanced reference for Android developers. Every command includes syntax, explanation, and real-world usage.

---

## Table of Contents

1. [Introduction to ADB](#1-introduction-to-adb)
2. [Installation & Setup](#2-installation--setup)
3. [Basic Commands (Beginner)](#3-basic-commands-beginner)
4. [Intermediate Commands](#4-intermediate-commands)
5. [Advanced Commands](#5-advanced-commands)
6. [Debugging & Logcat Usage](#6-debugging--logcat-usage)
7. [App Management](#7-app-management)
8. [Device & Emulator Control](#8-device--emulator-control)
9. [Networking & Port Forwarding](#9-networking--port-forwarding)
10. [File Transfer & Storage](#10-file-transfer--storage)
11. [Performance & Testing](#11-performance--testing)
12. [Troubleshooting Common Issues](#12-troubleshooting-common-issues)
13. [Pro Tips & Best Practices](#13-pro-tips--best-practices)

---

## 1. Introduction to ADB

**ADB (Android Debug Bridge)** is a command-line tool that lets you communicate with Android devices and emulators. It's part of the Android SDK Platform Tools and is essential for every Android developer.

### What ADB Can Do

- Install and uninstall apps
- Push and pull files between your machine and a device
- View real-time logs (Logcat)
- Run shell commands on the device
- Forward network ports for backend debugging
- Capture screenshots and screen recordings
- Simulate input (taps, swipes, text)
- Profile app performance

### How ADB Works

ADB is a client-server architecture:

```
Your Machine          USB / Wi-Fi         Android Device
┌──────────┐          ┌─────────┐         ┌─────────────┐
│ ADB      │◄────────►│ ADB     │◄───────►│ ADB Daemon  │
│ Client   │          │ Server  │         │ (adbd)      │
└──────────┘          └─────────┘         └─────────────┘
```

- **ADB Client**: CLI tool you type commands into
- **ADB Server**: Background process on your machine (port 5037)
- **ADB Daemon (adbd)**: Background process on your Android device

---

## 2. Installation & Setup

### Prerequisites

- Android Studio installed, **OR**
- Android SDK Platform Tools installed standalone

> **Download standalone tools:** https://developer.android.com/tools/releases/platform-tools

---

### Windows

#### Option A — Via Android Studio
1. Open Android Studio → `SDK Manager`
2. Go to `SDK Tools` tab
3. Check `Android SDK Platform-Tools`
4. Click `Apply`

#### Option B — Standalone Install
1. Download Platform Tools zip from Google
2. Extract to `C:\platform-tools\`
3. Add to PATH:
   - Open `System Properties` → `Environment Variables`
   - Edit `Path` → Add `C:\platform-tools`
4. Verify in a new terminal:

```bash
adb version
```

---

### macOS

#### Option A — Via Homebrew (Recommended)

```bash
brew install android-platform-tools
```

#### Option B — Via Android Studio
The tools are located at:
```
~/Library/Android/sdk/platform-tools/
```

Add to your `~/.zshrc` or `~/.bash_profile`:

```bash
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

Then reload:

```bash
source ~/.zshrc
```

---

### Linux

#### Ubuntu/Debian

```bash
sudo apt update
sudo apt install android-tools-adb android-tools-fastboot
```

#### Via Android Studio
Add to `~/.bashrc`:

```bash
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

Then reload:

```bash
source ~/.bashrc
```

---

### Enable Developer Options on Your Device

1. Go to `Settings` → `About Phone`
2. Tap `Build Number` **7 times**
3. Go back → `Developer Options`
4. Enable **USB Debugging**

> ⚠️ **Warning:** Always disable USB Debugging when not in use, especially on production devices.

---

### Verify ADB Installation

```bash
adb version
# Android Debug Bridge version 1.0.41
# Version 35.0.0-...

adb devices
# List of devices attached
# emulator-5554   device
# R3CN101XXXX     device
```

---

## 3. Basic Commands (Beginner)

### 3.1 Device Detection

| Command | Description |
|--------|-------------|
| `adb devices` | List all connected devices and emulators |
| `adb devices -l` | List devices with detailed info (model, transport) |
| `adb get-serialno` | Get serial number of the connected device |
| `adb get-state` | Get device state (`device`, `offline`, `bootloader`) |

```bash
# Check all connected devices
adb devices -l

# Example output:
# List of devices attached
# emulator-5554          device product:sdk_gphone_x86 model:Android_SDK_built_for_x86
# R3CN101ABCDE           device product:beyond1 model:SM_G973F
```

**Real-world use:** Before running any ADB command, verify your target device is connected and shown as `device` (not `offline`).

---

### 3.2 Targeting a Specific Device

When multiple devices are connected, specify the target using `-s`:

```bash
adb -s emulator-5554 shell
adb -s R3CN101ABCDE install myapp.apk
```

> 💡 **Tip:** Set a default device using the `ANDROID_SERIAL` environment variable:
> ```bash
> export ANDROID_SERIAL=R3CN101ABCDE
> ```

---

### 3.3 ADB Server Management

```bash
# Start ADB server
adb start-server

# Kill ADB server (restart it to fix connection issues)
adb kill-server

# Restart ADB server
adb kill-server && adb start-server
```

**Real-world use:** When your device shows as `offline` or isn't detected, restarting the ADB server often fixes the issue.

---

### 3.4 Basic Shell Access

```bash
# Open interactive shell on device
adb shell

# Run a single command and exit
adb shell ls /sdcard/

# Run as root (if device is rooted)
adb root
adb shell
```

---

### 3.5 Install & Uninstall APKs

```bash
# Install an APK
adb install myapp.apk

# Install and replace existing app
adb install -r myapp.apk

# Install to SD card
adb install -s myapp.apk

# Uninstall an app (by package name)
adb uninstall com.example.myapp

# Uninstall but keep data
adb uninstall -k com.example.myapp
```

**Real-world use:** During development, use `adb install -r` to update your debug build without losing app data.

---

### 3.6 Reboot Commands

```bash
adb reboot                # Normal reboot
adb reboot bootloader     # Reboot into bootloader (for flashing)
adb reboot recovery       # Reboot into recovery mode
adb reboot fastboot       # Reboot into fastboot mode
```

---

## 4. Intermediate Commands

### 4.1 File Transfer

```bash
# Copy file FROM device TO your machine
adb pull /sdcard/Download/report.pdf ~/Desktop/

# Copy file FROM your machine TO device
adb push ~/Desktop/config.json /sdcard/Download/

# Pull entire directory
adb pull /sdcard/DCIM/ ~/Desktop/phone-photos/
```

**Real-world use:** Pull crash logs, databases, or exported files from a device for local analysis.

---

### 4.2 Screenshot & Screen Record

```bash
# Take a screenshot
adb shell screencap /sdcard/screen.png
adb pull /sdcard/screen.png ~/Desktop/

# One-liner: take and pull screenshot
adb exec-out screencap -p > ~/Desktop/screen.png

# Record screen (max 3 minutes by default)
adb shell screenrecord /sdcard/demo.mp4

# Record with options
adb shell screenrecord --size 1080x1920 --bit-rate 4000000 /sdcard/demo.mp4

# Stop recording
# Press Ctrl+C — then pull the file
adb pull /sdcard/demo.mp4 ~/Desktop/
```

**Real-world use:** Capture bug report screenshots or record a demo of your app feature without needing a third-party tool.

---

### 4.3 Simulating Input

```bash
# Simulate tap at coordinates (x=500, y=1000)
adb shell input tap 500 1000

# Simulate swipe (startX startY endX endY duration_ms)
adb shell input swipe 300 1000 300 300 500

# Type text
adb shell input text "hello@example.com"

# Press a key by keycode
adb shell input keyevent 4    # Back button
adb shell input keyevent 3    # Home button
adb shell input keyevent 26   # Power button
adb shell input keyevent 24   # Volume Up
adb shell input keyevent 25   # Volume Down
adb shell input keyevent 82   # Menu
```

**Common Keycodes:**

| Action | Keycode |
|--------|---------|
| Back | 4 |
| Home | 3 |
| Recent Apps | 187 |
| Power | 26 |
| Volume Up | 24 |
| Volume Down | 25 |
| Enter | 66 |
| Delete | 67 |
| Camera | 27 |

**Real-world use:** Automate repetitive UI flows during manual testing or scripted demos.

---

### 4.4 Copy Text to Device Clipboard

```bash
adb shell am broadcast -a clipper.set -e text "Paste this text"
```

Or use input text directly into a focused field:

```bash
adb shell input text "My%sFull%sName"   # Use %s for spaces
```

---

### 4.5 Package Manager Queries

```bash
# List all installed packages
adb shell pm list packages

# List only third-party apps
adb shell pm list packages -3

# List system apps only
adb shell pm list packages -s

# Find a specific package
adb shell pm list packages | grep myapp

# Get path of installed APK
adb shell pm path com.example.myapp
```

---

### 4.6 System Information

```bash
# Get device model
adb shell getprop ro.product.model

# Get Android version
adb shell getprop ro.build.version.release

# Get Android API level
adb shell getprop ro.build.version.sdk

# Get all system properties
adb shell getprop

# Get device screen resolution
adb shell wm size

# Get device screen density
adb shell wm density

# Get device IMEI (requires phone permission)
adb shell service call iphonesubinfo 1
```

---

### 4.7 Start and Stop Activities

```bash
# Start an app by package and activity name
adb shell am start -n com.example.myapp/.MainActivity

# Start with intent action
adb shell am start -a android.intent.action.VIEW -d "https://example.com"

# Force stop an app
adb shell am force-stop com.example.myapp

# Clear app data
adb shell pm clear com.example.myapp
```

**Real-world use:** Launch your app directly to a deep-linked screen from the terminal, or clear app state to test onboarding flows.

---

## 5. Advanced Commands

### 5.1 ADB Over Wi-Fi (Wireless Debugging)

#### Android 11+ (Built-in Wireless Debugging)

1. Enable `Developer Options` → `Wireless Debugging`
2. Tap `Pair device with pairing code`
3. Run:

```bash
adb pair <ip>:<pairing-port>
# Enter pairing code shown on device
adb connect <ip>:<port>
```

#### Android 10 and Below (USB → Wi-Fi handoff)

```bash
# Step 1: Connect via USB first
adb devices

# Step 2: Enable TCP/IP mode on port 5555
adb tcpip 5555

# Step 3: Find device IP address
adb shell ip addr show wlan0

# Step 4: Disconnect USB and connect wirelessly
adb connect 192.168.1.100:5555

# Verify
adb devices

# Disconnect Wi-Fi session
adb disconnect 192.168.1.100:5555
```

**Real-world use:** Debug apps while the device is mounted on a stand, or test orientation and camera features without a USB cable in the way.

---

### 5.2 Port Forwarding (Backend Integration)

```bash
# Forward device port to host port
# (Access localhost:8080 on device via your machine's port 8080)
adb forward tcp:8080 tcp:8080

# Reverse forward: expose your machine's server to the device
adb reverse tcp:8080 tcp:8080

# List active forwards
adb forward --list

# Remove a specific forward
adb forward --remove tcp:8080

# Remove all forwards
adb forward --remove-all
```

**Real-world use:** Your Android app makes API calls to `http://localhost:8080`. Use `adb reverse tcp:8080 tcp:8080` so the device can reach your local dev server.

---

### 5.3 Dumping App Information

```bash
# Full dump of all app info (permissions, activities, services)
adb shell dumpsys package com.example.myapp

# Get current focused activity
adb shell dumpsys window | grep -E 'mCurrentFocus|mFocusedApp'

# Dump activity back stack
adb shell dumpsys activity activities

# Get app's memory usage
adb shell dumpsys meminfo com.example.myapp

# Get battery stats for app
adb shell dumpsys batterystats com.example.myapp

# Get all running services
adb shell dumpsys activity services
```

**Real-world use:** Use `dumpsys meminfo` to track memory leaks during QA testing.

---

### 5.4 Broadcast Intents

```bash
# Send a broadcast intent
adb shell am broadcast -a com.example.MY_CUSTOM_ACTION

# Send broadcast with extras
adb shell am broadcast -a com.example.REFRESH --es key "value"

# Simulate battery low broadcast
adb shell am broadcast -a android.intent.action.BATTERY_LOW

# Simulate connectivity change
adb shell am broadcast -a android.net.conn.CONNECTIVITY_CHANGE
```

**Real-world use:** Test how your app responds to system events like battery low or connectivity changes, without actually draining the battery.

---

### 5.5 Monkey Testing (Random UI Events)

```bash
# Send 500 random events to your app
adb shell monkey -p com.example.myapp -v 500

# Slow down events (100ms delay between each)
adb shell monkey -p com.example.myapp --throttle 100 -v 1000

# Use a fixed seed for reproducible test runs
adb shell monkey -p com.example.myapp -s 42 -v 500
```

**Real-world use:** Stress test your app by simulating random user interactions to find crashes.

---

### 5.6 SQLite Database Access

```bash
# Open the SQLite shell for your app's database
adb shell
run-as com.example.myapp
cd databases
sqlite3 myapp.db

# In SQLite shell:
.tables              # List tables
.schema users        # Show table schema
SELECT * FROM users; # Query data
.quit                # Exit
```

> ⚠️ **Note:** `run-as` only works on debuggable (debug build) apps.

---

### 5.7 Extract APK from Device

```bash
# Step 1: Find the APK path
adb shell pm path com.example.myapp
# package:/data/app/com.example.myapp-1/base.apk

# Step 2: Pull the APK
adb pull /data/app/com.example.myapp-1/base.apk ~/Desktop/myapp.apk
```

**Real-world use:** Extract an installed APK for backup or inspection.

---

### 5.8 System Settings via ADB

```bash
# Turn off screen
adb shell input keyevent 26

# Enable dark mode
adb shell cmd uimode night yes

# Disable dark mode
adb shell cmd uimode night no

# Change font scale
adb shell settings put system font_scale 1.2

# Reset font scale
adb shell settings put system font_scale 1.0

# Enable/Disable airplane mode
adb shell settings put global airplane_mode_on 1
adb shell am broadcast -a android.intent.action.AIRPLANE_MODE

# Show/Hide navigation bar
adb shell settings put global policy_control immersive.navigation=*
```

---

## 6. Debugging & Logcat Usage

Logcat is your real-time Android log stream — the most powerful debugging tool in your daily workflow.

### 6.1 Basic Logcat

```bash
# Stream all logs
adb logcat

# Clear log buffer before streaming
adb logcat -c && adb logcat

# Save logs to a file
adb logcat > ~/Desktop/device.log

# Dump current logs and exit (no streaming)
adb logcat -d > ~/Desktop/dump.log
```

---

### 6.2 Filter by Tag

```bash
# Show only logs from a specific tag
adb logcat -s MyAppTag

# Multiple tags
adb logcat -s MyAppTag:D NetworkModule:W

# Filter by package (Android 7+)
adb logcat --pid=$(adb shell pidof -s com.example.myapp)
```

---

### 6.3 Filter by Log Level

Log levels (lowest → highest severity):

| Flag | Level | Description |
|------|-------|-------------|
| `V` | Verbose | All messages |
| `D` | Debug | Debug messages |
| `I` | Info | Informational |
| `W` | Warning | Potential issues |
| `E` | Error | Errors only |
| `F` | Fatal | Fatal crashes |
| `S` | Silent | No output |

```bash
# Show only errors and above
adb logcat *:E

# Show warnings and errors for your app, suppress everything else
adb logcat MyAppTag:W *:S

# Show debug and above from your app
adb logcat MyAppTag:D *:S
```

---

### 6.4 Filter by Keyword (grep)

```bash
# macOS/Linux
adb logcat | grep "MyApp"
adb logcat | grep -i "exception"
adb logcat | grep -E "ERROR|CRASH"

# Windows (PowerShell)
adb logcat | Select-String "MyApp"
```

---

### 6.5 Formatted Logcat Output

```bash
# Default format
adb logcat -v brief

# Timestamp format (most useful for debugging)
adb logcat -v time

# Full format with all metadata
adb logcat -v long

# Thread + process IDs
adb logcat -v threadtime

# Color output (requires terminal support)
adb logcat -v color
```

**Recommended format for debugging:**
```bash
adb logcat -v time *:W MyAppTag:D
```

---

### 6.6 Filter Crash Logs

```bash
# See only crash/ANR logs
adb logcat -b crash

# Combine with time filter
adb logcat -b crash -v time

# Watch for ANRs specifically
adb logcat | grep -A 20 "ANR in"
```

---

### 6.7 Log Buffers

```bash
# Available buffers: main, system, radio, events, crash, all
adb logcat -b main
adb logcat -b crash
adb logcat -b all

# Get buffer size
adb logcat -g
```

---

### 6.8 Workflow: Debug an App Crash

```bash
# Step 1: Clear old logs
adb logcat -c

# Step 2: Start your app
adb shell am start -n com.example.myapp/.MainActivity

# Step 3: Stream logs filtered to your package
adb logcat --pid=$(adb shell pidof -s com.example.myapp) -v time

# Step 4: Reproduce the crash — logs appear in terminal
# Step 5: Ctrl+C and save
adb logcat -d > ~/Desktop/crash-log.txt
```

---

## 7. App Management

### 7.1 Install Options

```bash
# Basic install
adb install app.apk

# Reinstall (keep data)
adb install -r app.apk

# Allow version downgrade
adb install -r -d app.apk

# Install split APKs (app bundles)
adb install-multiple base.apk split_config.arm64.apk split_config.en.apk

# Grant all permissions at install
adb install -g app.apk
```

---

### 7.2 Uninstall Options

```bash
# Full uninstall
adb uninstall com.example.myapp

# Keep data and cache (useful for testing upgrades)
adb uninstall -k com.example.myapp
```

---

### 7.3 Runtime Permissions

```bash
# Grant a permission
adb shell pm grant com.example.myapp android.permission.CAMERA

# Revoke a permission
adb shell pm revoke com.example.myapp android.permission.CAMERA

# List granted permissions for an app
adb shell dumpsys package com.example.myapp | grep "granted=true"

# Reset all permissions for an app
adb shell pm reset-permissions com.example.myapp
```

**Common permissions:**

| Permission | String |
|-----------|--------|
| Camera | `android.permission.CAMERA` |
| Location (fine) | `android.permission.ACCESS_FINE_LOCATION` |
| Location (coarse) | `android.permission.ACCESS_COARSE_LOCATION` |
| Storage Read | `android.permission.READ_EXTERNAL_STORAGE` |
| Storage Write | `android.permission.WRITE_EXTERNAL_STORAGE` |
| Contacts | `android.permission.READ_CONTACTS` |
| Microphone | `android.permission.RECORD_AUDIO` |
| Phone | `android.permission.READ_PHONE_STATE` |
| Notifications | `android.permission.POST_NOTIFICATIONS` |

**Real-world use:** Test your permission-denied flows by revoking a permission before launching the app.

---

### 7.4 Enable & Disable Apps

```bash
# Disable an app (hides from launcher, doesn't uninstall)
adb shell pm disable-user --user 0 com.example.myapp

# Re-enable it
adb shell pm enable com.example.myapp
```

---

### 7.5 App Data & Cache

```bash
# Clear app data (same as "Clear Data" in settings)
adb shell pm clear com.example.myapp

# Clear only cache
adb shell rm -rf /data/data/com.example.myapp/cache/*
```

---

### 7.6 Backup & Restore App Data

```bash
# Backup an app (creates backup.ab file)
adb backup -f ~/Desktop/myapp-backup.ab com.example.myapp

# Backup with APK
adb backup -apk -f ~/Desktop/myapp-backup.ab com.example.myapp

# Restore from backup
adb restore ~/Desktop/myapp-backup.ab
```

---

## 8. Device & Emulator Control

### 8.1 Emulator Management

```bash
# List available AVDs
emulator -list-avds

# Start a specific AVD
emulator -avd Pixel_6_API_33

# Start AVD with no audio
emulator -avd Pixel_6_API_33 -no-audio

# Start AVD with writable system image (for installing system apps)
emulator -avd Pixel_6_API_33 -writable-system

# Wipe emulator data on start
emulator -avd Pixel_6_API_33 -wipe-data
```

---

### 8.2 Emulator-Specific Controls

```bash
# Set fake GPS location
adb shell geo fix <longitude> <latitude>
# Example:
adb shell geo fix -122.4194 37.7749    # San Francisco

# Simulate phone call
adb shell am broadcast -a android.intent.action.CALL -d tel:1234567890

# Send SMS to emulator
adb emu sms send 5551234 "Hello from ADB"

# Simulate network speed
adb shell settings put global tether_dun_required 0
```

---

### 8.3 Screen and Display

```bash
# Get current screen resolution
adb shell wm size

# Override screen resolution (emulator)
adb shell wm size 1080x1920

# Reset resolution
adb shell wm size reset

# Get screen density
adb shell wm density

# Override density
adb shell wm density 420

# Reset density
adb shell wm density reset
```

**Real-world use:** Test how your app UI adapts to different screen sizes by overriding the emulator resolution.

---

### 8.4 Battery Simulation

```bash
# Set battery level to 15%
adb shell dumpsys battery set level 15

# Simulate charging
adb shell dumpsys battery set status 2

# Simulate not charging (discharging)
adb shell dumpsys battery set status 3

# Reset to real battery values
adb shell dumpsys battery reset
```

**Real-world use:** Test your battery-aware code (low battery notifications, power saving mode triggers) without waiting for actual drain.

---

### 8.5 Network Simulation

```bash
# Simulate no network connectivity
adb shell svc wifi disable
adb shell svc data disable

# Re-enable network
adb shell svc wifi enable
adb shell svc data enable

# Simulate airplane mode
adb shell settings put global airplane_mode_on 1
adb shell am broadcast -a android.intent.action.AIRPLANE_MODE
```

---

## 9. Networking & Port Forwarding

### 9.1 Port Forwarding (Device → Host)

```bash
# Forward device port 8080 to host port 8080
adb forward tcp:8080 tcp:8080

# Access via http://localhost:8080 from your machine
# when your app opens a local server on port 8080
```

**Use case:** Your app runs an embedded web server (e.g., for WebView debugging or local API). Forward the port to inspect it from your browser.

---

### 9.2 Reverse Port Forwarding (Host → Device)

```bash
# Expose host port 8080 to device as localhost:8080
adb reverse tcp:8080 tcp:8080
```

**Use case:** Your Android app calls `http://localhost:8080` to talk to your backend. Your backend runs on your development machine. `adb reverse` bridges the gap.

**Step-by-step: Connect App to Local Dev Server**

```bash
# Step 1: Start your local dev server (e.g., Node.js on port 3000)
npm run dev

# Step 2: Set up reverse port forward
adb reverse tcp:3000 tcp:3000

# Step 3: In your Android app, make requests to:
# http://localhost:3000/api/...
# It will reach your machine's port 3000!
```

---

### 9.3 Manage Active Forwards

```bash
# List all active port forwards
adb forward --list

# List all active reverse forwards
adb reverse --list

# Remove specific forward
adb forward --remove tcp:8080

# Remove all forwards
adb forward --remove-all

# Remove all reverse forwards
adb reverse --remove-all
```

---

### 9.4 Network Info from Device

```bash
# Get device IP address (Wi-Fi)
adb shell ip addr show wlan0 | grep "inet "

# Shorter version
adb shell ifconfig wlan0

# Test connectivity from device
adb shell ping -c 4 google.com

# Show network interfaces
adb shell netstat

# Show open connections
adb shell netstat -tlnp
```

---

## 10. File Transfer & Storage

### 10.1 Push & Pull Files

```bash
# Push file to device
adb push ~/Desktop/test-data.json /sdcard/Download/

# Pull file from device
adb pull /sdcard/Download/export.csv ~/Desktop/

# Pull with progress
adb pull -a /sdcard/DCIM/ ~/Desktop/DCIM/

# Push entire directory
adb push ~/test-assets/ /sdcard/test-assets/
```

---

### 10.2 Navigate Device File System

```bash
# Open shell
adb shell

# Common directories
ls /sdcard/              # User storage (public)
ls /data/data/           # App private data (requires root)
ls /sdcard/Download/     # Downloads folder
ls /sdcard/DCIM/         # Camera photos
ls /sdcard/Android/data/com.example.myapp/  # App's external storage

# Navigate inside shell
cd /sdcard/Download
ls -la
pwd
```

---

### 10.3 File Operations via Shell

```bash
# Create directory
adb shell mkdir /sdcard/mytest/

# Delete file
adb shell rm /sdcard/mytest/old.txt

# Delete directory recursively
adb shell rm -rf /sdcard/mytest/

# Move/rename file
adb shell mv /sdcard/old.txt /sdcard/new.txt

# Copy file
adb shell cp /sdcard/source.txt /sdcard/dest.txt

# Check file size / disk usage
adb shell df -h
adb shell du -sh /sdcard/
```

---

### 10.4 Access App Private Storage (Debuggable Builds)

```bash
# Use run-as for debug builds
adb shell run-as com.example.myapp ls /data/data/com.example.myapp/

# Copy a database out
adb shell run-as com.example.myapp cp databases/myapp.db /sdcard/
adb pull /sdcard/myapp.db ~/Desktop/
```

---

### 10.5 Working with Large Files

```bash
# Compress before pulling (speeds up transfer)
adb shell tar -czf /sdcard/logs.tar.gz /data/data/com.example.myapp/files/logs/
adb pull /sdcard/logs.tar.gz ~/Desktop/

# Split large file
adb shell split -b 50m /sdcard/bigfile.mp4 /sdcard/chunk_

# Pull all chunks
adb pull /sdcard/ ~/Desktop/chunks/
```

---

## 11. Performance & Testing

### 11.1 CPU & Memory Profiling

```bash
# App memory info (summary)
adb shell dumpsys meminfo com.example.myapp

# Detailed memory stats
adb shell dumpsys meminfo com.example.myapp -d

# CPU usage
adb shell top -n 1

# CPU for specific app
adb shell top -n 1 | grep com.example.myapp

# Check for ANRs
adb shell dumpsys activity | grep -A 10 "ANR"
```

---

### 11.2 GPU & Frame Rate

```bash
# Enable GPU overdraw visualization (for debugging overdraw)
adb shell setprop debug.hwui.overdraw show

# Disable
adb shell setprop debug.hwui.overdraw false

# Show GPU rendering profile bars on screen
adb shell setprop debug.hwui.profile visual_bars

# Dump frame timing data
adb shell dumpsys gfxinfo com.example.myapp

# Reset frame data
adb shell dumpsys gfxinfo com.example.myapp reset
```

**Real-world use:** Analyze `dumpsys gfxinfo` output to count frame drops and jank in your app's animations.

---

### 11.3 StrictMode & Profiling

```bash
# Enable strict mode violations to be visible on screen
adb shell setprop debug.strictmode.penalty death

# Dump all app threads (stack traces)
adb shell kill -3 $(adb shell pidof com.example.myapp)
adb logcat | grep "Cmd line"
```

---

### 11.4 Automated Testing

```bash
# Run instrumented tests (JUnit/Espresso)
adb shell am instrument -w \
  com.example.myapp.test/androidx.test.runner.AndroidJUnitRunner

# Run a specific test class
adb shell am instrument -w \
  -e class com.example.myapp.LoginTest \
  com.example.myapp.test/androidx.test.runner.AndroidJUnitRunner

# Run a single test method
adb shell am instrument -w \
  -e class com.example.myapp.LoginTest#testSuccessfulLogin \
  com.example.myapp.test/androidx.test.runner.AndroidJUnitRunner
```

---

### 11.5 UI Automator Viewer

```bash
# Dump current UI hierarchy (XML)
adb shell uiautomator dump /sdcard/ui.xml
adb pull /sdcard/ui.xml ~/Desktop/

# Open in browser (human-readable)
cat ~/Desktop/ui.xml
```

**Real-world use:** Use this to find element IDs and bounds for UI automation scripts.

---

### 11.6 Battery Performance

```bash
# Get battery stats
adb shell dumpsys battery

# Get detailed battery history
adb shell dumpsys batterystats

# Reset battery stats (start fresh measurement)
adb shell dumpsys batterystats --reset

# Measure battery drain by your app
adb shell dumpsys batterystats com.example.myapp
```

---

## 12. Troubleshooting Common Issues

### 12.1 Device Not Detected

**Symptoms:** `adb devices` shows empty list or `unauthorized`

**Solutions:**

```bash
# Step 1: Restart ADB server
adb kill-server
adb start-server

# Step 2: Check USB connection & try different cable/port
adb devices

# Step 3: Revoke and re-grant USB Debugging
# Go to Developer Options → Revoke USB Debugging Authorizations
# Reconnect device and accept the dialog on your phone

# Step 4: Check ADB recognizes the device at OS level
# Linux: check USB permissions
lsusb    # Find your device
# Add udev rule if needed:
echo 'SUBSYSTEM=="usb", ATTR{idVendor}=="04e8", MODE="0666"' | sudo tee /etc/udev/rules.d/51-android.rules
sudo udevadm control --reload-rules
```

---

### 12.2 Device Shows as "Unauthorized"

```bash
# On the device:
# 1. Go to Settings → Developer Options
# 2. Tap "Revoke USB debugging authorizations"
# 3. Reconnect USB
# 4. Accept the "Allow USB debugging?" dialog on device

# On computer:
adb kill-server && adb start-server
adb devices
```

---

### 12.3 APK Install Failures

| Error | Cause | Fix |
|-------|-------|-----|
| `INSTALL_FAILED_ALREADY_EXISTS` | App exists | Use `adb install -r` |
| `INSTALL_FAILED_VERSION_DOWNGRADE` | Lower version | Use `adb install -r -d` |
| `INSTALL_FAILED_INSUFFICIENT_STORAGE` | No space | Free up device storage |
| `INSTALL_FAILED_INVALID_APK` | Corrupt APK | Rebuild the APK |
| `INSTALL_FAILED_ABORTED` | User cancelled | Retry / check USB |
| `INSTALL_PARSE_FAILED_NO_CERTIFICATES` | APK unsigned | Sign the APK |

---

### 12.4 ADB Wi-Fi Connection Issues

```bash
# If wireless connection drops
adb disconnect

# Reconnect
adb connect 192.168.1.100:5555

# If port changes after reboot, redo the handoff:
# 1. Plug in USB
adb tcpip 5555
# 2. Unplug USB
adb connect <device-ip>:5555
```

---

### 12.5 "Device Offline" Error

```bash
adb kill-server
adb start-server

# If still offline, try:
adb reconnect offline

# Or full device restart:
adb reboot
```

---

### 12.6 Logcat Showing No Output

```bash
# Clear buffer first
adb logcat -c

# Check if PID is correct
adb shell pidof com.example.myapp

# Try without filter first
adb logcat | head -20
```

---

### 12.7 "Permission Denied" in Shell

```bash
# Most /data paths require root — use run-as for debug apps
adb shell run-as com.example.myapp ls .

# Or root the ADB session (rooted devices only)
adb root
adb remount
```

---

## 13. Pro Tips & Best Practices

### 13.1 Create Shell Aliases

Add these to your `~/.zshrc` or `~/.bashrc`:

```bash
# ADB shortcuts
alias adbd='adb devices'
alias adbr='adb kill-server && adb start-server'
alias adblog='adb logcat -v time *:W'
alias adbss='adb exec-out screencap -p > ~/Desktop/screen_$(date +%Y%m%d_%H%M%S).png'
alias adbwifi='adb tcpip 5555'

# App-specific (change package name)
alias mylog='adb logcat --pid=$(adb shell pidof -s com.example.myapp) -v time'
alias mykill='adb shell am force-stop com.example.myapp'
alias myclear='adb shell pm clear com.example.myapp'
```

---

### 13.2 Multi-Device Script Template

```bash
#!/bin/bash
# Run command on ALL connected devices
adb devices | tail -n +2 | cut -sf 1 | xargs -I {} adb -s {} shell getprop ro.product.model
```

---

### 13.3 One-Liner Workflows

```bash
# Install latest debug APK and launch app
adb install -r app/build/outputs/apk/debug/app-debug.apk && \
adb shell am start -n com.example.myapp/.MainActivity

# Screenshot with timestamp
adb exec-out screencap -p > screenshot_$(date +%Y%m%d_%H%M%S).png

# Kill and clear app data, then relaunch
adb shell am force-stop com.example.myapp && \
adb shell pm clear com.example.myapp && \
adb shell am start -n com.example.myapp/.MainActivity

# Watch app memory every 2 seconds
watch -n 2 'adb shell dumpsys meminfo com.example.myapp | grep "TOTAL"'
```

---

### 13.4 Useful Developer Workflows

#### Workflow A: First-Run Debugging Setup

```bash
# 1. Confirm device is detected
adb devices

# 2. Clear old logs
adb logcat -c

# 3. Install fresh debug build
adb install -r app-debug.apk

# 4. Set up reverse port forward for local backend
adb reverse tcp:3000 tcp:3000

# 5. Launch app and stream filtered logs
adb shell am start -n com.example.myapp/.MainActivity
adb logcat MyAppTag:D *:S -v time
```

---

#### Workflow B: Investigating a Crash

```bash
# 1. Clear logs
adb logcat -c

# 2. Stream crash logs
adb logcat -b crash -v time &

# 3. Reproduce the crash

# 4. Pull full log dump
adb logcat -d > ~/Desktop/crash_$(date +%Y%m%d).log

# 5. Search for the exception
grep -A 30 "FATAL EXCEPTION" ~/Desktop/crash_$(date +%Y%m%d).log
```

---

#### Workflow C: Testing on Multiple Devices

```bash
# List all connected devices
adb devices -l

# Install on all devices
for device in $(adb devices | tail -n +2 | awk '{print $1}'); do
  echo "Installing on $device"
  adb -s $device install -r app-debug.apk
done
```

---

### 13.5 Security Best Practices

> ⚠️ **Security Reminders:**

- Always **disable USB Debugging** on production/personal devices when not actively developing.
- Never leave `adb tcpip 5555` active on an open/public network — anyone on that network can connect.
- Use `adb -s <serial>` explicitly in CI/CD scripts to avoid targeting the wrong device.
- Store ADB keys in `~/.android/adbkey` — don't share them.
- Regularly run `adb devices` to audit which machines have access.

---

### 13.6 Quick Reference Cheat Sheet

| Task | Command |
|------|---------|
| List devices | `adb devices -l` |
| Install APK | `adb install -r app.apk` |
| Uninstall app | `adb uninstall com.example.app` |
| Open shell | `adb shell` |
| Stream logs | `adb logcat -v time` |
| Filter logs | `adb logcat -s TAG:D *:S` |
| Take screenshot | `adb exec-out screencap -p > screen.png` |
| Record screen | `adb shell screenrecord /sdcard/demo.mp4` |
| Push file | `adb push file.txt /sdcard/` |
| Pull file | `adb pull /sdcard/file.txt .` |
| Clear app data | `adb shell pm clear com.pkg.name` |
| Force stop app | `adb shell am force-stop com.pkg.name` |
| Grant permission | `adb shell pm grant com.pkg.name android.permission.X` |
| Get device info | `adb shell getprop ro.product.model` |
| Enable Wi-Fi ADB | `adb tcpip 5555` then `adb connect <ip>:5555` |
| Reverse forward | `adb reverse tcp:3000 tcp:3000` |
| Restart ADB | `adb kill-server && adb start-server` |
| Simulate tap | `adb shell input tap 500 1000` |
| Type text | `adb shell input text "hello"` |
| Get app memory | `adb shell dumpsys meminfo com.pkg.name` |
| Run tests | `adb shell am instrument -w com.pkg.test/runner` |

---

*Guide last updated: May 2026 — covers Android API levels 21–35*
