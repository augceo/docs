# Chapter 0: System Architecture

Refinery employs a robust, distributed architecture to ensure both durability and transactional integrity for the Vibe ecosystem. The design separates the concerns of immutable data storage from the management of long-running, stateful processes.

## Core Components

### PostgreSQL for Transactional Integrity

The immutable source of truth in the system is the Vibe ledger itself, stored in a PostgreSQL database. All Vibe creation and modification are handled as atomic transactions to ensure data consistency and safety.

This transactional integrity guarantees that the graph of Vibes and their relationships remains consistent, even in a highly concurrent environment. Key operations, like the `refine` primitive, are built on this solid foundation. The database provides the durability and queryability needed to manage the Vibe ledger as a permanent, auditable record of all interactions.

### Temporal for Durable Processes

For long-running, stateful operations—specifically, the execution of `Process Vibes`—Refinery integrates with Temporal. Temporal workflows manage the ephemeral state and complex orchestration logic of a process, ensuring its durability and reliability even in the face of failures.

When a `Process Vibe` is executed, a Temporal workflow is initiated to carry out the defined steps. While Temporal handles the durable execution, the final resulting Vibe (the `solution` of the process) is committed transactionally to the PostgreSQL database via the `refine` primitive.

This separation of concerns is critical:

- **PostgreSQL** is the system of record for the "what" – the immutable facts and `solutions` represented by Vibes.
- **Temporal** is the system of record for the "how" – the durable, stateful execution of a process over time.

This dual-system approach allows Refinery to handle complex, long-running tasks without compromising the integrity of the core Vibe ledger, combining the best of transactional data management and durable workflow orchestration.
