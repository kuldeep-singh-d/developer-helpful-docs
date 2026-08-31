# React Native Mobile Permissions Troubleshooting

A practical guide for diagnosing camera, microphone, location, photos, notifications, contacts, and other protected-resource permissions on Android and iOS.

## Start with the feature, not the prompt

Before requesting access, confirm that the feature genuinely needs the protected resource. Prefer a platform feature that avoids broad permission access when it can satisfy the use case.

Ask only after the user starts the relevant action—for example, when they tap **Scan QR code**—and explain what the feature will do with the data.

## Use a clear permission-state model

Handle at least these application-level states:

| State | Application behavior |
|---|---|
| Not requested | Explain the feature, then request in context |
| Granted | Continue with the feature |
| Denied | Keep the app usable and explain how to retry |
| Blocked or never ask again | Offer a Settings shortcut without repeatedly prompting |
| Limited or partial | Use only the resources the user selected |
| Unavailable or restricted | Disable the feature and explain the platform limitation |

The exact states differ by platform and permission library. Do not map every non-granted result to the same user experience.

## Android: declaration and runtime request

Dangerous Android permissions generally require both:

1. A declaration in `android/app/src/main/AndroidManifest.xml`.
2. A runtime request on supported Android versions.

React Native provides `PermissionsAndroid` for Android runtime permissions:

```tsx
import {PermissionsAndroid, Platform} from 'react-native';

export async function requestCameraPermission(): Promise<boolean> {
  if (Platform.OS !== 'android') {
    return true;
  }

  const result = await PermissionsAndroid.request(
    PermissionsAndroid.PERMISSIONS.CAMERA,
    {
      title: 'Camera access',
      message: 'Camera access is needed to scan a QR code.',
      buttonPositive: 'Continue',
      buttonNegative: 'Not now',
    },
  );

  return result === PermissionsAndroid.RESULTS.GRANTED;
}
```

### Android troubleshooting checks

- Confirm the permission is present in the merged release manifest, not only a debug manifest.
- Check the device Android version and application target SDK.
- Verify the requested permission constant matches the feature.
- Handle `NEVER_ASK_AGAIN` separately from a normal denial.
- Do not assume permissions in the same group are granted together.
- Re-check permission immediately before using the protected API.

Inspect package permission state:

```bash
# Show package details, including requested and granted permissions.
adb shell dumpsys package com.example.app | grep -i -A 20 permission
```

Replace `com.example.app` with the real package name. If multiple devices are connected, add `-s DEVICE_ID` after `adb`.

## iOS: usage descriptions and native request

iOS protected resources require an appropriate purpose string in the application target's `Info.plist`. Examples include:

| Resource | Common purpose key |
|---|---|
| Camera | `NSCameraUsageDescription` |
| Microphone | `NSMicrophoneUsageDescription` |
| Photos read access | `NSPhotoLibraryUsageDescription` |
| Location while using app | `NSLocationWhenInUseUsageDescription` |
| Contacts | `NSContactsUsageDescription` |

The exact key depends on the API and access level. Confirm it in current Apple documentation and in every target that accesses the resource.

> [!WARNING]
> An iOS app can terminate when it accesses a protected resource without the required purpose string.

React Native core does not provide one universal iOS permission API. Follow the current instructions for the native module or permission library used by the project, including Podfile setup when required.

### iOS troubleshooting checks

- Confirm the purpose string exists in the built target, not only another plist file.
- Use a user-facing explanation specific to the feature.
- Check whether access is denied, restricted, limited, or previously granted.
- Test the real hardware capability; simulators cannot reproduce every permission flow.
- Verify application extensions have their own required configuration.

## Open application settings after blocking

When the operating system will no longer show a prompt, explain why Settings is needed and let the user choose whether to open it:

```tsx
import {Linking} from 'react-native';

export async function openApplicationSettings(): Promise<void> {
  await Linking.openSettings();
}
```

Do not send users to Settings after an ordinary first denial without context.

## Reset permission state for testing

Prefer changing the permission through device Settings. When a clean test is required, reinstalling or clearing app data also removes other local application state.

> [!WARNING]
> The following Android command deletes all local data for the selected application, including sessions, databases, files, and permission state.

```bash
# Reset one Android application's data and permissions.
adb shell pm clear com.example.app
```

For iOS Simulator, use **Settings → Privacy & Security** or the Simulator's reset controls only after preserving any needed test data.

## Common real-world failures

| Symptom | Likely investigation |
|---|---|
| Prompt never appears | Already decided, blocked state, missing declaration, unsupported resource, or request not reached |
| Android returns denied immediately | Manifest mismatch, OS policy, managed device, or previous denial |
| iOS app exits when feature opens | Missing or incorrect usage-description key |
| Debug works but release fails | Release manifest/plist, target, entitlement, or native-module configuration differs |
| Feature fails after grant | Permission succeeded but hardware/API initialization failed |
| Works on simulator only | Physical-device privacy, capability, or hardware behavior differs |
| Photos show only some items | User granted limited-library access |

## Recommended workflow

```text
Confirm feature needs access
        ↓
Check platform declaration
        ↓
Check current state
        ↓
Explain request in context
        ↓
Request once
        ↓
Handle granted, denied, blocked, limited, and unavailable
        ↓
Test on supported physical devices and release build
```

Never log private photos, contacts, precise location, microphone content, or other protected user data while debugging.

## Related guides

- [React Native Debugging Guide](react-native-debugging-guide.md)
- [React Native Release Build and Signing Troubleshooting](release-build-and-signing-troubleshooting.md)
- [Android Logcat Debugging](../android/android-logcat-debugging.md)
- [iOS Device and Simulator Logs](../ios/ios-device-and-simulator-logs.md)

## Official references

- [React Native PermissionsAndroid](https://reactnative.dev/docs/permissionsandroid)
- [Android: Request Runtime Permissions](https://developer.android.com/training/permissions/requesting)
- [Android: Declare App Permissions](https://developer.android.com/training/permissions/declaring)
- [Apple: Protecting the User's Privacy](https://developer.apple.com/documentation/uikit/protecting-the-user-s-privacy)
- [Apple: Protected Resources](https://developer.apple.com/documentation/bundleresources/protected-resources)
