# 304: Ideator/Watcher

> **Watcher:** Imagine a lookout who keeps an eye on a storage locker (`Storage`). This lookout, called a `Watcher`, is a special kind of helper that starts new tasks. When something changes in the locker (like a new version of an `Idea` is saved), the `Watcher` springs into action and kicks off a new job, usually by giving a `Plan` the new information to work with.
>
> — [Glossary](./000_glossary.md)

> Sidenote:
> - This depends on:
>   - [301: Ideator/Storage](./301_ideator_storage.md)
>   - [012: Agent/Plan](./012_agent_plan.md)

## 1. Introduction

This document explains the **Watcher Protocol**, which is a key part of building workflows that can react to events as they happen. A `Watcher` is like a permanent guard that watches a storage area (`Storage`) for any changes. When it sees one, it triggers a new process in response.

This is the main way we connect parts of our system that remember things with parts that don't, and how we handle big jobs that take a long time to finish.

## 2. The Watcher as a "Source" Helper

While the `Storage` service is like a destination where a task ends (a "sink"), a `Watcher` is the opposite: it's a starting point (a "source"). Its job isn't to finish a process, but to **start a new one**.

Here’s how it usually works:

1.  An `Idea` is saved into `Storage`, which completes the first job.
2.  The `Storage` service sends out a notification, like an alarm, telling everyone who's listening that something has changed.
3.  A `Watcher`, which signed up for these notifications, receives the alarm.
4.  The `Watcher` then starts a completely new, separate job. Usually, this means calling a `Plan` (a set of instructions) and giving it the new `Idea` to work on, kicking off a whole new workflow.

## 3. Handling Jobs That Take a Long Time

The `Watcher` is our secret weapon for managing long, complicated jobs without causing traffic jams.

Imagine a `Plan` has a step that could take hours or even days, like waiting for a person to approve something or chewing through a huge amount of data. A normal `Plan` can't just pause and wait that long.

Instead, the `Plan` can give the long-running task to someone else to handle and then finish its own job. When that other service is finally done, it saves its result back into `Storage`. A `Watcher`, set up to listen for that specific result, sees it and triggers a *new* `Plan` to pick up where the last one left off and continue the workflow.

This method lets us build very strong and flexible processes that aren't limited by the computer's memory or how long a single program can run. It breaks a big, long process down into a series of smaller, separate tasks that are triggered by events.