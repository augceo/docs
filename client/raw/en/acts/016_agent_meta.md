# 015: Agent/Meta

> [!DEFINITION] [Meta Properties](./000_glossary.md)
> A structured object that holds the explicit identity of a :term[Request]{canonical="Request"}, including its name, namespace, version, creation date, and branch associations. It provides a stable, machine-readable anchor for a request's lineage and addressability.

> Sidenote:
>
> - Requires:
>   - :term[001: Agent/Request]{href="./001_agent_request.md"}
> - Complemented by:
>   - :term[107: Concept/Identity]{href="./107_concept_identity.md"}
>   - :term[108: Concept/Visibility]{href="./108_concept_visibility.md"}
>   - :term[110: Concept/Addressing]{href="./110_concept_addressing.md"}

While many agent interactions can be ephemeral, building complex, stateful systems requires a way to uniquely identify and track a :term[Request]{href="./001_agent_request.md"} and its resulting :term[Solution]{href="./001_agent_request.md"}. The **`Meta`** object serves this purpose. It is the "business card" of a :term[Request]{canonical="Request"}, providing a stable, machine-readable handle that enables persistence, versioning, routing, and historical tracking.

Crucially, by making identity an explicit part of the `solution`, the `Meta` object empowers the LLM to mint new identities or branch existing ones in direct response to its context, turning a simple response into an act of creation.>

> Sidenote:
> :term[107: Concept/Identity]{href="./107_concept_identity.md"}

## The `Meta` Object in the Request Lifecycle

The `Meta` object is a first-class citizen in the :term[Request]{href="./001_agent_request.md"} lifecycle. It appears in the `context` to inform the LLM of the request's current identity, and in the `schema` as a required part of the `solution`.

- **In the `context`**: A `meta` message provides the LLM with the identity of the state or process it is currently updating.
- **In the `schema`**: The `solution`'s schema requires a `meta` property, compelling the LLM to consider and update the identity as part of its task.
- **In the `solution`**: The LLM generates a new `meta` object, often with an updated version, reflecting the evolution of the process.

This cycle transforms the LLM from a simple processor into an active participant in the lifecycle of a stateful, versioned entity.

> [!HEADSUP] Heads up: From Request to Idea
> This entire :term[Request]{href="./001_agent_request.md"} pipeline—the `context` (including the `Meta` message), the `schema`, and the resulting `solution`—forms a self-contained, reproducible unit. When saved, this unit is what the system refers to as an :term[Idea]{href="./101_concept_idea.md"}. The `Meta` object is the key that transforms an ephemeral `Request` into a persistent, addressable `Idea`.
>
> > Sidenote:
> > :term[101: Concept/Idea]{href="./101_concept_idea.md"}

::::columns
:::column{title="Example Request Structure"}

```json
// The LLM receives the current meta as context,
// and is tasked to produce a new one in the solution.
{
  "context": [
    {
      "type": "meta",
      "meta": {
        "domain": "reactor.ideas.services",
        "path": "/games/321",
        "version": "1.2.3",
        "branches": ["main"],
        "createdAt": "2025-10-26T10:00:00Z"
      }
    },
    {
      "type": "state",
      "state": {
        "...current game state..."
      }
    }
  ],
  "schema": {
    "type": "object",
    "properties": {
      "meta": {
        "$ref": "MetaSchema"
      },
      "output": {
        "$ref": "GameSchema"
      }
    }
  }
}
```

:::
:::column{title="Example Solution"}

```json
// The LLM's solution includes the next game state
// and the bumped version in the new meta object.
{
  "meta": {
    "domain": "reactor.ideas.services",
    "path": "/games/321",
    "version": "1.2.4",
    "branches": ["main"],
    "createdAt": "2025-10-26T10:05:00Z"
  },
  "output": {
    "...next game state..."
  },
  "calls": []
}
```

:::
::::

## Autonomous Evolution and Versioning

A key role of the `Meta` object is to enable the autonomous evolution of agentic processes. By including the current version and branches in the `context` and requiring a new version in the `solution`, the system tasks the LLM with version bumping. Generating a new `solution` against the same `schema` is considered a compatible change, and therefore it is imperative that the agent increments the minor version. This creates a new, immutable snapshot of its state and logic, establishing a complete and auditable lineage of the agent's actions over time.

> Sidenote:
>
> - The principles of autonomous improvement are described in :term[106: Concept/Evolution]{href="./106_concept_evolution.md"}.

This raises a challenge in distributed systems: if two independent processes react to different events simultaneously, they might both try to bump the version from `1.2.3` to `1.2.4`, creating a race condition.

The architecture resolves this using a branching mechanism. When creating a new line of development, a process adds a new branch to the `branches` array (e.g., `["main", "my-feature"]`) and creates a branched version. The versions become `1.2.3.branch-A.1` and `1.2.3.branch-B.1`, respectively. This allows for parallel, conflict-free evolution, with the different histories being merged or reconciled later.

> Sidenote:
>
> - This is detailed further in :term[108: Concept/Visibility]{href="./108_concept_visibility.md"}.

## Connecting Identity to Address

The fields within the `Meta` object provide all the necessary components to give a :term[Request]{canonical="Request"} a globally unique address. This allows a specific, versioned instance of a request/solution pair to be retrieved, routed, or referenced programmatically.

> Sidenote:
>
> - The concrete syntax for this addressing scheme is defined in :term[110: Concept/Addressing]{href="./110_concept_addressing.md"}.

The resolution process connects a dynamic request to a specific, immutable result. Here is a detailed breakdown:

- **Meta**: The properties of a specific, versioned instance.
  - _meta.domain:_ `my-project.com`
  - _meta.path:_ `bob`
  - _meta.version:_ `1.2.staging.1`
  - _meta.branches:_ `['staging']`
  - _meta.createdAt:_ `2025-10-26T10:00:00Z`

- **Reference**: A dynamic request for an entity.
  - `idea://my-project.com/bob?1.2`
  - _Search path:_ `['staging']`
  - _Cutoff date:_ `2025-10-26T10:00:00Z` (from `createdAt`)

- **Resolved**: The permanent, unambiguous URI pointing to the exact resource found.
  - `idea://my-project.com/~:staging/bob?1.2:1.2.staging.1`
  - _Branch:_ `~:staging`
    - _asked for:_ any branch in the search path
    - _got_: `staging`
  - _Version:_ `1.2:1.2.staging.1`
    - _asked for:_ latest compatible with `1.2`
    - _got:_ `1.2.staging.1`

This makes the entire request-solution unit self-describing and addressable.
