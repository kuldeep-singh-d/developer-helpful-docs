# CocoaPods Troubleshooting

A safe guide for installing and repairing iOS dependencies managed by CocoaPods.

## Install locked dependencies

```bash
# Enter the iOS project directory.
cd ios

# Install pods while respecting Podfile.lock.
pod install
```

Use `pod install` after cloning a project or changing its `Podfile`. Open the generated `.xcworkspace` afterward.

## Inspect the environment

```bash
# Display CocoaPods, Ruby, Xcode, and repository environment details.
pod env

# Show additional detail during installation.
pod install --verbose
```

Review output before sharing because local paths may contain usernames or project information.

## Update one dependency intentionally

> [!WARNING]
> `pod update` changes locked dependency versions. Review and test the resulting `Podfile.lock` changes.

```bash
# Check which pods have newer compatible versions.
pod outdated

# Update only the named pod.
pod update POD_NAME
```

Do not run `pod update` when you only need to install existing locked versions.

## Recreate local pod integration

> [!WARNING]
> This removes generated local CocoaPods integration. Keep `Podfile` and `Podfile.lock` unless a dependency change is intentional.

```bash
# Remove CocoaPods integration from the Xcode project.
pod deintegrate

# Reinstall the locked dependencies and integration.
pod install
```

## Recommended order

```text
pod install → inspect first error → pod env/--verbose → verify Ruby/Xcode → deintegrate and reinstall → update versions only intentionally
```
