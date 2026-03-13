# Expo Permissions Guide — Android & iOS

How to ask for ANY permission in Expo React Native.  
Simple language, real examples, step by step.

---

## What is a Permission?

A **permission** is when you ask the user:
> "Can my app use your camera / location / contacts / etc?"

The user can say **Yes** or **No**.

- If they say **Yes** → your app can use that feature
- If they say **No** → your app cannot use it

---

## How Permissions Work in Expo

There are **two ways** to handle permissions in Expo:

| Method | When to use |
|---|---|
| **expo-\* library** (e.g. `expo-camera`) | Most common — library handles permission for you |
| **expo-modules-core** or **expo-permissions** | When you need raw permission checks |

Most of the time you just install the feature library and it comes with its own permission function.

---

## Table of Contents

1. [Camera Permission](#1-camera-permission)
2. [Microphone Permission](#2-microphone-permission)
3. [Location Permission](#3-location-permission)
4. [Contacts Permission](#4-contacts-permission)
5. [Media Library / Gallery Permission](#5-media-library--gallery-permission)
6. [Notifications Permission](#6-notifications-permission)
7. [Calendar Permission](#7-calendar-permission)
8. [Bluetooth Permission](#8-bluetooth-permission)
9. [Motion / Pedometer Permission](#9-motion--pedometer-permission)
10. [How to Handle Permission Denied](#10-how-to-handle-permission-denied)
11. [Check Permission Without Asking](#11-check-permission-without-asking)
12. [app.json Setup for Each Permission](#12-appjson-setup-for-each-permission)
13. [Quick Reference Table](#13-quick-reference-table)

---

## 1. Camera Permission

### Install

```bash
npx expo install expo-camera
```

### Code

```tsx
import { CameraView, useCameraPermissions } from 'expo-camera';
import { View, Text, Pressable } from 'react-native';

export default function CameraScreen() {
  const [permission, requestPermission] = useCameraPermissions();

  // Still loading
  if (!permission) {
    return <View />;
  }

  // Permission not granted yet
  if (!permission.granted) {
    return (
      <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
        <Text>We need your camera permission</Text>
        <Pressable onPress={requestPermission}>
          <Text style={{ color: 'blue', marginTop: 10 }}>Allow Camera</Text>
        </Pressable>
      </View>
    );
  }

  // Permission granted — show camera
  return (
    <CameraView style={{ flex: 1 }} facing="back" />
  );
}
```

### What you get from `permission`

```tsx
permission.granted        // true or false
permission.status         // 'granted' | 'denied' | 'undetermined'
permission.canAskAgain    // true = can show popup again, false = user said never ask
```

---

## 2. Microphone Permission

### Install

```bash
npx expo install expo-av
```

### Code

```tsx
import { Audio } from 'expo-av';
import { useEffect } from 'react';

export default function MicrophoneExample() {
  useEffect(() => {
    askMicPermission();
  }, []);

  const askMicPermission = async () => {
    const { status } = await Audio.requestPermissionsAsync();

    if (status === 'granted') {
      console.log('Microphone allowed!');
    } else {
      console.log('Microphone denied');
    }
  };

  const startRecording = async () => {
    // Check first before recording
    const { status } = await Audio.getPermissionsAsync();
    if (status !== 'granted') {
      alert('Please allow microphone access first');
      return;
    }

    await Audio.setAudioModeAsync({ allowsRecordingIOS: true });
    const { recording } = await Audio.Recording.createAsync(
      Audio.RecordingOptionsPresets.HIGH_QUALITY
    );
    console.log('Recording started');
  };

  return null;
}
```

---

## 3. Location Permission

### Install

```bash
npx expo install expo-location
```

### Code

```tsx
import * as Location from 'expo-location';

export default function LocationExample() {

  // Step 1 — Ask for foreground (while app is open)
  const askForegroundLocation = async () => {
    const { status } = await Location.requestForegroundPermissionsAsync();

    if (status === 'granted') {
      const loc = await Location.getCurrentPositionAsync({});
      console.log('Your location:', loc.coords.latitude, loc.coords.longitude);
    } else {
      alert('Location permission denied');
    }
  };

  // Step 2 — Ask for background (when app is closed) — must do foreground first
  const askBackgroundLocation = async () => {
    const fg = await Location.requestForegroundPermissionsAsync();
    if (fg.status !== 'granted') return;

    const bg = await Location.requestBackgroundPermissionsAsync();
    if (bg.status === 'granted') {
      console.log('Background location allowed!');
    }
  };

  return null;
}
```

> **Rule:** Always ask foreground first. You cannot ask background without foreground.

---

## 4. Contacts Permission

### Install

```bash
npx expo install expo-contacts
```

### Code

```tsx
import * as Contacts from 'expo-contacts';

const getContacts = async () => {
  // Ask permission
  const { status } = await Contacts.requestPermissionsAsync();

  if (status !== 'granted') {
    alert('Contacts permission denied');
    return;
  }

  // Get contacts
  const { data } = await Contacts.getContactsAsync({
    fields: [Contacts.Fields.Name, Contacts.Fields.PhoneNumbers],
  });

  if (data.length > 0) {
    console.log('First contact:', data[0].name);
    console.log('All contacts count:', data.length);
  }
};
```

---

## 5. Media Library / Gallery Permission

### Install

```bash
npx expo install expo-media-library
```

### Code

```tsx
import * as MediaLibrary from 'expo-media-library';

const openGallery = async () => {
  // Ask permission to read photos
  const { status } = await MediaLibrary.requestPermissionsAsync();

  if (status !== 'granted') {
    alert('Gallery permission denied');
    return;
  }

  // Get photos from gallery
  const assets = await MediaLibrary.getAssetsAsync({
    mediaType: 'photo',
    first: 20, // get first 20 photos
  });

  console.log('Photos found:', assets.assets.length);
  assets.assets.forEach(photo => {
    console.log('Photo URI:', photo.uri);
  });
};

// Save a photo to gallery
const savePhotoToGallery = async (photoUri: string) => {
  const { status } = await MediaLibrary.requestPermissionsAsync();
  if (status !== 'granted') return;

  const asset = await MediaLibrary.createAssetAsync(photoUri);
  console.log('Saved to gallery:', asset.uri);
};
```

---

## 6. Notifications Permission

### Install

```bash
npx expo install expo-notifications
```

### Code

```tsx
import * as Notifications from 'expo-notifications';
import { Platform } from 'react-native';

const setupNotifications = async () => {
  // Ask permission
  const { status } = await Notifications.requestPermissionsAsync({
    ios: {
      allowAlert: true,
      allowBadge: true,
      allowSound: true,
    },
  });

  if (status !== 'granted') {
    alert('Notification permission denied');
    return;
  }

  console.log('Notifications allowed!');

  // Get push token (for sending remote notifications)
  const token = await Notifications.getExpoPushTokenAsync({
    projectId: 'your-project-id', // from app.json or EAS
  });

  console.log('Push token:', token.data);
  // Send this token to your server to send push notifications later
};

// Send a local notification right now
const sendLocalNotification = async () => {
  await Notifications.scheduleNotificationAsync({
    content: {
      title: 'Hello!',
      body: 'This is a test notification',
      sound: true,
    },
    trigger: null, // null = send immediately
  });
};
```

---

## 7. Calendar Permission

### Install

```bash
npx expo install expo-calendar
```

### Code

```tsx
import * as Calendar from 'expo-calendar';

const accessCalendar = async () => {
  // Ask permission
  const { status } = await Calendar.requestCalendarPermissionsAsync();

  if (status !== 'granted') {
    alert('Calendar permission denied');
    return;
  }

  // Get all calendars on the device
  const calendars = await Calendar.getCalendarsAsync(
    Calendar.EntityTypes.EVENT
  );
  console.log('Calendars:', calendars.map(c => c.title));

  // Get events from a calendar
  const startDate = new Date();
  const endDate = new Date();
  endDate.setDate(endDate.getDate() + 7); // next 7 days

  const events = await Calendar.getEventsAsync(
    [calendars[0].id],
    startDate,
    endDate
  );
  console.log('Upcoming events:', events.length);
};

// Ask permission to edit reminders (iOS only)
const accessReminders = async () => {
  const { status } = await Calendar.requestRemindersPermissionsAsync();
  if (status === 'granted') {
    console.log('Reminders access granted (iOS only)');
  }
};
```

---

## 8. Bluetooth Permission

### Install

```bash
npx expo install expo-bluetooth
# or for BLE (Bluetooth Low Energy)
npx expo install react-native-ble-plx
```

### Code using expo-modules approach

```tsx
import { PermissionsAndroid, Platform } from 'react-native';

// Android 12+ requires explicit Bluetooth permissions
const requestBluetoothPermission = async () => {
  if (Platform.OS === 'android') {
    const granted = await PermissionsAndroid.requestMultiple([
      PermissionsAndroid.PERMISSIONS.BLUETOOTH_SCAN,
      PermissionsAndroid.PERMISSIONS.BLUETOOTH_CONNECT,
      PermissionsAndroid.PERMISSIONS.ACCESS_FINE_LOCATION,
    ]);

    const allGranted = Object.values(granted).every(
      (status) => status === PermissionsAndroid.RESULTS.GRANTED
    );

    if (allGranted) {
      console.log('Bluetooth permissions granted');
    } else {
      console.log('Some Bluetooth permissions denied');
    }
  }

  // iOS handles Bluetooth permission automatically when you use it
};
```

### `app.json` for Bluetooth (Android)

```json
{
  "expo": {
    "android": {
      "permissions": [
        "BLUETOOTH",
        "BLUETOOTH_ADMIN",
        "BLUETOOTH_SCAN",
        "BLUETOOTH_CONNECT",
        "ACCESS_FINE_LOCATION"
      ]
    }
  }
}
```

---

## 9. Motion / Pedometer Permission

### Install

```bash
npx expo install expo-sensors
```

### Code

```tsx
import { Pedometer } from 'expo-sensors';
import { useEffect, useState } from 'react';

export default function StepCounter() {
  const [steps, setSteps] = useState(0);
  const [available, setAvailable] = useState(false);

  useEffect(() => {
    checkPedometer();
  }, []);

  const checkPedometer = async () => {
    // Check if permission is available
    const permission = await Pedometer.requestPermissionsAsync();

    if (permission.status !== 'granted') {
      alert('Motion permission denied');
      return;
    }

    // Check if device has a step counter
    const isAvailable = await Pedometer.isAvailableAsync();
    setAvailable(isAvailable);

    if (isAvailable) {
      // Watch steps in real time
      const subscription = Pedometer.watchStepCount((result) => {
        setSteps(result.steps);
      });

      return () => subscription.remove();
    }
  };

  return null; // add your UI here
}
```

---

## 10. How to Handle Permission Denied

When the user says NO, you need to handle it properly.

```tsx
import * as Location from 'expo-location';
import { Linking, Alert } from 'react-native';

const handlePermissionDenied = async () => {
  const { status, canAskAgain } = await Location.getForegroundPermissionsAsync();

  if (status === 'granted') {
    // All good!
    return;
  }

  if (canAskAgain) {
    // Can ask again — show the popup
    const { status: newStatus } = await Location.requestForegroundPermissionsAsync();
    if (newStatus !== 'granted') {
      showOpenSettingsAlert();
    }
  } else {
    // User clicked "Never ask again" — must open Settings manually
    showOpenSettingsAlert();
  }
};

const showOpenSettingsAlert = () => {
  Alert.alert(
    'Permission Required',
    'Please enable location permission in your phone Settings to use this feature.',
    [
      { text: 'Cancel', style: 'cancel' },
      {
        text: 'Open Settings',
        onPress: () => Linking.openSettings(), // opens phone settings
      },
    ]
  );
};
```

> **Key rule:** If `canAskAgain` is `false`, the popup will never show again. You must send the user to phone Settings with `Linking.openSettings()`.

---

## 11. Check Permission Without Asking

Sometimes you want to check if permission is already granted without showing a popup.

```tsx
import * as Location from 'expo-location';
import * as Camera from 'expo-camera';
import * as MediaLibrary from 'expo-media-library';

const checkAllPermissions = async () => {
  // Check location
  const location = await Location.getForegroundPermissionsAsync();
  console.log('Location:', location.status); // 'granted' | 'denied' | 'undetermined'

  // Check camera
  const camera = await Camera.getCameraPermissionsAsync();
  console.log('Camera:', camera.status);

  // Check gallery
  const gallery = await MediaLibrary.getPermissionsAsync();
  console.log('Gallery:', gallery.status);
};

// A reusable helper function
const isPermissionGranted = async (checkFn: () => Promise<{ status: string }>) => {
  const { status } = await checkFn();
  return status === 'granted';
};

// Usage
const hasCameraAccess = await isPermissionGranted(Camera.getCameraPermissionsAsync);
const hasLocationAccess = await isPermissionGranted(Location.getForegroundPermissionsAsync);
```

---

## 12. app.json Setup for Each Permission

Add these to your `app.json` based on which permissions you need.

```json
{
  "expo": {
    "plugins": [
      ["expo-camera", {
        "cameraPermission": "Allow app to use your camera."
      }],
      ["expo-location", {
        "locationWhenInUsePermission": "Allow app to use location while open.",
        "locationAlwaysPermission": "Allow app to track location in background.",
        "locationAlwaysAndWhenInUsePermission": "Allow app to use your location."
      }],
      ["expo-media-library", {
        "photosPermission": "Allow app to access your photos.",
        "savePhotosPermission": "Allow app to save photos.",
        "isAccessMediaLocationEnabled": true
      }],
      ["expo-contacts", {
        "contactsPermission": "Allow app to access your contacts."
      }],
      ["expo-notifications", {
        "icon": "./assets/notification-icon.png",
        "color": "#ffffff"
      }],
      ["expo-calendar", {
        "calendarPermission": "Allow app to access your calendar."
      }]
    ],
    "android": {
      "permissions": [
        "CAMERA",
        "RECORD_AUDIO",
        "READ_CONTACTS",
        "WRITE_CONTACTS",
        "ACCESS_FINE_LOCATION",
        "ACCESS_COARSE_LOCATION",
        "ACCESS_BACKGROUND_LOCATION",
        "READ_EXTERNAL_STORAGE",
        "WRITE_EXTERNAL_STORAGE",
        "READ_MEDIA_IMAGES",
        "READ_MEDIA_VIDEO",
        "FOREGROUND_SERVICE",
        "FOREGROUND_SERVICE_LOCATION",
        "BLUETOOTH",
        "BLUETOOTH_SCAN",
        "BLUETOOTH_CONNECT",
        "ACTIVITY_RECOGNITION"
      ]
    },
    "ios": {
      "infoPlist": {
        "NSCameraUsageDescription": "We need camera to take photos.",
        "NSMicrophoneUsageDescription": "We need microphone to record audio.",
        "NSLocationWhenInUseUsageDescription": "We need location while app is open.",
        "NSLocationAlwaysUsageDescription": "We need location in background.",
        "NSLocationAlwaysAndWhenInUseUsageDescription": "We need your location.",
        "NSContactsUsageDescription": "We need contacts to find your friends.",
        "NSPhotoLibraryUsageDescription": "We need gallery access to pick photos.",
        "NSPhotoLibraryAddUsageDescription": "We need to save photos to your gallery.",
        "NSCalendarsUsageDescription": "We need calendar access.",
        "NSRemindersUsageDescription": "We need reminders access.",
        "NSBluetoothAlwaysUsageDescription": "We need Bluetooth to connect to devices.",
        "NSMotionUsageDescription": "We need motion data to count your steps."
      }
    }
  }
}
```

> **Important:** On iOS every permission MUST have a description text (`NSXxxUsageDescription`). Without it, Apple will reject your app.

---

## 13. Quick Reference Table

| Feature | Install | Request Function | Check Function |
|---|---|---|---|
| Camera | `expo-camera` | `useCameraPermissions()` | `getCameraPermissionsAsync()` |
| Microphone | `expo-av` | `Audio.requestPermissionsAsync()` | `Audio.getPermissionsAsync()` |
| Location (fg) | `expo-location` | `requestForegroundPermissionsAsync()` | `getForegroundPermissionsAsync()` |
| Location (bg) | `expo-location` | `requestBackgroundPermissionsAsync()` | `getBackgroundPermissionsAsync()` |
| Contacts | `expo-contacts` | `requestPermissionsAsync()` | `getPermissionsAsync()` |
| Gallery | `expo-media-library` | `requestPermissionsAsync()` | `getPermissionsAsync()` |
| Notifications | `expo-notifications` | `requestPermissionsAsync()` | `getPermissionsAsync()` |
| Calendar | `expo-calendar` | `requestCalendarPermissionsAsync()` | `getCalendarPermissionsAsync()` |
| Pedometer | `expo-sensors` | `Pedometer.requestPermissionsAsync()` | `Pedometer.getPermissionsAsync()` |
| Bluetooth | `react-native-ble-plx` | `PermissionsAndroid.request()` | `PermissionsAndroid.check()` |

---

## The Pattern — Every Permission Works the Same Way

All permissions in Expo follow this exact same pattern:

```tsx
const askPermission = async () => {
  // 1. Ask for permission
  const { status } = await SomeLibrary.requestPermissionsAsync();

  // 2. Check if granted
  if (status === 'granted') {
    // 3. Do the thing
    doTheFeature();
  } else {
    // 4. Tell the user why they need it
    alert('We need this permission to use this feature');
  }
};
```

Once you understand this pattern, you can handle **any** permission in Expo.

---

## Common Mistakes to Avoid

| Mistake | Fix |
|---|---|
| Not adding plugin to `app.json` | Always add the plugin — without it the permission popup may not appear |
| Asking background location without foreground | Always ask foreground first |
| Not handling `canAskAgain = false` | Show an alert to open Settings |
| Forgetting iOS `NSXxxUsageDescription` | Add all descriptions in `infoPlist` |
| Testing background features on simulator | Always use a real device |
| Not running `expo prebuild` after changing `app.json` | Run `npx expo prebuild` to apply native changes |
