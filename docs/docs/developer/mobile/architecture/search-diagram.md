# Mobile Search & Discovery Diagram

This diagram visualizes the search functionality, including filters, pagination, and machine-learning-powered discovery.

## Search Class Diagram

```mermaid
classDiagram
    %% Layers
    namespace Presentation {
        class SearchPage
        class SearchFilterWidget
        class PaginatedSearchNotifier
        class SearchResult
    }

    namespace Domain {
        class SearchService
        class SearchFilter
    }

    namespace Infrastructure {
        class SearchRepository
        class PersonApiRepository
    }

    %% Relationships
    SearchPage ..> PaginatedSearchNotifier : ref.watch
    SearchFilterWidget ..> PaginatedSearchNotifier : updates filter
    
    PaginatedSearchNotifier --> SearchService : calls
    PaginatedSearchNotifier ..> SearchResult : manages UI state
    
    SearchService --> SearchRepository : executes search
    SearchService --> PersonApiRepository : fetches faces
    
    SearchService ..> SearchFilter : applies
```

## Search Flow
1.  **Input**: User enters text or selects a face in `SearchFilterWidget`.
2.  **ViewModel**: `PaginatedSearchNotifier` resets the page count and updates the `SearchFilter`.
3.  **Orchestration**: `SearchService` sends the request to the `SearchRepository`.
4.  **Result**: The `SearchResult` (containing assets and pagination metadata) is returned to the notifier, which updates the UI.
