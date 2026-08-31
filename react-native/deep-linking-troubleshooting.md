# React Native Deep Linking Troubleshooting

A practical guide for diagnosing custom URL schemes, Android App Links, iOS Universal Links, and navigation behavior in React Native apps.

## Know which link type you are testing

| Link type | Example | Important behavior |
|---|---|---|
| Custom scheme | `exampleapp://products/42` | Another app may claim the same scheme |
| Android App Link | `https://example.com/products/42` | Android verifies the website-to-app association |
| iOS Universal Link | `https://example.com/products/42` | iOS verifies the associated domain |
| Ordinary web link | `https://example.com/products/42` | Opens in browser when app association is absent or fails |

Prefer verified HTTPS links for public user journeys because they can fall back to the website and provide stronger ownership verification.

> [!WARNING]
> Treat every incoming link as untrusted input. Never include passwords, access tokens, one-time secrets, or sensitive personal data in a URL.

## Test cold and warm application states

React Native receives links through two paths:

- `Linking.getInitialURL()` when a link launches the app.
- A `Linking` `url` event when the app is already running.

```tsx
import {Linking} from 'react-native';

export async function readInitialLink(): Promise<string | null> {
  return Linking.getInitialURL();
}

export function subscribeToLinks(handler: (url: string) => void) {
  const subscription = Linking.addEventListener('url', event => {
    handler(event.url);
  });

  return () => subscription.remove();
}
```

Test these states separately:

1. App not running.
2. App in background.
3. App open on another screen.
4. User logged out.
5. Navigation container still initializing.

## Test a custom scheme

Android:

```bash
# Ask Android to open a custom-scheme URL.
adb shell am start -W -a android.intent.action.VIEW \
  -d 'exampleapp://products/42'
```

iOS Simulator:

```bash
# Ask the booted iOS Simulator to open a custom-scheme URL.
xcrun simctl openurl booted 'exampleapp://products/42'
```

If multiple Android apps can handle the scheme, Android may show a chooser. Custom schemes do not prove domain ownership.

## Test an Android App Link

```bash
# Open an HTTPS link and show activity-launch details.
adb shell am start -W -a android.intent.action.VIEW \
  -d 'https://example.com/products/42'

# Inspect Android's verified app-link state for the package.
adb shell pm get-app-links com.example.app
```

Check:

- Intent filter action, categories, scheme, host, and path match the URL.
- `android:autoVerify="true"` is configured where appropriate.
- The application ID and signing certificate match the website association.
- The device can retrieve the association file over public HTTPS.
- Redirects, staging domains, and debug/release signing fingerprints are handled intentionally.

Inspect the association response:

```bash
# Check status, content type, and body for Android's association file.
curl --include https://example.com/.well-known/assetlinks.json
```

## Test an iOS Universal Link

```bash
# Open an HTTPS link on the booted Simulator.
xcrun simctl openurl booted 'https://example.com/products/42'

# Check status, content type, and body for Apple's association file.
curl --include https://example.com/.well-known/apple-app-site-association
```

Check:

- The Associated Domains capability contains `applinks:example.com`.
- Team ID and bundle identifier match the installed build.
- The association file includes the intended paths or components.
- The file is reachable through HTTPS with the correct response and no authentication.
- The user has not previously chosen to open that domain in the browser.

Universal-link association can be cached by the operating system. Reinstalling the app may trigger a fresh association check, but it also removes local app data.

> [!WARNING]
> Do not repeatedly uninstall an app that contains valuable test data. Verify the hosted association file and entitlements first.

## Validate and route incoming URLs

Before navigation:

1. Parse the URL with a standards-based parser.
2. Allow only expected schemes and hosts.
3. Match an explicit route pattern.
4. Decode parameters once and validate their types and length.
5. Check authentication and authorization again inside the app.
6. Reject unknown or malformed routes safely.

A deep link is navigation input, not proof that the user may access the requested resource.

## Common failures

| Symptom | Likely investigation |
|---|---|
| Link opens browser | Association file, entitlement/intent filter, installed build, or user preference |
| Custom scheme does nothing | Scheme registration, URL syntax, target application, or simulator capability |
| Cold start loses link | `getInitialURL`, app initialization, splash/auth race, or navigation readiness |
| Warm app handles link twice | Duplicate subscriptions or notification-plus-link handling |
| Android chooser appears | Link is not verified or multiple apps claim the URL |
| `canOpenURL` returns false | Missing Android package visibility or iOS queried-scheme configuration |
| Debug works but release fails | Different package/bundle ID, signing fingerprint, domain, or entitlement |

## Recommended workflow

```text
Identify link type
        ↓
Test exact URL in terminal
        ↓
Verify native registration
        ↓
Verify hosted association for HTTPS links
        ↓
Log sanitized incoming URL
        ↓
Test cold and warm app states
        ↓
Validate route and authentication
        ↓
Test signed release build on physical devices
```

## Related guides

- [React Native Debugging Guide](react-native-debugging-guide.md)
- [Push Notification Troubleshooting](push-notification-troubleshooting.md)
- [React Native Release Build and Signing Troubleshooting](release-build-and-signing-troubleshooting.md)

## Official references

- [React Native Linking](https://reactnative.dev/docs/linking)
- [React Native Security: Deep Linking](https://reactnative.dev/docs/security#deep-linking)
- [Android: Add Intent Filters for App Links](https://developer.android.com/training/app-links/deep-linking)
- [Android: Verify App Links](https://developer.android.com/training/app-links/verify-android-applinks)
- [Apple: Supporting Associated Domains](https://developer.apple.com/documentation/xcode/supporting-associated-domains)
