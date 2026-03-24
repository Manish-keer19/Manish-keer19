# 🗺️ Google Maps Setup Guide — Expo React Native

> **Project:** `myapp` (Expo SDK 54)
> **Last Updated:** February 2026
> **Author:** Personal Reference Guide

---

## 📋 Table of Contents

| #   | Section                                                                 |
| --- | ----------------------------------------------------------------------- |
| 1   | [Packages Required](#1-packages-required)                               |
| 2   | [Get a Google Maps API Key](#2-get-a-google-maps-api-key)               |
| 3   | [Enable APIs in Google Cloud](#3-enable-the-right-apis-in-google-cloud) |
| 4   | [Configure app.json](#4-configure-appjson)                              |
| 5   | [Production Setup with .env](#5-production-setup--env-file)             |
| 6   | [Using MapView in Code](#6-using-mapview-in-code)                       |
| 7   | [Using expo-location for GPS](#7-using-expo-location-for-gps)           |
| 8   | [Places Autocomplete Search Bar](#8-places-autocomplete-search-bar)     |
| 9   | [Expo Go vs Dev Build](#9-expo-go-vs-development-build)                 |
| 10  | [Full E-Commerce Location Flow](#10-full-e-commerce-location-flow)      |
| 11  | [Common Errors & Fixes](#11-common-errors--fixes)                       |
| 12  | [Project Status Checklist](#12-project-status-checklist)                |

---

## 1. Packages Required

You need **3 packages** for a full maps + location + search experience:

```bash
# ✅ Install expo-compatible packages (use npx expo install — NOT npm install)
npx expo install react-native-maps expo-location

# ✅ Install places autocomplete
npm install react-native-google-places-autocomplete
```

| Package                                   | Purpose                                   |
| ----------------------------------------- | ----------------------------------------- |
| `react-native-maps`                       | The actual Google Map component           |
| `expo-location`                           | Get device GPS location + reverse geocode |
| `react-native-google-places-autocomplete` | Address search bar with suggestions       |

> ⚠️ **Important:** Always use `npx expo install` for Expo packages.
> It automatically picks the version compatible with your Expo SDK.
> Using `npm install` can cause version mismatch errors.

---

## 2. Get a Google Maps API Key

### Step 1 — Go to Google Cloud Console

👉 Open: **https://console.cloud.google.com/**

### Step 2 — Create a Project

1. Click the **project dropdown** at the top bar
2. Click **"New Project"**
3. Give it a name (e.g. `MyAppMaps`)
4. Click **Create**
5. Wait a few seconds, then select your new project

### Step 3 — Create an API Key

1. Go to: **APIs & Services → Credentials**
2. Click **"+ Create Credentials"**
3. Select **"API Key"**
4. Copy the key — it looks like this:
   ```
   AIzaSyBaiMZ2tMZrjH4E08wANDq2cmCwTCDhJiE
   ```

### Step 4 — Restrict the API Key (Security Best Practice)

> 🔐 Always restrict your key so no one can steal and misuse it.

1. Click on the key you just created
2. Under **"Application restrictions"** → Select **"Android apps"**
3. Click **"Add an item"** under Android apps
4. Enter your **Package Name** (from `app.json`):
   ```
   com.anonymous.myapp
   ```
5. Enter your **SHA-1 fingerprint** (see below how to get it)
6. Under **"API restrictions"** → Select **"Restrict key"**
7. Select only the APIs you use (see section 3)
8. Click **Save**

#### How to get your SHA-1 fingerprint (Debug):

```bash
# Run this in your terminal (requires Java JDK)
keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android
```

Look for the line starting with `SHA1:` and copy that value.

---

## 3. Enable the Right APIs in Google Cloud

Go to: **APIs & Services → Library**

Search and **Enable** each of these:

| API Name                    | Required For                                       |
| --------------------------- | -------------------------------------------------- |
| ✅ **Maps SDK for Android** | Showing the map on Android                         |
| ✅ **Maps SDK for iOS**     | Showing the map on iPhone                          |
| ✅ **Places API**           | Search bar autocomplete suggestions                |
| ✅ **Geocoding API**        | Converting coordinates → address (reverse geocode) |

> **How to enable:**
>
> 1. Search the API name in the Library
> 2. Click on it
> 3. Click the blue **"Enable"** button
> 4. Repeat for each API

---

## 4. Configure `app.json`

This is the **most critical step**.
Without the API key in `app.json`, the map shows as a **blank grey screen** on Android.

```json
{
  "expo": {
    "name": "myapp",
    "slug": "myapp",
    "version": "1.0.0",

    "android": {
      "package": "com.anonymous.myapp",

      "config": {
        "googleMaps": {
          "apiKey": "YOUR_GOOGLE_MAPS_API_KEY_HERE"
        }
      },

      "permissions": [
        "android.permission.ACCESS_FINE_LOCATION",
        "android.permission.ACCESS_COARSE_LOCATION"
      ]
    },

    "ios": {
      "supportsTablet": true,
      "config": {
        "googleMapsApiKey": "YOUR_GOOGLE_MAPS_API_KEY_HERE"
      }
    },

    "plugins": [
      [
        "expo-location",
        {
          "locationAlwaysAndWhenInUsePermission": "Allow $(BUNDLE_IDENTIFIER) to use your location for delivery."
        }
      ]
    ]
  }
}
```

### What Each Field Does:

| Field                              | What It Does                                                                 |
| ---------------------------------- | ---------------------------------------------------------------------------- |
| `android.config.googleMaps.apiKey` | Injects key into native `AndroidManifest.xml` — **required for map to show** |
| `ios.config.googleMapsApiKey`      | Injects key into iOS `Info.plist`                                            |
| `android.permissions`              | Declares GPS permissions in the Android app                                  |
| `plugins → expo-location`          | Adds the iOS location usage description (required for App Store)             |

---

## 5. Production Setup — `.env` File

> 🔐 **Never hardcode API keys in `app.json` when pushing to GitHub!**

### Step 1 — Create `app.config.js` (replaces `app.json`)

```js
// app.config.js
export default {
  expo: {
    name: "myapp",
    slug: "myapp",
    version: "1.0.0",

    android: {
      package: "com.anonymous.myapp",
      config: {
        googleMaps: {
          apiKey: process.env.GOOGLE_MAPS_API_KEY, // Read from .env
        },
      },
      permissions: [
        "android.permission.ACCESS_FINE_LOCATION",
        "android.permission.ACCESS_COARSE_LOCATION",
      ],
    },

    ios: {
      supportsTablet: true,
      config: {
        googleMapsApiKey: process.env.GOOGLE_MAPS_API_KEY,
      },
    },

    plugins: [
      [
        "expo-location",
        {
          locationAlwaysAndWhenInUsePermission:
            "Allow $(BUNDLE_IDENTIFIER) to use your location.",
        },
      ],
    ],
  },
};
```

### Step 2 — Create `.env` in project root

```env
GOOGLE_MAPS_API_KEY=AIzaSyBaiMZ2tMZrjH4E08wANDq2cmCwTCDhJiE
```

### Step 3 — Add `.env` to `.gitignore`

```
# .gitignore
.env
.env.local
```

---

## 6. Using MapView in Code

```tsx
import React, { useRef } from "react";
import { StyleSheet, View } from "react-native";
import MapView, { Marker, PROVIDER_GOOGLE, Region } from "react-native-maps";

export default function MyMapScreen() {
  const mapRef = useRef<MapView | null>(null);

  const handleRegionChange = (region: Region) => {
    console.log("Center:", region.latitude, region.longitude);
  };

  return (
    <View style={{ flex: 1 }}>
      <MapView
        ref={mapRef}
        style={styles.map} // ✅ StyleSheet — NOT className
        provider={PROVIDER_GOOGLE} // ✅ Force Google Maps (not Apple Maps)
        initialRegion={{
          latitude: 28.6139, // Starting latitude (New Delhi)
          longitude: 77.209, // Starting longitude
          latitudeDelta: 0.01, // Vertical zoom (smaller = more zoomed in)
          longitudeDelta: 0.01, // Horizontal zoom
        }}
        showsUserLocation={true} // Blue dot at user's GPS position
        showsMyLocationButton={false} // Hide default button (make your own)
        onRegionChange={() => {}} // Called while map is being dragged
        onRegionChangeComplete={handleRegionChange} // Called when drag stops
      />
    </View>
  );
}

const styles = StyleSheet.create({
  map: {
    flex: 1, // ✅ This makes the map fill the parent container
  },
});
```

### Adding a Marker (Pin at specific location):

```tsx
import { Marker } from 'react-native-maps';

<MapView ...>
  <Marker
    coordinate={{ latitude: 28.6139, longitude: 77.2090 }}
    title="My Location"
    description="Order will be delivered here"
    pinColor="#884cff"   // Custom pin color
  />
</MapView>
```

### Animate Map to a Location Programmatically:

```tsx
mapRef.current?.animateToRegion(
  {
    latitude: 28.6139,
    longitude: 77.209,
    latitudeDelta: 0.01,
    longitudeDelta: 0.01,
  },
  800,
); // 800ms animation duration
```

### ⚠️ Why You Must Use `style` NOT `className` on MapView

`react-native-maps` is a **native component** (not a pure React component).
NativeWind's `className` sometimes doesn't apply correctly to native components.

```tsx
// ❌ WRONG — may cause blank map
<MapView className="flex-1" />

// ✅ CORRECT — always works
<MapView style={{ flex: 1 }} />
// or
<MapView style={styles.map} />
```

---

## 7. Using `expo-location` for GPS

```tsx
import * as Location from "expo-location";

async function getUserLocation() {
  // ─── STEP 1: Ask for permission (REQUIRED on both Android & iOS) ───
  const { status } = await Location.requestForegroundPermissionsAsync();

  if (status !== "granted") {
    alert("Location permission is required!");
    return;
  }

  // ─── STEP 2: Get current GPS position ───
  const loc = await Location.getCurrentPositionAsync({
    accuracy: Location.Accuracy.High,
  });

  console.log(loc.coords.latitude); // e.g. 28.6139
  console.log(loc.coords.longitude); // e.g. 77.2090
  console.log(loc.coords.altitude); // Height in meters

  // ─── STEP 3: Reverse Geocode (coordinates → readable address) ───
  const addressResult = await Location.reverseGeocodeAsync({
    latitude: loc.coords.latitude,
    longitude: loc.coords.longitude,
  });

  const a = addressResult[0];
  console.log(a.name); // Building/street name e.g. "45 Main Street"
  console.log(a.street); // Street e.g. "Main Street"
  console.log(a.district); // District/area
  console.log(a.city); // City e.g. "Mumbai"
  console.log(a.region); // State e.g. "Maharashtra"
  console.log(a.country); // Country e.g. "India"
  console.log(a.postalCode); // Pin code e.g. "400001"
}
```

### Accuracy Levels (choose based on need):

| Constant                              | Accuracy       | Speed   | Battery |
| ------------------------------------- | -------------- | ------- | ------- |
| `Location.Accuracy.Lowest`            | ~3 km          | Fastest | Minimal |
| `Location.Accuracy.Low`               | ~1 km          | Fast    | Low     |
| `Location.Accuracy.Balanced`          | ~100 m         | Medium  | Medium  |
| `Location.Accuracy.High`              | ~10 m          | Slow    | High    |
| `Location.Accuracy.Highest`           | ~1 m           | Slowest | Max     |
| `Location.Accuracy.BestForNavigation` | ~1 m + compass | Slowest | Max     |

> 💡 For delivery address selection, `Location.Accuracy.High` is the sweet spot.

### Permission Types:

```tsx
// Foreground — only when app is open (use for delivery address)
await Location.requestForegroundPermissionsAsync();

// Background — even when app is closed (use for tracking)
await Location.requestBackgroundPermissionsAsync();
```

---

## 8. Places Autocomplete Search Bar

```tsx
import { GooglePlacesAutocomplete } from "react-native-google-places-autocomplete";
import MapView from "react-native-maps";
import { useRef } from "react";

export default function MapWithSearch() {
  const mapRef = useRef(null);

  return (
    <View style={{ flex: 1 }}>
      {/* 🔍 Search Box — must be ABOVE MapView in JSX */}
      <View
        style={{
          position: "absolute",
          top: 50,
          width: "100%",
          zIndex: 10,
          paddingHorizontal: 16,
        }}
      >
        <GooglePlacesAutocomplete
          placeholder="Search delivery address..."
          fetchDetails={true} // ✅ Required to get lat/lng
          onPress={(data, details = null) => {
            if (!details) return;

            const lat = details.geometry.location.lat;
            const lng = details.geometry.location.lng;
            const address = data.description;

            console.log("Selected:", address, lat, lng);

            // Animate map to searched location
            mapRef.current?.animateToRegion(
              {
                latitude: lat,
                longitude: lng,
                latitudeDelta: 0.01,
                longitudeDelta: 0.01,
              },
              800,
            );
          }}
          query={{
            key: "YOUR_API_KEY_HERE", // ✅ Needs Places API enabled
            language: "en",
            components: "country:in", // Restrict results to India (optional)
          }}
          styles={{
            container: { flex: 0 },
            textInputContainer: { backgroundColor: "#fff", borderRadius: 12 },
            textInput: { fontSize: 15, color: "#1F2937" },
            listView: { backgroundColor: "#fff", borderRadius: 8 },
          }}
          enablePoweredByContainer={false} // Hide "Powered by Google" branding
          minLength={2} // Min characters before showing results
          debounce={300} // Wait 300ms after typing to search
        />
      </View>

      {/* 🗺 Map */}
      <MapView ref={mapRef} style={{ flex: 1 }} provider="google" />
    </View>
  );
}
```

---

## 9. Expo Go vs Development Build

> This is the **most misunderstood part** of using maps in Expo.

### The difference:

| Feature               | Expo Go App        | Development Build      |
| --------------------- | ------------------ | ---------------------- |
| How to open           | Scan QR code       | `npx expo run:android` |
| Google Maps shows?    | ❌ Blank / Limited | ✅ Fully works         |
| API key injected?     | ❌ No              | ✅ Yes (from app.json) |
| Native code compiled? | ❌ No              | ✅ Yes                 |
| Good for              | UI testing         | Maps / Camera / BLE    |

### Why Expo Go can't show Google Maps:

```
Expo Go is a prebuilt app.
Your API key needs to be compiled INTO the native Android binary.
Expo Go can't do that — it's someone else's app.
Only YOUR OWN build has your API key embedded.
```

### How to run YOUR own build (Required for Maps):

```bash
# Option 1: Run on connected Android device or emulator (local)
npx expo run:android

# Option 2: Cloud build via EAS (Expo's build service)
npm install -g eas-cli
eas login
eas build --platform android --profile development
```

### Setting up an Android Emulator (for local testing):

1. Install **Android Studio**: https://developer.android.com/studio
2. Open Android Studio → **Virtual Device Manager**
3. Create a new **Pixel 7** device with **API level 34**
4. Start the emulator
5. Run `npx expo run:android` — it will auto-detect and deploy

---

## 10. Full E-Commerce Location Flow

This is the flow implemented in this project:

```
┌─────────────────────────────────────────────────────────────┐
│                   ProductDetail Screen                       │
│                                                              │
│  [Product Image Slider]                                      │
│  Product Title, Price, Description                           │
│                                                              │
│  [ Add to Cart ]  [ Buy Now ]       ← Tap either button     │
│                                              │               │
│  📍 Delivery to: [address shown here]        │               │
└──────────────────────────────────────────────┼──────────────┘
                                               │ navigation.navigate('Location')
                                               ▼
┌─────────────────────────────────────────────────────────────┐
│                   LocationScreen                             │
│                                                              │
│  ← Select Delivery Location           [Header]               │
│                                                              │
│  ┌─────────────────────────────────────────┐               │
│  │                                          │               │
│  │              Google Map                  │               │
│  │                    📍                    │               │  ← User drags map
│  │          (center pin fixed)              │               │    pin stays center
│  │                                          │               │
│  └─────────────────────────────────────────┘               │
│                                              [📍 My Location]│
│  ┌─────────────────────────────────────────┐               │
│  │  📍 Delivery Address                     │               │
│  │  [Address auto-filled from geocode...]   │  ← Bottom     │
│  │  28.61390, 77.20900                      │    Panel      │
│  │  [ ✅ Confirm This Location ]            │               │
│  └─────────────────────────────────────────┘               │
└──────────────────────────────────────────────────────────────┘
                                               │ navigation.goBack()
                                               │ + callback(selectedLocation)
                                               ▼
                           ProductDetail shows "📍 Delivery to: [address]"
```

### Key Code — Passing Location Back (Callback Pattern):

**In ProductDetail.tsx** (caller):

```tsx
navigation.navigate("Location", {
  onLocationSelected: (loc) => {
    // This runs AFTER user confirms location on map
    setSelectedLocation(loc);
  },
});
```

**In LocationScreen.tsx** (receiver):

```tsx
const handleConfirm = () => {
  const selectedLocation = {
    latitude: location.latitude,
    longitude: location.longitude,
    address: address,
  };

  // Call the callback passed from ProductDetail
  if (route?.params?.onLocationSelected) {
    route.params.onLocationSelected(selectedLocation);
  }

  navigation.goBack(); // Return to ProductDetail
};
```

---

## 11. Common Errors & Fixes

### ❌ Blank / Grey Map Screen

```
Cause:  Google Maps API key not in app.json
Fix:    Add android.config.googleMaps.apiKey to app.json
        Then rebuild: npx expo run:android
```

### ❌ "This app is not authorized to use Google Maps"

```
Cause:  API key restriction has wrong package name
Fix:    In Google Cloud Console → Credentials → your key
        Make sure package name exactly matches app.json:
        com.anonymous.myapp
```

### ❌ Map works in dev, blank in production

```
Cause:  SHA-1 fingerprint mismatch (release key != debug key)
Fix:    Add your RELEASE SHA-1 fingerprint to Google Cloud Console
        Get it from: keytool -list -v -keystore my-release-key.jks
```

### ❌ Location permission denied silently (Android)

```
Cause:  Permissions not declared in app.json
Fix:    Add to app.json android section:
        "permissions": [
          "android.permission.ACCESS_FINE_LOCATION",
          "android.permission.ACCESS_COARSE_LOCATION"
        ]
```

### ❌ Places autocomplete returns no results

```
Cause:  Places API not enabled in Google Cloud
Fix:    Go to APIs & Services → Library
        Search "Places API" → Enable it
```

### ❌ Reverse geocode returns empty array

```
Cause:  Geocoding API not enabled
Fix:    Go to APIs & Services → Library
        Search "Geocoding API" → Enable it
```

### ❌ Map not filling screen

```
Cause:  Missing flex:1 on MapView or parent container
Fix:
        const styles = StyleSheet.create({
          map: { flex: 1 }         // ✅
        });
        <View style={{ flex: 1 }}> // ✅ Parent also needs flex:1
          <MapView style={styles.map} />
        </View>
```

### ❌ NativeWind className not working on MapView

```
Cause:  MapView is a native component, not pure React
Fix:    Always use StyleSheet.create() for MapView styles
        Never use className="flex-1" on MapView
```

---

## 12. Project Status Checklist

Use this checklist every time you set up a new project:

- [ ] `npx expo install react-native-maps expo-location`
- [ ] `npm install react-native-google-places-autocomplete`
- [ ] Created Google Cloud project
- [ ] Created API key
- [ ] Enabled **Maps SDK for Android**
- [ ] Enabled **Maps SDK for iOS**
- [ ] Enabled **Places API**
- [ ] Enabled **Geocoding API**
- [ ] Added API key to `app.json` → `android.config.googleMaps.apiKey`
- [ ] Added API key to `app.json` → `ios.config.googleMapsApiKey`
- [ ] Added `android.permissions` for location
- [ ] Added `expo-location` plugin to `app.json`
- [ ] Using `PROVIDER_GOOGLE` on MapView component
- [ ] Using `StyleSheet` (not `className`) on MapView
- [ ] Running with `npx expo run:android` (NOT Expo Go)
- [ ] API key restricted to correct package name

---

## 📁 File Reference (This Project)

```
myapp/
│
├── app.json                          ← Google Maps API key configured here
├── app.config.js                     ← (Future) Use this for .env support
├── .env                              ← (Future) Store API key securely
│
├── src/
│   └── Screens/
│       ├── LocationScreen.tsx        ← Map screen with draggable pin
│       └── ProductDetail.tsx         ← Add to Cart → opens LocationScreen
│
└── docs/
    └── GOOGLE_MAPS_SETUP.md          ← 📄 This file
```

---

> 💡 **Quick Reminder:** After any change to `app.json`, always run:
>
> ```bash
> npx expo start -c   # Clear cache and restart
> # OR for full native rebuild:
> npx expo run:android
> ```

---

_Made with ❤️ for future reference — You've got this! 🚀_
