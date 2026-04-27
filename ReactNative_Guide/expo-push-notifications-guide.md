# 📲 Expo Push Notifications: Complete Guide
### Beginner → Intermediate → Production-Ready

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Beginner Setup — Minimum Working System ✅](#2-beginner-setup--minimum-working-system-)
   - [Frontend: Install & Configure](#21-frontend-install--configure)
   - [Backend: Simple Express Server](#22-backend-simple-express-server)
   - [Testing Your First Notification](#23-testing-your-first-notification)
3. [Intermediate Setup — Improving the Basics 🔧](#3-intermediate-setup--improving-the-basics-)
   - [Frontend Improvements](#31-frontend-improvements)
   - [Backend Improvements](#32-backend-improvements)
4. [Advanced / Production Setup 🚀](#4-advanced--production-setup-)
   - [Architecture Overview](#41-architecture-overview)
   - [Queue System with BullMQ + Redis](#42-queue-system-with-bullmq--redis)
   - [Notification Service Layer](#43-notification-service-layer)
   - [Firebase (Android FCM) Setup](#44-firebase-android-fcm-setup)
5. [Project Folder Structure](#5-project-folder-structure)
6. [Common Errors & Fixes](#6-common-errors--fixes)
7. [Optional Advanced Features](#7-optional-advanced-features)

---

## 1. Introduction

### What Are Push Notifications?

Push notifications are messages sent from a server to a user's device, even when the app is not open. Think of them as alerts, reminders, or updates that "push" information to the user without them needing to open the app.

**Examples:**
- "Your order has been shipped! 📦"
- "You have a new message from Alice 💬"
- "Your timer has ended ⏰"

### How Expo's Push System Works

Expo provides a unified push notification service that works across iOS and Android. Here is the simplified flow:

```
Your App (React Native)
    │
    │  1. App registers and gets a unique token
    │
    ▼
Expo Push Service (api.expo.dev)
    │
    │  2. Your backend sends a message to Expo's API
    │     using the token
    │
    ▼
FCM (Firebase) / APNs (Apple)
    │
    │  3. Platform-specific service delivers it
    │
    ▼
User's Device 📱
```

**Key concepts:**
- **Expo Push Token** — A unique string (like `ExponentPushToken[xxxx]`) that identifies a specific device/app installation. Your backend stores this.
- **FCM** — Firebase Cloud Messaging, used by Android to deliver notifications.
- **APNs** — Apple Push Notification service, used by iOS.
- **Expo Push API** — Expo's server that acts as a middle layer between your backend and FCM/APNs.

---

## 2. Beginner Setup — Minimum Working System ✅

> **Goal:** Get your first notification working end-to-end in under 30 minutes.

### 2.1 Frontend: Install & Configure

#### Step 1 — Install Dependencies

```bash
npx expo install expo-notifications expo-device expo-constants
```

#### Step 2 — Configure `app.json`

Add the following inside your `app.json` to enable notifications:

```json
{
  "expo": {
    "name": "MyApp",
    "slug": "my-app",
    "android": {
      "package": "com.yourcompany.myapp",
      "googleServicesFile": "./google-services.json"
    },
    "plugins": [
      [
        "expo-notifications",
        {
          "icon": "./assets/notification-icon.png",
          "color": "#ffffff"
        }
      ]
    ]
  }
}
```

#### Step 3 — Basic Notification Setup (Simple Version)

Create the file `src/notifications/registerForPushNotifications.js`:

```javascript
import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';

// This tells Expo how to handle notifications when the app is open
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: false,
  }),
});

export async function registerForPushNotificationsAsync() {
  // Push notifications only work on real devices, not emulators
  if (!Device.isDevice) {
    alert('Push notifications require a real device.');
    return null;
  }

  // Ask the user for permission
  const { status } = await Notifications.requestPermissionsAsync();

  if (status !== 'granted') {
    alert('Permission for notifications was denied.');
    return null;
  }

  // Get the Expo Push Token
  const tokenData = await Notifications.getExpoPushTokenAsync();
  const token = tokenData.data;

  console.log('Expo Push Token:', token);
  return token;
}
```

#### Step 4 — Use It in `App.js`

```javascript
import React, { useEffect } from 'react';
import { View, Text } from 'react-native';
import { registerForPushNotificationsAsync } from './src/notifications/registerForPushNotifications';

const BACKEND_URL = 'http://YOUR_LOCAL_IP:3000'; // Replace with your IP

export default function App() {
  useEffect(() => {
    async function setup() {
      const token = await registerForPushNotificationsAsync();

      if (token) {
        // Send the token to your backend so it can send notifications later
        await fetch(`${BACKEND_URL}/register-token`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ token }),
        });
      }
    }

    setup();
  }, []);

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text>Push Notifications Demo</Text>
    </View>
  );
}
```

> **Note:** To find your local IP on Mac/Linux run `ifconfig | grep "inet "`. On Windows run `ipconfig`. Use this IP instead of `localhost` — the device can't reach your computer's `localhost`.

---

### 2.2 Backend: Simple Express Server

#### Step 1 — Initialize the Project

```bash
mkdir push-backend && cd push-backend
npm init -y
npm install express expo-server-sdk
```

#### Step 2 — Create `server.js`

```javascript
const express = require('express');
const { Expo } = require('expo-server-sdk');

const app = express();
app.use(express.json());

// Simple in-memory token storage (for development only — not for production)
const tokens = [];

// Create an Expo SDK client
const expo = new Expo();

// ─── ROUTE: Register a device token ───────────────────────────────────────────
app.post('/register-token', (req, res) => {
  const { token } = req.body;

  if (!token) {
    return res.status(400).json({ error: 'Token is required' });
  }

  // Avoid duplicate tokens
  if (!tokens.includes(token)) {
    tokens.push(token);
    console.log('Token registered:', token);
  }

  res.json({ success: true });
});

// ─── ROUTE: Send a notification to ALL registered devices ────────────────────
app.post('/send-notification', async (req, res) => {
  const { title, body } = req.body;

  // Build the messages array
  const messages = tokens
    .filter(token => Expo.isExpoPushToken(token)) // validate each token
    .map(token => ({
      to: token,
      sound: 'default',
      title: title || 'Hello!',
      body: body || 'This is a push notification.',
      data: { someExtraData: 'goes here' },
    }));

  if (messages.length === 0) {
    return res.status(400).json({ error: 'No valid tokens registered' });
  }

  try {
    // Expo recommends sending in chunks of 100
    const chunks = expo.chunkPushNotifications(messages);
    const tickets = [];

    for (const chunk of chunks) {
      const ticketChunk = await expo.sendPushNotificationsAsync(chunk);
      tickets.push(...ticketChunk);
    }

    console.log('Notification tickets:', tickets);
    res.json({ success: true, tickets });
  } catch (error) {
    console.error('Error sending notifications:', error);
    res.status(500).json({ error: 'Failed to send notification' });
  }
});

// ─── START SERVER ─────────────────────────────────────────────────────────────
app.listen(3000, () => {
  console.log('Backend running on http://localhost:3000');
});
```

#### Step 3 — Run the Server

```bash
node server.js
```

---

### 2.3 Testing Your First Notification

#### Method 1 — Using `curl`

```bash
# First, make sure a device has registered a token by opening the app.
# Then trigger a notification from your terminal:

curl -X POST http://localhost:3000/send-notification \
  -H "Content-Type: application/json" \
  -d '{"title": "Hello World!", "body": "Your first push notification 🎉"}'
```

#### Method 2 — Using Expo's Push Notification Tool

Go to [https://expo.dev/notifications](https://expo.dev/notifications) and paste your Expo Push Token to send a test notification directly.

#### Expected Output

**Terminal (backend):**
```
Token registered: ExponentPushToken[xxxxxxxxxxxxxxxxxxxxxx]
Notification tickets: [ { status: 'ok', id: 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx' } ]
```

**Device:** You should see a notification appear at the top of the screen.

---

## 3. Intermediate Setup — Improving the Basics 🔧

> **Goal:** Handle real-world edge cases, use a database, and support multiple users.

### 3.1 Frontend Improvements

#### Handle Permission States Properly

Update `src/notifications/registerForPushNotifications.js`:

```javascript
import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';
import { Platform } from 'react-native';

Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: true,
  }),
});

export async function registerForPushNotificationsAsync() {
  if (!Device.isDevice) {
    console.warn('Must use a physical device for push notifications.');
    return null;
  }

  // Check existing permission status first
  const { status: existingStatus } = await Notifications.getPermissionsAsync();
  let finalStatus = existingStatus;

  // Only ask if not already determined
  if (existingStatus !== 'granted') {
    const { status } = await Notifications.requestPermissionsAsync();
    finalStatus = status;
  }

  if (finalStatus !== 'granted') {
    console.warn('Push notification permission not granted.');
    return null;
  }

  // Android requires a notification channel
  if (Platform.OS === 'android') {
    await Notifications.setNotificationChannelAsync('default', {
      name: 'default',
      importance: Notifications.AndroidImportance.MAX,
      vibrationPattern: [0, 250, 250, 250],
      lightColor: '#FF231F7C',
    });
  }

  const tokenData = await Notifications.getExpoPushTokenAsync({
    projectId: 'YOUR_EXPO_PROJECT_ID', // From app.json > extra > eas > projectId
  });

  return tokenData.data;
}
```

#### Token Refresh Handling

Expo push tokens can change (e.g., after app reinstall). Handle this with a listener:

```javascript
// src/hooks/usePushToken.js
import { useEffect, useRef, useState } from 'react';
import * as Notifications from 'expo-notifications';
import { registerForPushNotificationsAsync } from '../notifications/registerForPushNotifications';
import { sendTokenToBackend } from '../services/notificationService';

export function usePushToken() {
  const [token, setToken] = useState(null);

  useEffect(() => {
    // Register on mount
    registerForPushNotificationsAsync().then(async (newToken) => {
      if (newToken) {
        setToken(newToken);
        await sendTokenToBackend(newToken);
      }
    });

    // Listen for token refresh events
    const subscription = Notifications.addPushTokenListener(async (newToken) => {
      console.log('Token refreshed:', newToken.data);
      setToken(newToken.data);
      await sendTokenToBackend(newToken.data);
    });

    return () => subscription.remove();
  }, []);

  return token;
}
```

#### Notification Listeners (Foreground & Tap)

```javascript
// src/hooks/useNotificationListeners.js
import { useEffect, useRef } from 'react';
import * as Notifications from 'expo-notifications';
import { useNavigation } from '@react-navigation/native';

export function useNotificationListeners() {
  const navigation = useNavigation();
  const notificationListener = useRef();
  const responseListener = useRef();

  useEffect(() => {
    // Fires when a notification is received while app is OPEN (foreground)
    notificationListener.current = Notifications.addNotificationReceivedListener(
      (notification) => {
        console.log('Notification received in foreground:', notification);
        // You can update UI state here
      }
    );

    // Fires when the user TAPS the notification
    responseListener.current = Notifications.addNotificationResponseReceivedListener(
      (response) => {
        const data = response.notification.request.content.data;
        console.log('Notification tapped, data:', data);

        // Navigate based on notification data
        if (data.screen) {
          navigation.navigate(data.screen, data.params || {});
        }
      }
    );

    return () => {
      notificationListener.current?.remove();
      responseListener.current?.remove();
    };
  }, []);
}
```

#### Notification Service (API Layer)

```javascript
// src/services/notificationService.js
const BACKEND_URL = 'https://your-api.com';

export async function sendTokenToBackend(token, userId) {
  try {
    const response = await fetch(`${BACKEND_URL}/notifications/register`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${await getAuthToken()}`, // your auth token
      },
      body: JSON.stringify({ token, userId }),
    });

    if (!response.ok) throw new Error('Failed to register token');
    return await response.json();
  } catch (error) {
    console.error('Token registration error:', error);
  }
}
```

---

### 3.2 Backend Improvements

#### Install Additional Dependencies

```bash
npm install mongoose dotenv
```

#### Database Model (MongoDB)

```javascript
// models/DeviceToken.js
const mongoose = require('mongoose');

const deviceTokenSchema = new mongoose.Schema(
  {
    userId: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User',
      required: true,
      index: true,
    },
    token: {
      type: String,
      required: true,
      unique: true,
    },
    platform: {
      type: String,
      enum: ['ios', 'android'],
    },
    isActive: {
      type: Boolean,
      default: true,
    },
    lastUsedAt: {
      type: Date,
      default: Date.now,
    },
  },
  { timestamps: true }
);

module.exports = mongoose.model('DeviceToken', deviceTokenSchema);
```

**PostgreSQL Alternative Schema:**
```sql
CREATE TABLE device_tokens (
  id          SERIAL PRIMARY KEY,
  user_id     INTEGER NOT NULL REFERENCES users(id),
  token       TEXT NOT NULL UNIQUE,
  platform    VARCHAR(10),
  is_active   BOOLEAN DEFAULT TRUE,
  last_used_at TIMESTAMPTZ DEFAULT NOW(),
  created_at  TIMESTAMPTZ DEFAULT NOW(),
  updated_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_device_tokens_user_id ON device_tokens(user_id);
```

#### Notification Controller

```javascript
// controllers/notificationController.js
const { Expo } = require('expo-server-sdk');
const DeviceToken = require('../models/DeviceToken');

const expo = new Expo();

// Register or update a device token
async function registerToken(req, res) {
  const { token, platform } = req.body;
  const userId = req.user.id; // from auth middleware

  if (!Expo.isExpoPushToken(token)) {
    return res.status(400).json({ error: 'Invalid Expo push token' });
  }

  try {
    // Upsert: update if exists, create if not
    await DeviceToken.findOneAndUpdate(
      { token },
      { userId, token, platform, isActive: true, lastUsedAt: new Date() },
      { upsert: true, new: true }
    );

    res.json({ success: true });
  } catch (error) {
    console.error('Error registering token:', error);
    res.status(500).json({ error: 'Failed to register token' });
  }
}

// Send notification to one user (all their devices)
async function sendToUser(userId, message) {
  const deviceTokens = await DeviceToken.find({ userId, isActive: true });

  if (deviceTokens.length === 0) {
    return { sent: 0, error: 'No active devices found' };
  }

  const messages = deviceTokens.map((device) => ({
    to: device.token,
    sound: 'default',
    title: message.title,
    body: message.body,
    data: message.data || {},
  }));

  return await sendMessages(messages);
}

// Send notification to multiple users
async function sendToUsers(userIds, message) {
  const deviceTokens = await DeviceToken.find({
    userId: { $in: userIds },
    isActive: true,
  });

  const messages = deviceTokens.map((device) => ({
    to: device.token,
    sound: 'default',
    title: message.title,
    body: message.body,
    data: message.data || {},
  }));

  return await sendMessages(messages);
}

// Core sending function with error handling
async function sendMessages(messages) {
  const chunks = expo.chunkPushNotifications(messages);
  const tickets = [];

  for (const chunk of chunks) {
    try {
      const ticketChunk = await expo.sendPushNotificationsAsync(chunk);
      tickets.push(...ticketChunk);
    } catch (error) {
      console.error('Error sending chunk:', error);
    }
  }

  // Check receipts to find failed tokens
  await processReceipts(tickets);

  return { sent: tickets.length, tickets };
}

// Handle Expo receipt checking to find invalid tokens
async function processReceipts(tickets) {
  const receiptIds = tickets
    .filter((t) => t.status === 'ok')
    .map((t) => t.id);

  if (receiptIds.length === 0) return;

  const receiptChunks = expo.chunkPushNotificationReceiptIds(receiptIds);

  for (const chunk of receiptChunks) {
    try {
      const receipts = await expo.getPushNotificationReceiptsAsync(chunk);

      for (const [receiptId, receipt] of Object.entries(receipts)) {
        if (receipt.status === 'error') {
          console.error(`Receipt error: ${receipt.message}`);

          // Deactivate invalid/unregistered tokens
          if (
            receipt.details?.error === 'DeviceNotRegistered' ||
            receipt.details?.error === 'InvalidCredentials'
          ) {
            await DeviceToken.findOneAndUpdate(
              { token: receipt.details?.expoPushToken },
              { isActive: false }
            );
          }
        }
      }
    } catch (error) {
      console.error('Error processing receipts:', error);
    }
  }
}

module.exports = { registerToken, sendToUser, sendToUsers };
```

#### Notification Routes

```javascript
// routes/notificationRoutes.js
const express = require('express');
const router = express.Router();
const { registerToken, sendToUser } = require('../controllers/notificationController');
const authMiddleware = require('../middleware/auth');

router.post('/register', authMiddleware, registerToken);

router.post('/send', authMiddleware, async (req, res) => {
  const { userId, title, body, data } = req.body;
  const result = await sendToUser(userId, { title, body, data });
  res.json(result);
});

module.exports = router;
```

---

## 4. Advanced / Production Setup 🚀

> **Goal:** Build a scalable, resilient system that handles millions of notifications.

### 4.1 Architecture Overview

```
                     ┌─────────────────────────────────┐
                     │         Your API Server          │
                     │  (Express + Notification Routes) │
                     └────────────────┬────────────────┘
                                      │
                              Enqueues job
                                      │
                     ┌────────────────▼────────────────┐
                     │          Redis + BullMQ          │
                     │       (Notification Queue)       │
                     └────────────────┬────────────────┘
                                      │
                           Workers pick up jobs
                                      │
                     ┌────────────────▼────────────────┐
                     │       Notification Worker        │
                     │   (Processes & sends in bulk)    │
                     └────────────────┬────────────────┘
                                      │
                     ┌────────────────▼────────────────┐
                     │        Expo Push API             │
                     │  → FCM (Android) / APNs (iOS)   │
                     └─────────────────────────────────┘
```

**Why a queue?**
- Notifications don't block your API from responding
- Automatic retries on failure
- Handle spikes (e.g., 100,000 notifications at once)
- Monitor job status and failures

---

### 4.2 Queue System with BullMQ + Redis

#### Install Dependencies

```bash
npm install bullmq ioredis
```

#### Queue Definition

```javascript
// queue/notificationQueue.js
const { Queue } = require('bullmq');
const IORedis = require('ioredis');

const connection = new IORedis(process.env.REDIS_URL || 'redis://localhost:6379', {
  maxRetriesPerRequest: null, // Required for BullMQ
});

const notificationQueue = new Queue('notifications', {
  connection,
  defaultJobOptions: {
    attempts: 3,                    // Retry failed jobs 3 times
    backoff: {
      type: 'exponential',
      delay: 5000,                  // Start with 5 second delay, then 10s, 20s
    },
    removeOnComplete: { count: 100 }, // Keep last 100 completed jobs
    removeOnFail: { count: 500 },     // Keep last 500 failed jobs for debugging
  },
});

module.exports = { notificationQueue, connection };
```

#### Worker

```javascript
// queue/notificationWorker.js
const { Worker } = require('bullmq');
const { Expo } = require('expo-server-sdk');
const { connection } = require('./notificationQueue');
const DeviceToken = require('../models/DeviceToken');

const expo = new Expo();

const worker = new Worker(
  'notifications',
  async (job) => {
    const { userIds, title, body, data } = job.data;

    console.log(`Processing job ${job.id}: sending to ${userIds?.length || 1} user(s)`);

    // Fetch all active tokens for these users
    const deviceTokens = await DeviceToken.find({
      userId: { $in: userIds },
      isActive: true,
    });

    if (deviceTokens.length === 0) {
      console.log(`Job ${job.id}: No active tokens found.`);
      return { sent: 0 };
    }

    const messages = deviceTokens
      .filter((d) => Expo.isExpoPushToken(d.token))
      .map((d) => ({
        to: d.token,
        sound: 'default',
        title,
        body,
        data: data || {},
      }));

    // Send in chunks
    const chunks = expo.chunkPushNotifications(messages);
    let totalSent = 0;

    for (const chunk of chunks) {
      const tickets = await expo.sendPushNotificationsAsync(chunk);
      totalSent += tickets.filter((t) => t.status === 'ok').length;

      // Deactivate any invalid tokens immediately
      for (const ticket of tickets) {
        if (ticket.status === 'error' && ticket.details?.error === 'DeviceNotRegistered') {
          await DeviceToken.findOneAndUpdate(
            { token: ticket.details.expoPushToken },
            { isActive: false }
          );
        }
      }
    }

    return { sent: totalSent };
  },
  {
    connection,
    concurrency: 5, // Process up to 5 jobs at the same time
  }
);

worker.on('completed', (job, result) => {
  console.log(`Job ${job.id} completed. Sent: ${result.sent}`);
});

worker.on('failed', (job, err) => {
  console.error(`Job ${job.id} failed:`, err.message);
});

module.exports = worker;
```

#### Enqueue from Your Controller

```javascript
// In your notification controller or service
const { notificationQueue } = require('../queue/notificationQueue');

async function scheduleNotification(userIds, message) {
  const job = await notificationQueue.add('send', {
    userIds,
    title: message.title,
    body: message.body,
    data: message.data,
  });

  return { jobId: job.id };
}
```

#### Start the Worker (Separate Process)

```javascript
// worker.js (entry point — run separately from your API)
require('./queue/notificationWorker');
console.log('Notification worker started and listening for jobs...');
```

```bash
# Run your API and worker as separate processes
node server.js     # API server
node worker.js     # Notification worker
```

---

### 4.3 Notification Service Layer

```javascript
// services/notificationService.js
const { notificationQueue } = require('../queue/notificationQueue');
const DeviceToken = require('../models/DeviceToken');
const rateLimit = require('express-rate-limit');

class NotificationService {

  // Send to a single user (enqueue)
  async sendToUser(userId, { title, body, data }) {
    return await notificationQueue.add('send', {
      userIds: [userId],
      title,
      body,
      data,
    });
  }

  // Send to multiple users (bulk enqueue)
  async sendToUsers(userIds, { title, body, data }) {
    // Break into chunks of 1000 users per job
    const chunkSize = 1000;
    const jobs = [];

    for (let i = 0; i < userIds.length; i += chunkSize) {
      const chunk = userIds.slice(i, i + chunkSize);
      const job = await notificationQueue.add('send-bulk', {
        userIds: chunk,
        title,
        body,
        data,
      });
      jobs.push(job.id);
    }

    return { jobIds: jobs };
  }

  // Schedule a notification for a future time
  async scheduleNotification(userIds, message, sendAt) {
    const delay = new Date(sendAt).getTime() - Date.now();

    if (delay < 0) throw new Error('Cannot schedule notification in the past');

    return await notificationQueue.add(
      'scheduled',
      { userIds, ...message },
      { delay }
    );
  }

  // Clean up inactive tokens older than 30 days
  async cleanupStaleTokens() {
    const thirtyDaysAgo = new Date(Date.now() - 30 * 24 * 60 * 60 * 1000);

    const result = await DeviceToken.deleteMany({
      isActive: false,
      updatedAt: { $lt: thirtyDaysAgo },
    });

    console.log(`Cleaned up ${result.deletedCount} stale tokens.`);
    return result.deletedCount;
  }
}

module.exports = new NotificationService();
```

#### Rate Limiting Middleware

```javascript
// middleware/rateLimiter.js
const rateLimit = require('express-rate-limit');

exports.notificationRateLimit = rateLimit({
  windowMs: 60 * 1000,    // 1 minute
  max: 20,                // Max 20 notification sends per minute per IP
  message: { error: 'Too many notification requests. Please try again later.' },
});
```

---

### 4.4 Firebase (Android FCM) Setup

FCM is required for push notifications on Android bare workflow apps.

#### Step 1 — Create Firebase Project

1. Go to [https://console.firebase.google.com](https://console.firebase.google.com)
2. Click **Add project** → enter your project name
3. Click **Add app** → choose **Android**
4. Enter your package name (must match `app.json` → `android.package`)
5. Download `google-services.json`
6. Place it in the **root of your Expo project** (same level as `app.json`)

#### Step 2 — Configure `app.json`

```json
{
  "expo": {
    "android": {
      "package": "com.yourcompany.myapp",
      "googleServicesFile": "./google-services.json"
    }
  }
}
```

#### Step 3 — Gradle Configuration

In `android/build.gradle` (project-level):

```gradle
buildscript {
    dependencies {
        // Add this line:
        classpath 'com.google.gms:google-services:4.4.0'
    }
}
```

In `android/app/build.gradle` (app-level):

```gradle
// At the very bottom of the file, add:
apply plugin: 'com.google.gms.google-services'
```

#### Step 4 — Rebuild the App

After adding FCM config, you must do a full native rebuild:

```bash
npx expo run:android
```

> **Important:** Do not use Expo Go after adding FCM. The bare workflow (`npx expo run:android`) is required.

#### Step 5 — Add FCM Server Key to Expo (for managed service)

If you use Expo's push service (recommended), add your FCM Server Key in the Expo dashboard:

1. In Firebase Console → Project Settings → Cloud Messaging
2. Copy the **Server Key**
3. Go to [https://expo.dev](https://expo.dev) → Your Project → Credentials
4. Add the FCM Server Key under Android credentials

---

## 5. Project Folder Structure

### Frontend (Expo React Native)

```
src/
├── notifications/
│   ├── registerForPushNotifications.js   # Permission request & token retrieval
│   └── notificationHandler.js            # Foreground handler config
│
├── hooks/
│   ├── usePushToken.js                   # Register token on mount, handle refresh
│   └── useNotificationListeners.js       # Foreground & tap listeners
│
└── services/
    └── notificationService.js            # API calls to backend
```

### Backend (Node.js + Express)

```
backend/
├── server.js                             # App entry point
├── worker.js                             # Queue worker entry point
│
├── controllers/
│   └── notificationController.js         # Register token, send notification
│
├── routes/
│   └── notificationRoutes.js             # Express router
│
├── services/
│   └── notificationService.js            # Business logic, scheduling
│
├── models/
│   └── DeviceToken.js                    # Mongoose/DB schema
│
├── queue/
│   ├── notificationQueue.js              # BullMQ queue definition
│   └── notificationWorker.js             # BullMQ worker
│
├── middleware/
│   ├── auth.js                           # Authentication middleware
│   └── rateLimiter.js                    # Rate limiting
│
└── .env
    # REDIS_URL=redis://localhost:6379
    # MONGODB_URI=mongodb://localhost:27017/myapp
    # PORT=3000
```

---

## 6. Common Errors & Fixes

### ❌ Notifications Not Received

**Symptom:** No notification appears on device after calling the API.

**Checklist:**
1. Are you using a real device? Notifications often don't work on emulators without extra setup.
2. Did the device register a token? Check your backend logs for `Token registered: ExponentPushToken[...]`.
3. Is the token a valid `ExponentPushToken`? Check with `Expo.isExpoPushToken(token)`.
4. Is `expo-notifications` installed correctly? Run `npx expo install expo-notifications` again.
5. For Android bare workflow: did you rebuild with `npx expo run:android` after adding FCM?

---

### ❌ Notifications Not Working on Emulator

**Symptom:** Works on real device but not on Android emulator.

**Fix:** Android emulators can receive FCM notifications if they have Google Play Services installed. Use an AVD (Android Virtual Device) with "Google Play" in the System Image name.

```bash
# Use a physical device for reliable testing
adb devices  # Confirm device is connected
```

For iOS simulator, push notifications are **not supported at all**. Use TestFlight or a real device.

---

### ❌ Token Invalidation (DeviceNotRegistered)

**Symptom:** Receipt shows `"error": "DeviceNotRegistered"`.

**Cause:** The user uninstalled the app or the token expired.

**Fix:** Handle this in receipt processing (already shown in the intermediate controller):

```javascript
if (receipt.details?.error === 'DeviceNotRegistered') {
  await DeviceToken.findOneAndUpdate(
    { token: receipt.details.expoPushToken },
    { isActive: false }
  );
}
```

---

### ❌ Firebase Misconfiguration

**Symptom:** `Error: Failed to send a push notification. Error: ... FCM error`

**Checklist:**
1. Is `google-services.json` in the project root (not `android/app/`)?
2. Does the `package` in `app.json` match exactly what's registered in Firebase?
3. Did you rebuild with `npx expo run:android` after adding the file?
4. Is the FCM Server Key added to Expo's dashboard credentials?
5. Check `android/app/build.gradle` — is `apply plugin: 'com.google.gms.google-services'` at the bottom?

---

### ❌ "Must use physical device" Error

**Symptom:** App shows alert "Must use a physical device".

**Fix:** This is intentional in the beginner setup. If you want to test on an emulator, replace the check:

```javascript
// Only block on iOS simulators; Android emulators with Google Play can work
if (!Device.isDevice && Platform.OS === 'ios') {
  console.warn('iOS simulator does not support push notifications.');
  return null;
}
```

---

### ❌ Network Error When Registering Token

**Symptom:** `fetch` fails when trying to send token to backend.

**Fix:** Use your machine's actual local IP address, not `localhost`:

```javascript
// Wrong on a physical device:
const BACKEND_URL = 'http://localhost:3000';

// Correct:
const BACKEND_URL = 'http://192.168.1.42:3000'; // Your machine's local IP
```

---

## 7. Optional Advanced Features

### 📌 Topic-Based Notifications

Send to all users subscribed to a topic (e.g., "sports", "deals") without tracking individual tokens.

```javascript
// Backend: Subscribe a user's token to a topic
// Note: Expo doesn't natively support topics — use FCM HTTP v1 API directly

const admin = require('firebase-admin');
admin.initializeApp({ credential: admin.credential.applicationDefault() });

async function subscribeToTopic(tokens, topic) {
  const response = await admin.messaging().subscribeToTopic(tokens, topic);
  console.log(`Subscribed ${response.successCount} devices to topic: ${topic}`);
}

async function sendToTopic(topic, { title, body }) {
  await admin.messaging().send({
    topic,
    notification: { title, body },
  });
}
```

---

### ⏰ Scheduled Notifications

Send notifications at a specific future time using BullMQ's delay feature:

```javascript
// Schedule a notification 24 hours from now
async function remindUser(userId, message) {
  const oneDayMs = 24 * 60 * 60 * 1000;

  await notificationQueue.add(
    'reminder',
    { userIds: [userId], ...message },
    { delay: oneDayMs }
  );
}
```

For local device-side scheduled notifications (no server needed):

```javascript
// In React Native — schedule a local notification
import * as Notifications from 'expo-notifications';

await Notifications.scheduleNotificationAsync({
  content: {
    title: 'Daily Reminder',
    body: "Don't forget to check in today!",
  },
  trigger: {
    hour: 9,
    minute: 0,
    repeats: true, // Every day at 9:00 AM
  },
});
```

---

### 🔕 Silent Notifications (Background Data Push)

Silent notifications wake the app in the background without showing anything to the user. Useful for syncing data.

```javascript
// Backend: Send a silent notification
const messages = [{
  to: token,
  title: '',
  body: '',
  data: { action: 'sync', resource: 'messages' },
  _contentAvailable: true,  // iOS silent notification
  priority: 'normal',       // Android background priority
}];
```

```javascript
// Frontend: Handle silent notification in background
// In app.json, register a background task:
TaskManager.defineTask('BACKGROUND_NOTIFICATION_TASK', async ({ data, error }) => {
  if (error) return;
  console.log('Received silent notification:', data.notification.request.content.data);
  // Perform background sync here
});

Notifications.registerTaskAsync('BACKGROUND_NOTIFICATION_TASK');
```

---

### 🎯 User Segmentation

Send targeted notifications based on user properties (plan, location, behavior):

```javascript
// Notification service with segmentation
async function sendToSegment(segment, message) {
  // segment = { plan: 'premium', country: 'IN', lastActiveAfter: '2025-01-01' }

  const users = await User.find({
    ...(segment.plan && { plan: segment.plan }),
    ...(segment.country && { country: segment.country }),
    ...(segment.lastActiveAfter && {
      lastActiveAt: { $gte: new Date(segment.lastActiveAfter) },
    }),
  }).select('_id');

  const userIds = users.map((u) => u._id);
  return notificationService.sendToUsers(userIds, message);
}
```

---

## Quick Reference: Beginner → Advanced Checklist

| Step | Feature | Level |
|------|---------|-------|
| ✅ | Install `expo-notifications` | Beginner |
| ✅ | Request permission & get token | Beginner |
| ✅ | Send token to backend | Beginner |
| ✅ | Basic Express endpoint to send notification | Beginner |
| ✅ | Handle permission states properly | Intermediate |
| ✅ | Android notification channel | Intermediate |
| ✅ | Notification tap handler (navigation) | Intermediate |
| ✅ | Store tokens in database | Intermediate |
| ✅ | Multi-device support per user | Intermediate |
| ✅ | Receipt checking & token cleanup | Intermediate |
| ✅ | BullMQ + Redis job queue | Advanced |
| ✅ | Background worker process | Advanced |
| ✅ | Retry with exponential backoff | Advanced |
| ✅ | FCM full Android setup | Advanced |
| ✅ | Scheduled notifications | Advanced |
| ✅ | Rate limiting | Advanced |
| ✅ | User segmentation | Advanced |

---

*This guide covers Expo SDK 50+, expo-notifications 0.28+, and Node.js 18+. Always check the [Expo Notifications docs](https://docs.expo.dev/versions/latest/sdk/notifications/) for the latest API changes.*
