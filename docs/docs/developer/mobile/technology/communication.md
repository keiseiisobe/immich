# Communication: Real-time vs. Polling

To keep the UI synchronized with the server, Immich uses **WebSockets** instead of periodic API polling.

## WebSockets (socket.io)
-   **Real-time Updates**: When an asset is uploaded from the web or another device, the server pushes an `on_upload_success` event to the mobile app.
-   **Efficiency**: Reduces server load and saves mobile battery by avoiding constant HTTP "check-in" requests.
-   **State Consistency**: Immediate feedback for actions like trashing or archiving an asset on the server.

## When Polling is Used
Polling is only used in specialized, low-priority scenarios, such as log flushing or checking for app updates, typically using a self-scheduling `Timer` pattern to minimize overhead.

## Why WebSockets instead of BaaS (e.g., Firestore)?
Immich chooses WebSockets over managed services like Firebase/Firestore to align with its core mission:
-   **Self-Hosting & Sovereignty**: Firestore is a proprietary Google service. WebSockets run on the user's own hardware, ensuring full control and independence.
-   **LAN & Offline Support**: WebSockets work over local networks without requiring an active internet connection to a public cloud.
-   **Data Privacy**: Keeps all library activity and metadata encrypted between the client and the user's server, avoiding third-party data collection.
-   **Zero Operational Cost**: Avoids the "per-read/write" costs of BaaS providers, which would be significant for users with large photo libraries.
