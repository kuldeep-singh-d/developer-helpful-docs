# React Native Device and OS Fragmentation Testing

A practical guide for testing React Native apps across device performance levels, screen sizes, operating-system versions, accessibility settings, and manufacturer-specific behavior.

## Do not test every device equally

Build a risk-based device matrix from:

- Active-user analytics by model and OS version.
- Business-critical markets and user groups.
- Minimum supported Android and iOS versions.
- Low-memory and low-storage devices.
- Small phones, large phones, tablets, foldables, and split-screen windows.
- Devices involved in production crashes, ANRs, or support reports.
- Hardware required by important features, such as camera, NFC, biometrics, or GPS.

Analytics show existing users, not future users. Always include the minimum supported OS and at least one constrained device even if current usage is low.

## Maintain a small representative matrix

Example:

| Priority | Device category | Purpose |
|---|---|---|
| Required | Low-end Android on minimum supported OS | Memory, CPU, storage, and old-platform behavior |
| Required | Popular mid-range Android | Main Android user experience |
| Required | Current Android flagship | Latest platform behavior |
| Required | Smallest supported iPhone | Layout, keyboard, and memory constraints |
| Required | Current iPhone | Latest iOS behavior |
| Conditional | Tablet or foldable | Resizable windows and large layouts |
| Conditional | Manufacturer with production issues | Vendor-specific background or hardware behavior |

Record the exact model, OS, build type, locale, font scale, orientation, and result for every important test.

## Build layouts from available space

Use live window dimensions instead of selecting layouts by device model:

```tsx
import {useWindowDimensions} from 'react-native';

export function ResponsiveContent() {
  const {width, height, fontScale} = useWindowDimensions();
  const useTwoColumns = width >= 720;

  // Render from current window space; it can change at runtime.
  return null;
}
```

`useWindowDimensions` updates when the window or font scale changes. This matters for rotation, split-screen, tablets, foldables, and accessibility text settings.

Avoid:

- Hard-coded full-screen dimensions captured once at module load.
- Device-model checks such as “if iPhone X.”
- Fixed heights for text-heavy content.
- Assuming portrait orientation or one safe-area shape.
- Shrinking text to hide overflow.

## Test screen and input variations

For each critical screen, test:

- Smallest supported width and height.
- Portrait and landscape when supported.
- Split-screen or resizable window behavior.
- Notches, rounded corners, home indicators, and safe areas.
- Keyboard open, closed, and switched between input methods.
- Long translated strings and right-to-left layout where supported.
- Large accessibility text and display scaling.
- Screen reader focus order and touch-target size.
- Empty, loading, error, and maximum-content states.

A screen is not responsive merely because it fits several emulator presets.

## Test pixel density and images

React Native layout uses density-independent units, while image quality and decoded memory depend on physical pixels.

- Supply correctly sized image variants.
- Avoid downloading full-resolution media for thumbnails.
- Check thin borders and icons on multiple pixel densities.
- Verify screenshots, canvas content, maps, and native views separately.
- Test low-memory behavior with image-heavy screens.

Use `PixelRatio` only when physical pixel information is genuinely required; do not multiply normal layout sizes manually.

## Test OS-version behavior

Create a checklist for behavior that changes by OS version:

- Runtime permissions and limited/partial access.
- Notification permission and background delivery.
- Storage and media access.
- Background execution and battery restrictions.
- Deep-link and app-link verification.
- WebView, TLS, and certificate behavior.
- Status/navigation bars and edge-to-edge layout.
- SDK or native-library minimum versions.

Read platform behavior-change documentation before increasing the Android target SDK or building with a new Xcode SDK.

## Reproduce device-specific issues

Capture:

```text
App version and source commit
Build type and environment
Device manufacturer and exact model
OS version and security patch
Memory/storage state
Locale, font/display scale, theme, and orientation
Power-saving, VPN, work-profile, and accessibility state
Exact steps, logs, screenshot/video, and frequency
```

Do not label a failure “Samsung bug” or “iOS bug” until the smallest failing configuration is identified.

Useful Android details:

```bash
# Show manufacturer, model, Android release, and API level.
adb shell getprop ro.product.manufacturer
adb shell getprop ro.product.model
adb shell getprop ro.build.version.release
adb shell getprop ro.build.version.sdk

# Show physical memory information.
adb shell cat /proc/meminfo

# Show available filesystem space.
adb shell df -h
```

Review output before sharing because device names, mounted paths, or test-environment details may be sensitive.

## Test constrained conditions

Use safe test accounts and non-production environments to check:

- Low storage.
- Memory pressure and process recreation.
- Slow or interrupted network.
- Battery Saver or Low Power Mode.
- Backgrounding during a critical operation.
- App update over an older installed version.
- Permission revoked while the app is not running.
- System time, locale, or theme changed.

> [!WARNING]
> Simulating low storage, clearing data, uninstalling, or resetting devices can remove test data. Use dedicated test devices and preserve required evidence first.

## Use emulators and real devices together

Emulators and simulators are excellent for fast layout and OS-version coverage. Physical devices remain important for:

- Real memory and thermal limits.
- Camera, biometric, Bluetooth, NFC, sensor, and radio behavior.
- Manufacturer-specific Android behavior.
- Push notifications and background execution.
- Battery and performance measurements.

Cloud device labs can expand coverage, but results, videos, screenshots, and logs may contain application data. Use controlled accounts and review data-retention settings.

## Prioritize failures

Use a consistent score:

```text
Priority = affected users × severity × feature importance × reproducibility
```

A rare cosmetic defect and a rare data-loss defect should not receive the same priority. Track device-specific workarounds with an owner and removal condition so they do not become permanent unexplained branches.

## Recommended release matrix

```text
Every change: lint, unit tests, one Android, one iOS
        ↓
Pull request: critical flows on representative emulators/simulators
        ↓
Release candidate: minimum OS + popular devices + constrained device
        ↓
High-risk feature: relevant hardware, locale, accessibility, and background states
        ↓
Post-release: monitor failures by app version, OS, and device model
```

## Related guides

- [React Native App Performance and Memory Debugging](app-performance-and-memory-debugging.md)
- [React Native Mobile Permissions Troubleshooting](mobile-permissions-troubleshooting.md)
- [React Native Release Build and Signing Troubleshooting](release-build-and-signing-troubleshooting.md)

## Official references

- [React Native `useWindowDimensions`](https://reactnative.dev/docs/usewindowdimensions)
- [React Native `PixelRatio`](https://reactnative.dev/docs/pixelratio)
- [React Native `Dimensions`](https://reactnative.dev/docs/dimensions)
- [Firebase Test Lab](https://firebase.google.com/docs/test-lab)
- [Android Device Streaming](https://firebase.google.com/docs/test-lab/android/android-device-streaming)
