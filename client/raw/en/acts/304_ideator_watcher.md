# 304: Ideator/Watcher

> **Watcher:** A stateful "source" `Ideator` that subscribes to changes in a `Storage` service. When a change occurs (e.g., a new `Idea` version is saved), it initiates a new transaction, typically by invoking a `Plan` to process the new data.
>
> — [Glossary](./000_glossary.md)

> Sidenote:
>
> - Requires:
>   - [301: Ideator/Storage](./301_ideator_storage.md)
>   - [012: Agent/Plan](./012_agent_plan.md)

## 1. Introduction

The **Watcher Protocol** is a component for building asynchronous, event-driven workflows within the ecosystem. A `Watcher` (also referred to as a Subscriber) is a persistent service that observes a `Storage` provider for changes and triggers new processes in response.

It is the primary mechanism for bridging the gap between stateful and stateless services and for handling long-running tasks that cannot be managed in a simple request-response cycle.

## 2. The Watcher as a "Source" Ideator

In contrast to the `Storage` service, which acts as a transactional "sink," a `Watcher` is a "source." Its role is not to terminate a transaction, but to **initiate a new one**.

The typical workflow is as follows:

1.  An `Idea` is committed to `Storage`, completing the initial transaction.
2.  The `Storage` service emits an event notifying subscribers of the change.
3.  A `Watcher`, subscribed to these events, receives the notification.
4.  The `Watcher` initiates a new, independent transaction. This usually involves invoking a [012: Agent/Plan](./012_agent_plan.md) and passing the new `Idea` as context, kicking off a new workflow.

## 3. Handling Asynchronicity and Long-Running Processes

The `Watcher` is the key to managing long-running, asynchronous operations gracefully.

Consider a `Plan` that involves a step that may take hours or days to complete (e.g., waiting for human input, processing a large dataset). A synchronous `Plan` cannot simply `await` this result.

Instead, the `Plan` can delegate the long-running task to an external service and then terminate. That external service, upon completion, writes its result back to `Storage`. A `Watcher`, configured to listen for this specific result, can then trigger a _new_ `Plan` to continue the workflow.

This pattern allows for resilient and scalable processes that are not constrained by the memory or lifespan of any single runtime. It enables true asynchronicity by breaking down a long process into a series of smaller, event-triggered transactions.
