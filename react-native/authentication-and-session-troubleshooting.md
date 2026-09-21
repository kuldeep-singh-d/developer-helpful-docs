# React Native Authentication and Session Troubleshooting

A practical guide for diagnosing login failures, expired sessions, token-refresh races, OAuth redirects, biometric reauthentication, and incomplete logout behavior in React Native apps.

> [!IMPORTANT]
> Authentication is a client-and-server workflow. Hiding a screen or changing local state does not authorize a request. The backend must validate every protected operation.

## Map the complete authentication flow

Before changing code, write down the actual sequence:

```text
User submits credentials or starts OAuth
        ↓
Identity provider authenticates the user
        ↓
App receives a short-lived authorization result
        ↓
Backend validates or exchanges it securely
        ↓
App stores only the session material it needs
        ↓
API client attaches the access token
        ↓
Expired access token triggers one controlled refresh
        ↓
Logout revokes the session and removes user-specific state
```

Record which component owns each step: application code, native SDK, identity provider, API gateway, or backend service.

## Start with a failure classification

| Symptom | Likely area to inspect |
|---|---|
| Login request never leaves the device | Form state, network reachability, or client validation |
| Server returns `400` or `422` | Request shape, grant type, redirect URI, or missing field |
| Server returns `401` immediately | Credentials, token issuer/audience, clock, or environment mismatch |
| Login succeeds but the app returns to the login screen | Session restoration, navigation guard, or storage failure |
| Many refresh requests occur together | Missing refresh serialization |
| OAuth works on one platform only | Platform redirect registration or application configuration |
| Session fails after the app resumes | Token expiry, AppState handling, or background time |
| Another user's data appears after login | Incomplete logout/cache isolation; treat as a security incident |

Capture the HTTP status, sanitized response category, app state, platform, app version, and server request ID. Never capture passwords, raw tokens, authorization codes, cookies, or one-time links.

## Separate authentication states

Avoid using a single `isLoggedIn` boolean. Model startup explicitly:

```ts
type AuthState =
  | {status: 'restoring'}
  | {status: 'signedOut'}
  | {status: 'signedIn'; userId: string}
  | {status: 'reauthenticationRequired'};
```

Render a splash/loading state while restoring the session. Otherwise, protected navigation may briefly redirect to login before secure storage finishes reading.

## Diagnose ordinary login failures

Verify in this order:

1. The app points to the intended environment.
2. The device can reach the authentication host.
3. Request headers and body match the server contract.
4. Device date and time are reasonably accurate.
5. The account is active and allowed to use the client.
6. The server validates the expected issuer, audience, and redirect URI.
7. Error handling distinguishes invalid credentials from network or server failures.

```bash
# Show recent Android logs and filter common authentication signals.
adb logcat | rg -i "auth|oauth|token|401|403|network|ssl"

# Start the iOS Simulator log stream and filter common authentication signals.
xcrun simctl spawn booted log stream --level error --predicate 'eventMessage CONTAINS[c] "auth" OR eventMessage CONTAINS[c] "oauth"'
```

Sanitize logs before sharing them. Prefer correlation IDs over request bodies.

## Handle `401` and `403` differently

- `401 Unauthorized` usually means authentication is absent, expired, or invalid.
- `403 Forbidden` usually means the identity is known but lacks permission.

Do not refresh a token repeatedly for a real `403`. Do not log the user out for every temporary network failure.

## Prevent token-refresh races

When several requests receive `401` together, allow only one refresh operation. Other requests should wait for that result and retry at most once.

```ts
let refreshInFlight: Promise<string> | null = null;

async function getFreshAccessToken(): Promise<string> {
  if (!refreshInFlight) {
    refreshInFlight = refreshAccessToken()
      .finally(() => {
        refreshInFlight = null;
      });
  }

  return refreshInFlight;
}
```

The production implementation should also:

- Mark retried requests so they cannot enter an infinite loop.
- Avoid refreshing for login, refresh, or public endpoints.
- Update stored credentials atomically when tokens rotate.
- Move to a signed-out or reauthentication state when refresh is definitively rejected.
- Preserve the original request only when replay is safe.

> [!WARNING]
> Automatically replaying a non-idempotent request can duplicate a purchase, message, or mutation. Use idempotency keys or require the caller to decide whether replay is safe.

## Store session material safely

- Store access and refresh tokens using an evaluated Keychain/Keystore-backed solution.
- Keep ordinary UI preferences separate from credentials.
- Do not place tokens in Redux persistence, plain Async Storage, logs, analytics, crash breadcrumbs, or URLs.
- Treat build-time environment variables as bundled configuration, not secret storage.
- Define what happens when secure storage is locked, unavailable, corrupted, or invalidated.

See [Mobile Security and Sensitive Data Handling](mobile-security-and-sensitive-data-handling.md) for broader guidance.

## Restore a session safely at startup

A safe restoration flow is:

```text
Read credentials from secure storage
        ↓
No credentials? → signed out
        ↓
Check local expiry as an optimization
        ↓
Refresh or validate with the server when required
        ↓
Load minimum user profile
        ↓
Enter signed-in navigation
```

Do not trust decoded token claims as the final authorization decision. A locally decoded token may be expired, revoked, or forged.

## Troubleshoot OAuth and social login redirects

Check all of these together:

- The redirect URI matches exactly, including scheme, host, path, case, and trailing slash.
- Android intent filters and iOS URL types/associated domains are configured.
- The identity-provider dashboard contains the same release identifiers and redirect URIs.
- Development, staging, and production clients do not share accidental configuration.
- The app verifies OAuth `state`; the server/client flow uses PKCE where applicable.
- The authorization code is exchanged once and never logged.
- The installed release build uses the expected signing certificate and bundle/package identifier.

Test cold start, warm start, app already open, cancelled login, denied consent, and a redirect delivered twice.

See [Deep Linking Troubleshooting](deep-linking-troubleshooting.md) for platform link diagnostics.

## Handle application background and resume

Authentication can take place in a browser or native provider app. The original app may become inactive or stay in the background long enough for a token to expire.

```ts
import {AppState} from 'react-native';

const subscription = AppState.addEventListener('change', nextState => {
  if (nextState === 'active') {
    // Re-check time-sensitive session state without starting duplicate refreshes.
    void validateSessionIfNeeded();
  }
});

// Call subscription.remove() when the owning component or service is disposed.
```

Do not refresh on every focus event without checking expiry and current work.

## Design biometric reauthentication deliberately

Biometrics normally unlock a device-protected credential or approve a local action. They do not replace server authorization.

Test:

- User cancellation and repeated failure.
- Device passcode fallback policy.
- Changed biometric enrollment.
- Device without enrolled biometrics.
- App backgrounding during the prompt.
- Server requirement for recent or step-up authentication.

Never treat a stored `biometricEnabled` preference as proof that authentication occurred.

## Make logout complete and account-safe

A logout flow may need to:

1. Revoke or invalidate the refresh session on the server.
2. Stop refresh timers and reject waiting authenticated requests.
3. Delete credentials from secure storage.
4. Clear user-specific query, image, database, and file caches.
5. Remove queued offline mutations for that account.
6. Unregister or reassociate push-notification tokens as designed.
7. Reset sensitive navigation history and in-memory state.

> [!WARNING]
> Never allow one account to upload another account's queued offline data after logout. Namespace persisted data by account and clear or quarantine pending work before switching users.

Decide how partial logout failures behave. Local credentials should not remain active merely because server revocation temporarily failed; record a safe server-side cleanup strategy.

## Test the difficult cases

- Access token expires while multiple requests are active.
- Refresh token is rotated, revoked, or already used.
- Device time is incorrect.
- App is killed during login, refresh, or logout.
- Network disappears after the server accepts a request.
- User changes password or revokes the device elsewhere.
- User switches accounts with cached data present.
- OAuth callback arrives twice or with an invalid `state`.
- Secure storage becomes unavailable.
- Old app version talks to a newly deployed authentication service.

## Quick troubleshooting workflow

```text
Classify failure and affected environment
        ↓
Capture sanitized client and server request IDs
        ↓
Verify endpoint, redirect, clock, and release configuration
        ↓
Inspect storage/restoration and navigation state
        ↓
Check refresh serialization and retry limits
        ↓
Test logout and account switching
        ↓
Add a regression test for the failed state transition
```

## Related guides

- [Mobile Security and Sensitive Data Handling](mobile-security-and-sensitive-data-handling.md)
- [Network and API Debugging](network-and-api-debugging.md)
- [Deep Linking Troubleshooting](deep-linking-troubleshooting.md)
- [Offline Mode and Network Retry Strategies](offline-mode-and-network-retry-strategies.md)

## Official references

- [React Native Security](https://reactnative.dev/docs/security)
- [React Native AppState](https://reactnative.dev/docs/appstate)
- [React Native Linking](https://reactnative.dev/docs/linking)
- [OAuth 2.0 Security Best Current Practice](https://www.rfc-editor.org/rfc/rfc9700.html)
