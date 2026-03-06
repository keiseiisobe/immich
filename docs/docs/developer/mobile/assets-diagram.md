# Mobile Assets & Albums Diagram

This diagram visualizes how the application manages media assets, local/remote albums, and the synchronization between them.

## Asset Management Class Diagram

```mermaid
classDiagram
    %% Layers
    namespace Presentation {
        class HomePage
        class AlbumViewer
        class AssetNotifier
        class AlbumNotifier
    }

    namespace Domain {
        class AssetService
        class AlbumService
        class SyncService
        class Asset
        class Album
    }

    namespace Infrastructure {
        class AssetRepository
        class AssetApiRepository
        class AlbumRepository
        class AlbumApiRepository
        class Isar
        class Store
    }

    namespace Background_Process {
        class SyncIsolate
        class HashIsolate
    }

    %% Relationships
    HomePage ..> AssetNotifier : ref.watch
    AlbumViewer ..> AlbumNotifier : ref.watch
    
    AssetNotifier --> AssetService : calls
    AlbumNotifier --> AlbumService : calls
    
    AssetService --> SyncService : coordinates
    AssetService --> AssetRepository : local DB
    AssetService --> AssetApiRepository : remote API
    
    AlbumService --> AlbumRepository : local DB
    AlbumService --> AlbumApiRepository : remote API
    
    AssetRepository ..> Isar : uses
    AlbumRepository ..> Isar : uses

    SyncIsolate ..> Isar : initializes own instance
    SyncIsolate ..> Store : fetches token
    SyncIsolate --> AssetApiRepository : POST /assets
    SyncIsolate --> UserApiRepository : GET /users
    
    HashIsolate ..> Isar : saves local hashes
```

## Functional Flow
-   **Synchronization**: `SyncService` handles the complex logic of matching local device files with remote server assets.
-   **Background Safety**: `SyncIsolate` and `HashIsolate` run in separate memory spaces. They re-fetch the `accessToken` from the `Store` to ensure requests to the `AssetApiRepository` remain authenticated.
-   **Reactivity**: `AssetRepository` often returns `Streams` from Isar, allowing the UI to update automatically when the background sync adds new photos.
-   **Caching**: `AssetService` manages the logic of when to show a low-res thumbnail vs. fetching high-res data.

## Background Isolate APIs

Background tasks are "mini-apps" that stand on their own to perform data-heavy tasks. They primarily interact with:

1.  **Asset Management (`/assets`)**: 
    - `POST /assets`: Uploads the physical file and metadata.
    - `POST /assets/bulk-upload-check`: Fast deduplication check before uploading.
2.  **User Sync (`/users`)**: 
    - `GET /users`: Syncs profile and partner sharing settings to the local database.
3.  **Connectivity (`/server-info`)**:
    - `GET /server-info/ping`: Verifies server availability before initiating sync.
