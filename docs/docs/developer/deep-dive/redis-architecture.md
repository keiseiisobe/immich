# Redis Architecture & Communication

This document provides a deep dive into why Immich uses a separate Redis (Valkey) container and how it communicates with the main server.

## 1. Why a Separate Container?

In the default Immich setup, Redis (specifically the `valkey` image) runs as a dedicated service in `docker-compose.yml`. This architecture provides several key benefits:

*   **Job Queue Management (BullMQ):** Immich performs resource-intensive tasks (thumbnail generation, video transcoding, machine learning) asynchronously. Redis acts as the "shared brain" for these tasks, allowing multiple worker containers to pick up jobs from a central queue.
*   **Decoupling:** By keeping Redis separate from the main application server, the system is more resilient. If the main server restarts for an update, the Redis job queue remains intact.
*   **Scalability:** In large-scale deployments, you can move the Redis container to a dedicated high-memory server without changing any application code.

## 2. Communication Protocol: RESP vs. HTTP

A common question is whether `immich-server` communicates with Redis via HTTP. The answer is **no**.

Immich uses **RESP (Redis Serialization Protocol)**, which runs directly over TCP. Unlike HTTP, which is a text-based protocol with large headers (often 200–500 bytes for a tiny payload), RESP is a binary-safe protocol designed for extreme performance.

| Feature | Redis (RESP) | HTTP (1.1/2) |
| :--- | :--- | :--- |
| **Overhead** | Minimal (1-byte type prefix + length) | Large (hundreds of bytes for headers) |
| **Parsing** | O(1) per field | Complex (requires scanning for delimiters) |
| **Speed** | 100,000+ operations/sec | Significantly slower for the same hardware |

## 3. Network Latency & Performance

Does the network overhead of Docker containers slow down the cache?

Even though they are in separate containers, they communicate via a **virtual bridge network** on the same physical host.
*   **Latency:** Round-trip time (RTT) between containers on the same host is typically **0.05ms to 0.15ms**.
*   **Perception:** Humans perceive "instant" as anything under 100ms. A 0.1ms overhead is **1,000 times faster** than human perception and is negligible compared to the 20ms+ of a traditional database query.

## 4. Architectural Trade-offs

| Setup | Speed | Scaling | Resilience |
| :--- | :--- | :--- | :--- |
| **In-App RAM (Local)** | Nanoseconds | Impossible to share | Cache dies with App |
| **Redis (Separate)** | Microseconds | Easy (Global shared state) | Cache survives App restart |

Immich chooses the **Separate Container** approach because it provides the best balance of reliability and performance for self-hosted users while allowing the system to scale for high-demand environments.
