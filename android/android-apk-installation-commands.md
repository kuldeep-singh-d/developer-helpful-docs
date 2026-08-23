# Android APK Installation Commands

A practical reference for installing, replacing, and troubleshooting APK files with ADB.

## Check the target device

```bash
# List devices before installing an APK.
adb devices -l
```

## Install an APK

```bash
# Install an APK on the connected device.
adb install app-debug.apk

# Replace an installed compatible app while keeping its data.
adb install -r app-debug.apk
```

### Use case

Use `-r` during development when the package and signing key match the installed app.

## Install on one selected device

```bash
# Install when more than one device or emulator is connected.
adb -s emulator-5554 install -r app-debug.apk
```

## Allow a version-code downgrade

> [!WARNING]
> Downgrade behavior depends on the app build and Android version. Back up important app data first.

```bash
# Replace a debuggable app with an APK that has a lower version code.
adb install -r -d app-debug.apk
```

## Common installation errors

| Error | Likely cause | Safer next step |
|---|---|---|
| `INSTALL_FAILED_UPDATE_INCOMPATIBLE` | Signing key differs | Verify signing; uninstall only if losing app data is acceptable |
| `INSTALL_FAILED_VERSION_DOWNGRADE` | APK version code is lower | Use a newer APK or `-d` for an appropriate debug build |
| `INSTALL_FAILED_INSUFFICIENT_STORAGE` | Device storage is low | Remove unnecessary files or apps |
| `more than one device/emulator` | Multiple targets are connected | Add `-s DEVICE_ID` |

## Uninstall before a clean install

> [!WARNING]
> Uninstalling removes the app and its local data from the device.

```bash
# Remove the application package and its local data.
adb uninstall com.example.app

# Install the APK as a fresh application.
adb install app-debug.apk
```

## Quick reference

```bash
# Normal install.
adb install app-debug.apk

# Replace compatible installation.
adb install -r app-debug.apk

# Select a device.
adb -s DEVICE_ID install -r app-debug.apk
```
