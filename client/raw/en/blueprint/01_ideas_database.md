# Chapter 1 (Appendix): Vibe Database Structure

This document details the underlying PostgreSQL database structure that stores and manages Vibes, providing the foundation for the system's principles of immutability, content-addressability, and verifiable lineage. The design is centered around two core tables, `vibes` and `refinements`, and a powerful set of functions for dynamically resolving Vibe schemas.

## Core Tables

### The `vibes` Table

The `vibes` table is the heart of the system's storage. Every Vibe, regardless of its type (Record, Role, Process, or Capability), is stored as a row in this table. It is the immutable ledger of every interaction that has ever occurred.

Here are the key columns:

- `ref` (Primary Key): A unique, human-readable text reference for the Vibe, acting as its content-addressable identifier. This is the primary way Vibes are referenced throughout the system.
- `id`, `branches`, `revision`: These columns work together to manage versioning and identity. A Vibe's conceptual identity is its `id`, but it can exist on multiple `branches` (e.g., `main`, `dev/feature-x`) and have multiple `revision` numbers within those branches. This allows for powerful versioning and experimentation workflows.
- `authorRef`: A foreign key to the `ref` of the Vibe that created this one, establishing a direct lineage.
- `prototypeRef`: A reference to another Vibe that was used as a template, enabling creation by example.
- `refinementId`: A reference to the `refinements` table, linking the Vibe to the specific `refine` operation that produced it.
- `context`, `schema`, `solution`: These three `JSONB` columns store the core `{context, schema, solution}` triplet that defines a Vibe. The `schema` is stored directly, but as we'll see, it may contain references to other Vibes that need to be resolved.
- `schema_refs`: A `TEXT[]` column, automatically populated by a trigger, that stores an array of all the `aug:` references found within the `schema`. This provides a quick way to identify Vibes that have schema dependencies.

### The `refinements` Table

The `refinements` table acts as a transactional log for every `refine` operation. It captures the inputs and context of each state change, providing a complete and auditable history.

Key columns include:

- `id`: The primary key for the refinement record.
- `targetRefs`, `instructionRefs`, `budgetRefs`: These text arrays store the `ref`s of the Vibes that were used as inputs to the `refine` call. This explicitly records the dependencies of the operation.
- `authorRef`: The `ref` of the Vibe that initiated the refinement.

Together, these tables create a robust and interconnected graph of all activity in the system, where every Vibe is traceable to its author, its inputs, and the operation that created it.

### The `vibe_refs` Table: The Dependency Graph

To provide a complete and efficient way to query the relationships between Vibes, the system maintains a `vibe_refs` table. This table stores a denormalized, direct graph of all schema dependencies.

When a Vibe is created or updated, a trigger function scans its `schema` for any `aug:` references and populates this table.

Key columns include:

- `from_ref`: The `ref` of the Vibe containing the schema with the dependency.
- `to_ref`: The `ref` of the Vibe being referenced within the schema.
- `type`: The type of reference (e.g., 'schema').
- `branch`, `from_id`, `from_revision`, `to_id`, `to_revision`: These columns capture the specific versioning details of both ends of the dependency link.

This table is a performance optimization that allows the system to quickly answer questions like, "Which Vibes will be affected if we update this specific schema component?" without needing to scan the JSONB content of the entire `vibes` table. It makes the dependency graph an explicit and queryable part of the database.

## Dynamic Schema Resolution and Dependency Graphs

A key innovation in the database design is how Vibe schemas are handled. A Vibe's `schema` is not necessarily a static, self-contained document. It can be a composite structure built from other Vibes, enabling powerful reuse and composition. This is achieved through special `aug:` references and a database function that resolves them just-in-time.

### Composable Schemas with `aug:` References

Within a Vibe's `schema` field, we can embed references to other Vibes using a special URI format, like `aug:/common/schemas/address?1` or `aug:~dev/user-profile`. These references instruct the system to fetch the `schema` from the referenced Vibe and embed it into the current one during a resolution step.

This allows us to create a library of reusable schema components. For example, many different Record Vibes could all reference a canonical `address` schema instead of redefining it each time.

### The `resolveVibeSchema` Function and Reference Resolution

The magic happens in the `resolveVibeSchema` PostgreSQL function and the surrounding application logic. When a client submits a Vibe, its `schema` can contain unresolved references that need to be processed. The system is designed to handle several cases gracefully.

Inside a schema, a reference to another Vibe can be represented by an object containing specific keys:

- `$ref`: An `aug:` URI string pointing to a conceptual Vibe (e.g., `aug:/schemas/user`). This indicates that the system should find the **latest appropriate version** of that Vibe based on the current context (e.g., the current branch and timestamp).
- `$refOriginal`: An `aug:` URI string pointing to a **specific, versioned Vibe** (e.g., `aug:/schemas/user?1`). This is used when the client has already resolved the reference to a specific version and wants to use that exact one.
- `$refResolved`: This property is added by the system during the resolution process. It contains the canonical, fully-resolved `ref` of the Vibe version that was used to populate the schema, ensuring a permanent record of the exact dependency.

The resolution logic, typically in a database trigger or application service, works as follows:

1.  It traverses the incoming `schema` looking for objects that contain a `$ref` or `$refOriginal` key, but do **not** already have a `$refResolved` key.
2.  If an object has `$ref`, the system queries the database to find the correct version of the Vibe based on the URI and the current context (branch, date, etc.).
3.  If an object has `$refOriginal`, the system uses that exact Vibe version.
4.  Once the correct dependency Vibe is identified, its schema is recursively resolved and embedded.
5.  The system then adds the `$refResolved` property to the object, storing the canonical `ref` of the dependency that was used (e.g., `main@schemas/user?1`).

This ensures that all schemas are fully resolved and their dependencies are explicitly recorded at the time of creation or update. The process is idempotent; if a schema is processed again, the system sees the `$refResolved` property and knows not to re-resolve that dependency.

This explicit dependency information, captured in the final resolved schema, is invaluable. It provides a complete, verifiable dependency graph of how any given Vibe is constructed, underpinning the system's traceability and allowing us to understand the precise impact of any change.
