# 103: Concept/Ideator

> [!DEFINITION] [Ideator](./000_glossary.md)
> An :term[Idea]{canonical="Idea"} that accepts input, signified by a context message of `type: "input"`. It acts as a function, transforming input to output.

> Sidenote:
>
> - Requires:
>   - [101: Concept/Idea](./101_concept_idea.md)
>   - [006: Agent/Input](./006_agent_input.md)
> - Enables:
>   - [004: Agent/Call](./004_agent_call.md)
>   - [102: Concept/Sovereignty](./102_concept_sovereignty.md)

The protocol for :term[Ideators]{canonical="Ideator"} and :term[Idea Transformers]{canonical="Idea Transformer"} defines them as executable, service-based components. It builds upon the foundational [101: Concept/Idea](./101_concept_idea.md), which defines the core data structure, and describes how an :term[Idea]{canonical="Idea"} is transformed into a functional, invocable entity.

For details on the spectrum of hosting and deployment models, see [102: Concept/Sovereignty](./102_concept_sovereignty.md).

## From Idea to Ideator

An **:term[Ideator]{canonical="Ideator"}** is not a distinct entity, but a functional role any :term[Idea]{canonical="Idea"} can fulfill. It can be seen as a function that performs work by transforming input into output within a :term[latent space]{canonical="Latent Execution"}. This means its logic is not necessarily defined by explicit code, but is instead guided by the rich `context` of the :term[Idea]{canonical="Idea"}—its schemas, examples, and natural language instructions—which are interpreted by an LLM.

The definitive signal that an :term[Idea]{canonical="Idea"} is an :term[Ideator]{canonical="Ideator"} is the presence of a `context` message with `type: "input"`. This message defines the schema for the data the :term[Ideator]{canonical="Ideator"} expects. An _executable_ :term[Ideator]{canonical="Ideator"} may also include a `context` message with `type: "code"`, pointing to an explicit implementation.

### The Idea Transformer: A Special Case

A common pattern is an :term[Ideator]{canonical="Ideator"} whose input is itself another :term[Idea]{canonical="Idea"}. We call this specific type of :term[Ideator]{canonical="Ideator"} an **:term[Idea Transformer]{canonical="Idea Transformer"}**. This is what enables the compositional pipelines where :term[Ideas]{canonical="Idea"} are chained together and evolved.

## Implementations and Composition

The architectural principles outlined here define a **behavioral contract** for any :term[Ideator]{canonical="Ideator"} service. This contract is not for a single piece of software, but a standard for interoperability, allowing for a plurality of implementations and a rich, compositional ecosystem.

### A Plurality of Implementations

An :term[Ideator]{canonical="Ideator"} service's contract is fulfilled by honoring its public API (accepting an :term[Idea]{canonical="Idea"} and returning another). This allows for multiple concrete implementations, each suited for different use cases:

- **Managed Services**: A provider can offer hosting as a managed, cloud-based service, abstracting away the infrastructure, as described in the [:term[Sovereignty]{href="./102_concept_sovereignty.md"} Protocol](./102_concept_sovereignty.md).
- **Self-Hosted Instances**: A developer can run their own implementation of the service on their own infrastructure, giving them full control.
- **In-Memory Implementations**: For local development and testing, an :term[Ideator]{canonical="Ideator"}'s execution logic can be run as a simple in-memory function, bypassing the network entirely while still honoring the core contract.

### Composition and Higher-Order Systems

In this ecosystem, there is no concept of a "private API." All services are built to interact via their public, contract-based interfaces.

More sophisticated services, which can be thought of as **Higher-Order Systems**, are created by composing other, more primitive :term[Ideators]{canonical="Ideator"}. The internal logic of a higher-order service involves making calls to the public APIs of other :term[Ideators]{canonical="Ideator"}.

For example, the **:term[Reactor]{canonical="Reactor"}** system is a higher-order :term[Ideator]{canonical="Ideator"}. To manage a game, it might:

1.  Accept a game state :term[Idea]{canonical="Idea"} via its own public API.
2.  Internally call a public `Player` service to create and manage player identifiers.
3.  Call a public `Storage` service to log the game's history.
4.  Return the new game state :term[Idea]{canonical="Idea"} via its public API.

From the outside, the :term[Reactor]{canonical="Reactor"} is just another :term[Ideator]{canonical="Ideator"}. Its complexity is managed internally by composing other independent, public services.

## The Refiner: An Ideator for Evolution

While most :term[Ideators]{canonical="Ideator"} work within the constraints of a given `schema` to produce a :term[Solution]{canonical="Solution"}, a special class of :term[Idea Transformer]{canonical="Idea Transformer"} exists to evolve the `schema` itself. This is the **:term[Refiner]{canonical="Refiner"}**.

A :term[Refiner]{canonical="Refiner"} is a meta-operation that handles the structural evolution of an :term[Idea]{canonical="Idea"}. It takes an existing :term[Idea]{canonical="Idea"} and a prompt (e.g., "Add an 'author' field to this article") as input, and produces a _new_ :term[Idea]{canonical="Idea"} as output.

This new :term[Idea]{canonical="Idea"} has:

- An **updated `schema`**.
- An **updated :term[Solution]{canonical="Solution"}** that conforms to the new `schema`.
- **Migrated data** from the original :term[Solution]{canonical="Solution"}. The LLM, knowing both the old and new schemas, attempts to migrate the data intelligently.

The :term[Refiner]{canonical="Refiner"} is the primary mechanism for advancing an :term[Idea]{canonical="Idea"}'s **:term[lineage]{canonical="Lineage"}**. If the schema change is backward-incompatible, the new :term[Idea]{canonical="Idea"} will represent a new major version in its version chain. This allows for safe, explicit evolution of the system's core data structures, orchestrated by a dedicated, reusable :term[Ideator]{canonical="Ideator"}.

Now that an :term[Ideator]{canonical="Ideator"} provides a framework for execution, the next step is to understand how that execution actually happens. This leads to the concept of :term[Latent Execution]{canonical="Latent Execution"}, where an LLM interprets an :term[Idea]{canonical="Idea"}'s context to generate a result without explicit code.
