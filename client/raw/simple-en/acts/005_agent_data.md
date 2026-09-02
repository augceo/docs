# 005: Agent/Data

> [!DEFINITION] [Data Message](./000_glossary.md)
> A special message that holds onto information, like a sticky note that doesn't get thrown away. It has a piece of `data` and sometimes a `schema`, which is like a set of instructions explaining what the data means. It stays around while an AI agent works, giving it a stable set of facts to refer to.



> Sidenote:
> - Needs:
>   - [001: Agent/Request](./001_agent_request.md)
> - Makes Possible:
>   - [006: Agent/Input](./006_agent_input.md)
>   - [009: Agent/State](./009_agent_state.md)
> - Works Together With:
>   - [010: Agent/Loop](./010_agent_loop.md)
>   - [013: Agent/Instancing](./013_agent_instancing.md)

The :term[Data Protocol]{canonical="Data"} is a simple, basic way for the system to handle information that is organized and explains itself. Think of it as a set of rules for creating labeled containers for information. Other parts of the system, like those that handle the agent's starting instructions (:term[Input]{canonical="Input"}) or its memory (:term[State]{canonical="State"}), use these containers to keep their data neat and tidy inside the agent's workspace, called the `context`. Unlike quick, temporary messages that disappear, :term[Data]{canonical="Data"} messages stick around for many steps, giving the agent a reliable source of information for complex jobs.

## The Data Message

> Sidenote:
> - [001: Agent/Request](./001_agent_request.md)

A :term[Data]{canonical="Data"} message is a simple but really useful way to add organized information to a :term[Request]{canonical="Request"} you send to an agent. It's a message that contains these parts:

- **`data`**: Any piece of information, like a word, a number, a list, or an object. This is the actual stuff you want the agent to know.
- **`schema`**: An optional set of instructions, like a blueprint, that explains the `data`. It tells the agent what the data means, what format it should be in (like text or a number), and any rules it has to follow.
- **`kind`**: An optional label that says what the message is for (like `"state"` for memory, or `"input"` for instructions). This helps both the system and the AI agent quickly understand the purpose of different pieces of information.

By putting the data and its blueprint (`schema`) together, a :term[Data]{canonical="Data"} message makes information easy for a computer to understand. The `schema` guides the AI's thinking and also acts as a guide for humans, showing them what information they can change or add.

## Merging and Identity

The system is designed to handle many :term[Data]{canonical="Data"} messages at once in the agent's workspace (`context`). If the system sees multiple messages that seem to be about the same thing, it combines them into one neat package. This is super helpful when you're updating the agent's memory piece by piece.

What makes two messages "about the same thing" is their identity, which is mostly decided by their `kind` label. For example, all messages with the `kind: "state"` are usually seen as parts of the same memory. Other systems, like :term[Instancing]{canonical="Instancing"}, can give messages more specific identities, which helps the agent work on many things at once without mixing them up.

> Sidenote:
> - [013: Agent/Instancing](./013_agent_instancing.md)

When there are several messages that can be combined (like a few different updates to the agent's memory), the system can handle it in two ways. First, the agent's code can merge them into one single memory object before showing it to the AI, which makes it easier for the AI to understand. Second, the AI itself can be smart enough to mentally connect the dots, understanding that all these separate pieces of information are about the same concept.

:::::details{title="Example of how data message are seen by LLM" open=false}

- The normal `text` message is shown to the AI just as it is.
- The `data` messages are combined into a single, organized `text` message that includes the data, its label, a description, and the instructional blueprint (`schema`).

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

The basic :term[Data]{canonical="Data"} message is a general-purpose container for information. Other parts of the system give it a specific job by assigning it a `kind` label.

- **:term[Input Message]{canonical="Input Message"}:** This is a :term[Data]{canonical="Data"} message with `kind: 'input'`. It officially lists the information an agent needs to start a job, turning a simple request into a reusable tool, much like a function in programming.

  > Sidenote:
  > - [006: Agent/Input](./006_agent_input.md)

- **:term[State Message]{canonical="State Message"}:** This is a :term[Data]{canonical="Data"} message with `kind: 'state'`. It acts as the agent's long-term memory for a task. Its `schema` defines what this memory looks like and what kind of information can be stored or changed.

  > Sidenote:
  > - [009: Agent/State](./009_agent_state.md)

- **:term[Output]{canonical="Final Output"}:** The `_outputPath` feature creates new :term[Data]{canonical="Data"} messages to save the results from tools the agent uses. When a tool runs, its result is added back to the agent's workspace as a new :term[Data]{canonical="Data"} message (usually with `kind: 'state'`), so it can be used in later steps.

  > Sidenote:
  > - [008: Agent/Output](./008_agent_output.md)

- **:term[Instancing]{canonical="Instancing"}:** This system uses a special `_instance` property to tell :term[Data]{canonical="Data"} messages apart. This lets the agent work on a batch of similar tasks at the same time, keeping the data for each task separate. Messages with different `_instance` values are never merged.

  > Sidenote:
  > - [013: Agent/Instancing](./013_agent_instancing.md)

- **:term[Loop]{canonical="Loop"}:** The agent's work cycle, or :term[Execution Loop]{canonical="Execution Loop"}, depends on :term[Data]{canonical="Data"} messages to remember what's happening from one step to the next. The :term[State Message]{canonical="State Message"} is the main way it keeps track of information over time.

  > Sidenote:
  > - [010: Agent/Loop](./010_agent_loop.md)

- **:term[Variables]{canonical="Variable"}:** The :term[Variable]{canonical="Variable"} system is the main user of :term[Data]{canonical="Data"} messages. Special codes called **:term[Variable References]{canonical="Variable Reference"}** (which look like `†<kind>.<path>`) are used in tool instructions to pull information from :term[Data]{canonical="Data"} messages, like taking a value from the agent's memory (:term[State Message]{canonical="State Message"}) or its initial instructions (:term[Input Message]{canonical="Input Message"}).

  > Sidenote:
  > - [007: Agent/Variables](./007_agent_variables.md)

## From Generic Data to Specific Roles

The :term[Data]{canonical="Data"} message is a general container for organized information. But for an agent to use it well, it needs to know what that information is for. The next documents explain how this general container is used for specific jobs, like giving an agent its starting instructions.

The next document, :term[006: Agent/Input]{href="./006_agent_input.md"}, explains how a `Data` message is used to create a well-defined starting point for an agent's task.
