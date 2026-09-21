# React Native Mobile Security and Sensitive Data Handling

A practical guide for reducing common security and privacy risks in React Native apps without treating the mobile client as a trusted environment.

> [!IMPORTANT]
> This guide provides engineering practices, not legal or compliance advice. Requirements depend on the application's data, users, regions, industry, and current platform policies. Involve qualified security, privacy, and legal reviewers when needed.

## Begin with a small threat model

For each sensitive feature, document:

- Data collected, generated, stored, transmitted, and shared.
- Why the app needs it and how long it is retained.
- Who or what can access it.
- What happens if a device, account, network, log, backup, or third-party SDK is compromised.
- Abuse cases, such as stolen sessions, tampered deep links, screen capture, or automated requests.
- Required server-side authorization and audit controls.

Security is not a final release checklist. Revisit the model when features, SDKs, APIs, or regulations change.

## Assume the application bundle is inspectable

Anything shipped to a device can potentially be extracted, including:

- JavaScript bundles and source strings.
- Native resources and configuration files.
- API endpoints and feature flags.
- Embedded certificates and public keys.
- Values injected through build-time environment variables.

Never embed server secrets, private signing keys, database administrator credentials, unrestricted third-party keys, or credentials that authorize privileged operations.

If a provider requires a secret, keep it on a controlled backend and expose only the minimum authenticated operation the app needs.

## Separate configuration from secrets

Environment files can help select development, staging, and production configuration, but bundling a value from `.env` does not make it secret.

```bash
# Confirm that a local environment file is ignored by Git.
git check-ignore -v .env

# Review the exact staged changes before committing.
git diff --staged
```

Do not print all environment variables during debugging or CI; doing so can expose unrelated credentials.

## Store data according to sensitivity

| Data | Typical handling |
|---|---|
| Theme, onboarding flag, non-sensitive preference | Normal app-private preference/storage |
| Access or refresh token | Keychain/Keystore-backed secure-storage solution |
| Encryption key | Platform key-management APIs; avoid exporting raw key material |
| Offline private records | Minimize, encrypt where required, protect keys separately |
| Password | Do not persist unless a reviewed platform credential flow requires it |
| Server secret | Never ship in the app |

React Native does not include a universal secure-storage API. Evaluate maintained libraries or native implementations for platform support, accessibility level, backup behavior, biometric policy, migration, and failure handling.

Async key-value storage is appropriate for non-sensitive state, not tokens or secrets, unless the chosen implementation and threat model explicitly provide the required protection.

## Design token and session handling

- Keep access tokens short-lived where the backend architecture supports it.
- Protect refresh tokens more strongly and rotate them when supported.
- Serialize refresh attempts so multiple `401` responses do not create a race.
- Validate token issuer, audience, signature, and time claims on the server.
- Revoke or invalidate sessions after logout, password change, or suspected compromise when possible.
- Remove tokens, cached private data, queued operations, and user-specific notifications during logout.
- Never use a client-side role or flag as the final authorization decision.

Server authorization must check every protected operation even if the interface hides inaccessible actions.

## Handle biometrics correctly

Biometrics usually unlock locally protected credentials; they do not prove a fresh server authentication by themselves.

Define:

- Whether device passcode fallback is allowed.
- What happens when biometric enrollment changes.
- Which actions require recent authentication.
- How recovery works after repeated failure.
- Whether the operation needs server-side step-up authentication.

Do not store a plain `biometricEnabled=true` flag and treat it as proof that biometric authentication occurred.

## Protect logs, analytics, and crash reports

Never record:

- Passwords, tokens, cookies, authorization headers, or reset links.
- Full payment, health, identity, contact, location, or private-message data.
- Request/response bodies by default.
- Sensitive deep links or clipboard contents.
- Cryptographic key material.

Use structured events with allowlisted fields, irreversible correlation IDs, and centralized redaction. Review development logging, native logs, analytics, support attachments, and crash-report breadcrumbs before release.

See [Crash Reporting and Symbolication](crash-reporting-and-symbolication.md) for privacy-safe production diagnostics.

## Secure network communication

- Use HTTPS with correctly validated certificates.
- Do not globally disable Android cleartext protection or iOS App Transport Security.
- Scope development exceptions narrowly and exclude them from release when possible.
- Apply timeouts and bounded retries.
- Authenticate and authorize every sensitive API operation.
- Avoid sending private values in URL query strings, which may appear in logs and history.
- Plan certificate-pinning rotation and recovery before enabling pinning.

Pinning can block all older installed app versions after a certificate change. Use backup pins and an operational rotation plan when the risk model justifies it.

See [Network and API Debugging](network-and-api-debugging.md) for platform diagnostics.

## Treat deep links and notifications as untrusted

An incoming URL or notification payload can be forged or altered.

- Allowlist schemes, hosts, routes, and parameter formats.
- Re-check authentication and authorization after navigation.
- Never place session tokens or one-time credentials in custom-scheme URLs.
- Avoid executing sensitive actions directly from a link or notification tap.
- Require confirmation for destructive or financial actions.

Prefer verified Android App Links and iOS Universal Links for public web-to-app navigation.

## Protect sensitive screens and user actions

Consider the product's risk before:

- Showing private data in application-switcher snapshots.
- Allowing screenshots or screen recording.
- Copying secrets or personal data to the shared clipboard.
- Displaying sensitive notification previews on a locked device.
- Keeping private data visible after backgrounding or logout.
- Caching documents in shared storage.

Blocking screenshots is not complete protection and may reduce accessibility or supportability. Use it only for a defined threat and test platform behavior.

## Minimize permissions and collected data

- Request access only when the user starts the related feature.
- Collect the minimum precision, scope, and history required.
- Provide useful behavior when access is denied.
- Delete data when it is no longer needed.
- Keep privacy explanations consistent with actual SDK and backend behavior.
- Review third-party SDK collection, retention, and data sharing.

See [Mobile Permissions Troubleshooting](mobile-permissions-troubleshooting.md) for platform implementation details.

## Review WebView usage

When displaying web content:

- Load trusted HTTPS origins.
- Allowlist navigation destinations.
- Disable unnecessary JavaScript and native bridges.
- Validate every message crossing the JavaScript/native boundary.
- Do not inject secrets into arbitrary pages.
- Handle downloads, popups, file access, and external schemes deliberately.

A WebView bridge exposed to untrusted content can become a privileged interface into the app.

## Manage dependencies and native SDKs

- Remove unused dependencies.
- Review permissions, native code, network behavior, and data collection before adoption.
- Pin and update versions through the project's lockfile strategy.
- Monitor security advisories and abandoned packages.
- Re-test release builds after native dependency updates.
- Verify SDK initialization does not collect data before consent when consent is required.

Do not upgrade every dependency simultaneously during an incident; preserve a narrow, reviewable change.

## Handle compromised or modified devices realistically

Root/jailbreak detection and application-integrity signals can increase risk visibility but are bypassable. Do not use them as the only control protecting server data.

Use server-side authorization, rate limits, anomaly detection, device/app attestation where appropriate, and step-up verification for high-risk operations. Provide a deliberate policy for false positives and unsupported devices.

## Incident-ready checklist

```text
Identify exposed data and affected versions
        ↓
Revoke or rotate server-side credentials
        ↓
Disable vulnerable behavior remotely when possible
        ↓
Preserve sanitized evidence
        ↓
Patch client and backend
        ↓
Follow approved notification and reporting procedures
        ↓
Review root cause and preventive controls
```

Do not destroy logs or rotate signing identities impulsively. Coordinate changes with the people responsible for backend, store, security, privacy, and incident response.

## Pre-release security review

- [ ] No privileged secret is bundled in JavaScript or native resources.
- [ ] Tokens and sensitive local data use reviewed storage.
- [ ] Logout clears account-specific state and queued work.
- [ ] Server authorization is enforced independently of the client UI.
- [ ] Logs, analytics, and crash reports are redacted.
- [ ] HTTPS and release network policies are enabled.
- [ ] Deep links and notification payloads are validated.
- [ ] Permissions and data collection are minimized and explained.
- [ ] Third-party SDK behavior is reviewed.
- [ ] Release signing credentials remain private and access-controlled.
- [ ] Incident owners and rotation procedures are known.

## Related guides

- [React Native Release Build and Signing Troubleshooting](release-build-and-signing-troubleshooting.md)
- [React Native Network and API Debugging](network-and-api-debugging.md)
- [Deep Linking Troubleshooting](deep-linking-troubleshooting.md)
- [Offline Mode and Network Retry Strategies](offline-mode-and-network-retry-strategies.md)

## Official references

- [React Native Security](https://reactnative.dev/docs/security)
- [Android Keystore System](https://developer.android.com/privacy-and-security/keystore)
- [Apple: Using the Keychain to Manage User Secrets](https://developer.apple.com/documentation/security/using-the-keychain-to-manage-user-secrets)
- [Android Network Security Configuration](https://developer.android.com/privacy-and-security/security-config)
- [Apple: Protecting the User's Privacy](https://developer.apple.com/documentation/uikit/protecting-the-user-s-privacy)
