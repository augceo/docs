# 013: Agent/Delegate

> [!DEFINITION] [Delegate](./000_glossary.md)
> A protocol for isolating execution context. Invoked by a :term[Call]{canonical="Call"}'s `_delegate` property, it executes an :term[Activity]{canonical="Activity"} or a new :term[Request]{canonical="Request"} in an isolated environment, with the `_scopes` property providing controlled access to the parent context.

> Sidenote:
>
> - Requires:
>   - :term[004: Agent/Call]{href="./004_agent_call.md"}
> - Complemented by:
>   - :term[015: Agent/Scopes]{href="./015_agent_scopes.md"}

The **Delegation** pattern addresses the challenge of scaling and composing agent capabilities. It provides a mechanism for executing :term[Tools]{canonical="Tool"} in sandboxed contexts, preventing context bleeding and enabling true reusability. By delegating a :term[Call]{canonical="Call"} to an external delegate—either another :term[Request]{canonical="Request"} definition or an :term[Activity]{canonical="Activity"} in a sub-request—the system can build complex agentic behaviors from self-contained, independently developed components.

## Implementation as an Activity

Conceptually, delegation is not a new or separate protocol, but an application of the existing :term[Activity]{canonical="Activity"} system. The execution of a delegate is handled by a special, generic "catch-all" :term[Activity]{canonical="Activity"} that is registered by the system to handle any :term[Call]{canonical="Call"} containing the `_delegate` property.

This `Delegate Activity` receives the standard three arguments:

- **`call`**: The full call object, including the parameters intended for the sub-request.
- **`tool`**: The tool schema, from which the activity reads the `_delegate` property to know _what_ to invoke.
- **`context`**: The array of messages scoped from the parent, which will be forwarded to the sub-request.

The activity's logic is simple: it uses these three arguments to assemble and issue a new, isolated :term[Request]{canonical="Request"}. This implementation demonstrates the utility of the core protocols: by making the `Activity` signature so robust, even complex patterns like delegation can be built on top of it as a standard implementation rather than requiring a new set of rules.

## The Problem: Monolithic Tools and Context Bleeding

As agent capabilities grow, defining all :term[Tools]{canonical="Tool"} within a single, monolithic context becomes untenable.

1.  **Large Schemas**: LLMs have practical limits on the complexity of schemas they can handle in a single request. Combining numerous complex :term[Tools]{canonical="Tool"} can exceed these limits, preventing the LLM from correctly processing the available options.
2.  **Context Bleeding**: When all :term[Tools]{canonical="Tool"} operate in the same context, the LLM can be influenced by irrelevant information, leading to incorrect :term[Tool]{canonical="Tool"} selection or parameter filling.
3.  **Lack of Reusability**: A :term[Tool]{canonical="Tool"} defined for one agent is not easily portable to another without bringing its entire context with it.

Delegation solves these problems by introducing **Delegated Isolation**, a way to delegate a :term[Call]{canonical="Call"} to an external, isolated execution environment.

## Invoking a Delegate

Delegated execution is signaled by the `_delegate` property within a :term[Tool]{canonical="Tool"}'s schema. This property instructs the system to treat the :term[Call]{canonical="Call"} not as an inline operation, but as a request to an external delegate.

The `_delegate` property is a `string` and can be used in two ways:

- **Reference a saved :term[Request]{canonical="Request"}**: The string can be a path or URL to a JSON file that defines a self-contained :term[Request]{canonical="Request"}—a JSON object containing `context` and `schema` properties. This allows a :term[Tool]{canonical="Tool"} to delegate its execution to a completely separate set of instructions.

  > Sidenote:
  >
  > A saved, reusable :term[Request]{href="./001_agent_request.md"} is the most common form of an :term[Idea]{canonical="Idea"}. The Delegation pattern is the primary mechanism for composing these :term[Ideas]{canonical="Idea"} into more complex systems. See :term[101: Concept/Idea]{href="./101_concept_idea.md"} for details.

- **Create an anonymous delegate**: A string literal `'anonymous'` signals an anonymous delegate. This is used to create a fresh, isolated execution environment for a :term[Tool Call]{canonical="Tool Call"} (whether latent or explicit) without needing an intermediate JSON file. It automatically scopes a new, empty context for the :term[Call]{canonical="Call"}.

## Execution in an Isolated Environment

A delegate provides a "clean room" for execution. Instead of running inside the parent agent's bustling context, the :term[Call]{canonical="Call"} is processed in a new, self-contained session. The context for this sub-request is carefully constructed, not inherited.

This is where the :term[Scoped context]{canonical="scope" href="./015_agent_scopes.md"} becomes critical. The `_scopes` property on the :term[Tool]{canonical="Tool"} schema acts as a bridge, explicitly declaring which pieces of the parent context should be "imported" into the delegate's dedicated workspace. This gives the parent agent precise control over what the delegate can see, preventing context bleeding and enabling true encapsulation.

> Sidenote:
>
> - :term[015: Agent/Scopes]{href="./015_agent_scopes.md"}

## Handling Large Schemas

Delegation also provides a solution for managing :term[Tools]{canonical="Tool"} with very large or complex output schemas. Instead of including a massive `_output` schema in the main request—potentially crowding out other tools—a :term[Tool]{canonical="Tool"} can be defined with only its `input` parameters and a `_delegate` pointer.

The LLM can plan the :term[Call]{canonical="Call"} with just the input, and the complex output will be generated within the delegate's isolated sub-request. This allows an agent to reason about a sequence of complex operations without needing to "see" the entire, detailed schema for every step in a single context window. The LLM trusts that the delegate will produce the correct output, which it will receive and use in subsequent steps.

## Delegate Resolution Strategies

A :term[Tool]{canonical="Tool"} becomes a :term[Delegate]{canonical="Delegate"} simply by including the `_delegate` property in its schema. This signals that the :term[Call]{canonical="Call"} should be delegated. The key question is _when_ this delegation is resolved. The system supports two strategies, allowing a trade-off between strict safety and dynamic flexibility.

### 1. Execution-Time Resolution (Default)

The default and most flexible approach is to resolve the delegate at **execution time**, after an agent has already generated a :term[Call]{canonical="Call"}.

This method enables a paradigm that is not possible with traditional code: **the LLM acts as an intelligent glue layer.** An agent can generate a :term[Call]{canonical="Call"} with parameters that don't perfectly match the delegate's expected `input` schema. At execution time, the system assembles the delegate's context and the caller's provided input, and the LLM in the sub-request is tasked with bridging the gap.

This is a significant advantage, as it allows delegates to be updated and evolve independently. Even if a delegate changes its input structure, calling agents won't immediately break. The LLM will attempt to adapt the old :term[Call]{canonical="Call"} format to the new `input` schema, providing a level of resilience and loose coupling that is unique to this architecture.

The process is as follows:

1.  An agent generates a :term[Call]{canonical="Call"} to the modular :term[Tool]{canonical="Tool"}.
2.  The executor sees the `_delegate` property and initiates the delegation process.
3.  **Context Assembly**: The executor fetches the delegate definition file (if not anonymous) and assembles the base context. It then uses `_scopes` to append the caller's context.
4.  **Input Mapping**: The `params` from the :term[Call]{canonical="Call"} are packaged into an :term[Input Message]{canonical="Input Message"} and added to the context. This is where the LLM's "glue" capability comes into play, as it will use this input to fulfill the delegate's logic, even if the schemas don't align perfectly.
5.  **Execution**: A new, isolated :term[Request]{canonical="Request"} is made with the combined context. The result is returned as the output of the original :term[Call]{canonical="Call"}.

### 2. Upfront Resolution (Optional)

For scenarios requiring stricter guarantees, a delegate can be resolved **upfront**, before the initial :term[Request]{canonical="Request"} is sent to the agent.

In this mode, the system pre-fetches the delegate :term[Request]{canonical="Request"} definition and merges its `input` schema with the :term[Tool]{canonical="Tool"}'s parameter schema. This allows the agent's LLM to see the delegate's exact requirements from the start, ensuring that the generated :term[Call]{canonical="Call"} is perfectly formed and type-safe. Crucially, this upfront merge can also include the delegate's `_output` schema, providing a strict contract for the expected result.

This approach provides the safety of traditional API contracts, where both the inputs and outputs are known and validated. It sacrifices the flexibility of execution-time resolution and is best used for critical, well-defined integrations where loose coupling is not a desired feature.

:::::details{title="Example: Flexible Input Mapping at Execution Time"}

This example demonstrates the "LLM as glue" concept, where a delegate can be successfully executed even if the calling agent's :term[Call]{canonical="Call"} doesn't perfectly match the delegate's input schema. This is the default, execution-time resolution behavior.

::::columns
:::column{title="Caller's Perspective"}

An orchestrator agent needs to send a message. It knows about a `sendMessage` :term[Tool]{canonical="Tool"} that delegates to an external agent, identified by a URL. Based on its own context, it generates a :term[Call]{canonical="Call"} with `userId` and `text` parameters, without knowing the delegate's internal requirements.

```json
// CALL GENERATED BY THE ORCHESTRATOR
{
  "_tool": "sendMessage",
  "_delegate": "http://example.com/agents/speaker_EN",
  "userId": "u_123",
  "text": "Hello, world!"
}
```

:::
:::column{title="Delegate's Perspective & Final Context"}

The `speaker_EN` delegate is a separate :term[Request]{canonical="Request"} definition. At execution time, the system packages the caller's parameters into the `input` property of an :term[Input Message]{canonical="Input Message"}. Critically, it also includes the delegate's own `schema`, which does not match the provided `input`. The delegate's LLM is now responsible for bridging this semantic gap, intelligently mapping `userId` to `recipientId` and `text` to `messageBody`. This transformation isn't programmatic; it's a semantic mapping that occurs within the LLM's latent space.

```json
// FINAL CONTEXT FOR THE SUB-REQUEST
[
  {
    "type": "system",
    "message": "You are an expert in messaging in English."
  },
  {
    "type": "input",
    // This is the raw input provided by the calling agent.
    "input": {
      "userId": "u_123",
      "text": "Hello, world!"
    },
    // This is the expected input format for the  delegate
    "schema": {
      "type": "object",
      "properties": {
        "recipientId": { "type": "string" },
        "messageBody": { "type": "string" }
      }
    }
  }
]
```

:::
::::
:::::

:::details{title="Example: Music makers"}

Delegates enable composition by allowing :term[Request]{canonical="Request"} definitions to act as standalone services that can be orchestrated by other agents. This creates a clear, dynamic hierarchy: high-level agents can focus on orchestration while delegating specialized tasks to low-level, reusable delegates.

Consider a workflow with two specialist delegates: a **`Composer`** and a **`Sound-Designer`**.

- The **`Sound-Designer`** is a low-level expert. It's a self-contained :term[Request]{canonical="Request"} definition focused on the physics of sound, knowing how to operate synthesizers to produce specific audio data.

- The **`Composer`** is a mid-level specialist. Its primary job is to create a song. It uses its own inline tools to generate a melody and musical structure. To realize this vision, it then makes :term[Calls]{canonical="Call"} to the `Sound-Designer` delegate to synthesize the actual sounds.

This two-layer hierarchy is a common pattern. However, the true power of delegates lies in their dynamic, task-driven composition.

Now, let's introduce a high-level **`Producer`** agent. The `Producer`'s goal is to create a finished record. Based on the specific task, the `Producer` can orchestrate its delegates in different ways:

> Sidenote:
>
> This arrangement allows for flexible orchestration. A high-level `Producer` can delegate to a `Composer`, who in turn uses a `Sound-Designer`. However, the `Producer` can also bypass the `Composer` and interact with the `Sound-Designer` directly for specific tasks.
>
> ```mermaid
> graph TD
>     Producer --> Composer
>     Producer --> SoundDesigner(Sound-Designer)
>     Composer --> SoundDesigner(Sound-Designer)
> ```

- **Hierarchical Orchestration**: For creating a song, the `Producer` might make a single :term[Call]{canonical="Call"} to the `Composer` delegate. The `Producer` provides high-level direction ("I need a sad ballad"), and the `Composer` executes its entire internal workflow, including its own nested :term[Calls]{canonical="Call"} to the `Sound-Designer`. The `Producer` doesn't need to know about the `Sound-Designer`'s existence in this case.

- **Parallel Orchestration**: If the `Producer` also needs specific sound effects for the record (like foley or an ambient soundscape), it can make :term[Calls]{canonical="Call"} directly to the `Sound-Designer` delegate for those tasks, in parallel with its :term[Call]{canonical="Call"} to the `Composer`.

This demonstrates the key principle: the composition is not fixed within the tools themselves. The `Producer` can choose to treat the `Composer` as a black box or to interact with its constituent parts (`Sound-Designer`) directly, all depending on the needs of the moment. This flexibility allows the same set of expert delegates to be combined in various arrangements, creating a deeply composable and emergent system.

:::

## From Delegation to Scopes

Delegated execution provides an isolated environment, but for it to be useful, a delegate needs a way to receive information from its parent. The `_scopes` property provides this mechanism, acting as a controlled bridge between contexts. The specifics of how this bridge works are defined by the :term[Scoped context]{canonical="scope" href="./015_agent_scopes.md"} pattern.
