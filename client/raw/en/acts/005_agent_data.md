# 005: Agent/Data

> [!DEFINITION] [Data Message](./000_glossary.md)
> A persistent context message containing a `data` value and an optional `schema`. It is retained across an agent's execution loop to provide a stable, structured context.

> Sidenote:
>
> - Requires:
>   - [001: Agent/Request](./001_agent_request.md)
> - Enables:
>   - [006: Agent/Input](./006_agent_input.md)
>   - [009: Agent/State](./009_agent_state.md)
> - Complemented by:
>   - [010: Agent/Loop](./010_agent_loop.md)
>   - [013: Agent/Instancing](./013_agent_instancing.md)

The :term[Data Protocol]{canonical="Data"} is a low-level pattern for providing structured, self-describing information. It serves as a foundational mechanism used by other subsystems, such as :term[Input]{canonical="Input"} and :term[State]{canonical="State"}, to manage structured data within an agent's `context`. Unlike ephemeral messages, :term[Data]{canonical="Data"} messages are persistent and are retained across multiple steps of an agent's process, providing a stable context for multi-step tasks.

## The Data Message

> Sidenote:
>
> - [001: Agent/Request](./001_agent_request.md)

A :term[Data]{canonical="Data"} message is a simple construct for adding structured context to a :term[Request]{canonical="Request"}. It is a message object that contains the following properties:

- **`data`**: Any JSON value (e.g., string, number, object, array) that contains the payload.
- **`schema`**: An optional JSON Schema that defines the structure and semantic meaning of the `data`.
- **`kind`**: An optional string that identifies the message's role (e.g., `"state"`, `"input"`). This allows the system and the LLM to differentiate between various types of data within the same context.

By pairing data with an optional schema, a :term[Data]{canonical="Data"} message makes the context machine-readable. The `schema` acts as a blueprint, explaining to the LLM what each property means, what its type is, and what constraints apply. This not only guides the LLM's reasoning but also serves as a form of documentation, showing users what is possible to change or configure.

## Merging and Identity

The protocol is designed to handle multiple :term[Data]{canonical="Data"} messages within a single `context`. If the system decides that multiple messages share the same identity, they are merged into a single, coherent object. This is particularly useful for scenarios like applying a series of state patches.

A message's identity is primarily determined by its `kind`. For example, all messages of `kind: "state"` without any other distinguishing features are considered to have the same identity. This identity can be further specified by other protocols, most notably :term[Instancing]{canonical="Instancing"}, which allows for parallel processing by creating distinct contexts.

> Sidenote:
>
> - [013: Agent/Instancing](./013_agent_instancing.md)

When multiple mergeable messages are present (e.g., several `state` objects representing patches), the system can handle this in two ways. First, the agent's execution logic can explicitly merge these objects into a single, coherent state object before presenting it to the LLM. This reduces the cognitive load on the model. Second, the LLM itself can "mentally merge" the information in its latent space, understanding that the separate messages represent different facets of a single concept.

:::::details{title="Example of how data message are seen by LLM" open=false}

- The `text` message is passed through as-is.
- The `data` messages are merged and transformed into a ingle `text` message.

::::columns
:::column{title="What the code looks like"}

```typescript
Agent.Request(config, schema, [
  {
    type: 'text',
    text: "Update the user's city to Austin",
  },

  // Schema is serialized for LLM to understand semantics
  {
    type: 'data',
    kind: 'user',
    description: 'Represents the current user.',
    data: { name: 'John Doe' },
    schema: {
      type: 'object',
      properties: {
        name: { type: 'string' },
        age: { type: 'number' },
        city: { type: 'string' },
      },
    },
  },

  // Second `user` message will merge with the first
  {
    type: 'data',
    kind: 'user',
    data: { age: 30 },
  },
]);
```

:::
:::column{title="What the LLM Sees"}

```typescript
[
  {
    role: 'user',
    content: {
      type: 'text',
      text: "Update the user's city to Austin",
    },
  },
  {
    role: 'user',
    content: {
      type: 'text',
      text: `
        ## Data: ¶user
        {
          "name": "John Doe",
          "age": 30
        }
        Represents the current user.
        Schema for ¶user:
        {
          "type": "object",
          "properties": {
            "name": { "type": "string" },
            "age": { "type": "number" },
            "city": { "type": "string" }
          }
        }`,
    },
  },
];
```

:::
::::
:::::

## Specializing the Data Message

The generic :term[Data]{canonical="Data"} message is a foundational pattern. It is specialized for different roles by other parts of the system, often by assigning a specific `kind`.

- **:term[Input Message]{canonical="Input Message"}:** An :term[Input Message]{canonical="Input Message"} is a :term[Data]{canonical="Data"} message with `kind: 'input'`. It formally declares the parameters a :term[Request]{canonical="Request"} accepts, turning a static :term[Request]{canonical="Request"} into a reusable, function-like component.

  > Sidenote:
  >
  > - [006: Agent/Input](./006_agent_input.md)

- **:term[State Message]{canonical="State Message"}:** A :term[State Message]{canonical="State Message"} is a :term[Data]{canonical="Data"} message with `kind: 'state'`. It represents the persistent, evolving memory of a workflow. Its `schema` defines the structure of this memory, including what properties are available to be read from or written to.

  > Sidenote:
  >
  > - [009: Agent/State](./009_agent_state.md)

- **:term[Output]{canonical="Final Output"}:** The `_outputPath` mechanism creates new :term[Data]{canonical="Data"} messages to persist the results of :term[Tool Calls]{canonical="Tool Call"}. When a `Call` with an `_outputPath` is executed, its result is appended to the context as a new :term[Data]{canonical="Data"} message (often with `kind: 'state'`), making it available for subsequent steps.

  > Sidenote:
  >
  > - [008: Agent/Output](./008_agent_output.md)

- **:term[Instancing]{canonical="Instancing"}:** The :term[Instancing]{canonical="Instancing"} system uses the `_instance` property to distinguish :term[Data]{canonical="Data"} messages. This key scopes a message to a specific execution thread in a batch operation. :term[Data]{canonical="Data"} messages with different `_instance` values are treated as having different identities and will not be merged, ensuring data isolation.

  > Sidenote:
  >
  > - [013: Agent/Instancing](./013_agent_instancing.md)

- **:term[Loop]{canonical="Loop"}:** The :term[Execution Loop]{canonical="Execution Loop"} relies on :term[Data]{canonical="Data"} messages to maintain continuity. Specifically, the :term[State Message]{canonical="State Message"} is the primary vehicle for persisting information across the ticks of a :term[Loop]{canonical="Loop"}.

  > Sidenote:
  >
  > - [010: Agent/Loop](./010_agent_loop.md)

- **:term[Variables]{canonical="Variable"}:** The :term[Variable]{canonical="Variable"} system is the primary consumer of :term[Data]{canonical="Data"} messages. **:term[Variable References]{canonical="Variable Reference"}** (`†<kind>.<path>`) are used within :term[Tool Calls]{canonical="Call"} to dynamically read values from :term[Data]{canonical="Data"} messages in the context, such as an :term[Input Message]{canonical="Input Message"} or a :term[State Message]{canonical="State Message"}.

  > Sidenote:
  >
  > - [007: Agent/Variables](./007_agent_variables.md)

## From Generic Data to Specific Roles

The :term[Data]{canonical="Data"} message provides a generic container for structured information. However, for an agent to use this data effectively, it needs to understand the data's role in the process. The following chapters describe how this generic :term[Data]{canonical="Data"} message is specialized for specific purposes, such as providing initial parameters for a workflow.

:term[006: Agent/Input]{href="./006_agent_input.md"} describes how a `Data` message is used to create a structured prompt for an agent.
