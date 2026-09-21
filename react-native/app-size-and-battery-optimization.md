# React Native App Size and Battery Optimization

A measurement-first guide for reducing download/install size and preventing unnecessary battery use without breaking release builds or background features.

## Treat size and battery as separate budgets

Track both:

- Store download size by platform and device variant.
- Installed size and growth after normal use.
- JavaScript bundle, native code, resources, fonts, images, and media.
- Foreground and background energy use for critical journeys.
- Network bytes, wakeups, location activity, timers, and scheduled work.

Set budgets per release. “Smaller” and “uses less battery” are not verifiable goals without a baseline and test scenario.

## Measure the exact release artifact

Debug builds contain tooling and behave differently. Generate the same artifact intended for distribution.

Android:

```bash
# Build the release Android App Bundle from the project root.
npx react-native build-android --mode=release

# Show the local bundle file size.
du -h android/app/build/outputs/bundle/release/app-release.aab
```

The AAB file size is not identical to every user's Play download because stores generate optimized APKs for device configurations. Use Play Console size reports for delivered-size comparisons.

iOS:

```text
Xcode → Product → Archive → Distribute App
```

Use Xcode's distribution size report or App Store Connect metrics. Do not compare an uncompressed `.app`, archive, IPA, and store download as though they measure the same thing.

## Find what increased

When size changes, compare the same build type and inspect:

- New native SDKs and their transitive frameworks.
- Duplicate libraries or architectures.
- Images, videos, audio, fonts, and localization files.
- JavaScript dependencies and bundled data files.
- Debug resources accidentally included in release.
- Native symbols or artifacts packaged unnecessarily.

```bash
# Show the largest files under common source asset locations.
find . -type f \( -path './assets/*' -o -path './src/*' \) -print0 \
  | xargs -0 du -h \
  | sort -h \
  | tail -n 30
```

This is a source-tree clue, not a package analyzer. Confirm whether each file is actually present in the release artifact.

## Optimize assets safely

- Resize images close to their maximum rendered dimensions.
- Use an appropriate modern format supported by the target pipeline.
- Avoid bundling the same asset in several formats without a reason.
- Subset fonts only when localization and accessibility coverage remain correct.
- Stream or download large optional media rather than packaging it when product requirements allow.
- Remove unused launch images, sample files, and abandoned resources.

> [!WARNING]
> Lossy compression, font subsetting, and asset removal can damage visual quality, localization, accessibility, or offline behavior. Compare representative screens and languages before release.

## Review JavaScript and native dependencies

For each dependency, ask:

- Is it used in production code?
- Does a built-in platform or small local implementation already meet the need?
- Does it add native binaries to both platforms?
- Does it bundle large locale, icon, or data sets?
- Can imports target only the required feature?
- Is it maintained and compatible with the current React Native architecture?

Do not remove a package only because it looks unused in JavaScript; native initialization, configuration plugins, build scripts, or reflection may depend on it.

## Test Android shrinking in release

Android release optimization can remove unused code/resources and obfuscate symbols. Treat it as a release feature that needs testing.

```bash
# Build the Android release bundle with detailed failure output.
cd android
./gradlew bundleRelease --stacktrace
```

If optimization causes a crash, add the narrow keep rules required by the affected library. Preserve the release `mapping.txt` for crash deobfuscation.

See [Release Build and Signing Troubleshooting](release-build-and-signing-troubleshooting.md) before changing R8 configuration.

## Measure battery with a repeatable journey

Example:

```text
Charge and stabilize test device
→ Install the same release build
→ Run a 15-minute scripted journey
→ Include five minutes in background
→ Record CPU, network, location, wakeups, and energy
→ Repeat after one focused change
```

Use Android Studio Power Profiler/system tracing and Xcode Instruments. Avoid drawing conclusions from the battery percentage alone; it is coarse and affected by temperature, radio conditions, screen brightness, and other processes.

## Inspect Android battery state

```bash
# Show battery statistics associated with one package where supported.
adb shell dumpsys batterystats com.example.app
```

Look for repeated wakeups, long-running work, location usage, foreground services, and frequent network-radio activity. Compare identical time windows and device conditions.

Review output before sharing because device and application activity may be sensitive.

## Reduce background work

- Schedule deferrable work with platform-supported schedulers.
- Combine related uploads/downloads instead of waking the radio repeatedly.
- Use sensible constraints such as connectivity and charging when the task allows.
- Stop observers, timers, location updates, and sensors when no longer needed.
- Avoid JavaScript intervals as a guarantee of background execution.
- Keep push-triggered work short, idempotent, and resumable.
- Respect user battery, data, and background restrictions.

The operating system may defer background work. Design the feature so delayed execution is safe.

## Optimize network energy

Radio activation and repeated connections can cost more energy than the payload alone.

- Cache stable data with clear expiration rules.
- Batch safe operations.
- Compress appropriate payloads.
- Paginate large responses.
- Cancel requests whose result is no longer needed.
- Use bounded retry with backoff and jitter.
- Avoid polling when push or user-initiated refresh meets the requirement.
- Do not download high-resolution media before it is needed.

See [Offline Mode and Network Retry Strategies](offline-mode-and-network-retry-strategies.md) for safe retry behavior.

## Location, sensors, and animation

- Request only the accuracy and update frequency the feature needs.
- Stop location and sensor subscriptions promptly.
- Prefer significant-change or region-based behavior where platform APIs and requirements allow.
- Pause expensive animation/video when not visible.
- Test high-refresh-rate devices and thermal throttling.
- Provide reduced-motion behavior when supported.

Never silently reduce safety-critical accuracy. Make the tradeoff explicit in the product design.

## Common mistakes

- Measuring debug builds.
- Comparing unlike artifacts or different store variants.
- Optimizing only the JavaScript bundle while native SDKs dominate size.
- Deleting symbols or mapping files needed for production crash diagnosis.
- Adding broad R8 keep rules that disable most shrinking.
- Repeatedly polling or retrying while offline.
- Keeping high-accuracy location active after leaving the feature.
- Testing battery only on a simulator or plugged-in device.

## Recommended workflow

```text
Set size and energy budgets
        ↓
Build exact release artifact
        ↓
Measure one repeatable journey
        ↓
Identify largest asset/dependency or energy source
        ↓
Apply one focused change
        ↓
Rebuild and repeat measurement
        ↓
Test functionality, crash symbols, and constrained devices
        ↓
Track trend for every release
```

## Related guides

- [React Native App Performance and Memory Debugging](app-performance-and-memory-debugging.md)
- [Offline Mode and Network Retry Strategies](offline-mode-and-network-retry-strategies.md)
- [Push Notification Troubleshooting](push-notification-troubleshooting.md)
- [Device and OS Fragmentation Testing](device-and-os-fragmentation-testing.md)

## Official references

- [React Native: Optimizing JavaScript Loading](https://reactnative.dev/docs/optimizing-javascript-loading)
- [Android: Enable App Optimization with R8](https://developer.android.com/topic/performance/app-optimization/enable-app-optimization)
- [Android: Profile Types and Use Cases](https://developer.android.com/topic/performance/tracing/profile-types-overview)
- [Android Vitals](https://developer.android.com/topic/performance/vitals)
- [Apple: Reducing Your App's Size](https://developer.apple.com/documentation/xcode/reducing-your-app-s-size)
