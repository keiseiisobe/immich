# Data Persistence: The Store

While many Flutter apps use `SharedPreferences` for simple settings, Immich uses a custom **Reactive Key-Value Store** built on top of the local databases (Isar or Drift).

## Why a custom Store?
-   **Reactivity**: Other layers can `watch` a specific key (like `accessToken`) and react instantly when it changes.
-   **Type Safety**: Uses a `StoreKey<T>` system to ensure type consistency at compile-time.
-   **Performance**: Backed by an in-memory cache for synchronous reads, with asynchronous writes to the database.
-   **Background Safety**: Being database-backed makes it more reliable when accessed from background isolates (e.g., during image hashing or syncing).

## Usage Example
```dart
// Writing a value
await Store.put(StoreKey.serverUrl, "https://my-immich-instance.com");

// Synchronous read (from cache)
final url = Store.tryGet(StoreKey.serverUrl);

// Reactive watch (Streams)
Store.watch(StoreKey.accessToken).listen((token) {
  print("Token updated: $token");
});
```
