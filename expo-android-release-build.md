# Building a Release Android App (APK & AAB) from an Expo React Native Project

> A complete, beginner-friendly guide to generating signed release builds using native Gradle — covering everything from project setup to Play Store–ready bundles.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Step 1 — Convert to Native with `expo prebuild`](#step-1--convert-to-native-with-expo-prebuild)
4. [Step 2 — Generate a Keystore](#step-2--generate-a-keystore)
5. [Step 3 — Configure the Keystore in Gradle](#step-3--configure-the-keystore-in-gradle)
6. [Step 4 — Build the App](#step-4--build-the-app)
7. [Output File Locations](#output-file-locations)
8. [Install APK on a Device](#install-apk-on-a-device)
9. [Common Errors and Fixes](#common-errors-and-fixes)
10. [Best Practices](#best-practices)
11. [APK vs AAB — What's the Difference?](#apk-vs-aab--whats-the-difference)

---

## Introduction

When you're ready to publish your Expo React Native app to the Google Play Store — or distribute it directly to users — you need a **signed release build**. This guide walks you through the entire process using the **native Gradle build system**, giving you full control over your build pipeline.

There are two output formats:

- **APK** (Android Package) — A single installable file, perfect for direct distribution or testing on devices.
- **AAB** (Android App Bundle) — The format required by Google Play Store for new submissions. Google optimises delivery for each device automatically.

This guide covers both.

---

## Prerequisites

Before you begin, make sure the following tools are installed and configured on your machine.

### Required Software

| Tool | Version | Purpose |
|---|---|---|
| **Node.js** | 18 LTS or higher | JavaScript runtime for Expo CLI |
| **Java JDK** | 17 (recommended) | Required by Gradle and Android tooling |
| **Android Studio** | Latest stable | Android SDK, emulator, and build tools |
| **Expo CLI** | Latest | Project management and prebuild |
| **React Native CLI** | Latest | Optional but useful for native builds |

### Installation Checklist

**Node.js**

Download from [https://nodejs.org](https://nodejs.org) or use a version manager like `nvm`:

```bash
nvm install 18
nvm use 18
node --version   # should print v18.x.x
```

**Java JDK 17**

Download from [https://adoptium.net](https://adoptium.net) (Temurin/OpenJDK).

After installation, set the `JAVA_HOME` environment variable:

- **macOS / Linux** — add to `~/.zshrc` or `~/.bashrc`:

```bash
export JAVA_HOME=$(/usr/libexec/java_home -v 17)
export PATH=$JAVA_HOME/bin:$PATH
```

- **Windows** — set via System Properties → Environment Variables:

```
JAVA_HOME = C:\Program Files\Eclipse Adoptium\jdk-17.x.x
```

Verify:

```bash
java -version
# openjdk version "17.x.x" ...
```

**Android Studio & SDK**

1. Download Android Studio from [https://developer.android.com/studio](https://developer.android.com/studio).
2. During setup, install the **Android SDK**, **Android SDK Platform-Tools**, and **Android Emulator**.
3. Set the `ANDROID_HOME` environment variable:

- **macOS / Linux:**

```bash
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$ANDROID_HOME/emulator:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools:$PATH
```

- **Windows:**

```
ANDROID_HOME = C:\Users\<YourName>\AppData\Local\Android\Sdk
```

**Expo CLI**

```bash
npm install -g expo-cli
# or using the newer EAS CLI (optional)
npm install -g eas-cli
```

---

## Step 1 — Convert to Native with `expo prebuild`

Expo projects start in **Managed Workflow** — Expo handles the native code for you. To build with Gradle directly, you need to **eject to Bare Workflow** and generate the native `android/` and `ios/` folders.

> ⚠️ **Important:** After running `prebuild`, the `android/` folder is generated and managed by you. Re-running `prebuild` can overwrite your changes unless you use the `--clean` flag carefully.

### Run Prebuild

Navigate to your project root and run:

```bash
cd your-expo-project
npx expo prebuild --platform android
```

This will:

- Generate the `android/` directory with all native files
- Install required native dependencies
- Configure your `app.json` / `app.config.js` settings into native files

### What Gets Generated

```
your-expo-project/
├── android/
│   ├── app/
│   │   ├── build.gradle        ← app-level Gradle config
│   │   └── src/
│   ├── build.gradle            ← project-level Gradle config
│   ├── gradle.properties       ← keystore config goes here
│   └── gradlew                 ← Gradle wrapper (use this to build)
├── ios/
├── app.json
└── package.json
```

### Install JS Dependencies (if not done already)

```bash
npm install
# or
yarn install
```

---

## Step 2 — Generate a Keystore

A **keystore** is a binary file that holds your app's private signing key. Every release build must be signed with this key. **You must use the same keystore for every future update** — losing it means you cannot update your app on the Play Store.

### Generate Using `keytool`

`keytool` is bundled with the Java JDK. Run the following command from your project root (or any directory — just note where the file is saved):

```bash
keytool -genkeypair \
  -v \
  -keystore my-release-key.keystore \
  -alias my-key-alias \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000
```

You will be prompted to enter:

```
Enter keystore password:
Re-enter new password:
What is your first and last name?
What is the name of your organizational unit?
What is the name of your organization?
What is the name of your City or Locality?
What is the name of your State or Province?
What is the two-letter country code for this unit?
```

### Parameter Explanation

| Parameter | Value | Description |
|---|---|---|
| `-keystore` | `my-release-key.keystore` | Output filename for the keystore file. Use a descriptive name. |
| `-alias` | `my-key-alias` | A name to identify this specific key inside the keystore. Remember this — you'll need it in Gradle. |
| `-keyalg` | `RSA` | The encryption algorithm. RSA is the standard for Android signing. |
| `-keysize` | `2048` | Key length in bits. 2048 is the minimum recommended size. |
| `-validity` | `10000` | How many days the key is valid. 10,000 days ≈ 27 years. Google Play requires keys valid past October 2033. |
| `-v` | _(flag)_ | Verbose output — shows certificate details during generation. |

### Move the Keystore Into Your Project

Copy the generated `.keystore` file into the `android/app/` directory:

```bash
cp my-release-key.keystore android/app/my-release-key.keystore
```

> 🔐 **Security Note:** Never commit this file to a public repository. See [Best Practices](#best-practices) below.

---

## Step 3 — Configure the Keystore in Gradle

Gradle needs to know where your keystore is and how to use it when signing the release build.

### 3a — Add Credentials to `gradle.properties`

Open `android/gradle.properties` and add the following lines at the bottom:

```properties
MYAPP_UPLOAD_STORE_FILE=my-release-key.keystore
MYAPP_UPLOAD_KEY_ALIAS=my-key-alias
MYAPP_UPLOAD_STORE_PASSWORD=your_keystore_password
MYAPP_UPLOAD_KEY_PASSWORD=your_key_password
```

Replace the values with your actual alias and passwords from Step 2.

> ✅ Storing credentials in `gradle.properties` (instead of hardcoding in `build.gradle`) keeps sensitive data out of version-controlled source files — **as long as you add `gradle.properties` to `.gitignore`**.

### 3b — Configure `signingConfigs` in `build.gradle`

Open `android/app/build.gradle` and find the `android { ... }` block. Add a `signingConfigs` block and reference it in `buildTypes`:

```groovy
android {
    // ... existing config ...

    signingConfigs {
        release {
            if (project.hasProperty('MYAPP_UPLOAD_STORE_FILE')) {
                storeFile file(MYAPP_UPLOAD_STORE_FILE)
                storePassword MYAPP_UPLOAD_STORE_PASSWORD
                keyAlias MYAPP_UPLOAD_KEY_ALIAS
                keyPassword MYAPP_UPLOAD_KEY_PASSWORD
            }
        }
    }

    buildTypes {
        debug {
            signingConfig signingConfigs.debug
        }
        release {
            signingConfig signingConfigs.release  // ← reference the release config
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

### What Each Field Means

| Field | Description |
|---|---|
| `storeFile` | Path to your `.keystore` file, relative to `android/app/` |
| `storePassword` | Password for the keystore itself |
| `keyAlias` | The alias you set with `-alias` in `keytool` |
| `keyPassword` | Password for the specific key (often the same as `storePassword`) |
| `minifyEnabled` | Enables ProGuard/R8 code shrinking and obfuscation for release |
| `shrinkResources` | Removes unused resources from the final build |

---

## Step 4 — Build the App

All Gradle commands are run from inside the `android/` directory. Use the **Gradle wrapper** (`gradlew`) — do not use a globally installed Gradle.

```bash
cd android
```

> On **macOS / Linux**, use `./gradlew`. On **Windows**, use `gradlew.bat` or just `gradlew`.

### Build a Debug APK

Use this for testing on a device or emulator without release signing:

```bash
./gradlew assembleDebug
```

### Build a Release APK

A signed APK ready for direct distribution:

```bash
./gradlew assembleRelease
```

### Build a Release AAB (for Google Play Store)

The preferred format for Play Store submissions:

```bash
./gradlew bundleRelease
```

### Full Clean Build (Recommended Before Release)

Clears the build cache first to avoid stale artifact issues:

```bash
./gradlew clean assembleRelease
# or
./gradlew clean bundleRelease
```

---

## Output File Locations

After a successful build, your files will be located here:

| Build Type | Output Path |
|---|---|
| **Debug APK** | `android/app/build/outputs/apk/debug/app-debug.apk` |
| **Release APK** | `android/app/build/outputs/apk/release/app-release.apk` |
| **Release AAB** | `android/app/build/outputs/bundle/release/app-release.aab` |

### Verify the Release APK is Signed

```bash
# macOS / Linux
$ANDROID_HOME/build-tools/<version>/apksigner verify --verbose android/app/build/outputs/apk/release/app-release.apk

# Or use jarsigner
jarsigner -verify -verbose -certs android/app/build/outputs/apk/release/app-release.apk
```

A verified, signed APK will output: `jar verified.`

---

## Install APK on a Device

### Option 1 — ADB (Android Debug Bridge)

Connect your Android device via USB with **USB Debugging** enabled, then run:

```bash
# Install the release APK
adb install android/app/build/outputs/apk/release/app-release.apk

# If a previous version is installed, uninstall first
adb uninstall com.yourcompany.yourapp
adb install android/app/build/outputs/apk/release/app-release.apk

# Or use the -r flag to reinstall (keeps data)
adb install -r android/app/build/outputs/apk/release/app-release.apk
```

### Option 2 — Manual Transfer

1. Copy the `.apk` file to your phone (via USB, email, Google Drive, etc.).
2. On your Android device, go to **Settings → Security → Install Unknown Apps** and allow installation from your file manager.
3. Open the `.apk` file and tap **Install**.

### Option 3 — Wireless ADB (Android 11+)

1. Enable **Wireless Debugging** in Developer Options on your device.
2. Pair your device:

```bash
adb pair <device-ip>:<pairing-port>
```

3. Connect and install:

```bash
adb connect <device-ip>:<port>
adb install app-release.apk
```

---

## Common Errors and Fixes

### ❌ `JAVA_HOME is not set` or `Cannot find java`

**Cause:** The `JAVA_HOME` environment variable is missing or pointing to the wrong JDK.

**Fix:**

```bash
# macOS — find the correct path
/usr/libexec/java_home -V

# Set it in your shell profile (~/.zshrc or ~/.bashrc)
export JAVA_HOME=$(/usr/libexec/java_home -v 17)
source ~/.zshrc
```

On Windows, set `JAVA_HOME` via **System Properties → Environment Variables** and restart your terminal.

---

### ❌ `Gradle build failed: Could not find tools.jar`

**Cause:** Gradle is pointing to a JRE instead of a full JDK.

**Fix:** Ensure `JAVA_HOME` points to a **JDK** directory (which contains `bin/javac`), not just a JRE.

```bash
ls $JAVA_HOME/bin/javac   # Should exist
```

---

### ❌ `SDK location not found`

**Cause:** The `ANDROID_HOME` or `sdk.dir` path is not configured.

**Fix:** Create or edit `android/local.properties`:

```properties
sdk.dir=/Users/<your-username>/Library/Android/sdk
# Windows example:
# sdk.dir=C\:\\Users\\<YourName>\\AppData\\Local\\Android\\Sdk
```

---

### ❌ `Keystore file not found` or `wrong password`

**Cause:** The keystore path or credentials in `gradle.properties` are incorrect.

**Fix:**

1. Verify the keystore file exists: `ls android/app/my-release-key.keystore`
2. Double-check the filename, alias, and passwords in `android/gradle.properties`.
3. Test your keystore credentials manually:

```bash
keytool -list -v \
  -keystore android/app/my-release-key.keystore \
  -alias my-key-alias
```

---

### ❌ `Execution failed for task ':app:mergeReleaseResources'`

**Cause:** Duplicate or conflicting resource files.

**Fix:** Run a clean build:

```bash
cd android
./gradlew clean
./gradlew assembleRelease
```

---

### ❌ `Permission denied: ./gradlew`

**Cause:** The Gradle wrapper script is not executable (common on macOS/Linux after cloning).

**Fix:**

```bash
chmod +x android/gradlew
```

---

### ❌ `Could not resolve com.android.tools.build:gradle`

**Cause:** Network issues or outdated Gradle plugin version.

**Fix:**

1. Check your internet connection.
2. In `android/build.gradle`, update the Android Gradle Plugin to a compatible version:

```groovy
dependencies {
    classpath "com.android.tools.build:gradle:8.1.0"
}
```

3. Sync with a stable Gradle distribution by updating `android/gradle/wrapper/gradle-wrapper.properties`:

```properties
distributionUrl=https\://services.gradle.org/distributions/gradle-8.0-all.zip
```

---

## Best Practices

### 🔐 Keep Your Keystore Safe

- **Back up your keystore file** in a secure location (encrypted cloud storage, password manager, etc.).
- If you lose your keystore, you **cannot publish updates** to your existing Play Store app — you would need to create a new app listing.
- Store the keystore password in a password manager (e.g., 1Password, Bitwarden).

### 🚫 Never Upload Your Keystore to GitHub

Add these entries to your `.gitignore`:

```gitignore
# Android signing
android/app/*.keystore
android/app/*.jks
android/gradle.properties
```

> Alternatively, keep `gradle.properties` in `.gitignore` and use environment variables in CI/CD pipelines (GitHub Actions, Bitrise, Fastlane) to inject credentials at build time.

### ♻️ Always Use the Same Keystore for Updates

Google Play identifies your app by its signing certificate. Every update must be signed with the **same key** as the original release. Using a different keystore will cause Google Play to reject the update.

### 📌 Version Your Builds Properly

Before each release, increment `versionCode` (an integer, must increase with each upload) and `versionName` (a human-readable string) in `android/app/build.gradle`:

```groovy
android {
    defaultConfig {
        versionCode 2          // Increment by 1 for each Play Store upload
        versionName "1.1.0"    // Semantic versioning for display
    }
}
```

### 🛡️ Use Environment Variables in CI/CD

Never hardcode credentials in source files. In your CI pipeline, inject secrets:

```yaml
# Example: GitHub Actions
- name: Build Release AAB
  env:
    MYAPP_UPLOAD_STORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
    MYAPP_UPLOAD_KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
  run: |
    cd android
    ./gradlew bundleRelease
```

---

## APK vs AAB — What's the Difference?

| Feature | APK | AAB |
|---|---|---|
| **Full form** | Android Package | Android App Bundle |
| **File extension** | `.apk` | `.aab` |
| **Installation** | Direct — install on any Android device | Cannot install directly; requires Google Play |
| **File size** | Larger — contains resources for all screen densities and ABIs | Smaller — Google Play generates optimised APKs per device |
| **Play Store requirement** | Accepted, but legacy format | **Required for new apps** since August 2021 |
| **Best for** | Direct distribution, sideloading, beta testing | Google Play Store submissions |
| **Splitting** | No — all resources bundled together | Yes — Play Delivery splits by language, density, and ABI |

### Summary

- Use **APK** (`assembleRelease`) when you want to distribute your app directly (e.g., to testers, internal users, or via your own website).
- Use **AAB** (`bundleRelease`) when you are publishing to the **Google Play Store**. It reduces download size for end users and is the required format for new Play Store apps.

---

## Quick Reference — Build Commands

```bash
# From the android/ directory:

# Debug APK
./gradlew assembleDebug

# Release APK (signed)
./gradlew assembleRelease

# Release AAB (for Play Store)
./gradlew bundleRelease

# Clean + Release APK
./gradlew clean assembleRelease

# Clean + Release AAB
./gradlew clean bundleRelease
```

---

*Generated for Expo SDK 50+ with React Native 0.73+. Commands and paths may vary slightly for older SDK versions.*
