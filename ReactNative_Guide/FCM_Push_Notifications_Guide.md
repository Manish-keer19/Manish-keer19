# 🔔 Firebase Cloud Messaging (FCM) — Complete Implementation Guide
### React Native (Expo Bare Workflow) + Node.js Backend

> **Audience:** Developers with basic React Native and Node.js knowledge  
> **Difficulty:** Beginner → Advanced  
> **Stack:** React Native (Expo Bare) · TypeScript · Node.js · Firebase Admin SDK · MongoDB

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [How FCM Works Internally](#2-how-fcm-works-internally)
3. [Project Setup](#3-project-setup)
4. [Native Configuration (Android)](#4-native-configuration-android)
5. [Frontend Implementation (React Native)](#5-frontend-implementation-react-native)
6. [Backend Setup (Node.js + TypeScript)](#6-backend-setup-nodejs--typescript)
7. [Sending Notifications](#7-sending-notifications)
8. [Handling Notifications in App](#8-handling-notifications-in-app)
9. [Advanced Concepts](#9-advanced-concepts)
10. [Production Best Practices](#10-production-best-practices)
11. [Common Issues & Fixes](#11-common-issues--fixes)
12. [Comparison: FCM vs Expo Push Notifications](#12-comparison-fcm-vs-expo-push-notifications)
13. [Conclusion](#13-conclusion)

---

## 1. Introduction

### What is Firebase Cloud Messaging (FCM)?

**Firebase Cloud Messaging (FCM)** is a cross-platform messaging solution by Google (part of Firebase) that allows you to reliably send notifications and data messages to Android, iOS, and web applications — for free, at any scale.

FCM acts as a **reliable bridge** between your server and end-user devices. It handles the complex infrastructure of message routing, device targeting, delivery guarantees, and battery-efficient delivery.

> FCM handles over **hundreds of billions of messages** per day globally.

---

### Why Use FCM Instead of Expo Push Notifications?

Expo Push Notifications is a convenient wrapper, but it abstracts away too much for production apps. Here is a direct comparison of why FCM is preferred:

| Concern | Expo Push Notifications | FCM (Direct) |
|---|---|---|
| Control | Limited | Full |
| Token format | Expo token (`ExponentPushToken[...]`) | Native FCM token |
| Delivery guarantee | Via Expo's relay servers | Direct to Google/Apple |
| Data messages (silent) | Limited support | Full support |
| Topic messaging | Not supported | Supported |
| Token management | Managed by Expo | You control it |
| Works without Expo SDK | ❌ | ✅ |
| Production readiness | Moderate | High |
| Custom payload | Limited | Unlimited |

**Use FCM directly when:**
- You need full control over notification delivery
- You use data-only (silent) messages to trigger background logic
- You need topic-based broadcasting
- You want to decouple from Expo's infrastructure
- You are building a production app for millions of users

---

### High-Level Architecture Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                        YOUR APPLICATION                          │
│                                                                  │
│   ┌─────────────────┐          ┌──────────────────────────────┐  │
│   │  React Native   │          │      Node.js Backend         │  │
│   │  (Mobile App)   │          │   (REST API + Firebase Admin)│  │
│   └────────┬────────┘          └──────────────┬───────────────┘  │
│            │  FCM Token                       │  Send Notification│
│            │  (on app start)                  │  via Admin SDK    │
│            ▼                                  ▼                   │
│   ┌─────────────────────────────────────────────────────────┐    │
│   │                  Firebase (Google Servers)               │    │
│   │                                                         │    │
│   │   Token Registry  ──►  Message Router  ──►  Delivery   │    │
│   └─────────────────────────────────────────────────────────┘    │
│            │                                                      │
│            ▼                                                      │
│   ┌─────────────────────────────────────────────────────────┐    │
│   │             Google Play Services (Android)               │    │
│   │             APNs (iOS)                                  │    │
│   └──────────────────────────┬──────────────────────────────┘    │
│                              │                                    │
│                              ▼                                    │
│                     User's Device / App                           │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. How FCM Works Internally

### End-to-End Flow

```
Step 1: App Registration
────────────────────────
App starts on device
    │
    ▼
@react-native-firebase/messaging calls Firebase SDK
    │
    ▼
Firebase SDK contacts Google Play Services (Android)
    │
    ▼
Firebase server generates a unique FCM Registration Token
    │
    ▼
Token returned to app → App sends token to your backend
    │
    ▼
Backend stores token in database (linked to user ID)


Step 2: Sending a Notification
───────────────────────────────
Your backend prepares a notification payload
    │
    ▼
Backend calls Firebase Admin SDK → POST to FCM API
    │
    ▼
Firebase validates the token and routes the message
    │
    ▼
Firebase pushes message to Google Play Services on the device
    │
    ▼
Google Play Services delivers it to the app
    │
    ▼
App receives notification and handles it (foreground/background/quit)
```

---

### Role of Key Components

#### FCM Token
- A **unique string** identifying a specific app installation on a specific device.
- Format: long alphanumeric string (`dGhpcyBpcyBhbiBleGFtcGxlIHRva2Vu...`)
- **One token per app-device combination** — reinstalling the app generates a new token.
- Used by your backend to target a specific device.

#### Firebase Servers
- Act as a **message broker and router**.
- Authenticate your server using a service account key.
- Queue and retry message delivery if the device is offline.
- Provide delivery receipts and analytics.

#### Google Play Services (Android)
- A system-level service on every Android device.
- Maintains a **persistent TCP connection** to Google servers.
- Receives messages on behalf of all installed apps — this is why FCM is battery-efficient (one connection for all apps, not one per app).

---

### Token Lifecycle

```
Token Generated
     │
     │  (app install / first launch)
     ▼
Token Valid & Active ◄──────────────────────────────┐
     │                                              │
     │  Triggered by:                               │  Token refresh
     │  - App reinstall                             │  (onTokenRefresh)
     │  - Clear app data                            │
     │  - New device                                │
     │  - Token rotation (security)                 │
     ▼                                              │
Token Invalidated ──────────────────────────────────┘
     │
     ▼
Old token removed from DB + new token saved
```

**Key lifecycle events:**
- `getToken()` — get current token on startup
- `onTokenRefresh()` — fires when token is refreshed; **must update your backend**
- Token can expire silently — backend should handle `INVALID_REGISTRATION` errors

---

### Delivery Pipeline (Step by Step)

```
1. Your server calls: admin.messaging().send(message)
        │
        ▼
2. Firebase Admin SDK sends HTTP POST to:
   https://fcm.googleapis.com/v1/projects/{project_id}/messages:send
        │
        ▼
3. Firebase validates the service account credentials
        │
        ▼
4. Firebase looks up the FCM token → finds the target device
        │
        ▼
5. Firebase routes to the correct transport:
   - Android → Google Play Services (via XMPP or HTTP/2)
   - iOS     → Apple Push Notification service (APNs)
   - Web     → Browser Push API
        │
        ▼
6. If device is ONLINE:
   Message delivered immediately → App receives it
        │
   If device is OFFLINE:
   Firebase stores message (up to 4 weeks, configurable TTL)
   Delivered when device comes back online
        │
        ▼
7. Delivery receipt returned to Firebase
8. You can query delivery stats in Firebase Console
```

---

## 3. Project Setup

### Step 1 — Create a Firebase Project

1. Go to [https://console.firebase.google.com](https://console.firebase.google.com)
2. Click **"Add project"**
3. Enter a project name (e.g., `MyApp`)
4. Disable Google Analytics (optional for notifications)
5. Click **"Create project"**

---

### Step 2 — Add Android App

1. In Firebase Console, click the **Android icon** to add an Android app.
2. Enter your **Android package name** (must match your `app.json` / `build.gradle`):
   ```
   com.yourcompany.yourapp
   ```
3. Enter a nickname (optional) and **SHA-1** (needed for some features, not required for FCM).
4. Click **"Register app"**.

---

### Step 3 — Download and Place `google-services.json`

1. Download `google-services.json` from the Firebase Console.
2. Place it in your React Native project at:
   ```
   android/app/google-services.json
   ```

> ⚠️ **Never commit this file to a public repository.** Add it to `.gitignore` if needed, or use CI/CD secrets.

---

### Step 4 — Install Dependencies

```bash
# Install React Native Firebase core
npm install @react-native-firebase/app

# Install FCM messaging module
npm install @react-native-firebase/messaging

# Install notifee (for custom notification UI in foreground)
npm install @notifee/react-native

# Rebuild native code (Expo bare workflow)
npx expo prebuild
# or
cd android && ./gradlew clean && cd ..
```

---

## 4. Native Configuration (Android)

### `android/build.gradle` (Project-level)

```groovy
buildscript {
    dependencies {
        // Add the Google Services plugin
        classpath 'com.google.gms:google-services:4.4.0'
    }
}
```

### `android/app/build.gradle` (App-level)

```groovy
apply plugin: 'com.android.application'

// ADD THIS LINE at the bottom of the file
apply plugin: 'com.google.gms.google-services'

android {
    defaultConfig {
        applicationId "com.yourcompany.yourapp"
        // ...
    }
}

dependencies {
    // Firebase BoM (Bill of Materials) for consistent versions
    implementation platform('com.google.firebase:firebase-bom:32.7.0')
    implementation 'com.google.firebase:firebase-messaging'
}
```

---

### Required Permissions — `AndroidManifest.xml`

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- Required for notifications on Android 13+ -->
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>

    <!-- For receiving FCM messages -->
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.WAKE_LOCK"/>

    <application>

        <!-- FCM Service -->
        <service
            android:name="io.invertase.firebase.messaging.ReactNativeFirebaseMessagingService"
            android:exported="false">
            <intent-filter>
                <action android:name="com.google.firebase.MESSAGING_EVENT"/>
            </intent-filter>
        </service>

        <!-- Default notification channel (Android 8+) -->
        <meta-data
            android:name="com.google.firebase.messaging.default_notification_channel_id"
            android:value="default"/>

        <!-- Default notification icon -->
        <meta-data
            android:name="com.google.firebase.messaging.default_notification_icon"
            android:resource="@drawable/ic_notification"/>

        <!-- Default notification color -->
        <meta-data
            android:name="com.google.firebase.messaging.default_notification_color"
            android:resource="@color/notification_color"/>

    </application>
</manifest>
```

---

### Create Notification Channel (Required for Android 8+)

Create `android/app/src/main/java/com/yourapp/MainApplication.java` or add to existing:

```java
import com.google.firebase.messaging.FirebaseMessaging;
import android.app.NotificationChannel;
import android.app.NotificationManager;
import android.os.Build;

// In onCreate():
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
    NotificationChannel channel = new NotificationChannel(
        "default",
        "Default Channel",
        NotificationManager.IMPORTANCE_HIGH
    );
    channel.setDescription("Default notification channel");
    NotificationManager notificationManager = getSystemService(NotificationManager.class);
    notificationManager.createNotificationChannel(channel);
}
```

---

## 5. Frontend Implementation (React Native)

### `src/services/notificationService.ts`

```typescript
import messaging from '@react-native-firebase/messaging';
import { Alert, Platform } from 'react-native';
import axios from 'axios';

const API_BASE_URL = 'https://your-backend.com/api';

// ─────────────────────────────────────────────────
// 1. REQUEST NOTIFICATION PERMISSION
// ─────────────────────────────────────────────────
export async function requestNotificationPermission(): Promise<boolean> {
  if (Platform.OS === 'android' && Platform.Version >= 33) {
    // Android 13+ requires explicit permission
    const { PermissionsAndroid } = require('react-native');
    const result = await PermissionsAndroid.request(
      PermissionsAndroid.PERMISSIONS.POST_NOTIFICATIONS
    );
    return result === PermissionsAndroid.RESULTS.GRANTED;
  }

  if (Platform.OS === 'ios') {
    const authStatus = await messaging().requestPermission();
    const enabled =
      authStatus === messaging.AuthorizationStatus.AUTHORIZED ||
      authStatus === messaging.AuthorizationStatus.PROVISIONAL;
    return enabled;
  }

  return true; // Android < 13 doesn't need explicit permission
}

// ─────────────────────────────────────────────────
// 2. GET FCM TOKEN
// ─────────────────────────────────────────────────
export async function getFCMToken(): Promise<string | null> {
  try {
    // Check if app is registered for remote messages (iOS)
    if (!messaging().isDeviceRegisteredForRemoteMessages) {
      await messaging().registerDeviceForRemoteMessages();
    }

    const token = await messaging().getToken();
    console.log('FCM Token:', token);
    return token;
  } catch (error) {
    console.error('Error getting FCM token:', error);
    return null;
  }
}

// ─────────────────────────────────────────────────
// 3. SEND TOKEN TO BACKEND
// ─────────────────────────────────────────────────
export async function registerTokenWithBackend(
  userId: string,
  token: string
): Promise<void> {
  try {
    await axios.post(`${API_BASE_URL}/notifications/register-token`, {
      userId,
      token,
      platform: Platform.OS,
      deviceInfo: {
        os: Platform.OS,
        version: Platform.Version,
      },
    });
    console.log('Token registered with backend successfully');
  } catch (error) {
    console.error('Failed to register token with backend:', error);
  }
}

// ─────────────────────────────────────────────────
// 4. LISTEN FOR TOKEN REFRESH
// ─────────────────────────────────────────────────
export function listenForTokenRefresh(
  userId: string,
  onNewToken?: (token: string) => void
): () => void {
  return messaging().onTokenRefresh(async (newToken) => {
    console.log('FCM Token refreshed:', newToken);
    await registerTokenWithBackend(userId, newToken);
    onNewToken?.(newToken);
  });
}
```

---

### `src/hooks/useNotifications.ts`

```typescript
import { useEffect, useRef } from 'react';
import messaging, { FirebaseMessagingTypes } from '@react-native-firebase/messaging';
import notifee, { AndroidImportance } from '@notifee/react-native';
import {
  requestNotificationPermission,
  getFCMToken,
  registerTokenWithBackend,
  listenForTokenRefresh,
} from '../services/notificationService';

interface UseNotificationsOptions {
  userId: string;
  onForegroundMessage?: (message: FirebaseMessagingTypes.RemoteMessage) => void;
  onNotificationTap?: (message: FirebaseMessagingTypes.RemoteMessage) => void;
}

export function useNotifications({
  userId,
  onForegroundMessage,
  onNotificationTap,
}: UseNotificationsOptions) {
  const unsubscribersRef = useRef<Array<() => void>>([]);

  useEffect(() => {
    initializeNotifications();

    return () => {
      // Cleanup all listeners
      unsubscribersRef.current.forEach((unsub) => unsub());
    };
  }, [userId]);

  async function initializeNotifications() {
    // Step 1: Request permission
    const hasPermission = await requestNotificationPermission();
    if (!hasPermission) {
      console.warn('Notification permission denied');
      return;
    }

    // Step 2: Create notification channel (Android)
    await createNotificationChannel();

    // Step 3: Get and register FCM token
    const token = await getFCMToken();
    if (token) {
      await registerTokenWithBackend(userId, token);
    }

    // Step 4: Set up token refresh listener
    const unsubRefresh = listenForTokenRefresh(userId);
    unsubscribersRef.current.push(unsubRefresh);

    // Step 5: Handle foreground messages
    const unsubForeground = setupForegroundHandler();
    unsubscribersRef.current.push(unsubForeground);

    // Step 6: Handle background/quit state notification taps
    setupBackgroundHandler();
    setupQuitStateHandler();
  }

  // ──────────────────────────────────────────────────
  // FOREGROUND: App is open and visible
  // ──────────────────────────────────────────────────
  function setupForegroundHandler(): () => void {
    return messaging().onMessage(async (remoteMessage) => {
      console.log('Foreground message received:', remoteMessage);

      // FCM does NOT show a notification UI when app is in foreground
      // You must manually display it using notifee
      await displayLocalNotification(remoteMessage);

      onForegroundMessage?.(remoteMessage);
    });
  }

  // ──────────────────────────────────────────────────
  // BACKGROUND: App is open but in background
  // ──────────────────────────────────────────────────
  function setupBackgroundHandler() {
    messaging().onNotificationOpenedApp((remoteMessage) => {
      console.log('Background notification tapped:', remoteMessage);
      onNotificationTap?.(remoteMessage);
      handleDeepLink(remoteMessage);
    });
  }

  // ──────────────────────────────────────────────────
  // QUIT STATE: App was completely closed
  // ──────────────────────────────────────────────────
  async function setupQuitStateHandler() {
    const initialMessage = await messaging().getInitialNotification();
    if (initialMessage) {
      console.log('App opened from quit state notification:', initialMessage);
      onNotificationTap?.(initialMessage);
      handleDeepLink(initialMessage);
    }
  }

  // ──────────────────────────────────────────────────
  // LOCAL NOTIFICATION (for foreground display)
  // ──────────────────────────────────────────────────
  async function displayLocalNotification(
    remoteMessage: FirebaseMessagingTypes.RemoteMessage
  ) {
    const { notification, data } = remoteMessage;
    if (!notification) return;

    await notifee.displayNotification({
      title: notification.title,
      body: notification.body,
      android: {
        channelId: 'default',
        importance: AndroidImportance.HIGH,
        pressAction: { id: 'default' },
        smallIcon: 'ic_notification',
      },
      data,
    });
  }

  // ──────────────────────────────────────────────────
  // DEEP LINK HANDLER
  // ──────────────────────────────────────────────────
  function handleDeepLink(remoteMessage: FirebaseMessagingTypes.RemoteMessage) {
    const screen = remoteMessage.data?.screen as string | undefined;
    const id = remoteMessage.data?.id as string | undefined;

    if (screen) {
      // Use your navigation ref here
      // navigationRef.current?.navigate(screen, { id });
      console.log(`Navigate to: ${screen}, id: ${id}`);
    }
  }

  async function createNotificationChannel() {
    await notifee.createChannel({
      id: 'default',
      name: 'Default Notifications',
      importance: AndroidImportance.HIGH,
      vibration: true,
      sound: 'default',
    });
  }
}
```

---

### Register Background Message Handler — `index.js`

> ⚠️ **This must be in your root `index.js` file**, not inside a component.

```javascript
import { AppRegistry } from 'react-native';
import messaging from '@react-native-firebase/messaging';
import App from './App';
import { name as appName } from './app.json';

// Background message handler — runs even when app is killed
messaging().setBackgroundMessageHandler(async (remoteMessage) => {
  console.log('Background message received:', remoteMessage);
  // You can process data here, but avoid heavy operations
  // Navigation and UI updates are NOT possible here
});

AppRegistry.registerComponent(appName, () => App);
```

---

## 6. Backend Setup (Node.js + TypeScript)

### Install Dependencies

```bash
npm install firebase-admin express mongoose
npm install -D typescript @types/express @types/node ts-node nodemon
```

---

### Service Account Setup

1. In Firebase Console → **Project Settings** → **Service accounts**
2. Click **"Generate new private key"**
3. Download the JSON file → save as `serviceAccountKey.json`
4. **Never commit this file** — add to `.gitignore`

```
# .gitignore
serviceAccountKey.json
.env
```

Store credentials in environment variables instead:

```env
# .env
FIREBASE_PROJECT_ID=your-project-id
FIREBASE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"
FIREBASE_CLIENT_EMAIL=firebase-adminsdk-xxx@your-project.iam.gserviceaccount.com
MONGODB_URI=mongodb://localhost:27017/yourapp
PORT=3000
```

---

### `src/config/firebase.ts` — Initialize Firebase Admin SDK

```typescript
import * as admin from 'firebase-admin';
import dotenv from 'dotenv';

dotenv.config();

// Initialize once (guard against re-initialization)
if (!admin.apps.length) {
  admin.initializeApp({
    credential: admin.credential.cert({
      projectId: process.env.FIREBASE_PROJECT_ID,
      privateKey: process.env.FIREBASE_PRIVATE_KEY?.replace(/\\n/g, '\n'),
      clientEmail: process.env.FIREBASE_CLIENT_EMAIL,
    }),
  });
}

export const firebaseAdmin = admin;
export const fcm = admin.messaging();
```

---

### `src/models/User.ts` — MongoDB Schema

```typescript
import mongoose, { Schema, Document } from 'mongoose';

export interface IFCMToken {
  token: string;
  platform: 'android' | 'ios' | 'web';
  deviceInfo?: Record<string, unknown>;
  createdAt: Date;
  lastUsed: Date;
}

export interface IUser extends Document {
  _id: string;
  email: string;
  fcmTokens: IFCMToken[];
  notificationPreferences: {
    pushEnabled: boolean;
    topics: string[];
  };
}

const FCMTokenSchema = new Schema<IFCMToken>({
  token: { type: String, required: true, unique: true },
  platform: { type: String, enum: ['android', 'ios', 'web'], required: true },
  deviceInfo: { type: Schema.Types.Mixed },
  createdAt: { type: Date, default: Date.now },
  lastUsed: { type: Date, default: Date.now },
});

const UserSchema = new Schema<IUser>({
  email: { type: String, required: true, unique: true },
  fcmTokens: [FCMTokenSchema],
  notificationPreferences: {
    pushEnabled: { type: Boolean, default: true },
    topics: [String],
  },
});

export const User = mongoose.model<IUser>('User', UserSchema);
```

---

### `src/services/notificationService.ts` — Backend Notification Service

```typescript
import { fcm } from '../config/firebase';
import { User } from '../models/User';
import * as admin from 'firebase-admin';

export interface NotificationPayload {
  title: string;
  body: string;
  imageUrl?: string;
  data?: Record<string, string>;
  screen?: string;
}

// ─────────────────────────────────────────────────
// SEND TO A SINGLE USER (all devices)
// ─────────────────────────────────────────────────
export async function sendToUser(
  userId: string,
  payload: NotificationPayload
): Promise<{ successCount: number; failureCount: number }> {
  const user = await User.findById(userId);
  if (!user || user.fcmTokens.length === 0) {
    return { successCount: 0, failureCount: 0 };
  }

  if (!user.notificationPreferences.pushEnabled) {
    console.log(`User ${userId} has push notifications disabled`);
    return { successCount: 0, failureCount: 0 };
  }

  const tokens = user.fcmTokens.map((t) => t.token);
  return sendToTokens(userId, tokens, payload);
}

// ─────────────────────────────────────────────────
// SEND TO SPECIFIC TOKENS
// ─────────────────────────────────────────────────
export async function sendToTokens(
  userId: string,
  tokens: string[],
  payload: NotificationPayload
): Promise<{ successCount: number; failureCount: number }> {
  if (tokens.length === 0) return { successCount: 0, failureCount: 0 };

  const message: admin.messaging.MulticastMessage = {
    tokens,
    notification: {
      title: payload.title,
      body: payload.body,
      imageUrl: payload.imageUrl,
    },
    data: {
      ...payload.data,
      // Navigation data for deep linking
      ...(payload.screen && { screen: payload.screen }),
      clickedAt: new Date().toISOString(),
    },
    android: {
      priority: 'high',
      notification: {
        channelId: 'default',
        priority: 'high',
        defaultSound: true,
      },
    },
    apns: {
      payload: {
        aps: {
          sound: 'default',
          badge: 1,
        },
      },
    },
  };

  try {
    const response = await fcm.sendEachForMulticast(message);
    console.log(
      `Sent to ${tokens.length} tokens: ${response.successCount} success, ${response.failureCount} failures`
    );

    // Handle invalid tokens
    await cleanupInvalidTokens(userId, tokens, response);

    return {
      successCount: response.successCount,
      failureCount: response.failureCount,
    };
  } catch (error) {
    console.error('Error sending multicast message:', error);
    throw error;
  }
}

// ─────────────────────────────────────────────────
// SEND TO ALL USERS
// ─────────────────────────────────────────────────
export async function sendToAllUsers(
  payload: NotificationPayload,
  batchSize = 500
): Promise<{ total: number; success: number; failed: number }> {
  const users = await User.find({
    'notificationPreferences.pushEnabled': true,
    'fcmTokens.0': { $exists: true },
  }).select('_id fcmTokens');

  // Flatten all tokens
  const allTokens = users.flatMap((user) =>
    user.fcmTokens.map((t) => t.token)
  );

  let totalSuccess = 0;
  let totalFailed = 0;

  // Process in batches (FCM limit: 500 per multicast)
  for (let i = 0; i < allTokens.length; i += batchSize) {
    const batch = allTokens.slice(i, i + batchSize);
    try {
      const result = await sendToTokens('', batch, payload);
      totalSuccess += result.successCount;
      totalFailed += result.failureCount;
      // Small delay between batches to avoid rate limiting
      if (i + batchSize < allTokens.length) {
        await delay(200);
      }
    } catch (err) {
      console.error(`Batch ${i / batchSize + 1} failed:`, err);
      totalFailed += batch.length;
    }
  }

  return { total: allTokens.length, success: totalSuccess, failed: totalFailed };
}

// ─────────────────────────────────────────────────
// CLEANUP INVALID TOKENS
// ─────────────────────────────────────────────────
async function cleanupInvalidTokens(
  userId: string,
  tokens: string[],
  response: admin.messaging.BatchResponse
): Promise<void> {
  const tokensToRemove: string[] = [];

  response.responses.forEach((resp, index) => {
    if (!resp.success) {
      const errorCode = resp.error?.code;
      if (
        errorCode === 'messaging/invalid-registration-token' ||
        errorCode === 'messaging/registration-token-not-registered'
      ) {
        tokensToRemove.push(tokens[index]);
      }
    }
  });

  if (tokensToRemove.length > 0 && userId) {
    await User.updateOne(
      { _id: userId },
      { $pull: { fcmTokens: { token: { $in: tokensToRemove } } } }
    );
    console.log(`Removed ${tokensToRemove.length} invalid tokens for user ${userId}`);
  }
}

const delay = (ms: number) => new Promise((resolve) => setTimeout(resolve, ms));
```

---

### `src/routes/notifications.ts` — API Routes

```typescript
import { Router, Request, Response } from 'express';
import { User } from '../models/User';
import {
  sendToUser,
  sendToAllUsers,
} from '../services/notificationService';

const router = Router();

// ─────────────────────────────────────────────────
// POST /api/notifications/register-token
// Save FCM token for a user
// ─────────────────────────────────────────────────
router.post('/register-token', async (req: Request, res: Response) => {
  try {
    const { userId, token, platform, deviceInfo } = req.body;

    if (!userId || !token || !platform) {
      return res.status(400).json({ error: 'userId, token, and platform are required' });
    }

    const user = await User.findById(userId);
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }

    // Check if token already exists — update lastUsed
    const existingTokenIndex = user.fcmTokens.findIndex(
      (t) => t.token === token
    );

    if (existingTokenIndex >= 0) {
      user.fcmTokens[existingTokenIndex].lastUsed = new Date();
    } else {
      // Add new token (cap at 5 tokens per user)
      if (user.fcmTokens.length >= 5) {
        user.fcmTokens.shift(); // Remove oldest
      }
      user.fcmTokens.push({ token, platform, deviceInfo, createdAt: new Date(), lastUsed: new Date() });
    }

    await user.save();
    return res.json({ success: true, message: 'Token registered' });
  } catch (error) {
    console.error('Error registering token:', error);
    return res.status(500).json({ error: 'Internal server error' });
  }
});

// ─────────────────────────────────────────────────
// POST /api/notifications/send-to-user
// Send notification to a specific user
// ─────────────────────────────────────────────────
router.post('/send-to-user', async (req: Request, res: Response) => {
  try {
    const { userId, title, body, data, screen } = req.body;

    const result = await sendToUser(userId, { title, body, data, screen });
    return res.json({ success: true, ...result });
  } catch (error) {
    console.error('Error sending notification:', error);
    return res.status(500).json({ error: 'Failed to send notification' });
  }
});

// ─────────────────────────────────────────────────
// POST /api/notifications/broadcast
// Send notification to all users
// ─────────────────────────────────────────────────
router.post('/broadcast', async (req: Request, res: Response) => {
  try {
    const { title, body, data } = req.body;
    const result = await sendToAllUsers({ title, body, data });
    return res.json({ success: true, ...result });
  } catch (error) {
    console.error('Error broadcasting notification:', error);
    return res.status(500).json({ error: 'Failed to broadcast' });
  }
});

// ─────────────────────────────────────────────────
// DELETE /api/notifications/remove-token
// Remove a specific FCM token (e.g., on logout)
// ─────────────────────────────────────────────────
router.delete('/remove-token', async (req: Request, res: Response) => {
  try {
    const { userId, token } = req.body;

    await User.updateOne(
      { _id: userId },
      { $pull: { fcmTokens: { token } } }
    );

    return res.json({ success: true, message: 'Token removed' });
  } catch (error) {
    return res.status(500).json({ error: 'Failed to remove token' });
  }
});

export default router;
```

---

## 7. Sending Notifications

### Notification Payload vs Data Payload

| | Notification Payload | Data Payload |
|---|---|---|
| **What it is** | System-level notification shown by OS | Custom key-value data, handled by app |
| **Shown when app is in background** | ✅ Automatically | ❌ App must handle it |
| **Shown when app is in foreground** | ❌ Must handle manually | ❌ Must handle manually |
| **Can trigger background code** | Limited | ✅ Full execution |
| **Fields** | `title`, `body`, `imageUrl` | Any string key-value pairs |
| **Use case** | Standard alerts | Sync data, silent updates |

---

### Basic Notification Message

```typescript
const message: admin.messaging.Message = {
  token: 'USER_FCM_TOKEN_HERE',
  notification: {
    title: 'New Message! 💬',
    body: 'John sent you a message: "Are you coming tonight?"',
  },
  android: {
    priority: 'high',
    notification: {
      channelId: 'default',
      sound: 'default',
    },
  },
};

const messageId = await fcm.send(message);
console.log('Sent message ID:', messageId);
```

---

### Data-Only Message (Silent Notification)

```typescript
// Use for: syncing data, triggering background refresh, badge updates
const silentMessage: admin.messaging.Message = {
  token: 'USER_FCM_TOKEN_HERE',
  data: {
    type: 'CHAT_UPDATE',
    chatId: '12345',
    unreadCount: '5',
    refreshFeed: 'true',
  },
  android: {
    priority: 'high',
    // No notification block = silent/data-only message
  },
  apns: {
    headers: {
      'apns-push-type': 'background',
      'apns-priority': '5', // Lower priority for background
    },
    payload: {
      aps: {
        'content-available': 1, // Required for iOS background delivery
      },
    },
  },
};

await fcm.send(silentMessage);
```

---

### Multicast Messaging (Up to 500 Tokens)

```typescript
const multicastMessage: admin.messaging.MulticastMessage = {
  tokens: ['token1', 'token2', 'token3'], // Up to 500
  notification: {
    title: '🎉 Flash Sale!',
    body: 'Get 50% off in the next 2 hours',
  },
  data: {
    screen: 'SaleScreen',
    saleId: 'FLASH2024',
  },
};

const response = await fcm.sendEachForMulticast(multicastMessage);
console.log(`Success: ${response.successCount}, Failed: ${response.failureCount}`);
```

---

### Topic-Based Messaging

```typescript
// Subscribe a device to a topic (from backend)
await fcm.subscribeToTopic(['token1', 'token2'], 'sports-news');

// Send to all subscribers of a topic
const topicMessage: admin.messaging.Message = {
  topic: 'sports-news',
  notification: {
    title: '⚽ Goal Alert!',
    body: 'Manchester City scored in the 89th minute!',
  },
};

await fcm.send(topicMessage);
```

---

## 8. Handling Notifications in App

### All Notification States

```
┌─────────────────────────────────────────────────────────────────────┐
│                    APP NOTIFICATION STATES                          │
│                                                                     │
│  ┌─────────────┐    ┌─────────────────┐    ┌──────────────────┐   │
│  │  FOREGROUND │    │   BACKGROUND    │    │   QUIT / KILLED  │   │
│  │  (app open) │    │  (app in tray) │    │  (app closed)    │   │
│  └──────┬──────┘    └────────┬────────┘    └────────┬─────────┘   │
│         │                   │                       │             │
│         ▼                   ▼                       ▼             │
│  messaging()         System shows       System shows notification │
│  .onMessage()        notification       on device                 │
│  fires               automatically      (no code runs)           │
│         │                   │                                     │
│         │            User taps notification                       │
│         │                   │                       │             │
│         ▼                   ▼                       ▼             │
│  Must manually       .onNotification    .getInitialNotification() │
│  show UI with        OpenedApp()        fires on next app open   │
│  notifee             fires                                        │
└─────────────────────────────────────────────────────────────────────┘
```

---

### Complete Notification Handler

```typescript
// src/navigation/NotificationHandler.tsx
import { useEffect, useRef } from 'react';
import messaging, { FirebaseMessagingTypes } from '@react-native-firebase/messaging';
import { useNavigation } from '@react-navigation/native';

export function NotificationHandler() {
  const navigation = useNavigation<any>();
  const navigationReady = useRef(false);

  useEffect(() => {
    navigationReady.current = true;

    // ── FOREGROUND ──
    const unsubForeground = messaging().onMessage(async (remoteMessage) => {
      handleMessage(remoteMessage, false); // Don't navigate, just show notification
    });

    // ── BACKGROUND TAP ──
    messaging().onNotificationOpenedApp((remoteMessage) => {
      handleMessage(remoteMessage, true); // Navigate based on data
    });

    // ── QUIT STATE TAP ──
    messaging().getInitialNotification().then((remoteMessage) => {
      if (remoteMessage) {
        // Small delay to ensure navigation is ready
        setTimeout(() => handleMessage(remoteMessage, true), 500);
      }
    });

    return () => {
      unsubForeground();
    };
  }, []);

  function handleMessage(
    message: FirebaseMessagingTypes.RemoteMessage,
    shouldNavigate: boolean
  ) {
    const { data, notification } = message;

    console.log('Notification received:', {
      title: notification?.title,
      data,
    });

    if (shouldNavigate && data?.screen) {
      navigateToScreen(data.screen as string, data);
    }
  }

  function navigateToScreen(screen: string, params: Record<string, string>) {
    if (!navigationReady.current) return;

    switch (screen) {
      case 'ChatScreen':
        navigation.navigate('Chat', { chatId: params.chatId });
        break;
      case 'OrderScreen':
        navigation.navigate('Orders', { orderId: params.orderId });
        break;
      case 'ProfileScreen':
        navigation.navigate('Profile');
        break;
      default:
        navigation.navigate('Home');
    }
  }

  return null; // Render-less component
}
```

---

## 9. Advanced Concepts

### Token Management Strategy

For production apps with millions of users, use a robust token management approach:

```typescript
// src/services/tokenManager.ts

interface TokenRecord {
  token: string;
  userId: string;
  platform: string;
  createdAt: Date;
  lastSeen: Date;
  isActive: boolean;
}

export class TokenManager {
  // Store token with deduplication
  async upsertToken(userId: string, token: string, platform: string): Promise<void> {
    await User.findOneAndUpdate(
      { _id: userId, 'fcmTokens.token': token },
      {
        $set: { 'fcmTokens.$.lastUsed': new Date() },
      },
      { upsert: false }
    );

    // If not found (new token), add it
    await User.findOneAndUpdate(
      { _id: userId, 'fcmTokens.token': { $ne: token } },
      {
        $push: {
          fcmTokens: {
            $each: [{ token, platform, createdAt: new Date(), lastUsed: new Date() }],
            $slice: -5, // Keep only last 5 tokens
          },
        },
      }
    );
  }

  // Remove token on logout
  async removeToken(userId: string, token: string): Promise<void> {
    await User.updateOne(
      { _id: userId },
      { $pull: { fcmTokens: { token } } }
    );
  }

  // Purge tokens older than 30 days
  async purgeStaleTokens(): Promise<number> {
    const thirtyDaysAgo = new Date(Date.now() - 30 * 24 * 60 * 60 * 1000);
    const result = await User.updateMany(
      {},
      {
        $pull: {
          fcmTokens: { lastUsed: { $lt: thirtyDaysAgo } },
        },
      }
    );
    return result.modifiedCount;
  }
}
```

---

### Topic Subscriptions — Frontend

```typescript
import messaging from '@react-native-firebase/messaging';

// Subscribe to topics based on user preferences
export async function subscribeToTopics(topics: string[]): Promise<void> {
  for (const topic of topics) {
    await messaging().subscribeToTopic(topic);
    console.log(`Subscribed to topic: ${topic}`);
  }
}

// Unsubscribe when user changes preferences
export async function unsubscribeFromTopic(topic: string): Promise<void> {
  await messaging().unsubscribeFromTopic(topic);
}

// Example usage
await subscribeToTopics(['sports', 'breaking-news', 'offers']);
```

---

### Scheduling Notifications (Backend)

```typescript
import cron from 'node-cron';
import { sendToAllUsers } from './notificationService';

// Schedule a daily reminder at 9 AM
cron.schedule('0 9 * * *', async () => {
  await sendToAllUsers({
    title: '☀️ Good Morning!',
    body: 'Check out today\'s top deals',
    data: { screen: 'HomeScreen' },
  });
});

// One-time delayed notification
export async function scheduleNotification(
  userId: string,
  payload: NotificationPayload,
  sendAfterMs: number
): Promise<void> {
  setTimeout(async () => {
    await sendToUser(userId, payload);
  }, sendAfterMs);
}

// For persistent scheduling, use a queue (Bull/BullMQ)
import Queue from 'bull';

const notificationQueue = new Queue('notifications', process.env.REDIS_URL!);

notificationQueue.process(async (job) => {
  const { userId, payload } = job.data;
  await sendToUser(userId, payload);
});

export async function scheduleWithQueue(
  userId: string,
  payload: NotificationPayload,
  delay: number
): Promise<void> {
  await notificationQueue.add({ userId, payload }, { delay });
}
```

---

### Rate Limiting & Batching

```typescript
import Bottleneck from 'bottleneck';

// FCM rate limits: 600,000 messages per minute per project
const limiter = new Bottleneck({
  maxConcurrent: 10,
  minTime: 100, // 100ms between requests → max 10/sec
});

export async function sendWithRateLimit(
  message: admin.messaging.Message
): Promise<string> {
  return limiter.schedule(() => fcm.send(message));
}

// Batch processor
export async function processBatch(
  userIds: string[],
  payload: NotificationPayload
): Promise<void> {
  const chunks = chunkArray(userIds, 100); // Process 100 users at a time

  for (const chunk of chunks) {
    await Promise.all(chunk.map((userId) =>
      limiter.schedule(() => sendToUser(userId, payload))
    ));
  }
}

function chunkArray<T>(arr: T[], size: number): T[][] {
  return Array.from({ length: Math.ceil(arr.length / size) }, (_, i) =>
    arr.slice(i * size, i * size + size)
  );
}
```

---

## 10. Production Best Practices

### Security

```typescript
// ✅ CORRECT — Load credentials from environment
admin.initializeApp({
  credential: admin.credential.cert({
    projectId: process.env.FIREBASE_PROJECT_ID,
    privateKey: process.env.FIREBASE_PRIVATE_KEY?.replace(/\\n/g, '\n'),
    clientEmail: process.env.FIREBASE_CLIENT_EMAIL,
  }),
});

// ❌ WRONG — Never hardcode credentials
admin.initializeApp({
  credential: admin.credential.cert({
    projectId: 'my-project',
    privateKey: '-----BEGIN PRIVATE KEY-----\nABCDEF...',  // NEVER DO THIS
    clientEmail: 'firebase-adminsdk@my-project.iam.gserviceaccount.com',
  }),
});
```

---

### Error Handling & Retries

```typescript
import axios from 'axios';

export async function sendWithRetry(
  message: admin.messaging.Message,
  maxRetries = 3
): Promise<string | null> {
  let lastError: unknown;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const messageId = await fcm.send(message);
      return messageId;
    } catch (error: any) {
      lastError = error;
      const code = error?.errorInfo?.code;

      // Do not retry on permanent errors
      if (
        code === 'messaging/invalid-argument' ||
        code === 'messaging/registration-token-not-registered'
      ) {
        console.error(`Permanent FCM error (${code}), not retrying`);
        return null;
      }

      // Exponential backoff for transient errors
      if (attempt < maxRetries) {
        const backoffMs = Math.pow(2, attempt) * 1000;
        console.warn(`FCM send attempt ${attempt} failed, retrying in ${backoffMs}ms`);
        await delay(backoffMs);
      }
    }
  }

  console.error('All retry attempts failed:', lastError);
  return null;
}
```

---

### Logging & Monitoring

```typescript
import winston from 'winston';

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'notifications.log' }),
    new winston.transports.Console(),
  ],
});

export async function sendWithLogging(
  userId: string,
  payload: NotificationPayload
): Promise<void> {
  const startTime = Date.now();

  try {
    const result = await sendToUser(userId, payload);

    logger.info('Notification sent', {
      userId,
      title: payload.title,
      successCount: result.successCount,
      failureCount: result.failureCount,
      durationMs: Date.now() - startTime,
    });
  } catch (error) {
    logger.error('Notification failed', {
      userId,
      title: payload.title,
      error: error instanceof Error ? error.message : String(error),
      durationMs: Date.now() - startTime,
    });
  }
}
```

---

## 11. Common Issues & Fixes

### Token Not Generated

| Symptom | Cause | Fix |
|---|---|---|
| `getToken()` returns null | No permission granted | Call `requestPermission()` first |
| App crashes on `getToken()` | `google-services.json` missing or wrong location | Check `android/app/google-services.json` |
| Token generation works in debug, not release | SHA-1 fingerprint mismatch | Add release SHA-1 in Firebase Console |
| iOS: token is null | Device not registered | Call `registerDeviceForRemoteMessages()` |

```typescript
// Defensive token fetch
async function safeGetToken(): Promise<string | null> {
  try {
    if (Platform.OS === 'ios') {
      const registered = messaging().isDeviceRegisteredForRemoteMessages;
      if (!registered) {
        await messaging().registerDeviceForRemoteMessages();
      }
    }

    const token = await messaging().getToken();
    if (!token) throw new Error('Token is null');
    return token;
  } catch (err) {
    console.error('Token fetch failed:', err);
    return null;
  }
}
```

---

### Notifications Not Received

```
Checklist:
□ google-services.json is in android/app/ (not project root)
□ apply plugin: 'com.google.gms.google-services' is at bottom of app/build.gradle
□ Notification channel is created with IMPORTANCE_HIGH
□ Background handler is registered in index.js (not App.js)
□ Device is NOT in battery-saver or Doze mode (test with device plugged in)
□ Token sent to backend matches token in Firebase
□ Backend is using correct Firebase project
□ Emulator: Use a real device for push notification testing
```

---

### Foreground Notifications Not Showing

FCM does **not** automatically show notification UI when the app is in the foreground. You must manually display it using `@notifee/react-native`:

```typescript
messaging().onMessage(async (remoteMessage) => {
  // THIS IS REQUIRED — FCM does not show UI automatically in foreground
  await notifee.displayNotification({
    title: remoteMessage.notification?.title,
    body: remoteMessage.notification?.body,
    android: { channelId: 'default', importance: AndroidImportance.HIGH },
  });
});
```

---

### Android-Specific Issues

```typescript
// Issue: Notifications appear but without sound on Android 8+
// Fix: Create channel with sound before displaying any notification

await notifee.createChannel({
  id: 'default',
  name: 'Default',
  importance: AndroidImportance.HIGH,
  sound: 'default', // ← required
});

// Issue: Battery optimization kills background handler
// Fix: Guide user to disable battery optimization for your app
import { NativeModules, Linking } from 'react-native';

async function requestBatteryOptimizationExemption() {
  // On Android, prompt user to whitelist your app
  Linking.openSettings(); // Opens app settings where user can disable battery optimization
}

// Issue: Notifications not showing on Android 13+
// Fix: Request POST_NOTIFICATIONS permission at runtime
const { PermissionsAndroid } = require('react-native');
const granted = await PermissionsAndroid.request(
  PermissionsAndroid.PERMISSIONS.POST_NOTIFICATIONS
);
```

---

## 12. Comparison: FCM vs Expo Push Notifications

### Architecture Difference

```
Expo Push Notifications Flow:
──────────────────────────────
Your Backend → Expo's Servers → FCM/APNs → Device
              (intermediary)

FCM Direct Flow:
─────────────────
Your Backend → FCM/APNs → Device
              (direct)
```

### Feature Comparison

| Feature | Expo Push Notifications | FCM (Direct) |
|---|---|---|
| **Setup complexity** | Low | Medium |
| **Works without Expo** | ❌ | ✅ |
| **Data-only messages** | Limited | ✅ Full |
| **Topic messaging** | ❌ | ✅ |
| **Token format** | Expo token | Native FCM token |
| **Delivery reliability** | Depends on Expo servers | Direct to Google |
| **Customization** | Limited | Full |
| **Analytics** | Firebase Console | Firebase Console |
| **Cost** | Free | Free |
| **Production scale** | Moderate | High |
| **Debugging** | Easier | More involved |
| **SDK dependency** | Expo only | Any platform |

### When to Use Which

**Use Expo Push Notifications when:**
- Building a prototype or MVP quickly
- Using Expo managed workflow
- You don't need advanced features (topics, data messages)
- Your team is small and wants simplicity

**Use FCM directly when:**
- Building a production app
- Need data-only silent messages
- Need topic-based broadcasting
- Migrating away from Expo managed workflow
- Need maximum control and reliability
- Integrating with your own auth/user system

---

## 13. Conclusion

### Summary of Full Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       COMPLETE FCM FLOW SUMMARY                             │
│                                                                             │
│  1. APP STARTUP                                                             │
│     App opens → Request permission → Get FCM token → POST token to backend │
│                                                                             │
│  2. TOKEN STORAGE                                                           │
│     Backend saves token in MongoDB linked to user ID                       │
│                                                                             │
│  3. SEND NOTIFICATION                                                       │
│     Trigger (event/cron/user action) → Backend queries DB for token        │
│     → Firebase Admin SDK sends message → FCM delivers to device            │
│                                                                             │
│  4. RECEIVE NOTIFICATION                                                    │
│     Foreground: onMessage() fires → Show local notification via notifee    │
│     Background: OS shows notification → tap triggers onNotificationOpenedApp│
│     Quit state: OS shows notification → tap triggers getInitialNotification │
│                                                                             │
│  5. TOKEN MAINTENANCE                                                       │
│     onTokenRefresh() → Update backend with new token                       │
│     Invalid token errors → Auto-remove from DB                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### Recommended Architecture for Scaling

```
                           ┌──────────────────────┐
                           │   Load Balancer (Nginx)│
                           └──────────┬───────────┘
                                      │
              ┌───────────────────────┼────────────────────────┐
              ▼                       ▼                        ▼
     ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
     │  API Server 1   │   │  API Server 2   │   │  API Server 3   │
     │  (Express/TS)   │   │  (Express/TS)   │   │  (Express/TS)   │
     └────────┬────────┘   └────────┬────────┘   └────────┬────────┘
              │                     │                      │
              └─────────────────────┼──────────────────────┘
                                    │
                    ┌───────────────┴──────────────┐
                    │                              │
          ┌─────────▼──────────┐       ┌──────────▼──────────┐
          │  MongoDB Cluster   │       │   Redis (Bull Queue) │
          │  (Token Storage)   │       │   (Job Scheduling)   │
          └────────────────────┘       └─────────┬───────────┘
                                                 │
                                       ┌─────────▼───────────┐
                                       │  Worker Processes    │
                                       │  (Notification Jobs) │
                                       └─────────┬───────────┘
                                                 │
                                       ┌─────────▼───────────┐
                                       │  Firebase Admin SDK  │
                                       │  (FCM API calls)     │
                                       └─────────┬───────────┘
                                                 │
                                       ┌─────────▼───────────┐
                                       │  Firebase / FCM      │
                                       │  → Devices           │
                                       └─────────────────────┘
```

**Key principles for scale:**
- Use a **message queue (BullMQ + Redis)** to decouple notification sending from API requests
- Use **worker processes** to process the queue and call FCM
- **Batch multicast** messages — never send one token at a time at scale
- **Purge stale tokens** weekly to keep your database clean
- **Monitor delivery rates** in Firebase Console and set up alerts for drops
- Use **topics** for broad broadcasts instead of iterating over all user tokens
- Implement **circuit breakers** on the FCM API call to handle Firebase outages gracefully

---

> 📚 **Further Reading:**
> - [Firebase Cloud Messaging Documentation](https://firebase.google.com/docs/cloud-messaging)
> - [React Native Firebase Messaging](https://rnfirebase.io/messaging/usage)
> - [Notifee Documentation](https://notifee.app/react-native/docs/overview)
> - [Firebase Admin SDK Node.js](https://firebase.google.com/docs/admin/setup)

---

*Guide version: 1.0 | Last updated: 2024 | Stack: React Native 0.73+ · Firebase 21+ · Node.js 18+*
