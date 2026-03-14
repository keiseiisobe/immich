# Mobile Data Patterns: Models vs. Entities

In the Immich mobile app, we distinguish between **Models (DTOs)** and **Entities**. This separation follows clean architecture principles, ensuring that the local database schema (Persistence) is decoupled from the API and UI logic (Domain).

## 1. Entities (`.entity.dart`)

**Purpose:** Local Persistence (The "Storage" Layer)  
**Location:** `mobile/lib/entities/` or `mobile/lib/infrastructure/entities/`

Entities define the physical schema of the local databases on the mobile device (Isar or Drift). They are strictly tied to the requirements of the database engine.

*   **Database Integration:** Decorated with annotations like `@Collection` (Isar) or inherit from `Table` (Drift).
*   **Primary Key:** Must include a database-compatible ID (e.g., `Id id = Isar.autoIncrement`).
*   **Constraints:** Must use data types that the database can serialize directly.
*   **Lifecycle:** Entities represent the "Source of Truth" when the app is offline.

```dart
// Example: User Entity (Isar)
@Collection(inheritance: false)
class User {
  Id get isarId => fastHash(id); // Local database ID
  @Index(unique: true)
  final String id; // Server UUID
  final String email;
  final String name;
  // ...
}
```

## 2. Models / DTOs (`.model.dart`)

**Purpose:** Data Transfer & Business Logic (The "Communication" Layer)  
**Location:** `mobile/lib/domain/models/`

Models (often suffixed with `Dto`) represent data as it exists in the REST API or within the application's domain logic.

*   **Flexibility:** "Pure Dart" classes without database-specific annotations.
*   **API Alignment:** Automatically generated or manually mapped to match the OpenAPI specification of the Immich server.
*   **UI Helpers:** Often contain logic for the view layer (e.g., converting an enum to a Flutter `Color`).
*   **Lifecycle:** Models are transient and exist in RAM during API calls or UI rendering.

```dart
// Example: UserDto (Model)
class UserDto {
  final String id;
  final String email;
  final String name;
  final AvatarColor avatarColor;

  // Helper for the UI layer
  Color get color => avatarColor.toColor();
}
```

## 3. The Data Flow Pattern

Data typically flows through the following lifecycle in the mobile app:

1.  **API Response:** The server sends a JSON object.
2.  **Model (DTO):** The app parses the JSON into a **Model** (e.g., `UserDto`).
3.  **Entity Mapping:** The `Repository` converts the **Model** into an **Entity** (e.g., `User`).
4.  **Persistence:** The **Entity** is saved to the local database (Isar/Drift).
5.  **UI Retrieval:** The UI either reads the **Entity** directly (using it as a model) or the repository converts the **Entity** back into a **Model** for display.

## 4. Why Distinguish Between Them?

| Benefit | Description |
| :--- | :--- |
| **Decoupling** | API changes (Models) don't immediately force a database migration (Entities). |
| **Performance** | Entities are optimized for fast database queries; Models are optimized for complex UI logic. |
| **Offline-First** | By saving data as Entities, the app can display a full UI immediately on startup without waiting for a network response. |
| **Type Safety** | Models handle the "nullability" and "uncertainty" of API data, while Entities ensure the database always has valid, structured data. |

---

*Summary: Use **Entities** for anything that touches the local disk and **Models/DTOs** for anything that touches the network or the UI logic.*
