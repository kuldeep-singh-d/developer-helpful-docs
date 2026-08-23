# Gradle Cleanup and Troubleshooting

A safe troubleshooting sequence for Android and React Native Gradle builds. Run commands from the project's `android/` directory unless noted otherwise.

## Use the project wrapper

```bash
# Show the Gradle version selected by the project.
./gradlew --version

# List available tasks.
./gradlew tasks
```

Prefer `./gradlew` over a globally installed `gradle` command so the project uses its configured version.

## Get a useful error report

```bash
# Re-run the failing build with a stack trace.
./gradlew assembleDebug --stacktrace

# Add detailed logging when the stack trace is not enough.
./gradlew assembleDebug --info
```

## Stop Gradle daemons

```bash
# Stop daemons for the Gradle version used by this project.
./gradlew --stop
```

### Use case

Try this when a daemon is unresponsive or appears to be using the wrong Java environment.

## Clean project build output

> [!WARNING]
> `clean` deletes generated build output. The next build will take longer, but source files are preserved.

```bash
# Remove generated build output for this Gradle project.
./gradlew clean

# Build the Android debug variant again.
./gradlew assembleDebug
```

## Refresh dependencies

```bash
# Re-check configured repositories and refresh resolved dependencies.
./gradlew assembleDebug --refresh-dependencies
```

Use this only when dependency metadata or downloaded artifacts appear stale; it can require network access and take longer.

## React Native build command

```bash
# Return to the React Native project root.
cd ..

# Build and install the Android app.
npx react-native run-android
```

## Recommended order

```text
Read first error → Add --stacktrace → Verify JDK/Gradle versions → Stop daemon → Clean project → Refresh dependencies
```

Avoid manually deleting the entire global `~/.gradle` directory as a routine fix. It removes shared caches for every Gradle project and often hides the real problem.
