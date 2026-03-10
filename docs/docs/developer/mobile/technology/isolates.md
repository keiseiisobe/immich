# Background Isolate Safety

To ensure a smooth UI experience, heavy tasks like **Image Hashing** and **Asset Syncing** are offloaded to separate **Flutter Isolates**. 

## Initialization Sequence
1.  **First Launch**: The database is physically created on disk in the `main()` function via `await Bootstrap.initDB()` before the UI even appears.
2.  **Isolate Spawning**: When a background task is triggered (e.g., a sync job), a new Isolate is spawned with a fresh memory heap.
3.  **Local Re-Initialization**: Because Isolates cannot share memory handles, the newly spawned isolate must call `await Bootstrap.initDB()` at its own "startup" to establish its own native connection to the existing database files.

## How Isolates stay secure
-   **No Shared Memory**: Isolates do not share memory with the main UI thread. They cannot access the `AuthNotifier`'s in-memory state.
-   **Token Re-fetching**: Instead of receiving a token from the UI, the isolate fetches the `accessToken` directly from its own connection to the persistent **Store** (Local DB).
-   **Server Enforcement**: Every background API request is validated by the server. If the token has expired, the server returns a `401 Unauthorized`, and the background task terminates safely, preventing "ghost" operations.
