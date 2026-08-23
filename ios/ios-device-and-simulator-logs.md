# iOS Device and Simulator Logs

A practical reference for collecting iOS Simulator and connected-device logs.

## List simulator devices

```bash
# List available simulator names, UDIDs, and states.
xcrun simctl list devices available
```

## Stream booted Simulator logs

```bash
# Stream Debug-level logs from the currently booted simulator.
xcrun simctl spawn booted log stream --level debug
```

Press `Control+C` to stop streaming.

## Filter by process

```bash
# Replace MyApp with the executable process name.
xcrun simctl spawn booted log stream --level debug --predicate 'process == "MyApp"'
```

## Save recent Simulator logs

```bash
# Save the last five minutes of logs to a text file.
xcrun simctl spawn booted log show --last 5m > ios-simulator-log.txt
```

> [!WARNING]
> Logs can contain usernames, device details, URLs, identifiers, or application data. Review and redact them before sharing publicly.

## Physical devices

For a connected iPhone or iPad, use Xcode's device console:

```text
Xcode → Window → Devices and Simulators → Select device → Open Console
```

## Debugging flow

```text
Confirm target → Start focused log stream → Reproduce once → Stop capture → Redact sensitive information → Share relevant section
```
