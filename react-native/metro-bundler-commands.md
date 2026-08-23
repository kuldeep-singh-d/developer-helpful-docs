# Metro Bundler Commands

A quick reference for running and troubleshooting Metro in React Native projects.

## Start Metro

```bash
# Start Metro from the React Native project root.
npx react-native start
```

## Reset the transform cache

```bash
# Start Metro after clearing its transform cache.
npx react-native start --reset-cache
```

Use this only when Metro is serving stale transforms or reporting inconsistent module data.

## Use a different port

```bash
# Start Metro on port 8088.
npx react-native start --port 8088
```

The application must be configured to connect to the same port.

## Connect a physical Android device

```bash
# Forward the default Metro port from an Android device to this computer.
adb reverse tcp:8081 tcp:8081
```

## Check which process uses port 8081 on macOS

```bash
# Show the process listening on Metro's default port.
lsof -nP -iTCP:8081 -sTCP:LISTEN
```

Do not kill a process until you identify it and confirm it is safe to stop.

## Troubleshooting order

```text
Confirm project root → Confirm one Metro instance → Check port → Restart Metro → Reset cache
```
