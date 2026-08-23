# Android Logcat Debugging

A focused guide for reading and filtering Android system and application logs from Terminal.

## Confirm the device

```bash
# Confirm that ADB can see the target device.
adb devices
```

## Stream logs

```bash
# Stream new Android log messages until you press Control+C.
adb logcat
```

### Use case

Use this while reproducing a crash, startup failure, or native Android error.

## Show useful timestamps

```bash
# Display date, time, process ID, thread ID, priority, and tag.
adb logcat -v threadtime
```

## Filter by priority or tag

```bash
# Show Error-level messages while hiding unmatched tags.
adb logcat '*:E'

# Show ReactNativeJS messages at Verbose level and above.
adb logcat ReactNativeJS:V '*:S'
```

Quote filter expressions so the shell does not expand `*`.

## Show the crash buffer

```bash
# Display logs kept in Android's crash buffer.
adb logcat -b crash -d
```

## Save logs to a file

```bash
# Capture the current buffered logs for sharing or later analysis.
adb logcat -d -v threadtime > android-logcat.txt
```

Review the file before sharing it because logs can contain device, account, URL, or application data.

## Clear old logs

> [!WARNING]
> This removes the current Logcat buffer. Save important logs first.

```bash
# Clear buffered logs so the next capture contains only a fresh reproduction.
adb logcat -c
```

## Recommended debugging flow

```text
Confirm device → Clear old logs if needed → Start Logcat → Reproduce issue → Save relevant output
```
