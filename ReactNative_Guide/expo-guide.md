# The Complete Expo Router + TypeScript Field App Guide
### Building an offline-first, camera/audio/video/location-enabled production app from scratch

This guide walks through building a production-grade React Native app using the latest Expo SDK, Expo Router, and TypeScript. Every section explains the *why*, not just the *how*, and uses a running example — an **offline-first field survey app** (think: inspectors, agronomists, or auditors collecting data with photos/audio/video/GPS in areas with no signal) — to ground the concepts.

---

## Table of Contents

1. [Expo Project Structure](#1-expo-project-structure)
2. [Expo Router](#2-expo-router)
3. [Tab Navigation](#3-tab-navigation)
4. [Permissions](#4-permissions)
5. [Camera](#5-camera)
6. [Image & Video Picker](#6-image--video-picker)
7. [Audio Recording](#7-audio-recording)
8. [Video Player](#8-video-player)
9. [SQLite Database](#9-sqlite-database)
10. [File System](#10-file-system)
11. [Location](#11-location)
12. [Offline-First Architecture](#12-offline-first-architecture)
13. [Authentication](#13-authentication)
14. [API Layer](#14-api-layer)
15. [File Upload](#15-file-upload)
16. [Notifications](#16-notifications)
17. [Production Best Practices](#17-production-best-practices)
18. [Build & Deployment](#18-build--deployment)
19. [Scalable Project Structure](#19-scalable-project-structure)
20. [Cross-Cutting Best Practices Summary](#20-cross-cutting-best-practices-summary)

---

## 1. Expo Project Structure

### What is Expo?

Expo is a framework and set of tools built on top of React Native. It provides:

- A **build service** (EAS Build) that compiles native iOS/Android binaries without you owning a Mac or Android Studio setup (though you still can use them).
- A **unified SDK** (`expo-camera`, `expo-location`, `expo-sqlite`, etc.) — pre-built native modules with consistent JS APIs, so you rarely write native Swift/Kotlin code yourself.
- **Expo Router**, a file-based navigation system built on React Navigation.
- **Expo Go**, a sandbox app for quick iteration (limited to modules bundled in Expo Go).
- **Config plugins**, which let you modify native `Info.plist` / `AndroidManifest.xml` declaratively from `app.json`/`app.config.ts` instead of hand-editing native projects.

### How Expo works internally

Expo apps are still React Native apps. What Expo adds is:

- A **build orchestration layer** (`expo prebuild`) that generates the native `ios/` and `android/` folders from your JS config and installed config plugins. This is called the **Continuous Native Generation (CNG)** model — native folders are *derived*, not hand-maintained.
- **`expo-modules-core`**, a modern native-module architecture (Swift on iOS, Kotlin on Android) that most first-party Expo packages use, replacing the older bridge-based modules.
- At runtime, JS talks to native code either through the legacy bridge (JSON serialization) or the **New Architecture** (JSI + TurboModules + Fabric), which Expo SDK 51+ enables by default. The New Architecture allows synchronous JS↔native calls and better performance for things like camera preview and SQLite.

### Managed vs Bare workflow

| | Managed workflow | Bare workflow |
|---|---|---|
| Native folders | Not checked into git; generated via `prebuild` | Checked into git, hand-edited |
| Native code changes | Via config plugins | Directly in Xcode/Android Studio |
| Best for | 95% of apps, including this guide's app | Apps needing a custom native SDK Expo doesn't wrap |

**Recommendation for this guide:** use the **managed workflow with a Development Build** (not Expo Go) — this is the modern default. You get the CNG convenience of managed workflow, but you can still add any native module (via config plugins or `npx expo prebuild`) because you build your own dev client instead of relying on the fixed set of modules inside Expo Go.

```bash
npx create-expo-app@latest field-survey-app --template blank-typescript
cd field-survey-app
npx expo install expo-dev-client
```

### `app.json` vs `app.config.ts`

`app.json` is a static JSON file describing your app (name, icon, splash, plugins, permissions strings). `app.config.ts` is the same thing, but as executable TypeScript — use it when config needs to vary by environment (e.g., different bundle IDs for staging vs production) or read from `.env`.

```ts
// app.config.ts
import { ExpoConfig, ConfigContext } from 'expo/config';

export default ({ config }: ConfigContext): ExpoConfig => ({
  ...config,
  name: process.env.APP_VARIANT === 'staging' ? 'Field Survey (Staging)' : 'Field Survey',
  slug: 'field-survey-app',
  version: '1.0.0',
  orientation: 'portrait',
  icon: './assets/icon.png',
  scheme: 'fieldsurvey',
  userInterfaceStyle: 'automatic',
  splash: {
    image: './assets/splash.png',
    resizeMode: 'contain',
    backgroundColor: '#0f172a',
  },
  ios: {
    supportsTablet: true,
    bundleIdentifier:
      process.env.APP_VARIANT === 'staging'
        ? 'com.yourorg.fieldsurvey.staging'
        : 'com.yourorg.fieldsurvey',
    infoPlist: {
      ITSAppUsesNonExemptEncryption: false,
    },
  },
  android: {
    package:
      process.env.APP_VARIANT === 'staging'
        ? 'com.yourorg.fieldsurvey.staging'
        : 'com.yourorg.fieldsurvey',
    adaptiveIcon: {
      foregroundImage: './assets/adaptive-icon.png',
      backgroundColor: '#0f172a',
    },
    permissions: [], // fine-grained permissions are added by each plugin below
  },
  plugins: [
    'expo-router',
    'expo-sqlite',
    [
      'expo-camera',
      {
        cameraPermission: 'Allow $(PRODUCT_NAME) to access your camera to capture survey photos.',
        microphonePermission: 'Allow $(PRODUCT_NAME) to access your microphone to record video with audio.',
      },
    ],
    [
      'expo-location',
      {
        locationAlwaysAndWhenInUsePermission:
          'Allow $(PRODUCT_NAME) to use your location to tag survey entries and track field visits, even when the app is in the background.',
        isAndroidBackgroundLocationEnabled: true,
        isAndroidForegroundServiceEnabled: true,
      },
    ],
    [
      'expo-image-picker',
      {
        photosPermission: 'Allow $(PRODUCT_NAME) to access your photos to attach evidence to surveys.',
      },
    ],
    [
      'expo-av',
      {
        microphonePermission: 'Allow $(PRODUCT_NAME) to access your microphone to record audio notes.',
      },
    ],
    ['expo-notifications', { icon: './assets/notification-icon.png', color: '#0f172a' }],
    'expo-secure-store',
    'expo-background-task',
  ],
  extra: {
    apiUrl: process.env.API_URL ?? 'https://api.fieldsurvey.example.com',
    eas: { projectId: 'your-eas-project-id' },
  },
});
```

**Why `app.config.ts` here:** we need `process.env.APP_VARIANT` to switch bundle IDs for staging/production, and TypeScript catches typos in config keys — a plain JSON file can't do either.

### Folder structure (preview — full version in Section 19)

```
field-survey-app/
├── app/                 # Expo Router routes (screens)
├── src/
│   ├── components/
│   ├── hooks/
│   ├── services/
│   ├── database/
│   ├── context/
│   ├── constants/
│   ├── utils/
│   └── types/
├── assets/
├── app.config.ts
├── tsconfig.json
├── package.json
└── eas.json
```

### Best practices

- **Never edit `ios/`/`android/` directly** if you're on managed workflow with CNG — your changes get wiped on the next `prebuild`. Use config plugins instead.
- Pin your Expo SDK version and run `npx expo install` (not plain `npm install`) for Expo/React Native packages — it resolves the version compatible with your SDK.
- Run `npx expo-doctor` before every release to catch dependency mismatches.
- Keep `app.config.ts` free of secrets — API keys go in EAS Secrets / `.env` files excluded from git, referenced via `extra` at build time for public values only.

### How native modules work

A native module (e.g., `expo-camera`) ships:
1. JS/TS API surface (`import { CameraView } from 'expo-camera'`).
2. Native Swift (iOS) and Kotlin (Android) implementation, registered via `expo-modules-core`.
3. A **config plugin** (the `withCamera` function) that, during `prebuild`, injects the required `Info.plist` keys and `AndroidManifest.xml` permissions automatically — this is why you declare `cameraPermission` in `app.config.ts` instead of editing `Info.plist` by hand.

When you call `Camera.requestCameraPermissionsAsync()` in JS, it crosses the JSI bridge to native code, which shows the OS permission dialog and returns a promise that resolves with the result.

---

## 2. Expo Router

### What is Expo Router?

Expo Router is a **file-based routing** library for React Native, built on top of React Navigation. Every file you add under the `app/` directory automatically becomes a route — there's no manual `Stack.Screen` registration like in classic React Navigation.

### How file-based routing works

The router walks the `app/` directory tree at build time and turns it into a navigation tree:

| File | Route |
|---|---|
| `app/index.tsx` | `/` |
| `app/settings.tsx` | `/settings` |
| `app/survey/[id].tsx` | `/survey/123` |
| `app/survey/[id]/edit.tsx` | `/survey/123/edit` |
| `app/(tabs)/home.tsx` | `/home` (group doesn't affect URL) |
| `app/+not-found.tsx` | 404 fallback |

### Folder structure for the survey app

```
app/
├── _layout.tsx                 # Root layout (providers, fonts, splash)
├── +not-found.tsx
├── (auth)/
│   ├── _layout.tsx             # Stack for unauthenticated flow
│   ├── login.tsx
│   └── forgot-password.tsx
├── (tabs)/
│   ├── _layout.tsx             # Tab bar layout
│   ├── index.tsx               # Home tab
│   ├── surveys/
│   │   ├── _layout.tsx         # Nested stack for surveys
│   │   ├── index.tsx           # Survey list -> /surveys
│   │   ├── [id].tsx            # Survey detail -> /surveys/42
│   │   └── new.tsx             # New survey -> /surveys/new
│   ├── map.tsx
│   └── settings.tsx
└── modals/
    ├── _layout.tsx
    └── camera.tsx               # Presented as a modal
```

### `_layout.tsx` and the Root Layout

A `_layout.tsx` file defines the **navigator** (Stack, Tabs, or Drawer) for everything inside its folder. The root `_layout.tsx` wraps the entire app and is the right place for global providers.

```tsx
// app/_layout.tsx
import { useEffect, useState } from 'react';
import { Stack, SplashScreen } from 'expo-router';
import { useFonts } from 'expo-font';
import { GestureHandlerRootView } from 'react-native-gesture-handler';
import { SafeAreaProvider } from 'react-native-safe-area-context';
import { AuthProvider, useAuth } from '@/context/AuthContext';
import { DatabaseProvider } from '@/context/DatabaseContext';
import { SyncProvider } from '@/context/SyncContext';

SplashScreen.preventAutoHideAsync();

export default function RootLayout() {
  const [fontsLoaded] = useFonts({
    'Inter-Regular': require('@/assets/fonts/Inter-Regular.ttf'),
    'Inter-SemiBold': require('@/assets/fonts/Inter-SemiBold.ttf'),
  });

  useEffect(() => {
    if (fontsLoaded) SplashScreen.hideAsync();
  }, [fontsLoaded]);

  if (!fontsLoaded) return null;

  return (
    <GestureHandlerRootView style={{ flex: 1 }}>
      <SafeAreaProvider>
        <DatabaseProvider>
          <AuthProvider>
            <SyncProvider>
              <RootNavigator />
            </SyncProvider>
          </AuthProvider>
        </DatabaseProvider>
      </SafeAreaProvider>
    </GestureHandlerRootView>
  );
}

// Split out so useAuth() can be called *inside* a provider
function RootNavigator() {
  const { isAuthenticated, isLoading } = useAuth();

  if (isLoading) return null; // splash still visible logically, or a loader

  return (
    <Stack screenOptions={{ headerShown: false }}>
      <Stack.Protected guard={isAuthenticated}>
        <Stack.Screen name="(tabs)" />
      </Stack.Protected>
      <Stack.Protected guard={!isAuthenticated}>
        <Stack.Screen name="(auth)" />
      </Stack.Protected>
      <Stack.Screen name="modals" options={{ presentation: 'modal' }} />
      <Stack.Screen name="+not-found" />
    </Stack>
  );
}
```

**Why this structure:** `SplashScreen.preventAutoHideAsync()` keeps the native splash visible until fonts (and, in a real app, your auth/session bootstrap) finish loading, avoiding a flash of unstyled content. `Stack.Protected` (Expo Router v4+) declaratively gates route groups behind an auth condition instead of manually redirecting in `useEffect`, which avoids race conditions and flicker.

### Nested layouts

Any subfolder can have its own `_layout.tsx`, nesting a new navigator inside the parent's. `app/(tabs)/surveys/_layout.tsx` defines a **Stack** that lives inside the "Surveys" **Tab**, so pushing `[id].tsx` slides in from the right while the tab bar stays visible (unless you hide it per-screen).

```tsx
// app/(tabs)/surveys/_layout.tsx
import { Stack } from 'expo-router';

export default function SurveysLayout() {
  return (
    <Stack>
      <Stack.Screen name="index" options={{ title: 'Surveys' }} />
      <Stack.Screen name="[id]" options={{ title: 'Survey Detail' }} />
      <Stack.Screen name="new" options={{ title: 'New Survey', presentation: 'modal' }} />
    </Stack>
  );
}
```

### Route groups `(tabs)`

Parentheses around a folder name — `(tabs)`, `(auth)` — create a **group**: it organizes routes and layouts without adding a path segment to the URL. `app/(tabs)/index.tsx` is served at `/`, not `/(tabs)`. This is purely an organizational tool.

### Dynamic routes `[id].tsx`

A filename in square brackets captures a URL segment as a parameter, retrieved with `useLocalSearchParams()`:

```tsx
// app/(tabs)/surveys/[id].tsx
import { useLocalSearchParams, useRouter } from 'expo-router';
import { useSurvey } from '@/hooks/useSurvey';

export default function SurveyDetailScreen() {
  const { id } = useLocalSearchParams<{ id: string }>();
  const { survey, isLoading } = useSurvey(id);
  const router = useRouter();

  if (isLoading) return <LoadingView />;
  if (!survey) return <NotFoundView />;

  return <SurveyDetail survey={survey} onEdit={() => router.push(`/surveys/${id}/edit`)} />;
}
```

### Catch-all routes

`[...missing].tsx` matches any number of remaining segments — useful for a section-level 404 or for deep-link paths with variable depth (e.g., `app/docs/[...slug].tsx` matching `/docs/a/b/c`). `useLocalSearchParams()` then returns `slug` as a string array.

### Modal routes

Set `presentation: 'modal'` (iOS: card slides up; Android: configurable) on a `Stack.Screen`, either in the parent layout's `options` or by wrapping the modal routes in their own group with a dedicated layout, as shown in the folder structure above (`app/modals/`).

### Stack, Tab, and Drawer Navigation

- **Stack** (`expo-router` re-exports React Navigation's native stack): screens pushed on top of each other, back button pops. Default for most flows (list → detail → edit).
- **Tab** (`Tabs` from `expo-router`): persistent bottom/top bar switching between top-level sections. Covered in depth in Section 3.
- **Drawer** (`expo-router/drawer`, needs `react-native-reanimated` + `react-native-gesture-handler`): a slide-out side menu, useful for admin/settings-heavy apps with many top-level destinations that don't all fit in a tab bar.

```tsx
// Drawer example: app/(drawer)/_layout.tsx
import { Drawer } from 'expo-router/drawer';

export default function DrawerLayout() {
  return (
    <Drawer screenOptions={{ headerShown: true }}>
      <Drawer.Screen name="index" options={{ drawerLabel: 'Dashboard' }} />
      <Drawer.Screen name="reports" options={{ drawerLabel: 'Reports' }} />
    </Drawer>
  );
}
```

### Deep linking

Because Expo Router maps URLs to files, deep linking works out of the box for the `scheme` you set in `app.config.ts` (`fieldsurvey://surveys/42` opens `app/(tabs)/surveys/[id].tsx` with `id=42`). For universal links (`https://fieldsurvey.example.com/surveys/42`), add `associatedDomains` (iOS) / `intentFilters` (Android) — Expo Router's config plugin handles most of this when you set `origin` in your router config.

### Navigation methods

```tsx
import { router } from 'expo-router';

router.push('/surveys/42');        // push a new screen onto the stack
router.replace('/surveys/42');     // swap current screen, no back-history entry
router.back();                     // pop current screen
router.navigate('/surveys/42');    // push, but re-use an existing matching screen instead of duplicating
router.setParams({ tab: 'photos' }); // update params of current screen without navigating
```

| Method | Adds to history? | Use case |
|---|---|---|
| `push` | Yes | Drilling into detail from a list |
| `replace` | No (replaces top) | After login success → replace `/login` with `/home` so back doesn't return to login |
| `back` | Removes top entry | Cancel buttons, header back |
| `navigate` | Smart (reuses if already present) | Tab-like navigation where duplicate screens are undesirable |

### How navigation history works internally

Expo Router builds on React Navigation's state model: each navigator maintains a **stack of route objects** (`{ key, name, params }`) in memory. `push` appends a new route object; `back`/`pop` removes the top one; the visual transition animates between the previous and new top-of-stack screen. Route **state is preserved by default** — going back to a previous stack screen does not remount it unless you explicitly unmount (e.g., via `unmountOnBlur`). Deep links and typed routes are resolved by matching the incoming URL against the file tree at parse time, then that matched path is pushed onto the relevant navigator's stack, creating intermediate stack entries for any parent layouts along the way (so deep-linking straight to `/surveys/42` still gives you a working back button to `/surveys`).
