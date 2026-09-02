# 202: Idea/Vessel

> [!DEFINITION] [Vessel Idea](./000_glossary.md)
> A self-contained `Idea` that is both the definition of a reactive capability and the record of its chosen reaction. Its `schema` defines the full universe of possible `Tools`, and its `solution` captures the specific `Calls` (instances of those `Tools`) that were chosen in response to a stimulus.

> Sidenote:
>
> - Requires:
>   - [101: Concept/Idea](./101_concept_idea.md)
>   - [002: Agent/Tool](./002_agent_tool.md)
>   - [004: Agent/Call](./004_agent_call.md)

A **Vessel Idea** is a complete, self-contained snapshot of an agent's immediate reaction to a stimulus. It is more than just a log of what happened; it is a rich artifact that contains the full context of the decision, making it auditable, adaptable, and reusable.

## 1. The Anatomy of a Reaction

A `Vessel Idea` uses the core `Idea` triplet to capture both the potential for action and the action itself:

> Sidenote:
>
> - [101: Concept/Idea](./101_concept_idea.md)

- **`context` (The Stimulus):** This contains the input that triggered the agent's reaction.
- **`schema` (The Definition):** This defines the agent's entire reactive capability in that moment. It describes every `Tool` the agent _could have_ used, typically as a `oneOf` schema.
- **`solution` (The Instance):** This is the record of what the agent _actually did_. It's an array containing the specific `Calls`—concrete instances of the `Tools` from the `schema`—that were chosen and executed.

Because a `Vessel` contains both the universe of possibilities (`schema`) and the specific outcome (`solution`), it provides a complete picture of a decision-making event.

## 2. Self-Contained Decisions

This self-contained structure enables the system's capabilities. By packaging the definition with the instance, a `Vessel Idea` inherently supports:

- **Human Verification and Adjustment:** A person can review a `Vessel` and see not only what the agent did, but what it _could have_ done. They can then approve the action, or modify the `solution` by choosing different `Calls` from the provided `schema`.
- **Replanning:** If a `Call` fails, the agent can re-evaluate the situation. It already has the full `context` and the complete `schema` of its options, allowing it to easily select an alternative `Call` and generate a new `Vessel`.
- **Reusability:** A `Vessel` can be used as a template or example for future decisions in similar contexts.

### 2.1. Interactive Time Travel

Because a `Vessel Idea` is a complete, self-contained, and immutable snapshot, it enables a form of interactive time travel. By loading a `Vessel` from the past, one can interact with the agent exactly as it existed at that moment.

The `schema` guarantees that the agent's capabilities are frozen in time; even if the live agent has since been upgraded with new `Tools`, the historical `Vessel` will only present the options that were available when it was created. This allows a user or another agent to:

- **Replay History:** Perfectly reconstruct a past decision.
- **Explore Alternatives:** Provide a different `context` to the historical `Vessel` to run "what-if" scenarios, exploring how that past version of the agent would have reacted to different inputs.

This provides a mechanism for debugging, auditing, and understanding the evolution of an agent's behavior over time.

## 3. The Vessel in the Execution Loop

A `Vessel` represents a single, tactical tick of the execution loop. It is the decision made and recorded within one iteration. The results of its `Calls` are then fed back into the `context` for the next tick, potentially leading to a new `Vessel`.

> Sidenote:
>
> - [010: Agent/Loop](./010_agent_loop.md)

This reactive pattern is powerful for building agents that can respond intelligently to real-time events. However, it is not designed for orchestrating complex, multi-step tasks that require memory and long-term strategy.

## 4. From Reactive Moments to Proactive Plans

While a `Vessel` provides a complete record of a single reaction, achieving larger goals requires a structure for proactive, multi-step execution. This is the role of a **Process Idea**, which applies the same principles of self-containment to a strategic, stateful workflow.

:term[203: Idea/Process](./203_idea_process.md) describes this structure.
