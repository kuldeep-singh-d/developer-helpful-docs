# React Native Network and API Debugging

A practical guide for diagnosing API requests that work in a browser or API client but fail in a React Native app.

## Start with the exact failure

Before changing configuration, record:

- The request URL, method, and status code.
- Whether the problem affects Android, iOS, or both.
- Whether it occurs on a simulator, emulator, physical device, or release build.
- Whether the API is local, on a private network, or publicly reachable.
- The earliest useful error from JavaScript and native logs.

Do not record passwords, access tokens, cookies, API keys, or personal data.

## Confirm the API outside the app

```bash
# Request a non-sensitive health endpoint and include response headers.
curl --include --connect-timeout 10 https://api.example.com/health
```

### Use case

This separates a server, DNS, or TLS problem from an application-only problem. A successful request from the computer does not prove that a physical device can reach the same host.

## Understand `localhost`

Inside a mobile app, `localhost` normally refers to the simulator, emulator, or physical device—not automatically to the development computer.

| Runtime | Development server address |
|---|---|
| Android Emulator | Use `10.0.2.2` to reach the computer's loopback interface |
| iOS Simulator | `localhost` commonly reaches the Mac-hosted service |
| Physical Android or iOS device | Use a reachable LAN hostname or IP address |

Android Emulator example:

```text
http://10.0.2.2:3000
```

Physical-device example:

```text
http://192.168.1.20:3000
```

For a physical device, confirm that:

1. The device and computer are on a network that permits communication between them.
2. The development server listens on a reachable interface, not only `127.0.0.1`.
3. The operating-system firewall permits the required port.
4. A VPN, proxy, guest Wi-Fi, or corporate network is not isolating the devices.

> [!WARNING]
> Binding a development server to `0.0.0.0` can expose it to other devices on the network. Use a trusted network, avoid production credentials, and stop the server when finished.

## Check whether the port is listening

```bash
# Show the process listening on a local API port on macOS.
lsof -nP -iTCP:3000 -sTCP:LISTEN
```

Verify the process before stopping or reconfiguring it.

## Inspect requests in React Native DevTools

With Metro focused, press:

```text
j
```

In supported React Native versions, open the **Network** panel and reproduce the request. Inspect:

- Final URL and HTTP method.
- Request and response headers.
- Status code and response preview.
- Request timing and initiating call stack.

If the request does not appear, confirm that the app actually reached the request code. Some custom networking libraries or older React Native versions may require platform-native logging instead.

> [!WARNING]
> Network inspectors can display authorization headers, cookies, request bodies, and personal data. Redact sensitive values before sharing screenshots or logs.

## Interpret common HTTP results

| Result | Likely area to investigate |
|---|---|
| No request appears | Code path, validation, early exception, or unsupported inspector capture |
| Network request failed | Address, DNS, TLS, HTTP policy, firewall, VPN, or connectivity |
| `400` | Request body, query parameters, content type, or validation |
| `401` | Missing, expired, malformed, or wrong-environment credentials |
| `403` | Account permissions, server authorization, or environment restrictions |
| `404` | Base URL, route, API version, or path construction |
| `408` or timeout | Slow server, unreachable host, proxy, or timeout configuration |
| `429` | Rate limit or retry behavior |
| `500`–`599` | Server failure; correlate with backend logs and request ID |

## Diagnose authentication safely

Check that the app is using the intended development, staging, or production environment. Verify:

- The token exists before the request starts.
- The expected authorization scheme is used, such as `Bearer`.
- Device date and time are correct for time-sensitive tokens.
- A refresh flow is not replacing a valid token with an expired one.
- A `401` retry cannot loop forever.

Never print full tokens. If comparison is necessary, log only a non-sensitive label, token presence, or a short irreversible fingerprint.

## Android HTTP and certificate failures

Android can reject cleartext HTTP traffic depending on the app's target and network security configuration. Prefer HTTPS with a valid certificate.

For local development, use a narrowly scoped, debug-only Network Security Configuration rather than allowing cleartext traffic for every destination or release build.

> [!WARNING]
> Cleartext HTTP can be read or modified by other systems on the network. Never send real credentials or personal data over an insecure development connection.

Useful Android logs:

```bash
# Show common network, TLS, and cleartext-policy messages.
adb logcat | grep -iE 'cleartext|ssl|tls|unknownhost|connectexception|timeout'
```

If multiple devices are connected, add `-s DEVICE_ID` after `adb`.

## iOS App Transport Security failures

App Transport Security (ATS) expects secure connections that satisfy Apple's TLS requirements. Prefer correcting the server certificate or TLS configuration.

```bash
# Test how an HTTPS endpoint behaves under ATS requirements.
/usr/bin/nscurl --ats-diagnostics --verbose https://api.example.com
```

Use narrowly scoped development exceptions only when required. Avoid globally disabling ATS, especially in release builds.

Useful Simulator logs:

```bash
# Stream network-related messages from the booted iOS Simulator.
xcrun simctl spawn booted log stream --level debug \
  --predicate 'eventMessage CONTAINS[c] "network" OR eventMessage CONTAINS[c] "TLS" OR eventMessage CONTAINS[c] "ATS"'
```

## Test physical-device reachability

Open the API's non-sensitive health URL in the device browser. If it cannot connect:

1. Verify the LAN address and port.
2. Confirm the server is running and reachable from another device.
3. Temporarily disconnect VPN or proxy software if policy permits.
4. Check firewall and Wi-Fi client-isolation settings.
5. Try a different trusted network to isolate the environment.

Do not expose a private development API directly to the public internet as a quick fix.

## Handle timeouts and offline conditions

A production app should distinguish between:

- No network connection.
- DNS or TLS failure.
- Request timeout.
- Server error.
- User cancellation.

Retries should be limited and use delay or backoff. Avoid automatically retrying non-idempotent operations, such as payments or order creation, unless the API supports idempotency.

## Recommended debugging workflow

```text
Capture exact error
        ↓
Confirm URL and environment
        ↓
Test API from the computer
        ↓
Check emulator/device addressing
        ↓
Inspect request and response
        ↓
Check Android/iOS security logs
        ↓
Verify authentication and server logs
        ↓
Test release configuration separately
```

Change one thing at a time and retest the same request. Avoid clearing caches or resetting devices unless evidence points to stale local state.

## Quick reference

```bash
# Test a public HTTPS health endpoint.
curl --include --connect-timeout 10 https://api.example.com/health

# Check a local development port on macOS.
lsof -nP -iTCP:3000 -sTCP:LISTEN

# Check Android network-related logs.
adb logcat | grep -iE 'cleartext|ssl|tls|unknownhost|connectexception|timeout'

# Diagnose iOS ATS compatibility.
/usr/bin/nscurl --ats-diagnostics --verbose https://api.example.com
```

## Related guides

- [React Native Debugging Guide](react-native-debugging-guide.md)
- [Android Logcat Debugging](../android/android-logcat-debugging.md)
- [iOS Device and Simulator Logs](../ios/ios-device-and-simulator-logs.md)
- [Common React Native Build Errors](common-build-errors.md)

## Official references

- [React Native Networking](https://reactnative.dev/docs/network)
- [React Native DevTools](https://reactnative.dev/docs/react-native-devtools)
- [Android Emulator Network Address Space](https://developer.android.com/studio/run/emulator-networking-address)
- [Android Network Security Configuration](https://developer.android.com/privacy-and-security/security-config)
- [Apple App Transport Security](https://developer.apple.com/documentation/security/preventing-insecure-network-connections)
