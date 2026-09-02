# 003: Agent/Activity

> [!DEFINITION] [Activity](./000_glossary.md)
> The actual code that does the work for a :term[Tool]{canonical="Tool"}. It is how the system performs real-world actions—like checking a database or calling another website—that the AI's brain cannot do on its own.

> Sidenote:
> - Requires: :term[002: Agent/Tool]{href="./002_agent_tool.md"}

The **Activity Protocol** defines how :term[Tool]{canonical="Tool"}s are connected to real, working code. A :term[Tool]{canonical="Tool"} is like a labeled button on a control panel. An :term[Activity]{canonical="Activity"} is the wiring behind the panel that actually makes the machine move.

## The Dual Registry Architecture

The system uses two separate lists to keep the button detached from the wiring:

- **:term[Tool Registry]{canonical="Tool"}**: Stores the description of the :term[Tool]{canonical="Tool"}s (what the button looks like).
- **:term[Activity Registry]{canonical="Activity"}**: Stores the actual code (:term[Activities]{canonical="Activity"}) that makes the :term[Tool]{canonical="Tool"}s work.

Separating these two things makes the system highly adaptable. You can use a :term[Tool]{canonical="Tool"} purely in the AI's imagination (where the AI guesses the answer), or you can plug in real code to do the job perfectly, all without changing what the :term[Tool]{canonical="Tool"} looks like to the AI.

## Activity Registration

When you create an :term[Activity]{canonical="Activity"}, you give it a unique name so it can snap onto a :term[Tool]{canonical="Tool"}. The code behind it takes in three pieces of information:

- **`call`**: The specific request (:term[Call]{canonical="Call"}). This holds all the instructions for the task, plus extra hidden rules (properties that start with `_`) that guide the process.
- **`tool`**: The description of the :term[Tool]{canonical="Tool"}. This lets the activity read its own instructions, like knowing exactly what shape the `_output` data should be.
- **`context`**: A carefully selected bundle of past messages from the system. It does not get everything—only what is allowed through the :term[Scopes]{canonical="Scope"} rules. This keeps the code focused on exactly what it needs.

An `Activity` can hand back different things when it finishes. If it returns a properly formatted `Message`, the system's :term[Execution Loop]{canonical="Execution Loop"} throws it straight into the conversation. This gives the code total control over its final answer. If it returns something simple like a number or a piece of text, the system neatly wraps it up into a message and puts it exactly where the original request asked it to go (the `_outputPath`).

::::columns
:::column{title="Activity Implementation"}

```typescript
// Register an Activity implementation.
// By convention, an Activity can be bound to a Tool of the same name.
// Types are automatically inferred from the Tool.
Activity.register('weatherCheck', async (call, tool, context) => {
  const data = await weatherAPI.get(call.location);
  return { temperature: data.temp, conditions: data.desc };
});
```

:::
:::column{title="Corresponding Tool Schema"}

```typescript
Tool.register('weatherCheck', {
  type: 'object',
  description: 'Gets the current weather for a location.',
  properties: {
    _tool: { type: 'string', const: 'weatherCheck' },
    location: { type: 'string' },
    _output: {
      type: 'object',
      properties: {
        temperature: { type: 'number' },
        conditions: { type: 'string' },
      },
      required: ['temperature', 'conditions'],
    },
  },
  required: ['location'],
});
```

:::
::::

## Execution Modes: Latent vs. Explicit

The system has two completely different ways to handle a :term[Tool]{canonical="Tool"} request (a :term[Call]{canonical="Call"}):

- **Latent Execution**: The AI uses its own brain. It thinks through the problem and invents the answer immediately. This is what happens by default if no actual code (:term[Activity]{canonical="Activity"}) is hooked up to the :term[Tool]{canonical="Tool"}.
  > Sidenote:
  > - :term[104: Concept/Latent]{href="./104_concept_latent.md"}
- **Explicit Execution**: The system runs real, hardcoded logic. An :term[Activity]{canonical="Activity"} runs to lock down the exact mathematical or real-world answer. You need this to interact safely with the outside world (like checking a live website) or doing precise math.

### A Latent Step Settles on the Turn That Authors It

When we say the AI invents the answer "immediately," we mean it. This speed is why using the AI's brain (latent execution) is so powerful.

If a :term[Call]{canonical="Call"} has no `_activity` attached to it, the AI simply fills in the `_output` **right as it asks the question**. There is no waiting for external code to run. The system writes that answer at the :term[Output Path]{canonical="Output Path"}, which means **other steps in the exact same cycle can use that answer right away**.

Imagine you need the AI to decide which of five paragraphs sounds the most positive. You don't need external code for that. The AI can make that judgment and supply the answer in a single step. Latent execution isn't just a placeholder for unwritten code; it is a way to get the AI to evaluate something quickly, turning thoughts into a solid **value** other steps can read without having to guess what the AI meant later.

> Sidenote:
> - :term[011: Agent/Expressions]{href="./011_agent_expressions.md"} — a computed parameter usually grabs this fast AI answer on the exact same turn.
>
> If the AI asks for a latent :term[Call]{canonical="Call"} but does not fill in the `_output` right away, nothing happens on this turn. The request gets pushed to the next cycle for the AI to answer later. That costs you an entire turn. To avoid slowing down, always fill in the `_output` immediately if you already have the information, and only leave it blank if the data doesn't exist yet.

## Activity Resolution Strategy

The system works out how to run a :term[Tool]{canonical="Tool"} automatically using a simple set of rules. It looks at the `_activity` label on the :term[Tool]{canonical="Tool"} to decide what to do:

1.  **Written Rule**: If the :term[Tool]{canonical="Tool"} explicitly lists a name in its `_activity` label, the system grabs the matching :term[Activity]{canonical="Activity"} code.
2.  **Matching Names (Recommended)**: If the `_activity` label is completely blank, the system checks if there is code registered with the **exact same name** as the :term[Tool]{canonical="Tool"}. If they match, they automatically snap together.
3.  **AI Brain Fallback**: If it cannot find matching code, the system leaves the `_activity` field blank. This signals that the AI should just figure out the answer on its own (latent execution).

This completely automatic matching makes building easy:

- **To save time, give your :term[Activity]{canonical="Activity"} the exact same name as your :term[Tool]{canonical="Tool"}.**
- Any :term[Tool]{canonical="Tool"}s without matching code will safely fallback to the AI figuring it out.
- You can still connect one piece of code to multiple :term[Tool]{canonical="Tool"}s by typing its name manually in the `_activity` label.

## Interactions with Other Systems

An :term[Activity]{canonical="Activity"} does not work all alone. It needs information from other parts of the system to do its job safely and accurately.

- **:term[Call]{canonical="Call"}:** The request gives the :term[Activity]{canonical="Activity"} everything it needs to begin. It includes hidden properties like `_outputPath` to tell it exactly where to save the final answer, or `_instance` to tell it which exact file to look at in a massive list. This lets simple code handle heavy workflows.

  > Sidenote:
  > - :term[004: Agent/Call]{href="./004_agent_call.md"}
  > - :term[008: Agent/Output]{href="./008_agent_output.md"}
  > - :term[013: Agent/Instancing]{href="./013_agent_instancing.md"}

- **:term[Scopes]{canonical="Scope"}:** This acts like a security guard for the `context`. The `_scopes` rule decides exactly which previous messages (like :term[State]{canonical="State"} or :term[Input]{canonical="Input"}) the code is allowed to see. This stops the code from getting distracted by reading the entire history of the conversation, forcing it to just do its job.

  > Sidenote:
  > - :term[015: Agent/Scopes]{href="./015_agent_scopes.md"}

- **Resolving Branching Paths:** Sometimes, a plan can go in two different directions based on whether it succeeds or fails. The `_outputPath` handles this automatically. If an `Activity` gets a rule like `_outputPath: '†state.success <|> †state.error'`, the code figures out what happened, and then builds a response shaped perfectly to jump down the correct path. This gives the code absolute power over steering the ship.

## Why Separate Activities Matter

If we permanently glued the :term[Tool]{canonical="Tool"} (the button) and the :term[Activity]{canonical="Activity"} (the wiring) together, you could never upgrade one without ripping apart the other. If you wanted to swap an AI's guesswork for a live website link, you would have to rebuild every robot using that tool.

Keeping two separate lists fixes this. The AI always presses the same familiar :term[Tool]{canonical="Tool"} button, while you are totally free to change the wiring behind the wall. This means:

- **Upgrades don't break things**: You can swap out the AI's guesswork for real code without changing a single line of the AI's instructions.
- **Easy AI vs API tests**: You can race the AI against your own external code to see which finds the answer faster and more clearly.
- **Safe testing**: You can hand the new code to just a few systems while the rest safely use the old backup method.

## From Definition to Action

Splitting the "what" (:term[Tool]{canonical="Tool"}) from the "how" (:term[Activity]{canonical="Activity"}) gives the system massive flexibility. But building the parts is only the first step. The final piece is orchestration: organizing these :term[Call]{canonical="Call"}s, running them, and lining them up in the perfect order.

:term[004: Agent/Call]{href="./004_agent_call.md"} dives into the rules that drive this engine, turning blank definitions into real, moving parts.
