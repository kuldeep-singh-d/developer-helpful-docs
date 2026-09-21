# React Native Crash Reporting and Symbolication

A practical guide for turning production crashes, Android ANRs, iOS memory terminations, and JavaScript exceptions into actionable evidence.

## Identify the failure type first

| Failure type | Typical evidence |
|---|---|
| JavaScript exception | Message, JS stack, source-map symbolication, breadcrumbs |
| Android native/JVM crash | Logcat, tombstone/native stack, Play Console or crash service |
| Android ANR | Main-thread and process traces, Android vitals, user action |
| iOS native crash | Apple crash report with matching dSYM symbolication |
| iOS memory termination | Jetsam report and memory profile; often no ordinary crash stack |
| Intentional exit or watchdog | Lifecycle, timeout, resource, or explicit termination evidence |
| Server-visible failure | Correlation ID, request outcome, app version, sanitized backend logs |

Do not assume every disappearance is a crash. The operating system can terminate background or memory-heavy processes without producing the same report as an exception.

## Capture release identity

Every report should include:

- Application version and build number.
- Platform, OS version, device model, and architecture.
- Release channel/environment.
- Source commit and CI/build identifier.
- Whether code shrinking, obfuscation, Hermes, or architecture flags were enabled.
- User action immediately before failure.
- Sanitized correlation or session identifier.

Without exact build identity, the wrong source map, mapping file, or dSYM can produce misleading stacks.

## Protect user data in diagnostics

> [!WARNING]
> Crash reports, breadcrumbs, logs, screenshots, and support attachments can contain personal data, URLs, filenames, account identifiers, message content, or tokens.

- Use allowlisted diagnostic fields.
- Redact authorization headers, cookies, request bodies, deep links, and private local paths.
- Avoid attaching entire application state or persisted stores.
- Limit retention and access according to the project's policy.
- Verify third-party crash-service collection and privacy configuration.
- Never ask users to post unreviewed diagnostic files publicly.

## Preserve symbols for every distributed build

Archive together, using restricted storage and a release identifier:

- Source commit and dependency lockfiles.
- Android `mapping.txt` for every optimized variant.
- Native Android symbols when native code is stripped.
- iOS Xcode archive and dSYM files.
- JavaScript bundle and matching source map when produced by the pipeline.
- Build configuration needed to reproduce the artifact.

Do not rebuild later and assume the new symbols match. Build outputs may differ even from the same source.

## Capture an Android crash locally

```bash
# Show the Android crash log buffer currently retained by the device.
adb logcat -b crash -d

# Stream Error-level logs while reproducing a failure.
adb logcat '*:E'
```

If multiple devices are connected, add `-s DEVICE_ID` after `adb`.

Save the earliest exception, complete `Caused by` chain, process/package, thread, and native signal if present. Later errors may only describe the application shutting down.

## Retrace an obfuscated Android stack

R8 can shorten class and method names. Use the exact `mapping.txt` generated for that release:

```bash
# Retrace an obfuscated stack using the matching release mapping file.
$ANDROID_HOME/cmdline-tools/latest/bin/retrace \
  android/app/build/outputs/mapping/release/mapping.txt \
  trace.txt
```

The mapping output directory and variant name can differ by project. A mismatched mapping file can create a plausible but incorrect stack.

> [!CAUTION]
> Android mapping files can reveal original application structure and symbol names. Store and share them only through approved systems.

## Diagnose Android ANRs

An ANR means the application did not respond within a platform-defined time. Common causes include:

- Blocking disk, network, database, or native work on the main thread.
- Lock contention or deadlock.
- Slow broadcast/service handling.
- Excessive startup or rendering work.
- Native calls that synchronously block JavaScript or UI threads.

Use Android vitals or the selected monitoring service to group ANRs by app version, device, OS, and stack. Reproduce with system tracing or Android Studio profiling rather than fixing only the final blocked method.

## Diagnose Android native crashes

Check for:

- Signal such as `SIGSEGV` or `SIGABRT`.
- Faulting native library and ABI.
- Complete native backtrace.
- Matching unstripped native symbols.
- Device architecture and OS version.
- Third-party SDK ownership.

If the crash is inside a vendor library, preserve a minimal reproduction and matching symbols/configuration before upgrading or reporting it.

## Acquire iOS reports

Sources include:

- Xcode Organizer for distributed builds.
- Xcode **Devices and Simulators → View Device Logs**.
- App Store Connect and TestFlight diagnostics.
- An approved crash-reporting provider.
- A user-exported `.crash` or diagnostic file.

Preserve the complete report, not only a screenshot of the top frames.

## Symbolicate iOS crashes

Xcode is the preferred first option because it can locate available symbols and symbolicate imported crash reports.

```text
Xcode → Window → Devices and Simulators
      → Select device → View Device Logs
      → Import the .crash file
```

Verify that the report's binary UUID matches the dSYM:

```bash
# Display UUIDs contained in an application's dSYM bundle.
dwarfdump --uuid /secure/path/MyApp.app.dSYM
```

If frames remain as hexadecimal addresses or show only offsets, locate the exact archive/dSYM for that build. Third-party frameworks may require their own dSYMs.

## Read an iOS crash report systematically

Check:

1. App version, build, device, OS, and timestamp.
2. Exception type, termination reason, and triggered thread.
3. Application-specific message or last exception backtrace.
4. Fully symbolicated frames on every relevant thread.
5. Binary Images section and UUIDs.
6. Similar reports grouped by signature and release.

Do not focus only on the first app-owned frame; inspect thread state and exception context.

## Recognize iOS memory terminations

Jetsam reports describe system memory pressure and process memory use but do not provide an ordinary executing-thread crash stack.

Correlate:

- Peak and sustained memory.
- Large images/media.
- Repeated navigation or retained objects.
- Background tasks and extensions.
- Device class and available memory.
- The exact user journey before termination.

Use Instruments and a matching physical-device scenario. See [App Performance and Memory Debugging](app-performance-and-memory-debugging.md).

## Symbolicate JavaScript exceptions

Minified release JavaScript stacks require the source map produced for the exact JavaScript bundle.

Confirm that the build pipeline:

- Generates and securely retains the release bundle and source map.
- Labels them with app version, build, platform, and commit.
- Uploads them to the approved crash service when applicable.
- Prevents public source-map distribution unless intentionally required.
- Tests symbolication before a production incident.

Hermes, Metro, build tooling, and monitoring-service commands vary by React Native version. Follow the documentation matching the project version rather than copying an old source-map command.

## Add useful breadcrumbs

Good breadcrumbs describe state transitions without recording user content:

```text
screen=checkout
action=submit_order
network=timeout
retry_attempt=1
app_state=foreground
feature_flag=checkout_v2
```

Avoid free-form object dumps. Use stable event names, allowlisted fields, and correlation IDs that the backend can resolve under controlled access.

## Group and prioritize crashes

Prioritize using:

```text
Impact = affected users × frequency × severity × business-critical flow
```

Also consider:

- New regression versus long-standing issue.
- Rollout percentage and growth rate.
- Data loss, security, payment, or login impact.
- Device/OS concentration.
- Workaround availability.
- Reproducibility and diagnostic confidence.

Do not prioritize only by raw event count; one retry loop can generate many events from few users.

## Build a minimal reproduction

1. Match app version, environment, account state, device, and OS.
2. Reproduce the smallest user journey.
3. Compare debug and release builds.
4. Disable one suspected feature or dependency at a time.
5. Preserve logs and profile before clearing state.
6. Add a regression test when the failure becomes reproducible.

Avoid clearing caches, reinstalling, or upgrading dependencies before collecting the evidence that may explain the failure.

## Production crash workflow

```text
Confirm build identity
        ↓
Classify JS, native, ANR, memory, or server-visible failure
        ↓
Acquire complete privacy-reviewed report
        ↓
Match mapping/source map/dSYM
        ↓
Symbolicate before forming root-cause hypothesis
        ↓
Group impact by version, OS, device, and flow
        ↓
Reproduce with matching release configuration
        ↓
Ship focused fix and monitor staged rollout
```

## Release checklist

- [ ] Release artifacts are tagged with version, build, and commit.
- [ ] Android mapping files and native symbols are retained.
- [ ] Xcode archives and dSYMs are retained.
- [ ] JavaScript source maps are retained/uploaded securely.
- [ ] Crash collection is tested in a non-production release build.
- [ ] Privacy redaction and access controls are reviewed.
- [ ] Alerts distinguish crashes, ANRs, and memory terminations.
- [ ] On-call/release owners know how to pause rollout or disable a feature.

## Related guides

- [React Native Debugging Guide](react-native-debugging-guide.md)
- [React Native Release Build and Signing Troubleshooting](release-build-and-signing-troubleshooting.md)
- [App Performance and Memory Debugging](app-performance-and-memory-debugging.md)
- [App Store Submission and Release Management](app-store-submission-and-release-management.md)

## Official references

- [React Native: Debugging Release Builds](https://reactnative.dev/docs/debugging-release-builds)
- [Android Vitals](https://developer.android.com/topic/performance/vitals)
- [Android: Troubleshoot R8 Optimization](https://developer.android.com/topic/performance/app-optimization/troubleshoot-the-optimization)
- [Apple: Diagnosing Issues Using Crash Reports](https://developer.apple.com/documentation/xcode/diagnosing-issues-using-crash-reports-and-device-logs)
- [Apple: Adding Identifiable Symbol Names](https://developer.apple.com/documentation/xcode/adding-identifiable-symbol-names-to-a-crash-report)
