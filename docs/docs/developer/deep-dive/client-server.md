# Client-Server & Distributed Architecture

Immich follows a classic **Client-Server architecture** while internally operating as a **Distributed System**. This design ensures that the system is scalable, modular, and easy to maintain.

## 1. Macro-Level: The Client-Server Model

At its highest level, Immich fits the standard definition of a client-server architecture:

*   **The Clients:** These are the "requestors" of information. In Immich, this includes:
    *   **Mobile App:** (Android/iOS)
    *   **Web App:** (The browser interface)
    *   **CLI:** (Command-line tool for bulk uploads)
*   **The Server:** This is the "provider" of services. It manages the storage of your photos, performs metadata extraction, runs machine learning models, and handles authentication.

The clients are "thin," meaning they do not store your library locally or perform heavy processing. Instead, they send requests to the server and display the results.

## 2. Internal-Level: A Distributed System

While Immich appears as a single "server" to the user, it is internally a **Distributed Application** composed of several specialized services (containers) communicating over a virtual network:

1.  **`immich-server`**: Acts as the main API gateway for clients but serves as a client when communicating with the database or cache.
2.  **`immich-machine-learning`**: A dedicated service for AI-related tasks. It only "speaks" to the main server, not directly to the end-user clients.
3.  **`immich-microservices`**: Internal worker processes that pull jobs from Redis to perform background tasks like video transcoding.
4.  **Postgres & Redis**: These are specialized data and state "servers" that provide persistence and message queuing to the rest of the stack.

## 3. Why Not Other Architectures?

To understand Immich's design, it's helpful to compare it with other common patterns:

*   **Not Peer-to-Peer (P2P):** In a P2P system (like BitTorrent), your phone would talk directly to your computer. In Immich, all data and logic are centralized on the server to ensure consistency and availability across all your devices.
*   **Not a Monolith:** A monolith is a single, massive program. By splitting Immich into multiple containers, we gain **isolation**. If the machine learning service crashes, the main web interface remains functional.

## 4. Summary of the Relationship

| Layer | Component | Role | Protocol |
| :--- | :--- | :--- | :--- |
| **External** | Mobile/Web -> Server | Client-Server | HTTP (REST API) |
| **Internal** | Server -> Redis | Client-Server (Distributed) | RESP (TCP) |
| **Internal** | Server -> Machine Learning | Client-Server (Distributed) | HTTP (Internal) |
| **Internal** | Server -> Postgres | Client-Server (Distributed) | PostgreSQL Wire Protocol |

This "Distributed Client-Server" model allows Immich to handle thousands of photos and multiple users while remaining fast and reliable on a single home server or a large-scale cloud cluster.
