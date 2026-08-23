# React Native Development Environment on macOS

A verification-first checklist for a bare React Native environment targeting Android and iOS. Exact supported versions change, so confirm them in the current React Native documentation before installation.

## Required tools

- Node.js and the package manager used by the project
- Watchman
- A supported JDK
- Android Studio, Android SDK, Platform Tools, and an emulator
- Xcode, Xcode Command Line Tools, and an iOS Simulator
- CocoaPods for projects that use it

## Verify the environment

```bash
# Check JavaScript and package-manager versions.
node --version
npm --version

# Check Android and Java tools.
java -version
adb version
emulator -list-avds

# Check Apple development tools.
xcode-select -p
xcodebuild -version
xcrun simctl list devices available

# Check CocoaPods when the project uses it.
pod --version
```

## Android SDK environment

Configure `ANDROID_HOME` to the SDK location shown by Android Studio, and add the SDK's `emulator` and `platform-tools` directories to `PATH`. Do not blindly copy paths; verify the actual SDK location first.

```bash
# Confirm the configured Android SDK directory.
echo "$ANDROID_HOME"

# Confirm that required executables resolve from PATH.
command -v adb
command -v emulator
```

## Verify a new project

```bash
# Start Metro from the project root.
npx react-native start

# In separate terminals, build one platform at a time.
npx react-native run-android
npx react-native run-ios
```

## Troubleshooting order

```text
Check supported versions → Verify PATH → Verify device → Read first build error → Fix one platform → Clean caches only if evidence points to stale state
```
