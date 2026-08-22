# iOS Simulator Cleanup Commands

A clear reference for cleaning, resetting, and troubleshooting the iOS Simulator from Terminal or the VS Code terminal.

> These commands are mainly for macOS with Xcode installed.

---

## 1. List available iOS Simulators

```bash
# Shows all simulator devices installed on your Mac.
# Use this when you need to find the exact simulator name or UDID.
xcrun simctl list devices
```

To show only currently available devices:

```bash
# Hides unavailable/old simulator entries.
xcrun simctl list devices available
```

### Use case

Use this when:

- You do not know the simulator name.
- You need the simulator UDID.
- You have multiple iPhone/iPad simulators installed.

Example output may look like:

```text
iPhone 16 Pro (XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX) (Shutdown)
```

The long value inside parentheses is the simulator **UDID**.

---

## 2. Check currently booted simulator

```bash
# Shows the simulator that is currently running.
xcrun simctl list devices booted
```

### Use case

Useful before running commands against a simulator so you can confirm which device is active.

---

## 3. Shutdown all running simulators

```bash
# Stops every currently running iOS Simulator.
xcrun simctl shutdown all
```

### Use case

Use this when:

- Simulator is frozen.
- You want to reset simulator state cleanly.
- A simulator refuses to boot correctly.
- You want to perform a full erase.

---

## 4. Erase ALL iOS Simulator data

```bash
# WARNING: DESTRUCTIVE
# Factory-resets every installed simulator.
# Removes installed apps, app data, accounts, settings, permissions, etc.
xcrun simctl erase all
```

### Use case

Use this when you want a completely clean simulator environment.

Typical situations:

- React Native app has old/corrupted simulator data.
- Permissions are stuck in a strange state.
- Login/account/session data must be completely removed.
- Multiple simulators are behaving incorrectly.
- You want all simulators back to factory-reset state.

> This command resets **all simulator devices**, not only one.

---

## 5. Erase only ONE simulator

First find its UDID:

```bash
# Find the simulator UDID.
xcrun simctl list devices available
```

Then:

```bash
# WARNING: DESTRUCTIVE
# Replace SIMULATOR_UDID with the actual device UDID.
xcrun simctl erase SIMULATOR_UDID
```

Example:

```bash
# Example only — use your actual UDID.
xcrun simctl erase XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
```

### Use case

Use this instead of `erase all` when only one simulator needs a factory reset.

---

## 6. Erase the currently booted simulator

You can use the special identifier `booted`:

```bash
# WARNING: Simulator normally needs to be shutdown before erase.
xcrun simctl shutdown booted
```

Then erase it using its UDID.

A safer flow is:

```bash
# Step 1: See which simulator is booted.
xcrun simctl list devices booted

# Step 2: Shutdown running simulators.
xcrun simctl shutdown all

# Step 3: Erase the specific simulator using its UDID.
xcrun simctl erase SIMULATOR_UDID
```

---

## 7. Open the iOS Simulator application

```bash
# Opens Apple's Simulator app.
open -a Simulator
```

### Use case

Useful when you are working entirely from VS Code/Terminal and want to launch Simulator without opening Xcode.

---

## 8. Boot a specific simulator from Terminal

First list devices:

```bash
xcrun simctl list devices available
```

Then boot using its UDID:

```bash
# Starts the selected simulator.
xcrun simctl boot SIMULATOR_UDID
```

Then open the Simulator UI:

```bash
open -a Simulator
```

### Use case

Useful when you want to select exactly which iPhone/iPad simulator React Native should use.

---

## 9. Shutdown one specific simulator

```bash
# Stops only the simulator identified by this UDID.
xcrun simctl shutdown SIMULATOR_UDID
```

### Use case

Use this when multiple simulators exist and you do not want to shutdown all of them.

---

## 10. Uninstall only your app from the booted simulator

```bash
# Removes only your application from the currently booted simulator.
# Replace com.example.app with your actual iOS bundle identifier.
xcrun simctl uninstall booted com.example.app
```

Example:

```bash
xcrun simctl uninstall booted com.ebn
```

### Use case

Use this when:

- You want a fresh app installation.
- Old app data is causing issues.
- You do NOT want to factory-reset the whole simulator.

This is usually much less destructive than `simctl erase`.

---

## 11. Find the iOS Bundle Identifier

For a React Native project, open your iOS project settings in Xcode and check:

```text
Targets → Your App → General → Bundle Identifier
```

It may look like:

```text
com.company.appname
```

You may also find references inside project files such as:

```text
ios/<ProjectName>.xcodeproj/project.pbxproj
```

---

## 12. Reinstall/run your React Native app

From the React Native project root:

```bash
# Builds and launches the iOS version of the React Native app.
npx react-native run-ios
```

To run on a specific simulator by name:

```bash
# Replace the simulator name with one installed on your Mac.
npx react-native run-ios --simulator="iPhone 16 Pro"
```

### Use case

Use this after uninstalling your app or resetting the simulator.

---

## 13. Delete unavailable simulator entries

```bash
# Removes simulator records whose runtimes are no longer available.
xcrun simctl delete unavailable
```

### Use case

Useful after:

- Updating Xcode.
- Removing an old iOS runtime.
- Seeing duplicate/unavailable simulator entries.

This does NOT normally erase your active valid simulators.

---

## 14. Kill the Simulator application

```bash
# Force-closes the Simulator GUI application.
killall Simulator
```

### Use case

Useful when the Simulator app itself is frozen or not responding.

Then reopen it:

```bash
open -a Simulator
```

---

## 15. Restart CoreSimulator services

```bash
# Force-restarts Apple's CoreSimulator service for the current user.
killall -9 com.apple.CoreSimulator.CoreSimulatorService
```

Then reopen Simulator:

```bash
open -a Simulator
```

### Use case

Use this when:

- Simulator does not boot.
- `simctl` commands behave strangely.
- Simulator UI is stuck even after closing/reopening.
- React Native cannot communicate correctly with Simulator.

> Try normal shutdown/restart commands before using this.

---

## 16. Clear React Native Metro cache

This does not reset the iOS Simulator itself, but it is often useful during React Native debugging.

```bash
# Starts Metro and clears its transform cache.
npx react-native start --reset-cache
```

### Use case

Use this when:

- JavaScript changes are not appearing correctly.
- Metro is serving stale bundles.
- You see strange module/cache-related errors.

---

## 17. Clean iOS Xcode build files

From your React Native project:

```bash
# Go into the iOS directory.
cd ios

# Removes Xcode build output for the workspace/project.
xcodebuild clean

# Return to project root.
cd ..
```

### Use case

Useful when native iOS builds are failing because of stale build output.

This is different from resetting the simulator.

---

## 18. Remove Xcode DerivedData

```bash
# WARNING: Deletes Xcode's cached build/index data for all projects.
rm -rf ~/Library/Developer/Xcode/DerivedData/*
```

### Use case

Use this when:

- Xcode builds are behaving inconsistently.
- Native module changes are not being picked up.
- Build artifacts appear corrupted.
- `xcodebuild clean` does not fix the issue.

> This does not erase simulator user data, but the next Xcode build may take longer.

---

## 19. Clean CocoaPods and reinstall pods

From your React Native project:

```bash
# Move into the iOS project.
cd ios

# Remove installed Pods.
rm -rf Pods

# Remove the Pod lock file if you intentionally want a fresh dependency resolution.
rm -f Podfile.lock

# Install pods again.
pod install

# Return to project root.
cd ..
```

### Use case

Use this when:

- Native dependencies are broken.
- CocoaPods installation is corrupted.
- Pod/native module errors appear after package changes.

> Removing `Podfile.lock` can change dependency versions. Do it only when you actually want fresh pod resolution.

A safer first attempt is:

```bash
cd ios
pod install
cd ..
```

---

## 20. Quick reference — most useful commands

```bash
# List available simulators.
xcrun simctl list devices available

# Show booted simulator.
xcrun simctl list devices booted

# Open Simulator.
open -a Simulator

# Shutdown all simulators.
xcrun simctl shutdown all

# Factory reset ALL simulators.
xcrun simctl erase all

# Factory reset ONE simulator.
xcrun simctl erase SIMULATOR_UDID

# Uninstall only your app.
xcrun simctl uninstall booted com.example.app

# Remove unavailable simulator records.
xcrun simctl delete unavailable

# Force-close Simulator.
killall Simulator

# Restart CoreSimulator service.
killall -9 com.apple.CoreSimulator.CoreSimulatorService

# Run React Native app on iOS.
npx react-native run-ios

# Run on a specific simulator.
npx react-native run-ios --simulator="iPhone 16 Pro"

# Reset Metro cache.
npx react-native start --reset-cache
```

---

# Which command should I use?

| Situation | Command |
|---|---|
| Want a completely fresh simulator | `xcrun simctl erase SIMULATOR_UDID` |
| Want to reset every simulator | `xcrun simctl erase all` |
| Only want to reinstall/reset your app | `xcrun simctl uninstall booted com.example.app` |
| Simulator is frozen | `xcrun simctl shutdown all` |
| Simulator app itself is frozen | `killall Simulator` |
| Simulator service is broken | `killall -9 com.apple.CoreSimulator.CoreSimulatorService` |
| Old/unavailable simulators are listed | `xcrun simctl delete unavailable` |
| Metro cache seems stale | `npx react-native start --reset-cache` |
| Native Xcode build cache is broken | `rm -rf ~/Library/Developer/Xcode/DerivedData/*` |
| CocoaPods/native dependencies are broken | `cd ios && pod install` |
| Don't know simulator UDID/name | `xcrun simctl list devices available` |

---

# Safe recommended workflow for React Native iOS debugging

Try commands from least destructive to most destructive.

```bash
# 1. Check simulator state.
xcrun simctl list devices booted

# 2. Restart the running simulator.
xcrun simctl shutdown all
open -a Simulator

# 3. Try running the React Native app again.
npx react-native run-ios

# 4. If only your app state is bad, uninstall the app.
xcrun simctl uninstall booted com.example.app

# 5. Install/run it again.
npx react-native run-ios

# 6. If Metro appears stale, reset Metro cache.
npx react-native start --reset-cache

# 7. If native build files are stale, clean Xcode.
cd ios
xcodebuild clean
cd ..

# 8. If build caching is badly corrupted, remove DerivedData.
rm -rf ~/Library/Developer/Xcode/DerivedData/*

# 9. If the simulator itself is corrupted, shutdown it.
xcrun simctl shutdown all

# 10. Finally, factory-reset the affected simulator.
xcrun simctl erase SIMULATOR_UDID
```

---

# Important destructive commands

Be careful with these:

```bash
# Deletes ALL simulator user data across all simulators.
xcrun simctl erase all

# Deletes all user data from one simulator.
xcrun simctl erase SIMULATOR_UDID

# Deletes Xcode DerivedData for all projects.
rm -rf ~/Library/Developer/Xcode/DerivedData/*

# Removes CocoaPods installation and lock file.
rm -rf ios/Pods
rm -f ios/Podfile.lock
```

Before running destructive commands, make sure you understand what they remove.

---

# Recommended rule

For most React Native issues, use this order:

```text
Restart Simulator
        ↓
Re-run app
        ↓
Uninstall only the app
        ↓
Reset Metro cache
        ↓
Clean Xcode build
        ↓
Clear DerivedData
        ↓
Erase one simulator
        ↓
Erase all simulators only as a last resort
```

This avoids wiping more data than necessary.
