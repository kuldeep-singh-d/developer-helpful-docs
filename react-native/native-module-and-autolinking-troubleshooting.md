# React Native Native Module and Autolinking Troubleshooting

A practical guide for diagnosing native packages that install successfully but fail to link, compile, load, or behave correctly on Android or iOS.

## Understand what installation changes

A package containing native code crosses several layers:

```text
package.json and lockfile
        ↓
React Native autolinking configuration
        ↓
Gradle or CocoaPods dependency integration
        ↓
Native compilation and application binary
        ↓
JavaScript bundle calls the installed native implementation
```

Restarting Metro updates JavaScript, but it cannot add native code to an already installed binary. Rebuild the application after adding, removing, or changing a native dependency.

## Classify the symptom first

| Symptom | Start investigation here |
|---|---|
| Package import cannot be resolved | Package installation, exports, Metro resolution |
| `Native module is null` or not found | Autolinking, Pods/Gradle, stale installed binary |
| iOS linker error | Pod integration, frameworks, architectures, deployment target |
| Android duplicate class | Conflicting transitive dependencies or manual linking |
| Codegen/specification error | New Architecture configuration and typed spec |
| Debug works but release fails | Shrinking, build variants, conditional configuration, missing symbols |
| One platform works | Platform-specific setup, permissions, native source, or package support |

Copy the first meaningful native build error, not only the final `BUILD FAILED` line.

## Confirm the package and project state

```bash
# Confirm the package is present in the dependency tree.
npm ls package-name

# Ask the React Native CLI how it sees native dependencies and project platforms.
npx react-native config

# Show the React Native version used by this project.
npm ls react-native
```

Replace `package-name` with the actual dependency. Check its supported React Native versions, platforms, New Architecture status, and any manual native setup.

Do not assume a similarly named npm package supports React Native; some packages target only browsers or Node.js.

## Rebuild after native dependency changes

```bash
# Reinstall iOS Pods using the project Podfile configuration.
npx pod-install

# Rebuild and install the Android application.
npm run android

# Rebuild and install the iOS application.
npm run ios
```

If an old app binary remains installed, JavaScript may load the new package while the native implementation is still missing.

## Inspect autolinking output

Run:

```bash
# Print the CLI configuration, including discovered native dependencies.
npx react-native config
```

For the failing package, verify:

- It appears under dependencies.
- Android source directory, package import, and platform configuration are present when supported.
- iOS podspec/path is detected when supported.
- The package has not been excluded by `react-native.config.js`.
- A monorepo or custom path does not point to another copy.

If it is absent, inspect the package metadata and installation path before clearing caches.

## Remove obsolete manual linking

Modern React Native projects normally use autolinking. An old manual installation can cause duplicate registration or duplicate native classes.

Look for historical changes in:

- Android `settings.gradle`, app `build.gradle`, and application package lists.
- iOS `Podfile`, Xcode linked frameworks, library search paths, and build phases.
- Custom package initialization in application delegates.

Remove manual entries only after confirming the package supports autolinking and the entries are not required custom setup.

## Troubleshoot iOS integration

```bash
# Install Pods and display dependency-resolution output.
npx pod-install

# Show installed Pods that match the package name.
rg -i "package-name" ios/Podfile.lock

# List available Xcode schemes from the workspace.
xcodebuild -list -workspace ios/YourApp.xcworkspace
```

Replace `package-name` and `YourApp` with project values.

Check:

- Open the `.xcworkspace`, not only `.xcodeproj`, when CocoaPods is used.
- The pod appears in `Podfile.lock` and the intended application target.
- The iOS deployment target meets the package requirement.
- Swift, Objective-C, static/dynamic framework, and modular-header requirements are compatible.
- All application extensions link only dependencies they actually support.
- The selected scheme and configuration match the failing build.

If dependency resolution fails, fix the stated version conflict before deleting Pods or lockfiles.

## Troubleshoot Android integration

```bash
# Show Gradle projects and confirm the app module is available.
./android/gradlew -p android projects

# Print the app dependency tree for the debug runtime classpath.
./android/gradlew -p android app:dependencies --configuration debugRuntimeClasspath

# Compile the debug Android application and preserve the full native error.
./android/gradlew -p android app:assembleDebug --stacktrace
```

Check:

- The package supports the selected Android SDK and Java/Kotlin toolchain.
- Repositories are declared in the correct modern Gradle location.
- Only one compatible version of a conflicting native dependency is resolved.
- `namespace`, manifest placeholders, resources, and required permissions are configured.
- Product flavors use the expected application class and source set.
- The package is not both manually registered and autolinked.

Use dependency constraints or exclusions only after identifying why two versions conflict. A forced version can compile while causing a runtime crash.

## Check build variants and environment configuration

A module may work in debug but fail in release because:

- R8 removes classes reached through reflection.
- Release-only configuration values are missing.
- A development implementation is not packaged in release.
- Different flavors use different manifests, identifiers, or native files.
- JavaScript minification exposes unsafe assumptions.
- The release build enables the New Architecture while debug does not, or the reverse.

```bash
# Build the release variant locally and show detailed Gradle failures.
./android/gradlew -p android app:assembleRelease --stacktrace
```

Test a signed/distributed release build without Metro. See [Release Build and Signing Troubleshooting](release-build-and-signing-troubleshooting.md).

## Diagnose New Architecture and Codegen failures

Native Modules and Fabric Native Components may depend on Codegen. Confirm:

- The package version supports the project's React Native version.
- Typed specifications use supported names and types.
- `codegenConfig` points to the correct source directory.
- Generated interfaces are recreated by the supported native build process.
- Legacy manual registration is not mixed with generated registration.
- The package works under the architecture setting used by the release build.

```bash
# List Gradle tasks exposed by this project and locate Codegen-related tasks.
./android/gradlew -p android tasks --all | rg -i "codegen|schema"
```

Use the task names exposed by the installed React Native version instead of copying a task from an unrelated project.

> [!WARNING]
> Do not edit generated Codegen output as the permanent fix. Correct the typed specification or configuration and regenerate it.

## Handle monorepos and duplicate React Native copies

Symptoms include invalid hook calls, type mismatches, native registration problems, or packages resolving from an unexpected directory.

```bash
# Show every React Native version resolved in the dependency tree.
npm ls react-native --all

# Show why a package exists in an npm dependency tree.
npm explain package-name
```

Verify workspace hoisting, Metro watch folders, symlink behavior, native project paths, and peer dependency ranges. A reusable library should normally declare React and React Native as peer dependencies rather than bundling private copies.

## Use cache cleanup only after configuration checks

Start with the least destructive action:

1. Stop and restart Metro.
2. Rebuild the application.
3. Reinstall Pods when iOS native dependencies changed.
4. Clean the affected native build output.
5. Reinstall JavaScript dependencies only when the dependency tree is inconsistent.

See [React Native Cache Cleanup](react-native-cache-cleanup.md) for targeted commands and warnings.

> [!WARNING]
> Deleting lockfiles can select different dependency versions across the entire project. Preserve the lockfile unless intentionally updating and reviewing dependencies.

## Validate runtime behavior

After the build succeeds, test:

- Physical Android and iOS devices when the API depends on hardware.
- Permission denied, restricted, and unavailable states.
- App background/resume and process restart.
- Debug and release builds.
- Both New Architecture settings when supporting both is a project requirement.
- Upgrade from the previous app version.
- Failure behavior when the underlying native service is unavailable.

## Quick troubleshooting workflow

```text
Capture first native or runtime error
        ↓
Confirm package, React Native, and platform compatibility
        ↓
Inspect `npx react-native config`
        ↓
Remove accidental duplicate/manual integration
        ↓
Verify Pods or Gradle dependency graph
        ↓
Check architecture, Codegen, variant, and monorepo paths
        ↓
Rebuild the native binary and test release behavior
```

## Related guides

- [Common React Native Build Errors](common-build-errors.md)
- [React Native Cache Cleanup](react-native-cache-cleanup.md)
- [CocoaPods Troubleshooting](../ios/cocoapods-troubleshooting.md)
- [Gradle Cleanup and Troubleshooting](../android/gradle-cleanup-and-troubleshooting.md)

## Official references

- [React Native: Using Libraries](https://reactnative.dev/docs/libraries)
- [React Native: Native Platform](https://reactnative.dev/docs/native-platform)
- [React Native: Native Modules](https://reactnative.dev/docs/turbo-native-modules-introduction)
- [React Native: Codegen](https://reactnative.dev/docs/the-new-architecture/using-codegen)
