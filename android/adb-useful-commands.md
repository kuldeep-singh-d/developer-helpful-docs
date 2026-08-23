# ADB Useful Commands

A practical reference for working with Android devices and emulators through Android Debug Bridge (ADB).

## Check devices

```bash
# List connected devices and their connection state.
adb devices -l
```

### Use case

Run this first when an Android command cannot find a device.

## Target one device

```bash
# Run a command on one device when multiple devices are connected.
adb -s emulator-5554 shell getprop ro.product.model
```

Replace `emulator-5554` with an identifier from `adb devices`.

## Install an APK

```bash
# Install an APK and replace an existing compatible installation.
adb install -r app-debug.apk
```

See [Android APK Installation Commands](android-apk-installation-commands.md) for signing, downgrade, and installation-error guidance.

## Open an interactive shell

```bash
# Open a shell directly on the selected Android device.
adb shell
```

Use `exit` to return to the local terminal.

## Copy files

```bash
# Copy a local file to the device Download directory.
adb push sample.json /sdcard/Download/sample.json

# Copy a file from the device to the current local directory.
adb pull /sdcard/Download/sample.json .
```

## Capture a screenshot

```bash
# Save a device screenshot directly to the current directory.
adb exec-out screencap -p > android-screen.png
```

## Forward a React Native development port

```bash
# Allow a USB-connected device to reach Metro on the computer.
adb reverse tcp:8081 tcp:8081

# Remove that port mapping when it is no longer needed.
adb reverse --remove tcp:8081
```

## Restart ADB

```bash
# Restart the ADB server when devices are not detected correctly.
adb kill-server
adb start-server
adb devices
```

## Quick reference

| Task | Command |
|---|---|
| List devices | `adb devices -l` |
| Target a device | `adb -s DEVICE_ID COMMAND` |
| Install APK | `adb install -r app-debug.apk` |
| Open shell | `adb shell` |
| Copy to device | `adb push LOCAL REMOTE` |
| Copy from device | `adb pull REMOTE LOCAL` |
| Restart ADB | `adb kill-server` then `adb start-server` |

> [!WARNING]
> Commands such as `adb shell pm clear`, `adb uninstall`, and emulator wipe commands remove application or device data. See [Android Emulator Cleanup Commands](android-emulator-cleanup-commands.md) before using them.
