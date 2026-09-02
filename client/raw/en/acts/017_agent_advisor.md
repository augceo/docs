# 017: Agent/Advisor

> [!DEFINITION] [Advisor](./000_glossary.md)
> A specialized context message (with `kind: "advisor"`) that defines a persona or analytical model. It instructs the agent to generate a structured "opinion" or confidence score _before_ selecting a :term[Tool]{canonical="Tool"} or generating an :term[Output]{canonical="Output"}.

> Sidenote:
>
> - Requires:
>   - :term[001: Agent/Request]{href="./001_agent_request.md"}
> - Enhances:
>   - :term[002: Agent/Tool]{href="./002_agent_tool.md"}
>   - :term[010: Agent/Loop]{href="./010_agent_loop.md"}

The **Advisor Protocol** enforces a "deliberate-then-act" architecture. By registering **Advisors**, the agent assesses the context through specific strategic lenses—such as a "SecurityArchitect" or "ProductManager"—before committing to an Action.

## Undirected Thought

Unstructured reasoning (like Chain-of-Thought) is often chaotic. Agents wander without a map, mixing information gathering, reasoning, and decision-making into a single, noisy output.

This lack of structure bloats the context with excessive reasoning. Unguided deliberation causes the agent to vacillate, leading to shifting opinions and decreased accuracy. The unpredictability of this process degrades both speed and reliability. Furthermore, decisions derived from unstructured text are difficult to reconcile; a paragraph of generalist thought offers no quantitative handle for balancing competing priorities.

The Advisor Protocol formalizes a portion of the reasoning process to solve these issues. It structures the evaluation, making it predictable and precise. This structure also enhances the user experience. Instead of hiding "walls of text," the system can present structured, weighted outputs that clearly highlight divergences in opinion, strong preferences, and distinct recommendations. This creates a better ecosystem for decision-making and significantly improves reportability.

The Advisor Protocol does not replace latent reasoning; it structures a specific subset of it. It captures the evaluative process as a tangible artifact—a weighted opinion—that serves as a direct input for decision-making.

## Divergent Perspectives & Tool Weighting

By registering multiple Advisors, the system forces the agent to evaluate the situation from competing viewpoints (e.g., "Risk" vs. "Opportunity"). This divergence is captured through **Weighted Tool Selection**, where Advisors output explicit "calls" or confidence scores for specific :term[Tools]{canonical="Tool"}.

The Agent aggregates these weighted votes from its council. It does not merely "pick" a tool; it reconciles a vector of divergent opinions to reach a consensus. Activating a tool becomes a calculated resolution of competing advice.

## The Advisor Message

The **Advisor Message** defines the lens through which the agent views the problem.

- **`id`**: A unique identifier for the advisor (e.g., `"securityArchitect"`).
- **`persona`**: What this advisor is for — its domain and the perspective it argues from. It is named for the thing it holds, and named `persona` rather than `role` because `role` is the message envelope's own field: an Advisor authored as bare content carrying a `role` is indistinguishable from an envelope, and the primitive that tells them apart should not have to know about Advisors.
- **`answer`**: A JSON Schema defining the structure of the advisor's advice. This enables specific quantitative outputs (e.g., risk scores, budget estimates) or structural values beyond standard tool voting. It is named for what it holds — the shape of this advisor's answer — because in a library where every field is a schema, "schema" says nothing.
- **`on`**: Defines the participation strategy.
- **`scopes`**: An array of context keys (e.g., `["input", "state"]`) that focus the advisor's analysis.
- **`isInstanced`**: A boolean flag (default `false`). If `true`, the advisor definition includes an `_instance` property, ensuring individual consultation for each instance in a batch.

Both `persona` and `answer` are optional, and the declared `answer` reaches the response type: a field declared as a two-member enum arrives as that enum, not as `any`.

## One Advisor, Many Messages

An Advisor is not a message; it is every message sharing its `id`. A later message **patches** the standing definition field by field, so a request can refine an Advisor the surrounding system already registered:

```json
{
  "type": "advisor",
  "id": "riskAnalyst",
  "answer": {
    "type": "object",
    "properties": { "score": { "type": "number" } }
  }
}
```

That message names no `persona` and does not need to. Recency wins on any field both messages declare, and an `id` nobody has declared before simply mints a new Advisor — a declaration is an overlay onto nothing. The cost of that rule is worth stating plainly: a misspelled `id` quietly creates a persona-less Advisor instead of refining the intended one.

`on` is the exception, and is **set once**. It decides which mechanism delivers the Advisor — a property on the response, or the `ConsultAdvisor` tool — so refining it between turns would move the Advisor across cache boundaries and make the response type depend on which turn asked. A later message naming a *different* participation is an error. A message naming none inherits the standing one, which is what lets a silent Advisor be promoted to a participating one later.

## Execution Strategies

The `on` property controls participation:

- **`start` (Initial)**: Opines only at the start of a conversation to set direction. A conversation has started once the model has taken a turn of its own; until then a `start` Advisor is required, and afterwards it is offered like a stopping one. This is the only participation whose obligation depends on the history rather than on the declaration.
- **`finish` (Stopping)**: Consulted when determining if the loop needs to be stopped.
- **`request` (Continuous)**: Provides an opinion on start, end, and every iteration. Ideal for safety monitors or core strategic advisors.
- **`null` / `undefined` (On-Demand)**: Stays silent unless explicitly invoked. The system provides a meta-tool, `ConsultAdvisor`, allowing the LLM to invoke these advisors on demand in ambiguous situations.

## Advisory Order

When Advisors are active, the LLM generates their structured output _first_, within the same generation window. It adopts the advisor personas to analyze the state or vote on actions. The Agent then uses this output immediately to make the final decision on which :term[Tool]{canonical="Tool"} to invoke.

Quantitative data like "calls" is encoded as **Inline JSON** within a string field to prevent schema explosion while retaining rigor.

::::columns
:::column{title="Registering Advisors"}

```json
[
  {
    "type": "advisor",
    "id": "riskAnalyst",
    "on": "request",
    "scopes": ["†state.deploymentHistory"],
    "persona": "Analyze risks and Vote for deployment actions.",
    "answer": {
      "type": "object",
      "properties": {
        "thought": { "type": "string", "description": "Risk assessment of the deployment." }
      },
      "required": ["thought"]
    }
  }
]
```

:::
:::column{title="The LLM's Response"}

```json
{
  "advisors": [
    {
      "id": "riskAnalyst",
      "thought": "The new feature has passed unit tests but lacks integration tests. High risk of regression.",
      "calls": "{\"deploy\": 10, \"rollback\": 5, \"delay\": 95}"
    }
  ],
  "calls": [
    {
      "_tool": "delay",
      "_why": "RiskAnalyst strongly advises delaying due to insufficient testing."
    }
  ]
}
```

:::
::::

## Interactions with other systems

- **:term[Tool]{canonical="Tool"}:** Advisors do not execute actions; they vote on them. The `advisors` capability appears structurally similar to a tool call in the output schema, but it represents a meta-action—a "thought process"—that strictly precedes the execution of any real :term[Tool]{canonical="Tool"}. This separation ensures that advice is gathered before commitment.

  > Sidenote:
  >
  > - :term[002: Agent/Tool]{href="./002_agent_tool.md"}

- **:term[Loop]{canonical="Loop"}:** The `on` property integrates advisors into the agent's lifecycle. Advisors with `on: request` act as continuous monitors, running on every tick of the loop. Those with `on: null` (on-demand) act as safety valves, available to be called upon when the agent encounters ambiguity or difficult decisions during execution.

  > Sidenote:
  >
  > - :term[010: Agent/Loop]{href="./010_agent_loop.md"}

- **:term[Plan]{canonical="Plan"}:** Advisor advice is crucial during the planning phase. By consulting advisors with `on: start` or `on: request`, the agent can align its initial strategy and subsequent steps with the consensus of specialized personas, ensuring the :term[Plan]{canonical="Plan"} is robust and well-reasoned from the outset.

  > Sidenote:
  >
  > - :term[012: Agent/Plan]{href="./012_agent_plan.md"}

- **:term[Instancing]{canonical="Instancing"}:** Advisors support batched execution. By setting `isInstanced: true`, an advisor becomes instance-aware. The system will generate advice specific to each context (identified by `_instance`), ensuring that each item in a batch is evaluated by the advisor individually.

  > Sidenote:
  >
  > - :term[013: Agent/Instancing]{href="./013_agent_instancing.md"}

- **:term[Scopes]{canonical="Scopes"}:** The `scopes` property in an Advisor message provides a way to focus the advisor's attention. Unlike the strict data isolation seen in Delegates, scopes here act as **soft guidance**. They instruct the specific advisor persona to prioritize analysis of the listed context keys (e.g., `["†state.deploymentHistory"]`), enforcing a "separation of concerns" within the advisory phase without blocking access to the broader context.

  > Sidenote:
  >
  > - :term[015: Agent/Scopes]{href="./015_agent_scopes.md"}
