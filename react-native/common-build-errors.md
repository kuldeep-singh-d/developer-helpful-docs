# Common React Native Build Errors

A diagnosis-first checklist for common Android and iOS React Native build failures.

## No Android device found

```bash
# Confirm that ADB detects the device.
adb devices -l

# Restart ADB if the device is missing.
adb kill-server
adb start-server
```

## Android Gradle build fails

```bash
# Run the Android build with a useful stack trace.
cd android
./gradlew assembleDebug --stacktrace
```

Read the first `Caused by` section. Check JDK, Android SDK, Gradle, and plugin compatibility before clearing caches.

## iOS pods are missing or out of sync

```bash
# Install dependencies using versions locked by Podfile.lock.
cd ios
pod install
```

Open the generated `.xcworkspace`, not the `.xcodeproj`, when CocoaPods is used.

## Metro cannot resolve a module

```bash
# Reinstall locked npm dependencies.
npm ci

# Restart Metro with a fresh transform cache if necessary.
npx react-native start --reset-cache
```

First verify the import path, filename casing, and dependency declaration.

## App installs but cannot reach Metro

```bash
# Connect a USB Android device to Metro's default port.
adb reverse tcp:8081 tcp:8081
```

## Safe escalation order

```text
Read first error → Verify tools/paths → Reinstall locked dependencies → Clean affected platform → Reset caches → Reset device last
```
