# 800: Package/Schemistry

> [!DEFINITION] [Schemistry](./000_glossary.md)
> A library that establishes the **Schema** as the single source of truth for the application. It drives runtime validation, TypeScript types, LLM integration, and application logic from a unified definition, ensuring that runtime data and build-time types are always synchronized.

> Sidenote:
>
> - Enables:
>   - :term[001: Agent/Request]{href="./001_agent_request.md"}
>   - :term[101: Concept/Idea]{href="./101_concept_idea.md"}

**Schemistry** is the foundational library that powers the data structures of the entire ecosystem. Its primary mission is to concentrate power onto the **Schema**, making it the authoritative definition for everything from data validation to agent cognition.

## Philosophy & Core Concepts

### The Single Source of Truth

In many systems, there is a disconnect between the compile-time types (TypeScript interfaces) and the runtime data structures (JSON Schemas or validation logic). This often leads to duplication, drift, and bugs.

Schemistry solves this by establishing the **Schema** as the single source of truth. By defining data structures once in the schema, we automatically derive:

- **Runtime Validation:** Ensuring data integrity at the boundaries.
- **TypeScript Types:** Providing static safety during development.
- **LLM Integration:** Giving agents precise, structured descriptions of the data they handle.
- **Application Logic:** Driving dynamic behavior based on the schema's structure.

This approach supports **schema-driven development**, ensuring that the runtime reality and the build-time theory are always synchronized without manual intervention or complex code generation steps.

### The "From Schema" Paradigm

A central mechanism of the library is the "from schema" tool—a TypeScript macro that computes types directly from schema definitions.

- **Dynamic Type Inference:** We compute types dynamically from the JSON schema constant definition using the `FromSchema` utility.
- **Type Safety:** These inferred types are propagated throughout the codebase. If the schema changes, the types update automatically, and any code violating the new structure will fail to compile.

### The Schema Registry

To manage the complexity of a growing system, Schemistry includes a built-in registry mechanism.

- **Flat Structure:** Schemas are registered at the top level and reference each other via IDs (`$ref`). This prevents the "nesting hell" of deep, inline definitions.
- **Automatic Resolution:** Internal functions automatically handle these references, allowing developers to compose complex data structures from simple, reusable building blocks.

### Canonical Schema & Semantic Divergence

We embrace the concept of a **Canonical Schema**. Internally, a schema can be unwrapped into an expression optimized for library functions.

We accept that the TypeScript type describing a schema (inferred via `FromSchema`) and the schema's runtime structure may diverge structurally, provided they retain the **same semantic meaning**. For example, a schema might be structurally defined as an `allOf` intersection for validation purposes, but its inferred TypeScript type is a merged object. This divergence is intentional: it simplifies usage for the developer while strictly enforcing the data shape at runtime.

### Performance & Caching

Deep type inference is computationally expensive. To handle this, Schemistry implements a robust caching strategy:

- **Caching:** Helpers return objects with types pre-computed and cached.
- **Reusability:** Types are cached on the schema object itself. When chaining operations, we carry the cached type forward, avoiding the need to re-infer from scratch at every step.

## Evolution: From Composite to Strict

Initially, the library used a **composite, wide type** for all schemas, assuming any schema object could have any property. This approach, however, became a trap.

- **The "Composite" Trap:** The wide type prevented type narrowing. Mutually exclusive properties (like in discriminated unions) couldn't be correctly inferred because the wide type swallowed the specific constraints.
- **The Shift to Strictness:** We shifted to a strict approach where `schema.any` is a union of distinct types (`object`, `array`, `string`, etc.).
- **The Resolution:** We are moving away from the composite type entirely. Composite types should be "unwrapped" into their specific constituent types.

## Roadmap: Type-Safe Architecture

### Schema Manipulation

The library provides a suite of functional helpers for manipulating schemas, such as `Intersection`, `Pick`, `Omit`, and `Union`. These allow developers to compose and modify data structures dynamically, treating schemas as first-class citizens.

We are actively working to make these manipulation functions fully **type-safe**. The objective is to ensure that any transformation applied to a schema object at runtime is instantly and accurately reflected in its static TypeScript type.

### The `ToSchema` Strategy

Moving forward, Schemistry adopts a functional strategy for schema manipulation called `ToSchema`. This approach leverages the inherent **commutativity** between Schemas and TypeScript types.

The transformation pipeline follows a clear path:

1.  **Extract (`FromSchema`):** We lift the schema into the TypeScript type domain.
2.  **Operate:** We apply standard, composable TypeScript operations (e.g., `Intersection`, `Union`, `Pick`, `Omit`).
3.  **Reconstruct (`ToSchema`):** We lower the resulting type back into a Schema definition.

```typescript
Schema.Intersection<A, B>(a: A, b: B): ToSchema<FromSchema<A> & FromSchema<B>>
```

**Why This Works:**
Schemas act as a bridge between runtime values and static types. Because this relationship is commutative, we can confidently move between these domains. We prioritize structural correctness over metadata preservation, allowing us to treat schemas as pure, functional building blocks that can be composed, transformed, and reconstructed with mathematical certainty.

### Goals: New Helpers & Extensibility

This strategy allows us to implement complex operations that were previously difficult or impossible to type correctly:

- **Intersection & Union:** combining schemas with full type fidelity.
- **Pick & Omit:** manipulating object schemas while retaining strict typing.

**Maintainability:**
This approach significantly simplifies the implementation of new helpers. By focusing on the TypeScript type transformation rather than maintaining perfect schema structure alignment, we can create robust tools without the immense complexity of handling every schema permutation at the type level.

### Partial Schema Support

Currently, our strict typing requires schemas to be fully defined (e.g., an object must explicitly have `type: 'object'`). JSON Schema, however, allows for "partial" definitions (e.g., defining `properties` without a `type`), effectively inferring the type from the context.

- **Current Limitation:** Our `FromSchema` implementation limits us; it does not infer types from partial objects. A schema with just `properties` is not treated as a real object, which breaks type inference.
- **The Workaround:** We currently require **fully resolved objects and types** (explicit usage of the `type` property everywhere).
- **Internal State:** We use a `CouldBePartial` type internally to mark places where partial support might be added.
- **The Blocker:** We need to determine if adding support for partial inference in `FromSchema` would increase TypeScript type complexity and depth to an impractical level.
- **Future Goal:** If feasible without severe performance penalties, we aim to enable partial support across the ecosystem.

## Outro

By establishing the Schema as the unified language for both humans (developers) and machines (agents), Schemistry provides the robust foundation required for an AI-native system.

With the data structures defined, we can now look at how they are used to structure communication with agents. :term[001: Agent/Request]{href="./001_agent_request.md"} builds upon these schemas to define the protocol for LLM interaction.
