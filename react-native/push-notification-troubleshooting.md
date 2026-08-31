# React Native Push Notification Troubleshooting

A practical, provider-neutral guide for diagnosing push notifications that fail during registration, delivery, display, or user interaction on Android and iOS.

The exact JavaScript and native APIs depend on the notification library and provider used by the project. Verify implementation details against that library's current documentation.

## Separate the notification pipeline

Troubleshoot one stage at a time:

```text
Permission
   ↓
Platform registration
   ↓
FCM/APNs token
   ↓
Token stored by backend
   ↓
Provider accepts message
   ↓
Device receives message
   ↓
OS or app displays notification
   ↓
Tap opens correct screen
```

A successful provider response does not prove that the device displayed the notification.

## Record the exact test conditions

For every test, note:

- Android or iOS version and physical device model.
- Debug or release build.
- Foreground, background, or terminated application state.
- Notification, data, or mixed payload.
- Target environment: development, staging, or production.
- Provider response code and non-sensitive message identifier.

> [!WARNING]
> Push tokens identify application installations and can be used to target messages. Never publish full tokens, APNs keys, FCM service-account credentials, authorization headers, or user payloads.

## Check permission before delivery

- Android versions that require notification runtime permission must receive it before notifications can be shown.
- iOS requires notification authorization for alerts, sounds, and badges.
- A user can later disable notifications or individual categories in system Settings.
- Foreground notifications commonly require explicit application UI or local-notification handling.

Use [Mobile Permissions Troubleshooting](mobile-permissions-troubleshooting.md) for the full permission flow.

## Verify token registration

Check that:

1. Native platform registration succeeds.
2. The application receives the current FCM or APNs token.
3. The backend stores the token for the correct user and environment.
4. Token refresh updates the backend.
5. Logout removes or disassociates the token as required by the product.
6. Invalid or unregistered tokens are removed after provider feedback.

Log only token presence, environment, last-updated time, and a short irreversible fingerprint—not the full token.

## Understand application-state behavior

| App state | Typical behavior to verify |
|---|---|
| Foreground | Callback runs; visible notification may require explicit handling |
| Background | OS may display notification payload; data handling depends on platform and priority |
| Terminated | OS launches app after a tap; background execution is limited |
| Force-stopped or restricted | Delivery may be deferred or blocked by the platform/user |

On Android, notification messages received in the background are normally handled by the system tray, while foreground messages reach the application callback. Mixed notification/data payloads can produce different callback paths.

Do not rely on silent/background notifications for time-critical guaranteed execution. Mobile operating systems control background scheduling.

## Android troubleshooting

Check:

- `google-services.json` belongs to the release package name and Firebase project.
- Notification runtime permission is granted when required.
- The notification channel exists and has suitable importance.
- The user has not disabled the application or channel in Settings.
- Foreground handling intentionally displays or suppresses the notification.
- Battery restrictions, Data Saver, work profile, or vendor policies are not interfering.
- The release build includes required messaging services and code-shrinking rules.

```bash
# Show notification and FCM-related Android logs.
adb logcat | grep -iE 'firebase|messaging|notification|fcm'

# Show notification service state for the device.
adb shell dumpsys notification
```

Review logs before sharing because notification payloads may contain user data.

### Android channel problems

After a notification channel is created, some behavior is controlled by the user and cannot simply be overwritten by the app. During development, use a new channel ID only when the channel's meaning genuinely changes; do not continually create channels to bypass user preferences.

## iOS troubleshooting

Check:

- The application has the correct Push Notifications capability and entitlements.
- The bundle identifier matches the App ID, Firebase configuration, and provider topic.
- Development and production APNs environments are not mixed.
- The APNs authentication key or certificate is valid and authorized for the team.
- The app registers with APNs and forwards the token as required by its messaging library.
- Foreground presentation behavior is implemented intentionally.
- Background modes are enabled only for features that truly need them.

```bash
# Stream push and notification-related logs from the booted Simulator.
xcrun simctl spawn booted log stream --level debug \
  --predicate 'eventMessage CONTAINS[c] "push" OR eventMessage CONTAINS[c] "notification" OR eventMessage CONTAINS[c] "APNS"'
```

Always confirm final APNs behavior on supported physical devices and the actual distribution environment.

## Diagnose payload problems

Verify:

- Payload keys and types match the provider/platform specification.
- The payload stays within provider size limits.
- Custom data contains no secrets or unnecessary personal information.
- Android channel ID exists before it is referenced.
- iOS alert, sound, badge, category, and background fields are intentional.
- Navigation data is validated before opening a screen.

Never put passwords, session tokens, reset credentials, or sensitive personal data in notification text or deep-link parameters.

## Notification tap opens the wrong screen

Test all three situations separately:

1. App already open.
2. App in background.
3. App launched from a terminated state.

Make notification navigation idempotent so the same event does not push the screen twice. Wait until authentication and navigation state are ready, and validate every route and identifier from the payload.

See [Deep Linking Troubleshooting](deep-linking-troubleshooting.md) for URL and navigation handling.

## Recommended workflow

```text
Confirm permission
        ↓
Confirm current token and environment
        ↓
Send to one test device
        ↓
Record provider response
        ↓
Inspect platform logs
        ↓
Test foreground/background/terminated states
        ↓
Verify display, tap, and navigation separately
        ↓
Test the signed release build
```

## Official references

- [Firebase: FCM Troubleshooting and FAQ](https://firebase.google.com/docs/cloud-messaging/troubleshooting)
- [Firebase: Receive Messages on Android](https://firebase.google.com/docs/cloud-messaging/android/receive-messages)
- [Firebase: Receive Messages on Apple Platforms](https://firebase.google.com/docs/cloud-messaging/ios/receive-messages)
- [Android: Notification Runtime Permission](https://developer.android.com/develop/ui/views/notifications/notification-permission)
- [Apple: User Notifications](https://developer.apple.com/documentation/usernotifications)
