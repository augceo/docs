# 006: Agent/Input

> [!DEFINITION] [Input Message](./000_glossary.md)
> A specialized :term[Data Message]{canonical="Data Message"} (with `kind: "input"`) containing a `schema` and `input` data. It defines the expected inputs for a :term[Request]{canonical="Request"}, making it a reusable, function-like component.

> Sidenote:
>
> - Requires:
>   - [001: Agent/Request](./001_agent_request.md)
>   - [005: Agent/Data](./005_agent_data.md)
> - Enables: [002: Agent/Tool](./002_agent_tool.md)

The :term[Input Message]{canonical="Input Message"} is a specialized :term[Data Message]{canonical="Data Message"} that provides a structured prompt for a :term[Request]{canonical="Request"}. By formally defining the data that a :term[Request]{canonical="Request"} uses to shape its `solution`, the :term[Input]{canonical="Input"} message transforms a one-off invocation into a reusable, function-like component. This pattern is the key to turning a simple :term[Request]{canonical="Request"} into an executable :term[Tool]{canonical="Tool"} that agents can use.

> [!HEADSUP] Heads up
> A saved, reproducible :term[Request]{canonical="Request"} is what the system calls an :term[Idea]{canonical="Idea"}. When an :term[Input]{canonical="Input"} message is added to its context, it becomes an executable :term[Ideator]{canonical="Ideator"}.

## The :term[Input]{canonical="Input"} Message Type

The :term[Input]{canonical="Input"} message is a special type of context message designed to formally declare the data a :term[Request]{canonical="Request"} accepts. It is the mechanism that captures the structured input used to produce a given `solution`, providing a full, reproducible record of the computational process.

An :term[Input]{canonical="Input"} message contains two key properties:

1.  **`schema`**: A JSON Schema object that defines the structure, types, and constraints of the data the `Request` expects.
2.  **`input`**: A concrete data object that conforms to the `schema` and represents the actual values used for a specific execution.

By defining its inputs in this structured way, any :term[Request]{canonical="Request"} can become self-describing not only in its output (`solution` and `schema`) but also in what it requires to be generated. This is the core mechanism for creating a reproducible request.

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

The `Input` message enables the generation of interactive user interfaces for any :term[Request]{canonical="Request"}. Because a :term[Request]{canonical="Request"} is defined by structured schemas for both its input and its output, a user interface can be generated for it automatically.

> Sidenote:
>
> - [103: Concept/Ideator](./103_concept_ideator.md).

This UI has two parts:

1.  **The Form**: The `schema` within the :term[Input]{canonical="Input"} message provides a blueprint for the input form. A system can read it to instantly render interactive controls, complete with labels and validation.
2.  **The Result**: The main `schema` of the :term[Request]{canonical="Request"} provides the blueprint for the output. After the :term[Request]{canonical="Request"} is complete, its `solution` can be rendered in a rich, structured display instead of just raw data.

This transforms any :term[Request]{canonical="Request"} into an interactive playground. A user can experiment with different inputs in the form and immediately see how they shape the structured result. It democratizes the creation and use of powerful tools, turning abstract computational processes into tangible, interactive applications that anyone can explore.

> Sidenote:
>
> - [102: Concept/Sovereignty](./102_concept_sovereignty.md).

## Interactions with other systems

The :term[Input Message]{canonical="Input Message"} is a specialization of the :term[Data]{canonical="Data"} pattern, but it also interacts with several other systems to enable complex, dynamic workflows.

- **:term[Tool]{canonical="Tool"}:** The :term[Input Message]{canonical="Input Message"} is the key to turning a :term[Request]{canonical="Request"} into a reusable :term[Tool]{canonical="Tool"}. The `schema` from the :term[Input Message]{canonical="Input Message"} defines the :term[Tool]{canonical="Tool"}'s parameters (what it needs to run), and the main `schema` of the :term[Request]{canonical="Request"} defines the :term[Tool]{canonical="Tool"}'s `_output` (what it produces).

  > Sidenote:
  >
  > - [002: Agent/Tool](./002_agent_tool.md)

- **:term[Plan]{canonical="Plan"}:** A :term[Plan]{canonical="Plan"} message contains a graph of :term[Tool Calls]{canonical="Call"}. The :term[Input]{canonical="Input"} provides the initial parameters that are fed into the first :term[Tool Call]{canonical="Call"} in the graph, kicking off the entire process.

  > Sidenote:
  >
  > - [012: Agent/Plan](./012_agent_plan.md)

- **:term[Instancing]{canonical="Instancing"}:** When used with the :term[Instancing]{canonical="Instancing"} protocol, :term[Input]{canonical="Input"} messages can provide data for multi-instance requests, either as a global configuration for all instances or as a targeted input for a specific instance.

  > Sidenote:
  >
  > - [013: Agent/Instancing](./013_agent_instancing.md)

- **:term[Variables]{canonical="Variable"}:** The :term[Input]{canonical="Input"} message provides the initial, static data that kicks off a process. However, :term[Variables]{canonical="Variable"} are the mechanism that allows this data to be used dynamically. :term[Tool Calls]{canonical="Call"} use :term[Variable References]{canonical="Variable Reference"} to read values from the :term[Input]{canonical="Input"}, connecting the initial parameters to the executable steps of a :term[Plan]{canonical="Plan"}.
  > Sidenote:
  >
  > - [007: Agent/Variables](./007_agent_variables.md)

## From Static Inputs to Dynamic Connections

The :term[Input Message]{canonical="Input Message"} provides a formal mechanism for supplying structured data to an agent, turning a simple :term[Request]{canonical="Request"} into a reusable, function-like component. However, this only defines the starting point of a process. To build sophisticated workflows, this static input data needs to be connected to the tools that will operate on it.

:term[007: Agent/Variables]{href="./007_agent_variables.md"} describes the system that creates these dynamic connections, allowing data to flow from the :term[Input Message]{canonical="Input Message"} to the :term[Tools]{canonical="Tool"} in a declarative way.
