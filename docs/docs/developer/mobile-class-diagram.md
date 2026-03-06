# Mobile Class Relationship Diagram

This diagram visualizes the primary classes in the Immich Flutter application and how they interact across different architectural layers.

## High-Level Domain Diagram

```mermaid
classDiagram
    %% Layers Definitions
    namespace Presentation_Layer {
        class LoginPage
        class ProfilePage
        class SearchPage
        class AuthNotifier
        class UserNotifier
        class AssetNotifier
        class PaginatedSearchNotifier
    }

    namespace Domain_Layer {
        class AuthService
        class UserService
        class AssetService
        class SearchService
        class UserDto
        class Asset
        class AuthState
    }

    namespace Infrastructure_Layer {
        class AuthApiRepository
        class UserApiRepository
        class IsarUserRepository
        class AssetRepository
        class AssetApiRepository
        class SearchRepository
        class ApiService
    }

    %% UI to ViewModel (Notifier) Relationships
    LoginPage ..> AuthNotifier : ref.read/watch
    ProfilePage ..> UserNotifier : ref.watch
    SearchPage ..> PaginatedSearchNotifier : ref.watch

    %% ViewModel to Domain Service Relationships
    AuthNotifier --> AuthService : calls
    AuthNotifier --> UserService : calls
    UserNotifier --> UserService : calls
    AssetNotifier --> AssetService : calls
    AssetNotifier --> UserService : calls
    PaginatedSearchNotifier --> SearchService : calls

    %% Service to Repository Relationships
    AuthService --> AuthApiRepository : uses
    AuthService --> ApiService : uses
    UserService --> UserApiRepository : uses
    UserService --> IsarUserRepository : uses
    AssetService --> AssetRepository : uses
    AssetService --> AssetApiRepository : uses
    SearchService --> SearchRepository : uses

    %% Data Flow (Models)
    AuthApiRepository ..> UserDto : returns
    UserApiRepository ..> UserDto : returns
    IsarUserRepository ..> UserDto : returns
    AssetRepository ..> Asset : returns
    UserService ..> UserDto : orchestrates
    AuthNotifier ..> AuthState : manages

    %% Styling
    style Presentation_Layer fill:#f5f5f5,stroke:#333,stroke-width:2px
    style Domain_Layer fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    style Infrastructure_Layer fill:#fff3e0,stroke:#e65100,stroke-width:2px
```

## Layer Responsibilities

### 1. Presentation Layer (Grey)
- **Pages/Widgets**: The visual components. They are "passive" and only react to state changes in the Notifiers.
- **Notifiers (ViewModels)**: Maintain the UI-specific state (e.g., `loading`, `error`, `data`). They bridge the UI and the Business logic.

### 2. Domain Layer (Blue)
- **Services**: The "Brains" of the app. They contain the business rules (e.g., "when a user logs in, fetch their profile and sync the local DB").
- **Models (DTOs)**: Simple data containers like `UserDto` or `Asset` that travel between layers.

### 3. Infrastructure Layer (Orange)
- **Repositories**: The "Workers." They handle the technical details of communication.
    - **ApiRepositories**: Handle HTTP requests to the Immich Server.
    - **DatabaseRepositories**: Handle local persistence (Isar/Drift).
- **ApiService**: A global utility that manages authentication tokens, base URLs, and HTTP headers for all network requests.
