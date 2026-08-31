# React Native App Performance and Memory Debugging

A measurement-first guide for diagnosing slow startup, dropped frames, frozen interactions, excessive memory growth, and release-only performance problems.

## Define one reproducible symptom

Avoid starting with “the app is slow.” Record:

- Device model, OS version, and available storage.
- Debug or release build and source commit.
- Exact screen and interaction.
- Time until the problem appears.
- Whether CPU, memory, network, disk, image loading, or animation is involved.
- Whether the problem affects JavaScript, native UI, or both.

Test performance in a release build on representative physical devices. Development warnings, debugging hooks, and logging can significantly change results.

## Establish a repeatable scenario

Example:

```text
Cold launch
→ Sign in with test account
→ Open product list
→ Scroll from item 1 to item 100 at a steady speed
→ Open and close one detail screen 20 times
→ Record startup time, dropped frames, and memory
```

Change one variable at a time and compare against the same baseline.

## Use React Native DevTools

With Metro focused, press:

```text
j
```

In supported React Native versions, use the **Performance** panel to record the problematic interaction. Look for:

- Long JavaScript tasks.
- Repeated or unnecessary component renders.
- Network events delaying user-visible work.
- Expensive work during startup or navigation.
- A pattern that grows each time the scenario repeats.

Use the Components panel and React profiling tools when the delay is tied to rendering. Use Android Studio or Xcode tools when the bottleneck is native.

## Check Android process memory

```bash
# Show a memory summary for one running Android application.
adb shell dumpsys meminfo com.example.app
```

Capture the same summary before and after a controlled scenario. Memory increasing once can be normal caching; memory that keeps growing after repeated navigation and idle periods deserves investigation.

Useful native tools include Android Studio CPU, Memory, Network, and Energy profilers. Profile the same build type and scenario used to reproduce the problem.

## Check iOS performance and memory

Use Xcode's Instruments through:

```text
Xcode → Product → Profile
```

Useful templates include:

- Time Profiler for CPU hotspots.
- Allocations for allocation growth and object lifetimes.
- Leaks for certain native-memory leaks.
- Network and system instruments for I/O behavior.

Keep the matching archive and dSYM files when investigating a distributed release build.

## Common JavaScript causes

- Heavy synchronous work on the JavaScript thread.
- Parsing or transforming large responses during an interaction.
- Re-rendering large component trees because references change unnecessarily.
- Timers, listeners, subscriptions, or observers not cleaned up.
- Unbounded arrays, caches, logs, or navigation history.
- Large modules loaded during startup when they are not immediately needed.
- Excessive development logging left in release paths.

Prefer evidence from a profile before adding memoization or rewriting components.

## Large list problems

Check:

- Stable, unique keys.
- Bounded initial rendering and window sizes.
- Lightweight item components.
- Images sized close to their rendered dimensions.
- Pagination instead of keeping an unlimited dataset in memory.
- Avoiding new inline objects/functions only where profiling shows meaningful rerenders.
- Correct item layout estimates when the list API supports them.

Test realistic data volumes; a list of ten test items will not reveal production behavior.

## Image and media memory

Large decoded images can consume far more memory than their compressed file size.

- Request appropriately resized images from the server or image pipeline.
- Avoid retaining full-resolution camera photos for thumbnail views.
- Cancel obsolete loads when screens change.
- Bound memory and disk caches.
- Test repeated open/close flows and low-memory devices.

Do not solve memory pressure by clearing every cache on every launch; that can increase network, CPU, battery use, and startup time.

## Find lifecycle leaks

For every effect, subscription, listener, timer, or native observer, verify cleanup:

```tsx
useEffect(() => {
  const subscription = AppState.addEventListener('change', handleAppState);
  const timer = setInterval(refreshVisibleData, 60_000);

  return () => {
    subscription.remove();
    clearInterval(timer);
  };
}, []);
```

Also check callbacks that capture large objects, pending promises that update abandoned screens, and native modules that retain React contexts or view references.

## Startup performance

Classify startup before optimizing:

- Native process and framework initialization.
- JavaScript bundle loading.
- First React render.
- Authentication/session restoration.
- Initial API and local-database work.
- First meaningful screen becoming interactive.

Delay nonessential work and large components until needed. Avoid moving all initialization later if the first user action still blocks on it.

## Common diagnostic mistakes

- Profiling only a simulator or high-end development phone.
- Comparing debug and release numbers.
- Changing several optimizations before measuring again.
- Treating one memory snapshot as proof of a leak.
- Force-closing background apps to make benchmark results look better.
- Ignoring network and server latency while optimizing rendering.
- Publishing profiles or logs containing user data.

## Recommended workflow

```text
Define one symptom and device
        ↓
Reproduce in release build
        ↓
Record baseline
        ↓
Profile JavaScript and native layers
        ↓
Identify the largest measured bottleneck
        ↓
Apply one focused change
        ↓
Repeat identical scenario
        ↓
Check for regressions on Android and iOS
```

## Related guides

- [React Native Debugging Guide](react-native-debugging-guide.md)
- [React Native Network and API Debugging](network-and-api-debugging.md)
- [React Native Release Build and Signing Troubleshooting](release-build-and-signing-troubleshooting.md)

## Official references

- [React Native DevTools](https://reactnative.dev/docs/react-native-devtools)
- [React Native: Optimizing JavaScript Loading](https://reactnative.dev/docs/optimizing-javascript-loading)
- [Android Studio Profilers](https://developer.android.com/studio/profile)
- [Apple Instruments](https://developer.apple.com/tutorials/instruments)
