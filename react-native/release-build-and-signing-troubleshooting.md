# React Native Release Build and Signing Troubleshooting

A practical guide for diagnosing React Native apps that work in debug mode but fail to build, install, launch, or behave correctly in release mode.

This guide does not replace the current Google Play or Apple distribution instructions. Signing and store requirements change, so verify the final release workflow with the official platform documentation.

## Protect signing credentials first

> [!CAUTION]
> A signing private key can authorize application releases or updates. Never commit keystores, private keys, certificates with private keys, provisioning credentials, passwords, API keys, or store credentials.

Before debugging, confirm that sensitive files are ignored:

```bash
# Check whether a local Android keystore is ignored by Git.
git check-ignore -v android/app/upload-keystore.jks

# Search tracked filenames for common signing-file extensions.
git ls-files | grep -Ei '\.(jks|keystore|p12|mobileprovision)$'
```

No output from `git check-ignore` means the example path is not currently ignored. Add the project's actual secret paths to its private or repository ignore strategy before proceeding.

Do not place passwords directly in shell commands, screenshots, issue descriptions, build logs, or committed Gradle files.

## Start by classifying the failure

Record exactly where the release process fails:

| Stage | Typical symptoms |
|---|---|
| Compile or bundle | Gradle/Xcode error, missing module, JavaScript bundle failure |
| Sign | Missing key, certificate, private key, profile, alias, or entitlement mismatch |
| Package or archive | AAB/APK/archive is not produced or validation fails |
| Install | Signature mismatch, unsupported device, profile does not include device |
| Launch | Immediate release-only crash, blank screen, missing JavaScript bundle |
| Runtime | Wrong API, missing environment values, blocked network, stripped native code |
| Store upload | Duplicate version, wrong signing identity, invalid entitlement, policy validation |

Save the first meaningful error. Later messages often describe consequences rather than the original cause.

## Confirm the release environment

```bash
# Show React Native and related environment information.
npx react-native info

# Show the Java version used by Android builds.
java -version

# Show the active Xcode and command-line tools.
xcodebuild -version
xcode-select -p
```

Compare local versions with the project, CI, and the last successful release. Avoid upgrading build tools while diagnosing an unrelated signing failure.

## Android release troubleshooting

### Understand the Android keys

- The **app-signing key** signs APKs delivered to users and normally remains stable for the application's lifetime.
- The **upload key** signs the bundle or APK uploaded to Google Play when Play App Signing is used.
- A public certificate or fingerprint can be shared with an API provider; the corresponding private key must remain secret.

Do not create a new key merely because an existing key cannot be found. First identify whether the app already exists in a store and which certificate signs its installed releases.

### Build an Android App Bundle

From the React Native project root:

```bash
# Create a signed release Android App Bundle using project configuration.
npx react-native build-android --mode=release
```

The standard output location is:

```text
android/app/build/outputs/bundle/release/app-release.aab
```

If the command hides the useful Gradle error, run the underlying task:

```bash
# Enter the Android Gradle project.
cd android

# Build the release bundle and include a stack trace on failure.
./gradlew bundleRelease --stacktrace
```

### Use case

This reveals whether the failure occurs during JavaScript bundling, native compilation, resource processing, signing, or packaging.

### Build an APK for local release testing

```bash
# Build a release APK for direct installation testing.
cd android
./gradlew assembleRelease --stacktrace
```

The standard output location is:

```text
android/app/build/outputs/apk/release/app-release.apk
```

Verify its signature when Android SDK Build Tools are available on `PATH`:

```bash
# Verify the APK signature and print its public certificate details.
apksigner verify --verbose --print-certs app/build/outputs/apk/release/app-release.apk
```

Certificate fingerprints are not private keys, but review terminal output before sharing it publicly.

### Test the Android release build

> [!WARNING]
> Uninstalling an existing app removes its local data. Try an in-place compatible installation first.

```bash
# Return to the React Native project root.
cd ..

# Build and install the configured Android release variant.
npm run android -- --mode="release"
```

A release build contains its JavaScript bundle and should not depend on a running Metro server.

If installation reports a signature mismatch, compare the installed application's signing certificate with the new build. Do not uninstall until losing local data is acceptable and you know the new key is correct.

### Inspect an Android keystore safely

```bash
# Display aliases and certificate information; enter the password only at the prompt.
keytool -list -v -keystore /secure/path/upload-keystore.jks
```

Check:

- The configured keystore path exists on this machine or CI runner.
- The alias matches exactly.
- The store and key passwords are supplied through an approved secret mechanism.
- The expected certificate fingerprint matches Play Console or the previous release.
- The Gradle release build type uses the intended signing configuration.

Never add a password to the `keytool` command line because it may be saved in shell history or process output.

### Common Android release failures

| Error or symptom | Investigation |
|---|---|
| `Keystore file not found` | Resolve the path relative to the Gradle file and confirm CI secret installation |
| `Keystore was tampered with, or password was incorrect` | Verify secret values and keystore type; do not regenerate the key |
| `No key with alias ... found` | List aliases with `keytool` and check spelling/case |
| Upload certificate does not match | Compare Play Console upload certificate with the local upload key |
| Version code already used | Increase the Android `versionCode` for the next upload |
| Debug installs but release does not | Check signing, release API configuration, bundled JS, and device architecture |
| Release opens then crashes | Capture Logcat, test without code shrinking as a diagnosis, and inspect native dependencies |
| API works only in debug | Check release base URL, secrets, HTTPS policy, and environment-file loading |

### Diagnose R8 or code-shrinking problems

Code shrinking can remove or rename code that a native library expects dynamically.

1. Capture the release crash with Logcat.
2. Confirm whether shrinking is enabled for the release build.
3. Temporarily disable shrinking only as a diagnostic comparison.
4. If the crash disappears, add the narrow keep rules recommended by the affected library.
5. Re-enable shrinking and test the final release artifact.

Do not permanently disable optimization without understanding the size, performance, and security tradeoffs.

## iOS release troubleshooting

### Understand the iOS signing relationship

An iOS release normally requires these items to agree:

- Apple Developer team.
- Bundle identifier and registered App ID.
- Distribution method, such as TestFlight/App Store Connect or registered-device testing.
- Signing certificate with its private key in Keychain.
- Provisioning profile when signing is manually managed.
- Entitlements and capabilities used by the app and extensions.

With automatic signing, Xcode can manage distribution signing assets. With manual signing, the selected certificate, profile, App ID, and entitlements must match. Avoid mixing the two approaches accidentally across targets.

### Check available signing identities

```bash
# List code-signing identities that have accessible private keys.
security find-identity -v -p codesigning
```

If an expected identity is missing, check Keychain Access for both the certificate and its private key. Importing only a public certificate is not enough to sign an app.

Team names, certificate names, and local paths may identify an organization or user. Review output before sharing it.

### Verify the workspace and scheme

```bash
# List schemes available in a CocoaPods workspace.
cd ios
xcodebuild -list -workspace MyApp.xcworkspace
```

Replace `MyApp` with the real workspace name. When CocoaPods is used, archive the `.xcworkspace`, not the `.xcodeproj`.

In Xcode, verify every application and extension target under:

```text
Target → Signing & Capabilities
```

Check the team, bundle identifier, signing mode, capabilities, and release configuration.

### Create and validate an archive

The clearest first attempt is through Xcode:

```text
Select the application scheme and a generic iOS destination
        ↓
Product → Archive
        ↓
Window → Organizer
        ↓
Validate App
        ↓
Distribute App
```

Validation catches many signing, entitlement, version, and packaging problems before upload.

### Common iOS release failures

| Error or symptom | Investigation |
|---|---|
| No signing certificate found | Confirm Apple account, team access, certificate, and private key in Keychain |
| Provisioning profile does not match | Compare profile App ID, certificate, distribution type, and entitlements |
| Bundle identifier unavailable | Confirm the identifier belongs to the selected team and matches App Store Connect |
| Entitlement is not permitted | Enable the capability for the App ID and regenerate affected profiles if manually managed |
| Profile expired or invalid | Renew/regenerate through the team; do not bypass validation |
| Archive option is disabled | Select the correct shared app scheme and an archive-capable destination |
| App extension signing fails | Verify signing and bundle IDs for every extension target, not only the main app |
| Build number already used | Increase `CFBundleVersion` before uploading another build |
| Release-only crash | Test the archived configuration, collect device crash logs, and keep matching symbols |

### Avoid destructive certificate changes

> [!CAUTION]
> Revoking a certificate can break signing for teammates and CI systems. Do not revoke, delete, or replace signing assets merely to clear an error.

Before changing Apple signing assets:

1. Confirm whether Xcode manages signing automatically.
2. Check whether teammates or CI use the certificate.
3. Verify your Apple Developer role permits the required action.
4. Preserve approved backups of exportable credentials using secure organizational procedures.
5. Coordinate any certificate or profile rotation.

## Release-only application problems

### Verify environment configuration

Release builds often use different API URLs, feature flags, bundle identifiers, application IDs, or service configuration files.

Check that:

- The intended environment is selected at build time.
- Required values are present without being compiled into public JavaScript when they are secrets.
- Android and iOS service files match the release package/bundle identifier.
- Release network policies permit the production HTTPS endpoint.
- Debug-only mock servers and flags are disabled.

> [!WARNING]
> Values bundled into a mobile application can be extracted. Do not treat JavaScript variables, native resources, or application packages as secure secret storage.

### Capture release logs

```bash
# Capture Android release crashes and fatal errors.
adb logcat '*:E'

# Stream logs from the currently booted iOS Simulator.
xcrun simctl spawn booted log stream --level debug
```

Physical iOS release issues should also be investigated through Xcode's Devices and Simulators window and Organizer crash reports.

Redact tokens, user data, URLs, team details, and device identifiers before sharing logs.

### Keep symbols and release artifacts

For each distributed build, retain the information needed to match crashes to source:

- Application version and build number.
- Source commit.
- Android mapping file when code shrinking/obfuscation is enabled.
- iOS dSYM files and archive.
- Relevant JavaScript source maps when the build pipeline produces them.

Store artifacts according to the project's security and retention policy; do not commit large or sensitive release artifacts to this documentation repository.

## CI signing failures

When local release builds succeed but CI fails, compare:

- Tool and operating-system versions.
- Secret names and access permissions.
- Keystore/certificate/profile installation paths.
- Keychain availability and unlock duration.
- Selected scheme, configuration, flavor, and environment.
- Whether pull requests from forks are intentionally denied release secrets.
- Certificate and profile expiration dates.

Never add a command that prints all environment variables to diagnose CI. It can expose every secret available to the job.

## Recommended troubleshooting workflow

```text
Protect credentials
        ↓
Identify failing stage and first error
        ↓
Verify toolchain and release configuration
        ↓
Verify package/bundle ID and version
        ↓
Verify signing identity without changing it
        ↓
Build locally with detailed logs
        ↓
Install and test the exact release artifact
        ↓
Compare local and CI/store environments
        ↓
Rotate or regenerate signing assets only with authorization
```

## Pre-release checklist

- [ ] The exact release artifact was tested on representative physical devices.
- [ ] Package name or bundle identifier matches the store application.
- [ ] Version and build numbers are unique and intentional.
- [ ] Production API and service configuration are correct.
- [ ] No private signing files or credentials are tracked by Git.
- [ ] Android certificate fingerprints match required API and Play configuration.
- [ ] iOS capabilities, entitlements, team, and profiles agree across all targets.
- [ ] Release logs contain no unnecessary sensitive information.
- [ ] Crash symbols and mapping artifacts are stored securely.
- [ ] The final store validation succeeds before release.

## Related guides

- [Common React Native Build Errors](common-build-errors.md)
- [React Native Network and API Debugging](network-and-api-debugging.md)
- [Gradle Cleanup and Troubleshooting](../android/gradle-cleanup-and-troubleshooting.md)
- [Xcode Cleanup Commands](../ios/xcode-cleanup-commands.md)
- [CocoaPods Troubleshooting](../ios/cocoapods-troubleshooting.md)
- [iOS Device and Simulator Logs](../ios/ios-device-and-simulator-logs.md)

## Official references

- [React Native: Publishing to Google Play Store](https://reactnative.dev/docs/signed-apk-android)
- [Android Developers: Sign Your App](https://developer.android.com/studio/publish/app-signing)
- [Apple: Preparing Your App for Distribution](https://developer.apple.com/documentation/xcode/preparing-your-app-for-distribution)
- [Apple: Distributing Your App for Beta Testing and Releases](https://developer.apple.com/documentation/xcode/distributing-your-app-for-beta-testing-and-releases)
- [Apple: Certificates Overview](https://developer.apple.com/help/account/create-certificates/certificates-overview)
