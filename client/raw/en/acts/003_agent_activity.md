# 003: Agent/Activity

> [!DEFINITION] [Activity](./000_glossary.md)
> An explicit, asynchronous function that implements the logic for a :term[Tool]{canonical="Tool"}. It is the mechanism for executing actions that require external API calls, database operations, or any task that cannot be handled by an LLM's latent space.

> Sidenote:
>
> - Requires: :term[002: Agent/Tool]{href="./002_agent_tool.md"}

The **Activity Protocol** defines how :term[Tool]{canonical="Tool"}s are backed by concrete, executable code. While a :term[Tool]{canonical="Tool"} defines a capability's interface, an :term[Activity]{canonical="Activity"} provides its implementation.

## The Dual Registry Architecture

The agent system employs two complementary registries to separate a capability's interface from its implementation:

- **:term[Tool Registry]{canonical="Tool"}**: Stores the schema definitions for :term[Tool]{canonical="Tool"}s.
- **:term[Activity Registry]{canonical="Activity"}**: Stores the explicit `async` code functions (:term[Activities]{canonical="Activity"}) that implement :term[Tool]{canonical="Tool"}s.

This separation is the key to the system's flexibility. It allows :term[Tool]{canonical="Tool"}s to be defined and used in a latent-only mode (where the LLM generates the output directly) and allows for different implementations of an :term[Activity]{canonical="Activity"} to be swapped in without changing the :term[Tool]{canonical="Tool"}'s interface (e.g., for different environments like development and production).

## Activity Registration

An :term[Activity]{canonical="Activity"} is registered with a unique name, which is used to bind it to a :term[Tool]{canonical="Tool"}. The handler is an `async` function that receives three arguments:

- **`call`**: The concrete :term[Call]{canonical="Call"} object. This contains all parameters for the tool, including any meta-properties (prefixed with `_`) that guide execution.
- **`tool`**: The schema definition of the :term[Tool]{canonical="Tool"} being executed. This allows the activity to inspect its own interface, such as the expected `_output` schema.
- **`context`**: An array of messages selectively imported from the parent environment. This is not the full context of the parent, but a limited view controlled by the :term[Scopes]{canonical="Scope"} protocol, making contextual information an explicit, opt-in feature.

An `Activity`'s return value is flexible. If it returns a `Message` object (such as a `Data Message`), the :term[Execution Loop]{canonical="Execution Loop"} will append it to the context directly. This gives the `Activity` full control over its output, allowing it to construct the precise message needed for the workflow. If any other value is returned (e.g., a raw object or string), the loop will automatically wrap it in a `Data Message`, using the `_outputPath` from the original call to determine where to write the result.

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

The system supports two fundamentally different execution modes for a :term[Tool]{canonical="Tool"}'s :term[Call]{canonical="Call"}:

- **Latent Execution**: Uses the LLM's reasoning capabilities. The agent "thinks through" the problem and produces the output directly in the same invocation. This is the default mode when no :term[Activity]{canonical="Activity"} is found for a :term[Tool]{canonical="Tool"}.
  > Sidenote:
  >
  > - :term[104: Concept/Latent]{href="./104_concept_latent.md"}
- **Explicit Execution**: Delegates the :term[Call]{canonical="Call"} to deterministic code. An :term[Activity]{canonical="Activity"} function is invoked to compute the output. This is essential for interacting with the outside world (e.g., APIs, databases) or for tasks requiring precise, repeatable logic.

### A Latent Step Settles on the Turn That Authors It

"In the same invocation" is stronger than it reads, and it is what makes a latent step worth reaching for after the prototype is over.

A :term[Call]{canonical="Call"} whose `_activity` is empty carries an `_output` **the model fills in the same response that authors the call**. Nothing is dispatched, nothing is awaited — the answer arrived with the request for it. The loop writes that output at the call's :term[Output Path]{canonical="Output Path"} inside the same drain, and rescanning is what the drain already does, so **a sibling call reading that path becomes ready on the same turn**.

So a judgement with no code behind it — which of these passages read as positive, which of these documents is the endorsement — and everything computed from that judgement can be settled in one request rather than three. A latent step is not only the placeholder for an :term[Activity]{canonical="Activity"} nobody has written; it is how a run gets a reading it could not have computed, as a **value** other steps can read rather than as prose somebody has to interpret next turn.

> Sidenote:
>
> - :term[011: Agent/Expressions]{href="./011_agent_expressions.md"} — a computed parameter is the usual consumer of a latent output on the same turn.
>
> A latent :term[Call]{canonical="Call"} with no `_output` is not executable this turn: the loop never offers it for dispatch, and it rides into the next turn's :term[Plan]{canonical="Plan"} for the model to answer then. That is the cross-turn form, and it costs a turn — so fill `_output` on the turn you author the call whenever you already hold what it needs, and leave it out only when its input does not exist yet.

## Activity Resolution Strategy

The system uses a zero-configuration strategy to determine which execution mode to use when a :term[Tool]{canonical="Tool"} is called. The `_activity` field in a :term[Tool]{canonical="Tool"}'s schema signals the intent to use an explicit implementation. This field is resolved automatically during schema composition based on the following rules:

1.  **Explicit `_activity` Field**: If the :term[Tool]{canonical="Tool"} definition itself includes a non-empty `_activity` string, that value is used to look up the :term[Activity]{canonical="Activity"} in the registry.
2.  **Same-Name Convention (Recommended)**: If no `_activity` field is present on the :term[Tool]{canonical="Tool"}, the system checks if an :term[Activity]{canonical="Activity"} has been registered with the **same name** as the :term[Tool]{canonical="Tool"}. If found, the `_activity` field is automatically set to the :term[Tool]{canonical="Tool"}'s name.
3.  **Latent Fallback**: If no matching :term[Activity]{canonical="Activity"} is found by the above rules, the `_activity` field is set to an empty string, signaling that the :term[Call]{canonical="Call"} should be executed latently.

This convention-based approach simplifies development:

- **For zero-configuration, register your :term[Activity]{canonical="Activity"} under the same name as your :term[Tool]{canonical="Tool"}.**
- :term[Tool]{canonical="Tool"}s without a corresponding :term[Activity]{canonical="Activity"} will automatically and safely default to latent execution.
- An explicit `_activity` field in a :term[Tool]{canonical="Tool"} schema will always take precedence, allowing a single :term[Activity]{canonical="Activity"} to implement multiple :term[Tool]{canonical="Tool"} interfaces.

## Interactions with Other Systems

An :term[Activity]{canonical="Activity"} does not execute in isolation. It relies on other protocols to receive its parameters and contextual information in a structured and secure way.

- **:term[Call]{canonical="Call"}:** The `call` argument provides the :term[Activity]{canonical="Activity"} with the complete invocation object. This is more than just a simple dictionary of parameters; it's a rich structure that can include meta-properties like `_outputPath` to direct where the result should be stored or `_instance` to target a specific item in a batch process. This allows an :term[Activity]{canonical="Activity"} to participate in complex, stateful workflows.

  > Sidenote:
  >
  > - :term[004: Agent/Call]{href="./004_agent_call.md"}
  > - :term[008: Agent/Output]{href="./008_agent_output.md"}
  > - :term[013: Agent/Instancing]{href="./013_agent_instancing.md"}

- **:term[Scopes]{canonical="Scope"}:** The `context` argument is populated via the :term[Scopes]{canonical="Scope"} protocol. The `_scopes` property on a :term[Call]{canonical="Call"} acts as an allow-list, specifying which messages from the parent environment (like :term[State]{canonical="State"} or :term[Input]{canonical="Input"}) should be passed to the :term[Activity]{canonical="Activity"}. This provides a controlled and explicit mechanism for an :term[Activity]{canonical="Activity"} to access the information it needs without being exposed to the entire, potentially noisy, parent context.

  > Sidenote:
  >
  > - :term[015: Agent/Scopes]{href="./015_agent_scopes.md"}

- **Resolving Branching Paths:** The ability for an `Activity` to return a full `Message` is the key to proactive error handling and other complex branching logic. The `_outputPath` on a `call` now serves as a contract or guide. When an `Activity` receives a call with a branching expression like `_outputPath: '†state.success <|> †state.error'`, its internal logic can determine the outcome and then construct and return the correct `Data Message` to write the result to the chosen path (e.g., a message containing `{ data: { success: { ... } } }`). This gives deterministic code full control over directing the flow of the plan.

## Why Separate Activities Matter

Without separating :term[Tool]{canonical="Tool"} schemas from :term[Activity]{canonical="Activity"} implementations, the definition of a capability would be permanently tied to its execution logic. To switch from an LLM-based implementation to an external API, one would need to find and modify every agent that uses that :term[Tool]{canonical="Tool"}.

The dual registry architecture solves this by keeping :term[Tool]{canonical="Tool"} interfaces stable while allowing their underlying implementations to evolve. Agents interact with a consistent :term[Tool]{canonical="Tool"} schema, regardless of whether it is executed latently by an LLM or explicitly by an :term[Activity]{canonical="Activity"}. This means:

- **Implementation changes don't break agents**: You can switch from latent to explicit execution without touching the agent's code.
- **A/B testing execution strategies**: You can compare the performance of LLM reasoning versus an external API for the same capability.
- **Gradual rollouts**: You can deploy new :term[Activity]{canonical="Activity"} implementations to a subset of agents while others continue to use the old one or the latent fallback.

## From Definition to Action

By separating the "what" (:term[Tool]{canonical="Tool"}) from the "how" (:term[Activity]{canonical="Activity"}), the system gains flexibility. But this is only part of the story. With interfaces and implementations defined, the final piece is orchestration: how these :term[Call]{canonical="Call"}s are managed, executed, and sequenced.

:term[004: Agent/Call]{href="./004_agent_call.md"} explores the protocol that governs this execution, turning abstract definitions into concrete, stateful actions.
