# Expo Location Tracking Guide

A complete guide for foreground & background location in Expo — including storing data locally with AsyncStorage and SQLite, and sending it to a server.

---

## Table of Contents

1. [Installation](#installation)
2. [App Config Setup](#app-config-setup)
3. [Permissions](#permissions)
4. [Foreground Location](#foreground-location)
5. [Background Location](#background-location)
6. [Distance Calculation](#distance-calculation)
7. [Store Locally — AsyncStorage](#store-locally--asyncstorage)
8. [Store Locally — SQLite](#store-locally--sqlite)
9. [Send Data to Server](#send-data-to-server)
10. [Full Example](#full-example)

---

## Installation

```bash
npx expo install expo-location expo-task-manager
npx expo install @react-native-async-storage/async-storage
npx expo install expo-sqlite
```

---

## App Config Setup

Add this to your `app.json` — required for both iOS and Android:

```json
{
  "expo": {
    "plugins": [
      [
        "expo-location",
        {
          "locationAlwaysAndWhenInUsePermission": "Allow the app to use your location.",
          "locationAlwaysPermission": "Allow the app to track your location in the background.",
          "locationWhenInUsePermission": "Allow the app to use your location while open."
        }
      ]
    ],
    "android": {
      "permissions": [
        "ACCESS_FINE_LOCATION",
        "ACCESS_COARSE_LOCATION",
        "ACCESS_BACKGROUND_LOCATION",
        "FOREGROUND_SERVICE",
        "FOREGROUND_SERVICE_LOCATION"
      ]
    }
  }
}
```

> **Note:** After editing `app.json`, run `npx expo prebuild` to apply changes.

---

## Permissions

You must request **foreground first**, then **background**. You cannot skip foreground.

```tsx
import * as Location from 'expo-location';

// Step 1 — Foreground permission
const requestForegroundPermission = async (): Promise<boolean> => {
  const { status } = await Location.requestForegroundPermissionsAsync();
  if (status !== 'granted') {
    console.log('Foreground permission denied');
    return false;
  }
  return true;
};

// Step 2 — Background permission (only after foreground is granted)
const requestBackgroundPermission = async (): Promise<boolean> => {
  const { status } = await Location.requestBackgroundPermissionsAsync();
  if (status !== 'granted') {
    console.log('Background permission denied');
    return false;
  }
  return true;
};

// Check existing permissions without prompting
const checkPermissions = async () => {
  const fg = await Location.getForegroundPermissionsAsync();
  const bg = await Location.getBackgroundPermissionsAsync();
  console.log('Foreground:', fg.status);
  console.log('Background:', bg.status);
};
```

---

## Foreground Location

Foreground location works while the app is open and visible.

### Get location once

```tsx
import * as Location from 'expo-location';

const getLocationOnce = async () => {
  const granted = await requestForegroundPermission();
  if (!granted) return;

  const location = await Location.getCurrentPositionAsync({
    accuracy: Location.Accuracy.High,
  });

  console.log('Latitude:', location.coords.latitude);
  console.log('Longitude:', location.coords.longitude);
  console.log('Speed:', location.coords.speed);        // m/s
  console.log('Altitude:', location.coords.altitude);  // meters
  console.log('Timestamp:', location.timestamp);
};
```

### Watch location continuously

```tsx
import * as Location from 'expo-location';
import { useEffect, useRef, useState } from 'react';

const useForegroundLocation = () => {
  const [location, setLocation] = useState<Location.LocationObject | null>(null);
  const [error, setError] = useState<string>('');
  const watchRef = useRef<Location.LocationSubscription | null>(null);

  const startWatching = async () => {
    const granted = await requestForegroundPermission();
    if (!granted) {
      setError('Permission denied');
      return;
    }

    watchRef.current = await Location.watchPositionAsync(
      {
        accuracy: Location.Accuracy.High,
        timeInterval: 5000,    // update every 5 seconds
        distanceInterval: 10,  // or every 10 meters (whichever comes first)
      },
      (newLocation) => {
        setLocation(newLocation);
      }
    );
  };

  const stopWatching = () => {
    watchRef.current?.remove();
    watchRef.current = null;
  };

  useEffect(() => {
    startWatching();
    return () => stopWatching(); // cleanup on unmount
  }, []);

  return { location, error, stopWatching };
};
```

### Location accuracy options

| Accuracy | Description | Use case |
|---|---|---|
| `Accuracy.Lowest` | ~3000m | City-level |
| `Accuracy.Low` | ~1000m | Rough tracking |
| `Accuracy.Balanced` | ~100m | General use |
| `Accuracy.High` | ~10m | Navigation |
| `Accuracy.Highest` | ~1m | Precise tracking |
| `Accuracy.BestForNavigation` | ~1m + sensors | Turn-by-turn nav |

---

## Background Location

Background location works even when the app is closed or in the background. Requires a **TaskManager task** defined at the top level of your file.

```tsx
import * as Location from 'expo-location';
import * as TaskManager from 'expo-task-manager';

const BACKGROUND_LOCATION_TASK = 'background-location-task';

// IMPORTANT: Define task OUTSIDE the component, at the top level of the file
TaskManager.defineTask(BACKGROUND_LOCATION_TASK, async ({ data, error }) => {
  if (error) {
    console.log('Background location error:', error.message);
    return;
  }

  if (data) {
    const { locations } = data as { locations: Location.LocationObject[] };
    const location = locations[0];

    const coords = {
      latitude: location.coords.latitude,
      longitude: location.coords.longitude,
      speed: location.coords.speed,
      altitude: location.coords.altitude,
      timestamp: location.timestamp,
    };

    console.log('Background location received:', coords);

    // Save locally and/or send to server
    await saveToAsyncStorage(coords);
    await saveToSQLite(coords);
    await sendToServer(coords);
  }
});

// Start background tracking
const startBackgroundTracking = async () => {
  // 1. Request foreground first
  const fgGranted = await requestForegroundPermission();
  if (!fgGranted) return;

  // 2. Request background
  const bgGranted = await requestBackgroundPermission();
  if (!bgGranted) return;

  // 3. Check if already running
  const isRunning = await Location.hasStartedLocationUpdatesAsync(
    BACKGROUND_LOCATION_TASK
  );
  if (isRunning) {
    console.log('Background tracking already running');
    return;
  }

  // 4. Start background updates
  await Location.startLocationUpdatesAsync(BACKGROUND_LOCATION_TASK, {
    accuracy: Location.Accuracy.High,
    timeInterval: 10000,   // every 10 seconds
    distanceInterval: 20,  // or every 20 meters
    pausesUpdatesAutomatically: false,
    showsBackgroundLocationIndicator: true, // iOS — shows blue bar at top
    foregroundService: {
      // Android — required to keep the task alive
      notificationTitle: 'Tracking your route',
      notificationBody: 'Your location is being tracked',
      notificationColor: '#3699DC',
    },
  });

  console.log('Background tracking started');
};

// Stop background tracking
const stopBackgroundTracking = async () => {
  const isRunning = await Location.hasStartedLocationUpdatesAsync(
    BACKGROUND_LOCATION_TASK
  );
  if (isRunning) {
    await Location.stopLocationUpdatesAsync(BACKGROUND_LOCATION_TASK);
    console.log('Background tracking stopped');
  }
};
```

> **Important:** Background location must be tested on a **physical device**. It will not work on simulators or emulators.

---

## Distance Calculation

Use the Haversine formula to calculate distance between two GPS coordinates.

```tsx
interface Coords {
  latitude: number;
  longitude: number;
}

// Returns distance in meters
const getDistanceMeters = (coord1: Coords, coord2: Coords): number => {
  const R = 6371000; // Earth radius in meters
  const toRad = (deg: number) => deg * (Math.PI / 180);

  const lat1 = toRad(coord1.latitude);
  const lat2 = toRad(coord2.latitude);
  const dLat = toRad(coord2.latitude - coord1.latitude);
  const dLon = toRad(coord2.longitude - coord1.longitude);

  const a =
    Math.sin(dLat / 2) * Math.sin(dLat / 2) +
    Math.cos(lat1) * Math.cos(lat2) *
    Math.sin(dLon / 2) * Math.sin(dLon / 2);

  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
  return R * c;
};

// Calculate total distance from an array of coordinates
const calculateTotalDistance = (locations: Coords[]): number => {
  let total = 0;
  for (let i = 1; i < locations.length; i++) {
    total += getDistanceMeters(locations[i - 1], locations[i]);
  }
  return total; // meters
};

// Helper formatters
const toKilometers = (meters: number) => (meters / 1000).toFixed(2) + ' km';
const toMiles = (meters: number) => (meters / 1609.34).toFixed(2) + ' mi';
```

---

## Store Locally — AsyncStorage

AsyncStorage is a simple key-value store. Good for small amounts of data.

```tsx
import AsyncStorage from '@react-native-async-storage/async-storage';

const LOCATIONS_KEY = 'tracked_locations';

interface LocationRecord {
  latitude: number;
  longitude: number;
  speed: number | null;
  altitude: number | null;
  timestamp: number;
}

// Save a single location point
const saveToAsyncStorage = async (coords: LocationRecord): Promise<void> => {
  try {
    const existing = await AsyncStorage.getItem(LOCATIONS_KEY);
    const locations: LocationRecord[] = existing ? JSON.parse(existing) : [];
    locations.push(coords);
    await AsyncStorage.setItem(LOCATIONS_KEY, JSON.stringify(locations));
  } catch (error) {
    console.log('AsyncStorage save error:', error);
  }
};

// Get all saved locations
const getLocationsFromStorage = async (): Promise<LocationRecord[]> => {
  try {
    const data = await AsyncStorage.getItem(LOCATIONS_KEY);
    return data ? JSON.parse(data) : [];
  } catch (error) {
    console.log('AsyncStorage read error:', error);
    return [];
  }
};

// Get total distance from stored locations
const getTotalDistanceFromStorage = async (): Promise<number> => {
  const locations = await getLocationsFromStorage();
  return calculateTotalDistance(locations);
};

// Clear all stored locations
const clearStoredLocations = async (): Promise<void> => {
  await AsyncStorage.removeItem(LOCATIONS_KEY);
};
```

> **Limitation:** AsyncStorage is not suitable for large datasets (thousands of location points). Use SQLite for heavy tracking.

---

## Store Locally — SQLite

SQLite is a full SQL database — better for large amounts of location data with queries, filters, and aggregation.

```tsx
import * as SQLite from 'expo-sqlite';

// Open (or create) the database
const db = SQLite.openDatabaseSync('locations.db');

// Initialize the table — run this once on app start
const initDatabase = async (): Promise<void> => {
  await db.execAsync(`
    CREATE TABLE IF NOT EXISTS locations (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      latitude REAL NOT NULL,
      longitude REAL NOT NULL,
      speed REAL,
      altitude REAL,
      timestamp INTEGER NOT NULL,
      session_id TEXT,
      synced INTEGER DEFAULT 0
    );
  `);
  console.log('Database initialized');
};

// Save a location to SQLite
const saveToSQLite = async (coords: LocationRecord, sessionId?: string): Promise<void> => {
  try {
    await db.runAsync(
      `INSERT INTO locations (latitude, longitude, speed, altitude, timestamp, session_id, synced)
       VALUES (?, ?, ?, ?, ?, ?, 0)`,
      [
        coords.latitude,
        coords.longitude,
        coords.speed ?? null,
        coords.altitude ?? null,
        coords.timestamp,
        sessionId ?? null,
      ]
    );
  } catch (error) {
    console.log('SQLite save error:', error);
  }
};

// Get all locations
const getAllLocations = async (): Promise<LocationRecord[]> => {
  try {
    const rows = await db.getAllAsync(
      'SELECT * FROM locations ORDER BY timestamp ASC'
    );
    return rows as LocationRecord[];
  } catch (error) {
    console.log('SQLite read error:', error);
    return [];
  }
};

// Get locations by session
const getLocationsBySession = async (sessionId: string): Promise<LocationRecord[]> => {
  try {
    const rows = await db.getAllAsync(
      'SELECT * FROM locations WHERE session_id = ? ORDER BY timestamp ASC',
      [sessionId]
    );
    return rows as LocationRecord[];
  } catch (error) {
    console.log('SQLite session read error:', error);
    return [];
  }
};

// Get unsynced locations (not yet sent to server)
const getUnsyncedLocations = async (): Promise<any[]> => {
  try {
    const rows = await db.getAllAsync(
      'SELECT * FROM locations WHERE synced = 0 ORDER BY timestamp ASC'
    );
    return rows;
  } catch (error) {
    console.log('SQLite unsynced read error:', error);
    return [];
  }
};

// Mark locations as synced after sending to server
const markAsSynced = async (ids: number[]): Promise<void> => {
  if (ids.length === 0) return;
  const placeholders = ids.map(() => '?').join(',');
  await db.runAsync(
    `UPDATE locations SET synced = 1 WHERE id IN (${placeholders})`,
    ids
  );
};

// Get total distance from SQLite
const getTotalDistanceFromSQLite = async (): Promise<number> => {
  const locations = await getAllLocations();
  return calculateTotalDistance(locations);
};

// Delete old locations (e.g. older than 7 days)
const deleteOldLocations = async (daysOld: number = 7): Promise<void> => {
  const cutoff = Date.now() - daysOld * 24 * 60 * 60 * 1000;
  await db.runAsync('DELETE FROM locations WHERE timestamp < ?', [cutoff]);
};
```

---

## Send Data to Server

### Send a single location point

```tsx
import axios from 'axios';

const API_URL = 'https://your-api.com';

const sendToServer = async (coords: LocationRecord): Promise<void> => {
  try {
    await axios.post(`${API_URL}/api/location`, {
      latitude: coords.latitude,
      longitude: coords.longitude,
      speed: coords.speed,
      altitude: coords.altitude,
      timestamp: coords.timestamp,
    });
  } catch (error) {
    console.log('Server send error — will retry later:', error);
    // Don't throw — location is already saved locally
  }
};
```

### Send in bulk (batch sync)

Batch syncing is more efficient — collect locations locally and send them together periodically.

```tsx
const syncUnsyncedToServer = async (): Promise<void> => {
  try {
    const unsynced = await getUnsyncedLocations();
    if (unsynced.length === 0) return;

    const res = await axios.post(`${API_URL}/api/locations/batch`, {
      locations: unsynced,
    });

    if (res.status === 200) {
      const ids = unsynced.map((loc: any) => loc.id);
      await markAsSynced(ids);
      console.log(`Synced ${ids.length} locations to server`);
    }
  } catch (error) {
    console.log('Batch sync failed — will retry:', error);
  }
};

// Call this when app opens or when connection is available
const syncOnAppOpen = async () => {
  await initDatabase();
  await syncUnsyncedToServer();
};
```

### With auth token

```tsx
import axiosInstance from './services/axiosInstance';

const sendLocationWithAuth = async (coords: LocationRecord): Promise<void> => {
  try {
    await axiosInstance.post('/location/track', {
      latitude: coords.latitude,
      longitude: coords.longitude,
      timestamp: coords.timestamp,
    });
  } catch (error) {
    console.log('Auth send error:', error);
  }
};
```

---

## Full Example

A complete working component combining everything above.

```tsx
import React, { useEffect, useRef, useState } from 'react';
import { View, Text, Pressable } from 'react-native';
import * as Location from 'expo-location';
import * as TaskManager from 'expo-task-manager';

const TASK_NAME = 'background-location-task';

// Define background task at top level
TaskManager.defineTask(TASK_NAME, async ({ data, error }) => {
  if (error || !data) return;
  const { locations } = data as { locations: Location.LocationObject[] };
  const loc = locations[0];

  const coords = {
    latitude: loc.coords.latitude,
    longitude: loc.coords.longitude,
    speed: loc.coords.speed,
    altitude: loc.coords.altitude,
    timestamp: loc.timestamp,
  };

  await saveToSQLite(coords);           // save locally
  await sendToServer(coords);           // try to send immediately
});

export default function LocationTracker() {
  const [tracking, setTracking] = useState(false);
  const [distance, setDistance] = useState(0);
  const [currentLocation, setCurrentLocation] = useState<any>(null);
  const prevCoordsRef = useRef<any>(null);
  const watchRef = useRef<Location.LocationSubscription | null>(null);

  useEffect(() => {
    initDatabase();        // setup SQLite on mount
    syncOnAppOpen();       // sync any unsynced data
  }, []);

  const startTracking = async () => {
    // Foreground watching
    const fgGranted = await requestForegroundPermission();
    if (!fgGranted) return;

    watchRef.current = await Location.watchPositionAsync(
      { accuracy: Location.Accuracy.High, distanceInterval: 10 },
      async (loc) => {
        const coords = {
          latitude: loc.coords.latitude,
          longitude: loc.coords.longitude,
          speed: loc.coords.speed,
          altitude: loc.coords.altitude,
          timestamp: loc.timestamp,
        };

        setCurrentLocation(coords);

        if (prevCoordsRef.current) {
          const d = getDistanceMeters(prevCoordsRef.current, coords);
          setDistance((prev) => prev + d);
        }
        prevCoordsRef.current = coords;

        await saveToSQLite(coords);
        await sendToServer(coords);
      }
    );

    // Background tracking
    await startBackgroundTracking();
    setTracking(true);
  };

  const stopTracking = async () => {
    watchRef.current?.remove();
    await stopBackgroundTracking();
    setTracking(false);
  };

  return (
    <View style={{ padding: 20 }}>
      <Text style={{ fontSize: 18, fontWeight: 'bold' }}>Location Tracker</Text>

      {currentLocation && (
        <View style={{ marginTop: 10 }}>
          <Text>Lat: {currentLocation.latitude.toFixed(6)}</Text>
          <Text>Lng: {currentLocation.longitude.toFixed(6)}</Text>
          <Text>Speed: {currentLocation.speed?.toFixed(1) ?? '0'} m/s</Text>
        </View>
      )}

      <Text style={{ marginTop: 10, fontSize: 16 }}>
        Distance: {toKilometers(distance)}
      </Text>

      <Pressable
        onPress={tracking ? stopTracking : startTracking}
        style={{
          marginTop: 20,
          padding: 16,
          backgroundColor: tracking ? '#e74c3c' : '#3699DC',
          borderRadius: 10,
          alignItems: 'center',
        }}
      >
        <Text style={{ color: '#fff', fontSize: 16, fontWeight: 'bold' }}>
          {tracking ? 'Stop Tracking' : 'Start Tracking'}
        </Text>
      </Pressable>
    </View>
  );
}
```

---

## Quick Reference

| Feature | Method |
|---|---|
| Ask foreground permission | `requestForegroundPermissionsAsync()` |
| Ask background permission | `requestBackgroundPermissionsAsync()` |
| Get location once | `getCurrentPositionAsync()` |
| Watch live location | `watchPositionAsync()` |
| Start background tracking | `startLocationUpdatesAsync()` |
| Stop background tracking | `stopLocationUpdatesAsync()` |
| Check if background running | `hasStartedLocationUpdatesAsync()` |
| Save to AsyncStorage | `AsyncStorage.setItem()` |
| Save to SQLite | `db.runAsync()` |
| Batch sync to server | `axios.post('/locations/batch', ...)` |

---

## Important Notes

- **Background location requires a physical device** — simulators will not work
- **iOS App Store** requires a valid justification for always-on location
- **Android** requires `FOREGROUND_SERVICE` permission for background tracking
- **TaskManager task must be defined at the top level** of the file — never inside a component
- **Always save locally first**, then send to server — never rely on network alone
- **Batch sync** is more efficient than sending each point individually
- **SQLite** is preferred over AsyncStorage for large datasets (1000+ location points)
