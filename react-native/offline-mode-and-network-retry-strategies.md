# React Native Offline Mode and Network Retry Strategies

A practical design and troubleshooting guide for unreliable connectivity, cached data, queued writes, retries, and synchronization conflicts in mobile apps.

## Connectivity is not binary

A device can be connected to Wi-Fi while still unable to reach the API because of captive portals, DNS, VPN, TLS, firewall, or server problems.

Use connectivity state as a useful signal, but treat the actual request result as the source of truth for whether an operation succeeded.

## Classify application data

Decide explicitly which data is:

- Read-only and safe to cache.
- User-created and must survive process termination.
- Sensitive and requires protected storage.
- Time-sensitive and should expire.
- Safe to overwrite.
- Conflict-prone and requires server resolution.
- Too important to queue without user confirmation, such as payments.

Offline behavior should be a product decision, not an automatic retry wrapper around every request.

## Design readable offline states

The interface should distinguish:

- Showing current server data.
- Showing cached data that may be stale.
- Saving locally and waiting to sync.
- Syncing now.
- Sync failed and needs attention.
- Conflict requires user or server resolution.

Keep the user's content visible when possible, and provide a clear retry action rather than an endless spinner.

## Choose what to retry

| Result | Typical retry decision |
|---|---|
| Offline, DNS, connection reset | Retry later with a limit |
| Timeout before confirmed response | Retry safe reads; mutations require idempotency |
| `400` validation failure | Do not retry unchanged request |
| `401` unauthenticated | Refresh credentials once, then require sign-in |
| `403` forbidden | Do not retry until authorization changes |
| `404` | Usually do not retry unless eventual creation is expected |
| `408` | Retry when operation is safe and policy permits |
| `409` conflict | Resolve version/data conflict before retrying |
| `429` rate limited | Respect `Retry-After` and back off |
| `500`, `502`, `503`, `504` | Retry a bounded number of times with backoff |

Do not retry all errors immediately. Retry storms make outages worse and waste battery and mobile data.

## Use bounded exponential backoff with jitter

Conceptual delay:

```text
delay = min(maxDelay, baseDelay × 2^attempt) + randomJitter
```

Every retry policy should define:

- Maximum attempts or total elapsed time.
- Which HTTP methods and errors are retryable.
- Backoff range and jitter.
- Cancellation when the user leaves or signs out.
- Behavior when the app moves to the background.
- Logging that excludes credentials and personal data.

## Make mutations idempotent

A timeout does not prove the server failed to process a request. Retrying a purchase, order, message, or form submission can create duplicates.

For important mutations:

1. Generate a stable operation ID before the first attempt.
2. Persist it with the queued operation.
3. Send it as the server-supported idempotency key or operation identifier.
4. Reuse it for retries of the same logical action.
5. Let the server return the original result for duplicates.

Idempotency requires server support; a client-generated key alone cannot guarantee safety.

## Build a persistent write queue

A robust queued operation commonly stores:

- Stable operation ID.
- Operation type and validated payload.
- Creation and last-attempt time.
- Attempt count and next eligible retry time.
- User/account and environment ownership.
- Dependency on earlier operations.
- Current state and last safe error category.

> [!WARNING]
> Queued payloads may contain private user data. Store only necessary fields, use platform-appropriate protected storage, redact logs, and delete data on logout or account removal according to product policy.

Do not store access tokens inside every queued record. Resolve current authorization securely when the operation runs.

## Synchronize in a safe order

```text
App starts or connectivity may be restored
        ↓
Confirm authenticated account and environment
        ↓
Acquire one queue-processing lock
        ↓
Load next eligible operation
        ↓
Send with stable operation ID
        ↓
Persist success before removing item
        ↓
Back off, pause, or surface conflict on failure
```

Prevent multiple screens, listeners, or app lifecycle events from draining the same queue concurrently.

## Resolve conflicts intentionally

Common strategies include:

- Server wins.
- Client wins for explicitly owned fields.
- Last write wins using trusted server versions/timestamps.
- Field-level merge.
- User reviews both versions.
- Domain-specific rule, such as additive counters or append-only events.

Do not use device clock time as the only source of truth; device clocks can be wrong or manipulated.

## Background execution limitations

Mobile operating systems may suspend or terminate an app and do not guarantee immediate background execution.

- Persist work before starting it.
- Use supported native background schedulers for deferred work.
- Keep background tasks short and resumable.
- Do not require a JavaScript timer to remain alive indefinitely.
- Sync again when the app becomes active.
- Communicate when the user must keep the app open for a critical transfer.

## Test real failure scenarios

Test on physical Android and iOS devices:

1. Launch with no connection and cached data.
2. Lose connectivity during a read.
3. Lose connectivity during a write after the server may have responded.
4. Move between Wi-Fi and mobile data.
5. Restore connectivity with several queued operations.
6. Sign out while work is queued.
7. Expire credentials before retry.
8. Receive `429`, conflict, and server errors.
9. Kill and relaunch the app during synchronization.
10. Modify the same record from two devices while offline.

Never run failure tests against real payment, messaging, or production user accounts unless the environment and test plan explicitly permit it.

## Observability without sensitive data

Record:

- Operation category and irreversible correlation ID.
- Attempt count and delay.
- Connectivity signal and application state.
- Sanitized error category and HTTP status.
- Queue age and size.
- Success, cancellation, conflict, or permanent failure.

Do not log tokens, cookies, full payloads, private URLs, precise locations, or personal records.

## Recommended workflow

```text
Classify data and operations
        ↓
Define cached and offline UX
        ↓
Define retryable errors and limits
        ↓
Add idempotency for mutations
        ↓
Persist and secure queued work
        ↓
Handle auth, conflicts, and logout
        ↓
Test interruption and process death
        ↓
Monitor bounded, sanitized outcomes
```

## Related guides

- [React Native Network and API Debugging](network-and-api-debugging.md)
- [React Native App Performance and Memory Debugging](app-performance-and-memory-debugging.md)
- [React Native Release Build and Signing Troubleshooting](release-build-and-signing-troubleshooting.md)

## Official references

- [React Native Networking](https://reactnative.dev/docs/network)
- [Android: Build an Offline-First App](https://developer.android.com/topic/architecture/data-layer/offline-first)
- [Android WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager)
- [Apple: Background Tasks](https://developer.apple.com/documentation/backgroundtasks)
