# 101: Concept/Idea

> [!DEFINITION] [Idea](./000_glossary.md)
> A self-contained, stateful data triplet (`schema`, `solution`, `context`) representing a unit of knowledge. It's a persistent, computational primitive, not an ephemeral prompt.

> Sidenote:
>
> - Requires:
>   - [001: Agent/Request](./001_agent_request.md)
> - Enables:
>   - [103: Concept/Ideator](./103_concept_ideator.md)

A decentralized web of living documents is enabled by the architecture outlined here, which covers the protocol's core data structure (the **Idea**) and the decentralized discovery mechanism for publishing and resolving them via **DNS**.

For details on how `Ideas` become executable services, see [103: Concept/Ideator](./103_concept_ideator.md). For details on the spectrum of hosting models, see [102: Concept/Sovereignty](./102_concept_sovereignty.md).

## The Mechanics of a Living Web

The architecture is built on a principle: **the content is the protocol.** The system's entire grammar consists of a single unit: a self-contained "triplet" called the **Idea**. This structure enables true ownership and portability; because there is no hidden state, you are never locked in.

- **Context:** All the instructions, source material, and references used to generate the solution.
- **Schema:** The `jsonschema` blueprint that gives the Idea's data a universal, semantic meaning, allowing any AI to understand and modify it.

  > Sidenote:
  >
  > - More at [json-schema.org/](https://json-schema.org/)

- **Solution:** The output, result, or content of the Idea.

Ideas are **immutable by design**. The protocol has only one action: sharing an Idea. To evolve a thought, a new Idea is created that references the old, preserving a pristine, unbreakable chain of creation.

## Beyond the Prompt: A New Computational Primitive

At first glance, an `Idea` might seem like a prompt for a large language model (LLM). The key difference lies in the shift from single, ephemeral interactions to a system of persistent, composable assets.

Unlike a simple, ephemeral request to a chatbot, an `Idea` is a self-contained, stateful artifact. It packages the `input`, the `output` (`solution`), the rules (`schema`), and the entire `context` of its creation into a single, portable unit. It's not just a question; it's the question, the answer, and the complete formula that connects them, enabling a persistent, composable system, not just a one-off transaction.

This makes an `Idea` a true computational primitive—a building block for creating complex, evolving systems. You don't just "run" an Idea; you can fork it, remix it, feed it into other Ideas, and build entire pipelines of logic, all without writing traditional code. It's a platform, not a prompt.

## Core Invariants

To ensure the protocol remains robust, transparent, and portable, all implementations must adhere to four core invariants.

### Deterministic Provenance

An `Idea` is designed for reproducibility. By feeding the same `context` and `schema` to a capable LLM, a comparable `solution` can be regenerated. This principle ensures that we are striving for a reproducible web of ideas. While variations from different providers or model settings are expected, the fundamental goal is that the output is a direct, traceable function of its inputs.

### Transparent Context

The entire `context` triplet is visible to the LLM during execution. This means it cannot be used as a container for arbitrary state unless that state is directly relevant to the computation and intended for the LLM to process. This constraint is critical to prevent indiscriminate use of the `context`, ensuring it remains a focused, purposeful part of the `Idea`.

### Schema-Bound State

The `solution` is the state. Because every `solution` must conform to its `schema`, the state of any `Idea` is fundamentally determined and validated by its schema. This follows from the principle of a transparent context, ensuring that all state is explicit, structured, and universally understandable.

### Immutability

An `Idea` with a different `context` or `schema` is a different `Idea`. An `Idea`
is considered backward-compatible if it adds new fields to the `schema`. An `Idea` that changes the `schema` requires bumping the version.

## Publication & Discovery

For an `Idea` to be useful, it must be published and discoverable. To achieve this in a decentralized way, every `Idea` is given a globally unique, resolvable name, which serves as its identity. This decouples the `Idea` from any specific storage location, making it truly portable and sovereign.

This is achieved by giving each `Idea` a unique domain name and using DNS records to point to its canonical definition. The specific mechanics of this decentralized identity and the progressive layers of hosting that build upon it are detailed in [102: Concept/Sovereignty](./102_concept_sovereignty.md).
