# Developer Helpful Docs

A collection of practical developer documentation, useful commands, troubleshooting guides, setup notes, and quick references for everyday development.

The goal of this repository is to keep commonly used developer commands and troubleshooting steps organized in one place so they are easy to find, understand, and reuse.

---

## 📚 Documentation

### Android

Useful Android development, ADB, emulator, and troubleshooting references.

- [Android Emulator Cleanup Commands](android/android-emulator-cleanup-commands.md)
- [ADB Useful Commands](android/adb-useful-commands.md)
- [Android Logcat Debugging](android/android-logcat-debugging.md)
- [Android APK Installation Commands](android/android-apk-installation-commands.md)
- [Gradle Cleanup and Troubleshooting](android/gradle-cleanup-and-troubleshooting.md)

### iOS

Useful iOS Simulator, Xcode, CocoaPods, and troubleshooting references.

- [iOS Simulator Cleanup Commands](ios/ios-simulator-cleanup-commands.md)
- [Xcode Cleanup Commands](ios/xcode-cleanup-commands.md)
- [CocoaPods Troubleshooting](ios/cocoapods-troubleshooting.md)
- [iOS Device and Simulator Logs](ios/ios-device-and-simulator-logs.md)

### React Native

React Native development, debugging, cache cleanup, build errors, and useful commands.

- [React Native Cache Cleanup](react-native/react-native-cache-cleanup.md)
- [React Native Debugging Guide](react-native/react-native-debugging-guide.md)
- [Metro Bundler Commands](react-native/metro-bundler-commands.md)
- [Common React Native Build Errors](react-native/common-build-errors.md)

### Git

Common Git commands, undo operations, branch management, and everyday Git workflows.

- [Git Common Commands](git/git-common-commands.md)
- [Git Undo Commands](git/git-undo-commands.md)

### macOS

Useful macOS terminal commands and developer environment references.

- [macOS Developer Commands](macos/macos-developer-commands.md)

### VS Code

Useful VS Code settings, shortcuts, extensions, and development tips.

- [VS Code React Native Setup](vscode/vscode-react-native-setup.md)

### GitHub

- [GitHub Pull Request Workflow](github/github-pull-request-workflow.md)

### Environment Setup

- [React Native Development Environment on macOS](setup/react-native-development-environment-macos.md)

---

## 🗂 Repository Structure

```text
developer-helpful-docs/
│
├── README.md
│
├── LICENSE
│
├── CONTRIBUTING.md
│
├── CODE_OF_CONDUCT.md
│
├── .gitignore
│
├── .github/
│   └── pull_request_template.md
│
├── android/
│   ├── android-emulator-cleanup-commands.md
│   ├── adb-useful-commands.md
│   ├── android-logcat-debugging.md
│   ├── android-apk-installation-commands.md
│   └── gradle-cleanup-and-troubleshooting.md
│
├── ios/
│   ├── ios-simulator-cleanup-commands.md
│   ├── xcode-cleanup-commands.md
│   ├── cocoapods-troubleshooting.md
│   └── ios-device-and-simulator-logs.md
│
├── react-native/
│   ├── react-native-cache-cleanup.md
│   ├── react-native-debugging-guide.md
│   ├── metro-bundler-commands.md
│   └── common-build-errors.md
│
├── git/
│   ├── git-common-commands.md
│   └── git-undo-commands.md
│
├── macos/
│   └── macos-developer-commands.md
│
├── vscode/
│   └── vscode-react-native-setup.md
│
├── github/
│   └── github-pull-request-workflow.md
│
└── setup/
    └── react-native-development-environment-macos.md
```

---

## 🎯 Purpose

As developers, we often use the same commands repeatedly for things like:

- Cleaning Android emulators
- Resetting iOS simulators
- Fixing React Native build problems
- Clearing caches
- Debugging ADB issues
- Cleaning Xcode builds
- Managing Git branches and commits
- Fixing common development environment issues

Finding the correct command again can take unnecessary time.

This repository keeps those commands documented with explanations, examples, use cases, and warnings so they can be reused quickly and safely.

---

## ✨ Documentation Style

The documentation in this repository focuses on being:

- Clear and beginner-friendly
- Practical and easy to follow
- Focused on real development use cases
- Easy to copy and use from Terminal
- Organized by technology
- Safe for troubleshooting workflows

Commands usually include comments explaining what they do.

Example:

```bash
# Show all Android Virtual Devices installed on your machine.
emulator -list-avds
```

When a command can remove data or make significant changes, the documentation includes a warning.

Example:

```bash
# WARNING:
# This will completely reset the selected Android emulator
# and remove installed apps, accounts, settings, and application data.

emulator -avd Pixel_9_Pro -wipe-data
```

---

## 🛠 Topics Covered

This repository will gradually include documentation for:

- Android
- Android Emulator
- ADB
- iOS
- iOS Simulator
- Xcode
- CocoaPods
- React Native
- Metro Bundler
- Gradle
- Git
- GitHub
- macOS
- VS Code
- Terminal commands
- Build troubleshooting
- Development environment setup
- Debugging workflows
- Useful developer utilities

---

## 🚀 Example Guides

### Android Emulator Cleanup

Learn how to:

- List Android emulators
- Factory-reset an emulator
- Clear individual application data
- Uninstall applications
- Restart ADB
- Cold boot an emulator
- Start without Quick Boot snapshots

See:

[Android Emulator Cleanup Commands](android/android-emulator-cleanup-commands.md)

---

### iOS Simulator Cleanup

Learn how to:

- List available simulators
- Check running simulators
- Shutdown simulators
- Reset one simulator
- Reset all simulators
- Uninstall applications
- Restart CoreSimulator
- Clean Xcode builds
- Clear DerivedData
- Reset Metro cache

See:

[iOS Simulator Cleanup Commands](ios/ios-simulator-cleanup-commands.md)

---

## ⚠️ Command Safety

Some developer commands can delete:

- Emulator data
- Simulator data
- Application data
- Build files
- Caches
- DerivedData
- Installed applications

Always read the warning and command description before running destructive commands.

Whenever possible, the guides recommend trying the least-destructive troubleshooting option first.

For example:

```text
Restart application
        ↓
Clear app data
        ↓
Restart ADB / Simulator
        ↓
Clear build cache
        ↓
Reset emulator / simulator
```

A full reset should normally be used only when simpler troubleshooting steps do not solve the issue.

---

## 🧑‍💻 Who Is This Repository For?

This repository can be useful for:

- React Native developers
- Android developers
- iOS developers
- Mobile application developers
- Developers learning Terminal commands
- Developers troubleshooting local development environments
- Anyone who wants quick development command references

---

## 🤝 Contributions

Contributions, corrections, and improvements are welcome.

If you find:

- An incorrect command
- A missing explanation
- A safer approach
- A useful developer command
- A common troubleshooting solution

feel free to open an issue or submit a pull request.

Read [CONTRIBUTING.md](CONTRIBUTING.md) before preparing a contribution.

Please keep contributions:

- Simple
- Practical
- Well explained
- Safe to use
- Beginner-friendly

---

## 📄 File Naming

Documentation files use lowercase names with hyphens.

Example:

```text
android-emulator-cleanup-commands.md
ios-simulator-cleanup-commands.md
react-native-cache-cleanup.md
git-common-commands.md
```

This keeps the repository consistent and URLs easy to read.

---

## 🔄 Repository Status

This repository is actively being expanded.

More documentation will be added over time as useful commands, troubleshooting scenarios, and development workflows are encountered.

---

## ⭐ Support

If you find these docs useful, consider starring the repository.

It helps make the project easier for other developers to discover.

---

## 📄 License

This project is intended to be open and reusable for developers.

See the [LICENSE](LICENSE) file for licensing details.
