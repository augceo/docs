# 004: Agent/Call

> [!DEFINITION] [Call](./000_glossary.md)
> A concrete, executable instance of a :term[Tool]{canonical="Tool"}, with specific values for its `params`. It's an invocation-focused request for what _should be done_.

> Sidenote:
>
> - Requires:
>   - [002: Agent/Tool](./002_agent_tool.md)
> - Enables:
>   - [103: Concept/Ideator](./103_concept_ideator.md)
>   - [015: Agent/Scopes](./015_agent_scopes.md)
>   - [013: Agent/Instancing](./013_agent_instancing.md)
>   - [202: Idea/Vessel](./202_idea_vessel.md)
>   - [203: Idea/Process](./203_idea_process.md)
> - Extended by:
>   - [014: Agent/Delegate](./014_agent_delegate.md)

The :term[Tool]{canonical="Tool"} protocol establishes the foundational schema-driven interface that enables agents to understand structured capabilities, and the **:term[Call]{canonical="Call"}** protocol builds upon :term[Tools]{canonical="Tool"} to define how execution happens.

A :term[Call]{canonical="Call"} is a concrete instance of a :term[Tool]{canonical="Tool"} with specific parameter values, ready for execution. Where :term[Tools]{canonical="Tool"} define _what can be done_, :term[Calls]{canonical="Call"} define _how it gets executed_.

> [!HEADSUP] Heads up
> A :term[Request]{href="./001_agent_request.md"} that results in a set of :term[Calls]{canonical="Call"} is a :term[Vessel]{href="./202_idea_vessel.md"}. A :term[Vessel]{canonical="Vessel"} represents a single, reactive moment of decision-making where an agent selects from available :term[Tools]{canonical="Tool"} to form a response.
>
> > Sidenote:
> >
> > - [001: Agent/Request](./001_agent_request.md)
> > - [202: Idea/Vessel](./202_idea_vessel.md)

## Call Results and Final Output

> [!DEFINITION] [Final Output](./000_glossary.md)
> The terminal `output` field in a :term[Solution]{canonical="Solution"} object that fulfills the goal schema of an entire agent execution loop. It is generated only when the agent determines its task is complete.

When a :term[Request]{canonical="Request"} is made, the goal is to receive a **:term[Final Output]{canonical="Final Output"}**. The LLM analyzes the available context and makes a decision:

- If the context contains sufficient information, it generates the :term[Final Output]{canonical="Final Output"} directly, completing the process.
- If the context is insufficient, the LLM determines that more information is needed. It nullifies the `output` and instead generates a set of :term[Calls]{canonical="Call"} to gather the missing data.

After these :term[Calls]{canonical="Call"} are executed, their results are added to the context. A new :term[Request]{canonical="Request"} is then made with this enriched context, giving the LLM another opportunity to produce the :term[Final Output]{canonical="Final Output"}. This iterative cycle of requests and calls continues until the goal is achieved.

> Sidenote:
>
> - [010: Agent/Loop](./010_agent_loop.md)

This clear separation between intermediate results (handled via :term[Output Path]{canonical="Output Path"}) and the final, goal-oriented output is what enables the construction of complex, multi-step agents.

## Interactions with other systems

A :term[Call]{canonical="Call"} by itself is a simple data structure. Its utility comes from its composition with other protocols that manage its execution environment. These protocols are activated by special meta-properties (prefixed with `_`) within a :term[Tool]{canonical="Tool"}'s schema, allowing a single :term[Call]{canonical="Call"} object to trigger a variety of execution behaviors.

By giving these meta-properties clear semantic meaning, we enable the LLM to be an active participant in the composition. It can reason about the various permutations of these properties to construct complex and novel execution chains, moving beyond simple tool selection to dynamic workflow orchestration. This interaction between :term[Calls]{canonical="Call"} and the various execution protocols is the foundation of the agent's ability to perform complex, multi-step tasks.

> [!TIP]
> The following section explains connections to other parts of the system that will be detailed later. There's no need to jump ahead, as we'll cover everything in a logical sequence. You can come back to this section later.

- **Explicit Execution (`_activity`)**: The most fundamental extension is connecting a :term[Call]{canonical="Call"} to a deterministic code function. The `_activity` property signals that the :term[Call]{canonical="Call"} should be executed by an :term[Activity]{canonical="Activity"} rather than by the LLM's latent reasoning.

  > Sidenote:
  >
  > - [003: Agent/Activity](./003_agent_activity.md).

- **Delegated Execution (`_delegate`)**: A :term[Call]{canonical="Call"} can be delegated to an external :term[Delegate]{canonical="Delegate"}. The `_delegate` property typically contains a reference to a saved :term[Request]{canonical="Request"} (often as a path to a JSON file), allowing that request to be invoked as a reusable tool. This provides an isolated "clean room" for execution, preventing context bleeding and enabling true encapsulation.

  > Sidenote:
  >
  > - [014: Agent/Delegate](./014_agent_delegate.md).

- **Context Bridging (`_scopes`)**: The :term[Scopes]{canonical="Scope"} protocol controls what context is available to a :term[Call]{canonical="Call"}. Its primary use is to focus an LLM's attention during a latent execution by specifying which parts of the parent context it should consider. This prevents context bleeding and leads to more reliable outputs. When used with a `_delegate`, its role becomes even more powerful: it strictly defines the _entire_ context for the delegate's isolated execution.

  > Sidenote:
  >
  > - [015: Agent/Scopes](./015_agent_scopes.md).

- **Stateful Execution (`_outputPath`)**: A :term[Call]{canonical="Call"} can be made stateful by instructing it where to write its output. The :term[Output Path]{canonical="Output Path"} property specifies a path within a persistent :term[State]{canonical="State"} where the result of the :term[Call]{canonical="Call"} should be stored. This allows for the creation of multi-step workflows where the output of one :term[Call]{canonical="Call"} can be used as the input for another.

  > Sidenote:
  >
  > - [009: Agent/State](./009_agent_state.md).
  > - [008: Agent/Output](./008_agent_output.md).

- **Instanced Execution (`_instance`)**: A :term[Call]{canonical="Call"} can be directed to operate on a specific item within a batch process, allowing a single request to manage multiple data contexts in parallel. The `_instance` property acts as a unique identifier, focusing all operations of that :term[Call]{canonical="Call"} (like reading inputs from and writing outputs to the :term[State]{canonical="State"}) on a particular context. This enables efficient, parallel processing of multiple states with the same set of tools.
  > Sidenote:
  >
  > - [013: Agent/Instancing](./013_agent_instancing.md).

## From Actions to Data

While :term[Calls]{canonical="Call"} define what an agent does, they are only one half of the equation. The other half is the structured data that flows into and out of them. A :term[Call]{canonical="Call"} needs structured data to operate on, and its results must be stored in a structured way for the next :term[Call]{canonical="Call"} to use.

:term[005: Agent/Data]{href="./005_agent_data.md"} describes the foundational protocol for structuring and managing this data within the agent's context.
