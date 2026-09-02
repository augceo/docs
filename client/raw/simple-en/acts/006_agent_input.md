# 006: Agent/Input

> [!DEFINITION] [Input Message](./000_glossary.md)
> An :term[Input Message]{canonical="Input Message"} is a special kind of information packet. Think of it like a recipe card for an AI :term[Request]{canonical="Request"}: it has a `schema` (the list of required ingredients and rules) and an `input` (the actual ingredients you're using right now). This turns a simple one-time command into a reusable tool, like a function in programming.

> Sidenote:
> - Needs these docs first:
>   - [001: Agent/Request](./001_agent_request.md)
>   - [005: Agent/Data](./005_agent_data.md)
> - Makes this doc possible: [002: Agent/Tool](./002_agent_tool.md)

The :term[Input Message]{canonical="Input Message"} is like a structured set of instructions given to an AI :term[Request]{canonical="Request"}. By clearly defining the information a :term[Request]{canonical="Request"} needs to create its `solution`, the :term[Input]{canonical="Input"} message turns a one-off command into a reusable tool. This is how a simple :term[Request]{canonical="Request"} gets promoted into an executable :term[Tool]{canonical="Tool"} that other AI agents can use.

> [!HEADSUP] Heads up
> A saved, repeatable :term[Request]{canonical="Request"} is what the system calls an :term[Idea]{canonical="Idea"}. When you add an :term[Input]{canonical="Input"} message to that :term[Idea]{canonical="Idea"}, it becomes a runnable app called an :term[Ideator]{canonical="Ideator"}.

## The :term[Input]{canonical="Input"} Message Type

The :term[Input]{canonical="Input"} message is a special instruction used to officially state what kind of information an AI :term[Request]{canonical="Request"} needs to work. It's the part that captures the exact data used to build a `solution`, creating a complete and repeatable record of what happened.

An :term[Input]{canonical="Input"} message has two main parts:

1.  **`schema`**: This is the blueprint. It’s a set of rules (a JSON Schema) that describes the shape and type of data the :term[Request]{canonical="Request"} expects. For example, it might say, "I need a user's name, which must be text, and their age, which must be a number."
2.  **`input`**: This is the actual data that follows the `schema`'s rules for a specific task. For example, `{ "userName": "Jane", "age": 30 }`.

By defining its needs this way, any :term[Request]{canonical="Request"} can describe not just what it creates (its `solution` and output `schema`), but also what it needs to get started. This is the secret to making a request that can be run again and again with perfect consistency.

:::::details{title="Example: Using an Input for a Structured Prompt"}

::::columns
:::column{title="What the code looks like"}

```typescript
Agent.Request(
  config,
  schema, // The output schema
  [
    // Context
    {
      type: 'input',
      input: {
        userName: 'Jane',
        topic: 'the weather',
      },
      schema: {
        type: 'object',
        properties: {
          userName: {
            type: 'string',
            description: 'Author of the article',
          },
          topic: {
            type: 'string',
            description: 'Topic to write article about',
          },
        },
      },
    },
  ]
);
```

:::
:::column{title="What the LLM Sees"}

```typescript
{
  role: 'user',
  content: {
    type: 'text',
    text:
      `## Data: ¶input
      Input data MUST be treated as a structured prompt
      Schema: {
        "type": "object",
        "properties": {
          userName: {
            type: "string",
            description: "Author of the article"
          },
          topic: {
            type: "string",
            description: "Topic to write article about"
          },
        }
      }

      {
        "userName": "Jane",
        "topic": "the weather"
      }`
  }
}
```

:::
::::

:::::

## A Gateway to Usability: UI by Default

The `Input` message is more than a technical detail—it's the key that automatically creates a user-friendly, interactive experience for any :term[Request]{canonical="Request"}. Because a :term[Request]{canonical="Request"} knows the structure for both its ingredients (input) and its final cake (output), a user interface can be built for it automatically.

> Sidenote:
> - [103: Concept/Ideator](./103_concept_ideator.md).

This user interface (UI) has two parts:

1.  **The Form**: The `schema` in the :term[Input]{canonical="Input"} message is a blueprint for an input form. The system reads it and instantly creates fields and dropdowns for the user to fill out, complete with hints and rules.
2.  **The Result**: The main `schema` of the :term[Request]{canonical="Request"} is the blueprint for the output. After the :term[Request]{canonical="Request"} finishes, its `solution` is shown in a clean, organized way instead of just a raw block of text or data.

This turns any :term[Request]{canonical="Request"} into an interactive app. A user can play around with different inputs in the form and see right away how they change the final result. It makes powerful AI tools easy to create and use, turning complex code into something anyone can explore.

> Sidenote:
> - [102: Concept/Sovereignty](./102_concept_sovereignty.md).

## Interactions with other systems

The :term[Input Message]{canonical="Input Message"} is a special version of the :term[Data]{canonical="Data"} message, but it also works with several other parts of the system to create powerful workflows.

- **:term[Tool]{canonical="Tool"}:** The :term[Input Message]{canonical="Input Message"} is what turns a :term[Request]{canonical="Request"} into a reusable :term[Tool]{canonical="Tool"}. The `schema` from the :term[Input Message]{canonical="Input Message"} defines the :term[Tool]{canonical="Tool"}'s settings (what it needs to run), and the main `schema` of the :term[Request]{canonical="Request"} defines the :term[Tool]{canonical="Tool"}'s `_output` (what it will create).

  > Sidenote:
  > - [002: Agent/Tool](./002_agent_tool.md)

- **:term[Plan]{canonical="Plan"}:** A :term[Plan]{canonical="Plan"} message outlines a series of :term[Tool Calls]{canonical="Call"} to be executed in order. The :term[Input]{canonical="Input"} provides the starting information for the very first :term[Tool Call]{canonical="Call"} in the plan, kicking off the whole process.

  > Sidenote:
  > - [012: Agent/Plan](./012_agent_plan.md)

- **:term[Instancing]{canonical="Instancing"}:** When working on many things at once (:term[Instancing]{canonical="Instancing"}), :term[Input]{canonical="Input"} messages can provide data for all of them (like a general setting) or give specific instructions to just one of them.

  > Sidenote:
  > - [013: Agent/Instancing](./013_agent_instancing.md)

- **:term[Variables]{canonical="Variable"}:** The :term[Input]{canonical="Input"} message provides the starting data for a process. But :term[Variables]{canonical="Variable"} are how that data is actually used throughout the workflow. :term[Tool Calls]{canonical="Call"} use references to these :term[Variables]{canonical="Variable"} to grab the values from the :term[Input]{canonical="Input"}, connecting the initial settings to the steps in a :term[Plan]{canonical="Plan"}.
  > Sidenote:
  > - [007: Agent/Variables](./007_agent_variables.md)

## From Static Inputs to Dynamic Connections

The :term[Input Message]{canonical="Input Message"} offers a formal way to feed structured data to an AGI, turning a simple :term[Request]{canonical="Request"} into a reusable tool. However, this just defines the starting line. To build truly smart workflows, this starting data needs to be connected to the tools that will use it.

The next document, :term[007: Agent/Variables]{href="./007_agent_variables.md"}, explains the system that makes these connections, allowing data to flow from the :term[Input Message]{canonical="Input Message"} to the :term[Tools]{canonical="Tool"} in a smart and organized way.
