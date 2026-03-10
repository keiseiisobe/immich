# Mobile Class Relationship Diagrams

The Immich mobile architecture is modular and layered. To better understand how the different components interact, we have separated the class diagrams by functional area:

### 1. [Authentication & Session Management](./auth-diagram.md)
Visualizes how the app handles server validation, user login, and persistent session state.

### 2. [Media Assets & Albums](./assets-diagram.md)
Visualizes the core of the app: managing media, synchronizing local/remote files, and album organization.

### 3. [Search & Discovery](./search-diagram.md)
Visualizes the search engine, including filters, pagination, and facial recognition integration.

---

## Architectural Principles (Summary)

Regardless of the function, all modules follow these core principles:

-   **MVVM Pattern**: Riverpod Notifiers act as the ViewModels, isolating UI state from business logic.
-   **Granular Repositories**: Repositories are typically restricted to a single data source (API or DB).
-   **Service Orchestration**: Domain Services manage the complex rules of when to use which repository.
-   **Immutable State**: Application state is managed via immutable data classes and `copyWith` patterns to ensure predictable UI updates.
