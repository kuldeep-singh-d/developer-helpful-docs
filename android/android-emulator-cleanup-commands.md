# Android Emulator Cleanup Commands

A clear reference for wiping, resetting, and cleaning an Android Emulator from Terminal or the VS Code terminal.

> Example AVD used below: `Pixel_9_Pro`

---

## 1. List available Android Virtual Devices (AVDs)

```bash
# Shows all emulator names available on your machine.
# Use this when you are not sure what your AVD name is.
emulator -list-avds
```

Example output:

```text
Pixel_9_Pro
```

Use the exact name shown in the output for the commands below.

---

## 2. Fully wipe emulator data

```bash
# Factory-resets the selected emulator.
# Removes installed apps, app data, accounts, settings, and other user data.
# Use this when the emulator is behaving strangely or you want a completely clean device.
emulator -avd Pixel_9_Pro -wipe-data
```

### Use case

Use this when you want the emulator to behave like a fresh Android device.

Typical situations:

- React Native app is behaving differently because of old cached/device data.
- Old installed apps or settings are causing problems.
- Login/account data needs to be completely removed.
- You want to test the app from a clean emulator state.

---

## 3. Stop a running emulator

```bash
# Closes the currently connected Android Emulator.
# Useful before running a full wipe.
adb emu kill
```

Then start it again with a clean wipe:

```bash
# Starts Pixel_9_Pro after deleting its previous user data.
emulator -avd Pixel_9_Pro -wipe-data
```

### Recommended full-reset flow

```bash
# Step 1: Close the running emulator.
adb emu kill

# Step 2: Start it again with all user data wiped.
emulator -avd Pixel_9_Pro -wipe-data
```

---

## 4. Clear data for only one Android app

```bash
# Clears data only for the specified Android package.
# The rest of the emulator remains unchanged.
adb shell pm clear com.example.app
```

Example:

```bash
# Replace this package name with your actual app package.
adb shell pm clear com.example
```

### Use case

Use this when you only want to reset your app without resetting the whole emulator.

This removes things such as:

- App login/session
- SharedPreferences
- Local databases
- App files
- Cached app-specific data

It does **not** delete other apps or emulator settings.

---

## 5. Find your app package name

For React Native projects, check:

```text
android/app/build.gradle
```

Look for something similar to:

```gradle
applicationId "com.example.app"
```

You can also list installed packages:

```bash
# Lists all installed Android package names.
adb shell pm list packages
```

Filter the list:

```bash
# Replace "example" with part of your app/package name.
adb shell pm list packages | grep example
```

---

## 6. Uninstall only your app

```bash
# Completely uninstalls the app from the emulator.
# Useful when you want a clean reinstall.
adb uninstall com.example.app
```

Then reinstall/run your React Native app:

```bash
# Runs the Android app again from your React Native project.
npx react-native run-android
```

### Use case

Use this when:

- `pm clear` is not enough.
- You want to test a fresh installation.
- Native app installation/state seems corrupted.

---

## 7. Check connected Android devices/emulators

```bash
# Shows devices/emulators currently visible to ADB.
adb devices
```

Example:

```text
List of devices attached
emulator-5554    device
```

### Use case

Use this before running ADB commands if you are unsure whether the emulator is connected.

---

## 8. Restart ADB

```bash
# Stops the ADB server.
adb kill-server

# Starts the ADB server again.
adb start-server
```

Then verify:

```bash
adb devices
```

### Use case

Useful when:

- Emulator is running but `adb devices` does not show it.
- React Native says no Android device is connected.
- ADB is behaving inconsistently.

---

## 9. Restart emulator without wiping data

```bash
# Close the current emulator.
adb emu kill

# Start it normally without deleting data.
emulator -avd Pixel_9_Pro
```

### Use case

Use this for a normal emulator restart when you do **not** want to lose apps or settings.

---

## 10. Start emulator without loading an old snapshot

```bash
# Starts the emulator without restoring the previous Quick Boot snapshot.
# Existing user data is kept.
emulator -avd Pixel_9_Pro -no-snapshot-load
```

### Use case

Useful when the emulator gets stuck because of a broken or stale snapshot, but you do not want a full factory reset.

---

## 11. Cold boot from Terminal

```bash
# Prevents loading the previous snapshot and starts the device fresh.
# This is similar to Android Studio's "Cold Boot Now".
emulator -avd Pixel_9_Pro -no-snapshot-load
```

This keeps normal emulator data but does not restore the previous suspended state.

---

## 12. Most useful commands — quick reference

```bash
# Show available emulator names.
emulator -list-avds

# Check connected emulator/device.
adb devices

# Close running emulator.
adb emu kill

# Start emulator normally.
emulator -avd Pixel_9_Pro

# Full factory reset + start.
emulator -avd Pixel_9_Pro -wipe-data

# Start without previous snapshot.
emulator -avd Pixel_9_Pro -no-snapshot-load

# Clear only one app's data.
adb shell pm clear com.example.app

# Uninstall only one app.
adb uninstall com.example.app

# Restart ADB.
adb kill-server
adb start-server
```

---

# Which command should I use?

| Situation                             | Command                                       |
| ------------------------------------- | --------------------------------------------- |
| Want a completely fresh emulator      | `emulator -avd Pixel_9_Pro -wipe-data`        |
| Only want to reset your app           | `adb shell pm clear com.example.app`          |
| Want to completely reinstall your app | `adb uninstall com.example.app`               |
| Emulator is frozen/stuck              | `adb emu kill` then restart                   |
| Snapshot/Quick Boot is causing issues | `emulator -avd Pixel_9_Pro -no-snapshot-load` |
| ADB cannot detect emulator            | `adb kill-server` + `adb start-server`        |
| Don't know emulator name              | `emulator -list-avds`                         |

---

# Safe recommended workflow for React Native debugging

When you have an Android/React Native issue, try these steps from least destructive to most destructive:

```bash
# 1. Confirm emulator is connected.
adb devices

# 2. Clear only your app data first.
adb shell pm clear com.example.app

# 3. If needed, uninstall the app.
adb uninstall com.example.app

# 4. Run/install the app again.
npx react-native run-android

# 5. If the emulator itself still has issues, close it.
adb emu kill

# 6. Finally, perform a complete emulator reset.
emulator -avd Pixel_9_Pro -wipe-data
```

> `-wipe-data` is destructive. It resets the emulator and removes its user-installed apps, accounts, settings, and app data. Use it when a full clean device is actually needed.
