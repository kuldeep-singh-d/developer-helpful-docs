# React Native App Store Submission and Release Management

A practical guide for preparing, submitting, rolling out, monitoring, and supporting Android and iOS releases without treating upload as the final step.

Store policies, forms, SDK requirements, and review processes change. Verify current Apple and Google guidance for every release, especially for payments, privacy, children, health, finance, user-generated content, and regional distribution.

## Separate build readiness from store readiness

A technically valid binary can still be rejected or delayed because of:

- Incomplete or misleading metadata.
- Missing privacy disclosures or policy URLs.
- Broken review credentials or inaccessible features.
- Payment or subscription-rule problems.
- Missing account-deletion or support flows where required.
- Unexplained background access, permissions, or sensitive capabilities.
- Crashes, placeholder content, or incomplete functionality.
- Bundle/package identifiers, versions, entitlements, or signing mismatches.

Assign an owner for binary quality, store metadata, privacy answers, review communication, rollout monitoring, and incident decisions.

## Define the release before building

Record:

```text
Release version and build numbers
Source commit and CI run
Target environment and API compatibility
Included features and feature-flag defaults
Database/storage migrations
Minimum supported OS versions
Required backend deployment order
Known limitations and support notes
Rollout, pause, and incident owners
```

Do not build multiple different artifacts with the same public version/build identity.

## Verify version compatibility

Mobile users do not all update immediately. Before release, confirm:

- The new app works with the currently deployed backend.
- The new backend continues supporting older active app versions.
- API and local-database migrations are forward and backward compatible where required.
- Feature flags default safely when configuration is unavailable.
- Forced-update behavior is reserved for justified compatibility or security needs.
- Deep links, push payloads, and shared content remain compatible across versions.

A mobile release usually cannot be rolled back instantly on every installed device. Prefer additive server changes and staged migration.

## Build the final Android artifact

```bash
# Build the release Android App Bundle from the project root.
npx react-native build-android --mode=release

# Verify that the expected bundle exists.
ls -lh android/app/build/outputs/bundle/release/app-release.aab
```

Before upload:

- Confirm application ID, version name, and unique version code.
- Verify upload-key certificate and Play App Signing configuration.
- Test the release build without Metro.
- Preserve R8 `mapping.txt` and relevant JavaScript source maps.
- Review Play-generated device APKs or internal-track installation, not only a locally installed APK.

## Build and validate the iOS archive

```text
Select release scheme and distribution destination
        ↓
Xcode → Product → Archive
        ↓
Organizer → Validate App
        ↓
Distribute App → App Store Connect
```

Before upload:

- Confirm bundle identifier, marketing version, and unique build number.
- Verify team, capabilities, entitlements, and every extension target.
- Preserve the archive and dSYM files.
- Test through TestFlight using the distributed build.
- Confirm production services, APNs environment, URLs, and privacy behavior.

See [Release Build and Signing Troubleshooting](release-build-and-signing-troubleshooting.md) for signing failures.

## Prepare store metadata as release content

Review:

- App name, subtitle/short description, full description, and category.
- Screenshots for required device classes and current interface.
- Support, marketing, and privacy-policy URLs.
- Contact details and review notes.
- Age/content rating and content declarations.
- Data-safety/privacy answers based on actual app and SDK behavior.
- Encryption/export and regional declarations where applicable.
- In-app products, subscriptions, pricing, and localization.

Metadata must describe the submitted build. Do not promise unavailable features or hide material limitations.

## Give reviewers a complete test path

When authentication or special hardware is required, provide:

- A working review account with appropriate sample data.
- Clear steps to reach non-obvious features.
- Explanation of required permissions and background behavior.
- Instructions for demo mode, hardware, region, or server prerequisites.
- Contact information monitored during review.

> [!WARNING]
> Use a dedicated least-privileged review account. Never place production administrator credentials, employee accounts, private customer data, or reusable secrets in review notes.

Keep the account active until review completes, but monitor it and rotate/remove access according to the approved process afterward.

## Test through platform distribution

Use Google Play internal/closed testing and Apple TestFlight to verify:

- Store signing and installation/update path.
- Production-like configuration.
- In-app purchases and subscriptions in the correct test environment.
- Push notification and deep-link behavior.
- Background tasks and permissions.
- Crash symbol uploads and analytics release tagging.
- Upgrade from the previous public version without data loss.

An app installed directly from a development machine does not test every store-delivery condition.

## Common rejection and delay categories

| Category | Prevention |
|---|---|
| Crash or broken flow | Test signed distributed build with reviewer path |
| Incomplete metadata | Use an owner and pre-submission checklist |
| Privacy mismatch | Compare declarations with code, SDKs, network traffic, and retention |
| Permission not justified | Request in context and explain actual use |
| Payment/subscription issue | Review current platform rules before implementation and submission |
| Login inaccessible | Maintain tested review credentials and instructions |
| User-generated content | Provide required moderation, reporting, blocking, and support behavior |
| Misleading screenshots/description | Capture the submitted build and describe current functionality |
| Duplicate/outdated build | Verify selected version/build and processing status |

When rejected, address the exact cited issue. Do not resubmit an unchanged build repeatedly or argue from outdated policy summaries.

## Plan a staged rollout

Before starting:

- Define the initial percentage or audience.
- Set minimum observation time between stages.
- Choose crash, ANR, startup, API, conversion, and support thresholds.
- Confirm who may expand, pause, or halt rollout.
- Keep backend and feature flags capable of disabling risky behavior.
- Ensure monitoring is segmented by app version, OS, and device.

Store rollout controls reduce exposure but do not replace tested rollback/mitigation plans.

## Decide whether to pause, mitigate, or hotfix

| Situation | Typical response |
|---|---|
| Feature defect controlled by safe flag | Disable or limit feature, then investigate |
| Backend incompatibility | Restore compatible backend behavior when safe |
| Rollout-only emerging crash | Pause rollout and preserve diagnostics |
| Security/privacy exposure | Activate incident process immediately |
| Severe client bug already widely installed | Prepare focused hotfix and server-side mitigation |
| Minor cosmetic issue | Document and include in normal release unless impact justifies risk |

Avoid bundling unrelated dependency upgrades or refactors into an urgent hotfix.

## Manage feature flags safely

- Provide safe defaults when remote configuration is unavailable.
- Separate release deployment from feature exposure.
- Scope flags by app version and platform when needed.
- Protect administrative flag changes with access control and audit history.
- Remove expired flags and both code paths after rollout stabilizes.
- Never use a client flag as a security authorization control.

## Monitor after release

Track by version and rollout stage:

- Crash-free users/sessions and top crash groups.
- Android ANRs and iOS memory terminations.
- Startup, network, and critical-flow success.
- Authentication, payment, sync, and push failures.
- Ratings, reviews, support contacts, and refund signals.
- Backend load and errors from old and new clients.

Preserve release artifacts and record decisions when expanding or pausing rollout.

## Release evidence to retain

- Source commit and reproducible build reference.
- Submitted AAB and Xcode archive according to retention policy.
- Android mapping file, native symbols, iOS dSYMs, and JS source maps.
- Store metadata and privacy answers used for the release.
- Test results and known issues.
- Approval/rejection correspondence.
- Rollout timeline and monitoring notes.

Do not commit signing secrets or private store credentials to this repository.

## Recommended release workflow

```text
Define release and compatibility
        ↓
Build signed artifacts from reviewed source
        ↓
Test upgrade and critical flows
        ↓
Validate store metadata and privacy answers
        ↓
Distribute to internal/beta testers
        ↓
Submit with complete review instructions
        ↓
Start staged rollout
        ↓
Monitor version-specific health
        ↓
Expand, pause, mitigate, or hotfix using agreed thresholds
```

## Related guides

- [React Native Release Build and Signing Troubleshooting](release-build-and-signing-troubleshooting.md)
- [Device and OS Fragmentation Testing](device-and-os-fragmentation-testing.md)
- [Mobile Security and Sensitive Data Handling](mobile-security-and-sensitive-data-handling.md)
- [Push Notification Troubleshooting](push-notification-troubleshooting.md)

## Official references

- [Apple App Review](https://developer.apple.com/app-store/review/)
- [Apple: Submitting Apps to the App Store](https://developer.apple.com/app-store/submitting/)
- [Apple: Distributing Your App](https://developer.apple.com/documentation/xcode/distributing-your-app-for-beta-testing-and-releases)
- [Google Play: Publish Your App](https://support.google.com/googleplay/android-developer/answer/9859751)
- [Google Play: Helpful Publishing Tips](https://support.google.com/googleplay/android-developer/answer/15191715)
