# React Native Mobile Testing and CI Strategy

A practical guide for choosing valuable tests, running them consistently in continuous integration, controlling flaky failures, and building release confidence across JavaScript, Android, and iOS.

## Build a testing pyramid around risk

Use several layers because each catches different failures:

| Layer | Best for | Limitations |
|---|---|---|
| Static analysis | Types, syntax, unsafe patterns | Does not execute behavior |
| Unit tests | Business rules and transformations | Can miss integration problems |
| Component tests | Rendering and user interaction | Does not fully exercise native platforms |
| Integration tests | Storage, networking, and module boundaries | More setup and slower execution |
| End-to-end tests | Critical flows in a real app binary | Slowest and most prone to environmental flakiness |
| Manual/device testing | Hardware, visual quality, exploratory risks | Expensive and difficult to repeat consistently |

Do not pursue test counts alone. Automate high-impact behavior and use production incidents to improve coverage.

## Start with critical user journeys

List flows whose failure would block release or seriously harm users:

- Sign in, session restoration, logout, and account switching.
- Main product action and data synchronization.
- Purchase, subscription, or payment confirmation.
- Permission request and denial recovery.
- Deep-link and push-notification navigation.
- Offline creation and later retry.
- Upgrade with existing local data.
- Accessibility of core interactions.

Give each journey an owner, supported platforms, data prerequisites, expected result, and the fastest reliable test layer.

## Establish fast local checks

Use the scripts defined by the project. Typical examples are:

```bash
# Run the project's lint rules.
npm run lint

# Run TypeScript checking when the project exposes this script.
npm run typecheck

# Run the JavaScript test suite once for CI-style output.
npm test -- --runInBand
```

`--runInBand` can reduce resource contention in small CI runners, but parallel tests may be faster when isolation is correct. Measure before standardizing it.

## Test behavior rather than implementation details

Prefer assertions based on what a user can see, hear, or do:

```tsx
it('shows an error when sign-in fails', async () => {
  render(<SignInScreen />);

  await user.type(screen.getByLabelText('Email'), 'person@example.com');
  await user.press(screen.getByRole('button', {name: 'Sign in'}));

  expect(await screen.findByText('Unable to sign in')).toBeVisible();
});
```

This is illustrative. Adapt it to the testing library and user-event API installed by the project.

Avoid asserting private component state, calling internal methods, or relying on large snapshots for important logic. Such tests often pass while the user flow is broken.

## Keep business logic easy to test

- Separate formatting, validation, and state transitions from rendering.
- Inject clocks, random values, API clients, and storage boundaries.
- Return explicit success/failure results from important operations.
- Keep platform-specific adapters behind small interfaces.
- Avoid global mutable state shared between tests.

Use real logic whenever practical. Mock network, time, storage, and native modules at clear boundaries rather than mocking every internal function.

## Test native boundaries explicitly

JavaScript tests cannot prove that Android and iOS native configuration is correct. Add device or native integration coverage for:

- Native modules and SDK initialization.
- Permissions and operating-system dialogs.
- Universal/App Links and custom schemes.
- Push notification delivery and tap handling.
- Camera, biometrics, location, Bluetooth, and background execution.
- Release signing/configuration and code shrinking.

Rebuild the app after native dependency changes. See [Native Module and Autolinking Troubleshooting](native-module-and-autolinking-troubleshooting.md).

## Keep end-to-end coverage focused

End-to-end tests should cover a small number of high-value paths, not every visual variant.

Good candidates:

- Cold launch and session restoration.
- One successful and one failed authentication path.
- The primary create/read/update workflow.
- Offline queue and recovery.
- Deep link into an authenticated screen.
- Upgrade of important persisted data.

Use stable accessibility labels or user-visible roles/text. Do not depend on screen coordinates, arbitrary sleep calls, test order, or live production accounts.

## Design deterministic test data

- Give each CI run unique account/data identifiers.
- Reset state through supported test APIs or isolated fixtures.
- Freeze time when testing expiry or date logic.
- Stub third-party services at a controlled boundary.
- Avoid shared mutable accounts across parallel jobs.
- Clean up created server data when safe and necessary.

> [!WARNING]
> Never place production passwords, signing keys, API tokens, or customer data in the repository, test fixtures, screenshots, or CI logs. Use the CI provider's protected secret storage and least-privileged test accounts.

## Use a practical CI pipeline

```text
Install locked dependencies
        ↓
Lint and type-check
        ↓
Run unit and component tests
        ↓
Build Android and iOS artifacts
        ↓
Run selected integration/E2E tests
        ↓
Collect test reports and safe diagnostics
        ↓
Publish immutable candidate artifacts
```

Fail early on inexpensive checks, then run costly native and device jobs. Build release-like artifacts before declaring a release candidate healthy.

## Install dependencies reproducibly

```bash
# Install exactly the npm dependency versions recorded in package-lock.json.
npm ci
```

Commit and preserve the project's lockfile. Pin runner images and important build tool versions where possible. Record Node.js, Java, Ruby, Xcode, Android SDK, and React Native versions in CI diagnostics.

## Validate native builds in CI

```bash
# Compile the Android debug application without starting Metro.
./android/gradlew -p android app:assembleDebug

# Compile the Android release application without publishing it.
./android/gradlew -p android app:assembleRelease
```

For iOS, use the workspace, scheme, SDK, and signing mode defined by the project. Keep simulator builds separate from distribution archives; a simulator success does not validate device signing or App Store configuration.

## Control flaky tests systematically

A flaky test passes and fails without a relevant code change. Treat it as lost signal, not ordinary noise.

When a test flakes:

1. Save logs, screenshots, video, device state, seed, and timing information.
2. Reproduce under the same runner and app configuration.
3. Identify uncontrolled time, data, animation, network, or test-order dependencies.
4. Fix the synchronization or isolation problem.
5. Quarantine only with an owner, issue, and removal deadline.

Retries may expose frequency but should not silently turn an unreliable test green. Track first-attempt failure separately.

## Wait for conditions, not arbitrary time

Avoid:

```ts
// Fragile: assumes every device and CI runner finishes in two seconds.
await sleep(2000);
```

Prefer waiting for a visible element, network-idle signal, completed state, or explicit test hook with a bounded timeout. Disable or account for animations when they prevent stable interaction.

## Choose a useful device matrix

Cover differences that can change behavior:

- Minimum and current supported OS versions.
- At least one lower-memory or lower-performance Android profile.
- Common screen size plus a small-screen case.
- Release architecture and ABI combinations used in production.
- Devices required for hardware-only capabilities.
- Representative language, RTL, font scale, and accessibility settings.

Run a small pull-request matrix and a broader scheduled or release matrix. Use production analytics and business impact to revise it.

See [Device and OS Fragmentation Testing](device-and-os-fragmentation-testing.md).

## Test upgrades, not only clean installs

A release candidate should be installed over the previous production build with realistic data. Verify:

- Local schema and key-value migrations.
- Authentication/session restoration.
- Pending offline operations.
- Push registration and deep links.
- User preferences and downloaded files.
- Safe behavior if the migration is interrupted.

Clean-install tests cannot detect most upgrade failures.

## Define pull-request and release gates

Example pull-request gates:

- Lint and type checks pass.
- Unit/component tests pass.
- Android and iOS compile where affected.
- No unexplained test quarantine or snapshot expansion.

Example release gates:

- Signed candidate tested without Metro.
- Critical E2E journeys pass on both platforms.
- Upgrade path and offline recovery pass.
- Crash reporting and source maps/symbols are configured.
- Security, privacy, accessibility, and store checklists are complete.

Document who can override a gate and require a written risk decision.

## Preserve useful test evidence

On failure, retain according to the project's privacy policy:

- Test name, runner image, app version, and commit.
- Sanitized device and application logs.
- Screenshots/video without sensitive user data.
- Test report and timing.
- Native build output and dependency versions.

Avoid printing all environment variables or uploading entire app data directories.

## Review test value over time

Regularly ask:

- Which production defects escaped and where should they have been caught?
- Which tests fail for irrelevant reasons?
- Which critical flows have no device-level validation?
- Which suite is too slow for its value?
- Which unsupported devices or old fixtures can be removed?
- Are quarantined tests being fixed or forgotten?

## Quick CI checklist

- [ ] Dependency installation uses the committed lockfile.
- [ ] Lint, types, and JavaScript tests run on every relevant change.
- [ ] Native projects compile in CI.
- [ ] Critical paths have focused device-level tests.
- [ ] Test data is isolated and contains no production secrets.
- [ ] Flaky tests have owners and deadlines.
- [ ] Upgrade paths are tested with historical data.
- [ ] Release artifacts are tested without Metro.
- [ ] Failure evidence is useful and privacy-safe.

## Related guides

- [Device and OS Fragmentation Testing](device-and-os-fragmentation-testing.md)
- [App Store Submission and Release Management](app-store-submission-and-release-management.md)
- [App Performance and Memory Debugging](app-performance-and-memory-debugging.md)
- [Crash Reporting and Symbolication](crash-reporting-and-symbolication.md)

## Official references

- [React Native Testing Overview](https://reactnative.dev/docs/testing-overview)
- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [Android: Test Your App](https://developer.android.com/studio/test)
- [Apple: Testing Your Apps in Xcode](https://developer.apple.com/documentation/xcode/testing-your-apps-in-xcode)
