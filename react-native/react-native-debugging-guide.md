# React Native Debugging Guide

A practical workflow for diagnosing JavaScript and native React Native problems.

## Read the first useful error

Start with the earliest application error rather than later cascading failures. Check Metro, the app's LogBox, and the native build terminal.

## Open the Developer Menu

```bash
# Open the React Native Developer Menu on Android through ADB.
adb shell input keyevent 82
```

On iOS Simulator, use `Control+Command+Z`. On Android Emulator, use `Command+M` on macOS or `Control+M` on Windows/Linux.

## Open React Native DevTools

```text
With Metro focused, press: j
```

Use DevTools for console messages, breakpoints, component inspection, and performance investigation.

## Read native logs

```bash
# Stream Android logs for React Native JavaScript messages.
adb logcat ReactNativeJS:V '*:S'

# Stream logs from the currently booted iOS Simulator.
xcrun simctl spawn booted log stream --level debug
```

See [Android Logcat Debugging](../android/android-logcat-debugging.md) and [iOS Device and Simulator Logs](../ios/ios-device-and-simulator-logs.md).

## Recommended flow

```text
Reproduce once → Read first error → Identify JS/Android/iOS layer → Inspect focused logs → Apply smallest fix → Re-test
```

> [!NOTE]
> Developer Menu, LogBox, and React Native DevTools are development features and are unavailable in release builds.
