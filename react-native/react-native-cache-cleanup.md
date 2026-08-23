# React Native Cache Cleanup

A least-destructive cleanup sequence for stale React Native development state.

## 1. Restart Metro normally

Stop Metro with `Control+C`, then restart it:

```bash
# Start Metro without deleting caches.
npx react-native start
```

## 2. Reset Metro's transform cache

```bash
# Start Metro and rebuild its JavaScript transform cache.
npx react-native start --reset-cache
```

### Use case

Use this when JavaScript changes are stale or Metro reports inconsistent module data.

## 3. Reinstall JavaScript dependencies

> [!WARNING]
> Removing `node_modules` deletes locally installed packages. Keep the lockfile and use the package manager already selected by the project.

```bash
# Remove installed packages while preserving package-lock.json.
rm -rf node_modules

# Reinstall the exact versions from package-lock.json.
npm ci
```

For Yarn or pnpm projects, use their lockfile-aware install command instead.

## 4. Clean platform builds

```bash
# Clean Android-generated build output.
cd android
./gradlew clean
cd ..

# Reinstall iOS pods without changing locked versions.
cd ios
pod install
cd ..
```

More focused guidance:

- [Gradle Cleanup and Troubleshooting](../android/gradle-cleanup-and-troubleshooting.md)
- [CocoaPods Troubleshooting](../ios/cocoapods-troubleshooting.md)
- [iOS Simulator Cleanup Commands](../ios/ios-simulator-cleanup-commands.md)

## Recommended order

```text
Restart Metro → Reset Metro cache → Reinstall dependencies → Clean one platform → Reset device only as a last resort
```
