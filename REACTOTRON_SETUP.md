# 🔭 Reactotron Integration Guide

> **Reusable guide** for adding Reactotron debugging to any React Native / Expo app.
> Stack: **Expo + Axios + Redux Toolkit + TypeScript + Physical Android Device via ADB**

---

## 📖 What is Reactotron?

Reactotron is a **desktop debugging app** for React Native. It lets you:
- See all **network requests/responses** in real time
- View and inspect **Redux state** and dispatched actions
- Read/write **AsyncStorage** key-value pairs
- Log **custom debug messages** from anywhere in your code
- Track **JS errors and warnings**

Think of it as Chrome DevTools but for React Native.

---

## 🧩 What Was Done in This Project

Here's a breakdown of every file that was created/modified and **why**:

### 1. `src/config/reactotron.ts` — The Core Setup File

This is the **main Reactotron configuration file**. It:
- Connects the app to the Reactotron desktop app
- Enables **networking plugin** (captures HTTP calls)
- Enables **AsyncStorage tracking** (watch storage changes)
- Enables **error tracking** (JS crashes reported in Reactotron)
- Enables **Redux plugin** (watch state + dispatched actions)
- Ignores noisy Metro/Expo internal URLs
- Patches `console.tron` so you can log from anywhere

```ts
// src/config/reactotron.ts
import Reactotron from "reactotron-react-native";
import AsyncStorage from "@react-native-async-storage/async-storage";
import { reactotronRedux } from "reactotron-redux";

Reactotron
  .configure({
    name: "YourAppName",   // shown in Reactotron desktop
    host: "localhost",     // "localhost" when using ADB reverse proxy
    port: 9090,            // default Reactotron port
  })
  .setAsyncStorageHandler(AsyncStorage)  // track storage reads/writes
  .useReactNative({
    asyncStorage: { ignore: ["secret"] },  // ignore sensitive keys
    networking: {
      ignoreUrls: /symbolicate|hot-update|\/logs|\.bundle\?|metro/,
    },
    errors: { veto: () => false },  // send all JS errors to Reactotron
    overlay: false,
  })
  .use(reactotronRedux())   // Redux state integration
  .connect();

console.tron = Reactotron;  // makes console.tron available everywhere
export default Reactotron;
```

---

### 2. `src/types/reactotron.d.ts` — TypeScript Type Fix

Without this file, TypeScript throws an error:
> `Property 'tron' does not exist on type 'Console'`

This file **extends** TypeScript's built-in `Console` interface to include `console.tron`:

```ts
// src/types/reactotron.d.ts
import Reactotron from "reactotron-react-native";

declare global {
  interface Console {
    tron: typeof Reactotron;
  }
}
export {};
```

---

### 3. `src/services/axiosInstance.ts` — Axios Interceptors with Logging

The **request interceptor** and **response interceptor** were updated to log:

| Interceptor | Logs |
|-------------|------|
| Request | method, full URL, headers, body, params |
| Response (success) | status code, URL, response data |
| Response (error) | status code, URL, error message, full error body |

All logs are wrapped in `if (__DEV__)` — **zero overhead in production**.
Logs go to both `console.log` and `console.tron.display()` (visible in Reactotron Timeline).

```ts
axiosInstance.interceptors.request.use((config) => {
  if (__DEV__) {
    console.log(`[Axios ➜ Request] ${config.method?.toUpperCase()} ${config.url}`);
    console.tron?.display({
      name: "🚀 Axios Request",
      value: { method: config.method, url: config.url, body: config.data },
    });
  }
  return config;
});
```

---

### 4. `src/store/index.ts` — Redux Enhancer

Reactotron's Redux plugin needs a **store enhancer** to hook into dispatched actions.
It's added conditionally — only in `__DEV__` mode:

```ts
const reactotronEnhancer =
  __DEV__ && console.tron
    ? (console.tron as any).createEnhancer?.()
    : undefined;

export const store = configureStore({
  reducer: { auth: authReducer },
  enhancers: (getDefaultEnhancers) =>
    reactotronEnhancer
      ? getDefaultEnhancers().concat(reactotronEnhancer)
      : getDefaultEnhancers(),
});
```

---

### 5. `App.tsx` — Import Order (Critical!)

The Reactotron config **must be loaded before everything else** — before Redux store, before any component. This is handled at the very top of `App.tsx`:

```ts
// App.tsx — MUST be the very first lines
if (__DEV__) {
  require("./src/config/reactotron");
}

import { NavigationContainer } from "@react-navigation/native";
// ... rest of imports
```

> ⚠️ If Reactotron is imported after the Redux store, the Redux enhancer won't connect properly.

---

## 📦 Packages Installed

```bash
# Already included in reactotron-react-native (no extra install needed):
# - networking plugin
# - asyncStorage plugin
# - error tracking

# Extra packages needed:
npm install reactotron-redux --save-dev
```

### Full devDependency list for Reactotron:
```json
"devDependencies": {
  "reactotron-react-native": "^5.1.18",
  "reactotron-redux": "^3.1.3"
}
```

---

## 🔌 Connection Method: ADB Reverse Proxy (Physical Android Device)

### Why ADB Reverse?
- ✅ No need to know your PC's IP address
- ✅ Works even if phone and PC aren't on the same Wi-Fi
- ✅ More stable than Wi-Fi connection
- ✅ Works via USB cable

### How it works:
```
Phone (USB) ──adb reverse tcp:9090 tcp:9090──▶ PC Reactotron (port 9090)
```
The phone thinks it's connecting to its own `localhost:9090`, but ADB silently tunnels it through USB to your PC.

### Setup Steps:
```powershell
# 1. Enable USB Debugging on your Android phone:
#    Settings → Developer Options → USB Debugging → ON

# 2. Connect phone via USB and run:
adb reverse tcp:9090 tcp:9090   # Reactotron
adb reverse tcp:3000 tcp:3000   # Your backend API
adb reverse tcp:8081 tcp:8081   # Metro bundler (optional)

# 3. Start Expo with localhost flag:
npx expo start --localhost
```

> 🔁 Re-run `adb reverse` every time you **replug the USB cable**.

---

## 🗂️ Complete File Summary

```
myapp/
├── App.tsx                          ← require reactotron FIRST inside __DEV__
├── src/
│   ├── config/
│   │   └── reactotron.ts           ← Main Reactotron setup
│   ├── types/
│   │   └── reactotron.d.ts         ← TypeScript fix for console.tron
│   ├── services/
│   │   └── axiosInstance.ts        ← Axios with request/response/error logging
│   └── store/
│       └── index.ts                ← Redux store with Reactotron enhancer
```

---

## ♻️ Reusing in a New App — Step-by-Step Checklist

Follow these steps exactly when integrating Reactotron into a **brand new project**:

### Step 1 — Install packages
```bash
npm install reactotron-react-native --save-dev
npm install reactotron-redux --save-dev
```

### Step 2 — Create `src/config/reactotron.ts`
Copy the config from the section above. Change:
- `name:` → your app name
- `host: "localhost"` → keep as-is for ADB reverse

### Step 3 — Create `src/types/reactotron.d.ts`
Copy the type declaration from the section above. No changes needed.

### Step 4 — Update `App.tsx` (or `_layout.tsx` for Expo Router)
Add at the **very top** before all other imports:
```ts
if (__DEV__) {
  require("./src/config/reactotron");
}
```

### Step 5 — Update your Axios instance
Add `if (__DEV__)` blocks inside request and response interceptors.
Use `console.tron.display(...)` to send named log entries to Reactotron.

### Step 6 — Update your Redux store
Add the Reactotron enhancer using the pattern shown above.

### Step 7 — ADB Reverse (run every session)
```powershell
adb reverse tcp:9090 tcp:9090
adb reverse tcp:3000 tcp:3000
```

### Step 8 — Start Reactotron desktop app first, then your Expo app
```bash
npx expo start --localhost
```

---

## 🖥️ Using Reactotron Desktop App

### Timeline Tab (main view)
| Entry | Meaning |
|-------|---------|
| 🚀 Axios Request | Outgoing API call |
| ✅ Axios Response | Successful response |
| 🔴 Axios Error | Failed request |
| `console.log` output | Regular logs |
| Redux action | State change |

### Logging from your code
```ts
// Simple log
console.tron.log("Debug message", { data: 123 });

// Named display block (easier to spot in timeline)
console.tron.display({
  name: "🔑 Auth Check",
  value: { token: "...", userId: 5 },
  important: true,  // appears highlighted in RED
});

// Warning
console.tron.warn("Slow network detected");
```

### Tips
- 🗑️ Click the **trash icon** to clear logs
- 🔍 Use the **filter bar** to search by name (e.g. type "Axios")
- 📌 Enable **Always on Top** in the View menu for side-by-side debugging

---

## 🛠️ Troubleshooting

| Problem | Solution |
|---------|----------|
| "Waiting for connection..." | Run `adb reverse tcp:9090 tcp:9090` |
| App crashes on startup | Check that `reactotron-redux` is installed |
| TypeScript error on `console.tron` | Check `src/types/reactotron.d.ts` exists |
| No axios logs in Timeline | Check `axiosInstance.ts` has the interceptors |
| Redux actions not showing | Check store `enhancers` setup in `store/index.ts` |
| Works on emulator but not device | Use ADB reverse instead of IP address |

### Allow port 9090 through Windows Firewall (if needed):
```powershell
# Run as Administrator
New-NetFirewallRule -DisplayName "Reactotron" -Direction Inbound -Protocol TCP -LocalPort 9090 -Action Allow
```

---

*Guide created for RanneetiSurvey project — March 2026*
