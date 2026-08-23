# Xcode Cleanup Commands

A focused reference for cleaning Xcode build output without resetting simulator data.

## Clean the current project

```bash
# Move into the iOS project directory.
cd ios

# Remove build products through Xcode's build system.
xcodebuild clean
```

For a workspace with multiple schemes:

```bash
# Replace names with the project's actual workspace and scheme.
xcodebuild clean -workspace MyApp.xcworkspace -scheme MyApp
```

## Inspect available schemes

```bash
# List project or workspace build information, including schemes.
xcodebuild -list
```

## Remove DerivedData

> [!WARNING]
> This removes cached builds and indexes for every Xcode project. The next build and indexing pass will take longer.

```bash
# Delete the contents of Xcode's shared DerivedData directory.
rm -rf ~/Library/Developer/Xcode/DerivedData/*
```

## Recommended order

```text
Rebuild → xcodebuild clean → Reinstall pods if relevant → Remove DerivedData → Reset simulator only if device state is broken
```

Simulator reset commands are documented separately in [iOS Simulator Cleanup Commands](ios-simulator-cleanup-commands.md).
