# 301: Ideator/Storage

**System: Storage** is a core utility service within the Idea ecosystem. The Storage service functions as a specialized **Idea Transformer** whose primary purpose is to provide a persistent, content-addressable repository for `Ideas`. It is the foundational layer for memory and recall, allowing `Ideas` to be saved and versioned for later use by other services, such as [System: Resolver](./302_ideator_resolver.md).

## 2. Core Concept: Storage as an Idea Transformer

The Storage service adheres to the `Idea Transformer` pattern, accepting an `Idea` as input and returning an `Idea` as output.

- **Input**: A valid `Idea` triplet (`schema`, `solution`, `context`).
- **Process**: The service receives the `Idea`, assigns it a unique, addressable identifier (which incorporates versioning), and persists it to a durable backend (e.g., a PostgreSQL database).
- **Output**: The service returns the original `Idea` as an acknowledgment of successful storage. This output may be augmented with metadata, such as the assigned identifier and version, confirming its persistence.

## 3. Storage as a Transactional Boundary

A critical role of the Storage service is to act as a **transactional boundary**. When an external actor (e.g., a user or another service) submits an `Idea` and receives a successful acknowledgment, that initial transaction is considered complete.

This decoupling is fundamental to the architecture's resilience and scalability. It allows the system to build complex, event-driven workflows. Once an `Idea` is committed to `Storage`, other services (like a `Watcher` or `Subscriber`) can react to this change, initiating new, independent transactions to continue a process. This prevents the need for long-running, monolithic transactions and creates a clear, auditable hand-off between different phases of a workflow.

## 4. Key Characteristics

### 4.1. Black Box Persistence

The Storage service is designed as a managed "black box." Users send `Ideas` to be persisted without control over the underlying storage mechanism (e.g., database type, sharding) or data retention policies. The service guarantees that an `Idea`, once accepted, will be stored and made available for resolution. The specific implementation details are left to the service provider.

### 4.2. Versioning and Immutability

The system is built on the principle of immutability. When an existing `Idea` is "updated," the storage service does not modify the original record. Instead, it creates and stores a new version of the `Idea`, preserving a complete, unbroken chain of its history. This ensures that any reference to a specific version of an `Idea` will always resolve to the exact same content. References can point to a specific version or to a dynamic branch (e.g., `latest`).

### 4.3. Content-Addressable

While the exact addressing scheme is an implementation detail, the principle is that every stored `Idea` is uniquely identifiable and retrievable via its identifier and version. This allows for a robust and decentralized system where `Ideas` can be referenced with precision.

## 5. Public Interface

The public interface for the Storage system is simple and write-oriented, aligning with its focused role as a "sink" for `Ideas`.

- **Primary Interaction**: The sole public interaction is the submission of an `Idea`.

- **Decoupling from Retrieval**: To maintain a clean separation of concerns, the Storage service does not expose a public, general-purpose retrieval or query API. The responsibility of fetching and resolving `Ideas` is explicitly delegated to the **Resolver** service. This keeps each service focused on a single task: Storage handles persistence, and the Resolver handles retrieval and composition.
