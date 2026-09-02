# 015: Agent/Meta

> [!DEFINITION] Meta Properties
> Imagine every task in our system has its own unique ID card. This card, which we call a "Meta Property," holds all the important details: its name, where it belongs, what version it is, when it was made, and what project branches it's part of. It's a clear, computer-friendly label that lets us track a task's history and find it easily.

> Sidenote:
> - You should know:
>   - :term[001: Agent/Request]{href="./001_agent_request.md"}
> - Related ideas:
>   - :term[107: Concept/Identity]{href="./107_concept_identity.md"}
>   - :term[108: Concept/Visibility]{href="./108_concept_visibility.md"}
>   - :term[110: Concept/Addressing]{href="./110_concept_addressing.md"}

While some conversations with an AI can be forgotten, creating complex systems requires a way to keep track of every :term[Request]{href="./001_agent_request.md"} and its final :term[Solution]{href="./001_agent_request.md"}. The **`Meta`** object is that tracking system. Think of it as the "business card" for a :term[Request]{canonical="Request"}. It provides a permanent, computer-readable ID that allows us to save, version, find, and look up the history of any task.

Most importantly, by making this ID card a required part of the final answer, the AI can create brand new IDs or split off new versions on its own. This turns a simple response into a creative act, like starting a new project.

> Sidenote:
> :term[107: Concept/Identity]{href="./107_concept_identity.md"}

## The `Meta` Object's Job

The `Meta` object is a core part of how a :term[Request]{href="./001_agent_request.md"} is handled from start to finish. It shows up in the information given to the AI and is a required part of the AI's answer.

- **In the `context` (the information given)**: A `meta` message tells the AI the identity of the task it's currently working on.
- **In the `schema` (the rules for the answer)**: The rules say that the final `solution` *must* include a `meta` section, forcing the AI to think about the task's identity and update it.
- **In the `solution` (the final answer)**: The AI creates a new `meta` object, usually with an updated version number, to show how the task has changed.

This process turns the AI from a simple calculator into an active manager that helps keep track of a system's history and versions.

> [!HEADSUP] Heads up: From a Request to an Idea
> The whole package—the `context` (including the `Meta` ID card), the `schema` (the rules), and the final `solution`—creates a single, complete unit. When we save this unit, the system calls it an :term[Idea]{href="./101_concept_idea.md"}. The `Meta` object is the key that turns a temporary `Request` into a permanent, findable `Idea`.
>
> > Sidenote:
> > :term[101: Concept/Idea]{href="./101_concept_idea.md"}

::::columns
:::column{title="Example of a Request"}

```json
// The AI gets the current ID card as context
// and is told to create a new one in its answer.
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
    "type": 'object',
    "properties": {
      "meta": {
        "$ref": "MetaSchema"
      },
      "output": {
        "$ref": 'GameSchema'
      }
    }
  }
}
```

:::
:::column{title="Example of a Solution"}

```json
// The AI's answer includes the next game state
// and an updated version in the new ID card.
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

## Growing and Versioning on Its Own

A key job of the `Meta` object is to let the system's tasks grow and change by themselves. By giving the AI the current version and telling it to create a new one in its answer, we make the AI responsible for updating the version number. When it makes a simple update, it should change the last number in the version (like going from 1.2.3 to 1.2.4). This creates a permanent snapshot of its state and work, building a complete and clear history of everything the AI has done over time.

> Sidenote:
> - The ideas behind self-improvement are explained in :term[106: Concept/Evolution]{href="./106_concept_evolution.md"}.

This can cause a problem in big systems. Imagine two different AIs working on the same task at the same time. They might both try to update the version from `1.2.3` to `1.2.4` simultaneously, creating a conflict.

Our system solves this with a branching feature. Think of it like a tree. Instead of two people trying to build on the same branch, they each create their own new branch. In our system, the versions would become `1.2.3.branch-A.1` and `1.2.3.branch-B.1`. This lets them work at the same time without getting in each other's way. Later on, their work can be combined.

> Sidenote:
> - This is explained in more detail in :term[108: Concept/Visibility]{href="./108_concept_visibility.md"}.

## Turning an ID into an Address

The information inside the `Meta` object gives us everything we need to create a unique address for any :term[Request]{canonical="Request"}, just like a website URL. This allows us to find, send, or refer to a specific, versioned task and its solution using code.

> Sidenote:
> - The exact way this addressing system is written is defined in :term[110: Concept/Addressing]{href="./110_concept_addressing.md"}.

The process of finding the right file connects a general request to a specific, permanent result. Here’s how it works:

- **Meta**: The ID card for a specific, versioned file.
  - _meta.domain:_ `my-project.com`
  - _meta.path:_ `bob`
  - _meta.version:_ `1.2.staging.1`
  - _meta.branches:_ `['staging']`
  - _meta.createdAt:_ `2025-10-26T10:00:00Z`

- **Reference**: A general request for a file.
  - `idea://my-project.com/bob?1.2`
  - _Where to look:_ `['staging']`
  - _Don't look past this date:_ `2025-10-26T10:00:00Z`

- **Resolved**: The permanent, exact address pointing to the file that was found.
  - `idea://my-project.com/~:staging/bob?1.2:1.2.staging.1`
  - _Branch:_ `~:staging`
    - _You asked for:_ any branch in the search path
    - _You got_: `staging`
  - _Version:_ `1.2:1.2.staging.1`
    - _You asked for:_ the newest version that works with `1.2`
    - _You got:_ `1.2.staging.1`

This makes the entire request-and-solution package easy to understand and find.