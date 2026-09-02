# 002: Agent/Tool

> [!DEFINITION] [Tool](./000_glossary.md)
> A set of rules describing something an AI agent can do. Think of a :term[Tool]{canonical="Tool"} like an item on a restaurant menu. The menu shows the AI what is available and what details it needs to provide to place an order. When the AI decides to use it, it creates a :term[Call]{canonical="Call"}—like writing down an order slip with all the needed details. The system then processes this order, either by letting the AI figure it out itself or by handing it to a specific piece of code called an :term[Activity]{canonical="Activity"}.

> Sidenote:
> - Requires: :term[001: Agent/Request]{href="./001_agent_request.md"}
> - Complemented by: :term[003: Agent/Activity]{href="./003_agent_activity.md"}

A :term[Tool]{canonical="Tool"} acts as the instruction manual for the AI. It provides a clear, organized list of possible actions so the AI understands exactly what it can do and how to execute it.

## What Are Tools?

**Tools are the foundation** of action. They allow agents to look at a situation, check their available options, and choose the right move.

Tools provide:

- **Clear Rules**: Blueprints that agents can easily read and understand.
- **Exact Requirements**: A strict list of what information must go in and what will come out.
- **Building Blocks**: Simple actions that can be chained together to solve tough problems.
- **AI-Friendly Design**: Formats explicitly designed for the AI to reason about and correctly choose from.

When an agent provides the exact details a Tool asks for, it creates a :term[Call]{canonical="Call"}—a specific request saying, "I want to use this tool, and here is all the information needed to run it."

> Sidenote:
> :term[004: Agent/Call]{href="./004_agent_call.md"}
>

## When to Use the Tool System

Give your agents tools when you want them to:

- **Pick the best action** based on what is happening right now.
- **Choose from different options** to reach a final goal.
- **Use different methods** for the same type of job (like choosing between different search engines).
- **Combine AI thinking** with strict, reliable computer code.

## The Blueprint as an Interface

The system relies on a very important rule: **Tools are just blueprints.** They describe *what* an action requires, but they don't contain the code to actually do the work. Keeping the description separate from the actual work makes the system incredibly flexible.

A `Tool` is written in a standard format called a JSON Schema. Any normal word in this blueprint is a detail the AI needs to provide. Words that start with a special underscore (`_`) are secret instructions for the system, telling it how to handle the tool behind the scenes.

A Tool's blueprint clearly maps out its job:

> Sidenote:
> Extensions:
>
> - **`_activity`**: Connects the tool to a strict piece of code to do actual work outside the AI. See :term[003: Agent/Activity]{href="./003_agent_activity.md"}
> - **`_delegate`**: Hires a completely separate, independent agent to do this specific tool's job. See :term[014: Agent/Delegate]{href="./014_agent_delegate.md"}
> - **`_outputPath`**: Remembers the tool's result by saving it permanently into a data folder. See :term[008: Agent/Output]{href="./008_agent_output.md"}
> - **`_instance`**: Aims the tool at one specific target when the AI is juggling many tasks at once. See :term[013: Agent/Instancing]{href="./013_agent_instancing.md"}

- **`title`**: A simple name for humans to read (optional).
- **`description`**: Tells the AI exactly what the tool accomplishes.
- **`properties`**: The required information (inputs) the AI must provide to make it work.
- **`_tool`**: A unique ID name so the system knows exactly which tool is being used.
- **`_output`**: What the final result will look like after the tool finishes.
- **`_why`**: A space the system adds so the AI can explain its reasoning for choosing this action.

Higher-level systems use these basic blueprints to build bigger workflows and manage tasks.

## Tool Definition

Tools are defined as JSON templates. The example below shows a `Tool` for reading the emotion or sentiment of text. This specific `Tool` is meant to be done in the AI's head (latent execution), because the AI already knows how to understand language and doesn't need external code to figure it out.

::::columns
:::column{title="Tool Definition"}

```typescript
Tool.register('sentimentAnalysis', {
  type: 'object',
  description: 'Analyzes text sentiment',
  properties: {
    _tool: { type: 'string', const: 'sentimentAnalysis' },
    text: { type: 'string', description: 'Text to analyze' },
    _output: {
      type: 'object',
      properties: {
        sentiment: { type: 'string' },
        confidence: { type: 'number' },
      },
    },
  },
});
```

:::
:::column{title="Example LLM Output"}

```json
// Prompt: "What is the sentiment of 'This is the best!'"
{
  "_tool": "sentimentAnalysis",
  "text": "This is the best!",
  "_output": {
    "sentiment": "positive",
    "confidence": 0.99
  }
}
```

:::
::::

## Tool Overriding

Every tool uses its `_tool` name as its unique ID. Sometimes, during a conversation, the AI might receive multiple blueprints that have the exact same ID. When this happens, **the newest version always wins.**

This "last-one-wins" rule is incredibly useful:

- **Standard & Custom**: You can give an agent a standard set of tools, then suddenly change how one works for a specific job just by sending a new blueprint over it.
- **Adapting to the Moment**: You can update a tool's instructions mid-conversation to help the AI handle a changing situation.

If there is a conflict, older descriptions are safely ignored and hidden entirely from the AI.

## Building the Final Instruction Manual

An agent does more than just use tools; it eventually needs to deliver a final answer. To set this up, the system gathers all the available :term[Tool]{canonical="Tool"} blueprints and combines them with a master "output" blueprint. This creates one massive instruction manual that is sent to the AI in a single :term[Request]{canonical="Request"}.

This master manual gives the AI a choice. Depending on what you asked it to do, it can:

- **Only use tools (`meta` and `calls`):** If the job requires multi-step planning, the AI will invoke a tool, update the project's version in the `meta` section, and leave the `output` blank for now.
- **Only give the answer (`meta` and `output`):** If the AI already knows the answer, it gives you the final result directly in `output`, updates `meta`, and skips using any tools.
- **Do both at once:** Sometimes the AI will use a tool and also give you a final answer in the exact same step.

To stop the AI from getting confused or making up false answers (hallucinating), the system secretly changes the blueprints right before handing them over:

- If a :term[Tool]{canonical="Tool"} is **latent** (meaning the AI will solve it in its own head without external code), the `_output` section is left alone. The AI needs to see it because it is responsible for guessing the answer.
- If a :term[Tool]{canonical="Tool"} is **explicit** (meaning it will be handed off to a real script, an :term[Activity]{canonical="Activity"}), the system **completely removes** the `_output` requirement before the AI sees it. This sets a very clear boundary: the AI's *only* job is to provide the inputs. Information about what the output looks like is left safely in the text `description` to help the AI plan, but it is not allowed to fill in the answer itself.

This setup handles everything cleanly. The developer provides the `Tool` blueprints separately, and the system snaps them all together so the AI always knows exactly what its choices are.

The example below shows how a :term[Tool]{canonical="Tool"} blueprint and a final output blueprint merge together.

::::columns
:::column{title="Agent Configuration"}

```typescript
Agent.Request(
  config, // Configuration for the request (e.g., model, temperature)
  {
    // Output Schema
    type: 'object',
    properties: {
      summary: { type: 'string' },
    },
    required: ['summary'],
  },
  [
    // Context
    {
      type: 'tool',
      tool: {
        greetUser: {
          type: 'object',
          properties: {
            userName: { type: 'string' },
          },
          required: ['userName'],
        },
      },
    },
    { type: 'text', text: 'some prompt here' },
  ]
);
```

:::
:::column{title="Composed Schema (for the LLM)"}

```json
{
  "type": "object",
  "properties": {
    "meta": {
      "type": "object",
      "description": "Metadata about the idea, including version and identity. The LLM is expected to update this, for example by bumping the version.",
      "properties": {
        "path": {
          "type": "string"
        },
        "version": {
          "type": "string"
        }
      }
    },
    "output": {
      "type": ["object", "null"],
      "properties": {
        "summary": { "type": "string" }
      },
      "required": ["summary"],
      "additionalProperties": false
    },
    "calls": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "_tool": { "const": "greetUser" },
          "userName": { "type": "string" }
        },
        "required": ["_tool", "userName"]
      }
    }
  },
  "required": ["meta", "calls", "output"]
}
```

:::
::::

## Enhancing Tools with Meta-Properties

You can attach extra instructions to a tool right when the AI decides to use it. This is a very clever way to add new powers to a basic tool without rewriting its core blueprint.

When an agent picks a tool, it creates a :term[Call]{canonical="Call"}—the filled-out order slip. This slip includes the required details, but you can also slap special stickers on it using meta-properties (prefixed with `_`). These properties tell the system's engine exactly how to process the order beyond the basic steps.

> Sidenote:
> - :term[004: Agent/Call]{href="./004_agent_call.md"}

This keeps the main blueprint simple and perfectly reusable. The :term[Call]{canonical="Call"} ends up containing *what* needs to be done (the inputs) and *how* to process it (the meta-properties). The final step is understanding the actual ways the order can be carried out.

## Knowing It vs. Doing It

Once the AI fills out the :term[Call]{canonical="Call"} order slip, the system has to actually do the work. Remember, the `Tool` blueprint is just a static description—it has no ability to run real code.

The work gets done in one of two ways. The default way is **latent execution**. This is like doing math in your head. The AI uses its own built-in intelligence to guess the result. This works beautifully for reading text, summarizing, or answering knowledge questions.

However, if the tool requires interacting with the real world—like checking a live weather API or saving a file to a database—it must be handed off to a strictly programmed computer function. This explicit code execution is called an **:term[Activity]{canonical="Activity"}**.

Keeping the abstract :term[Tool]{canonical="Tool"} blueprint separate from the solid :term[Activity]{canonical="Activity"} code is the secret to this design. It lets the AI understand a tool completely based on language, even while the development team drastically changes the hidden code running behind it. :term[003: Agent/Activity]{href="./003_agent_activity.md"} describes how :term[Activities]{canonical="Activity"} provide the actual programming muscles to make :term[Tool]{canonical="Tool"}s work.

> Sidenote:
> - :term[003: Agent/Activity]{href="./003_agent_activity.md"}.
