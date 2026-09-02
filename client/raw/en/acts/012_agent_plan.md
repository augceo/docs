# 012: Agent/Plan

> [!DEFINITION] [Plan](./000_glossary.md)
> A context message carrying a data-flow graph of :term[Tool Calls]{canonical="Tool Call"} that represents an agent's strategy. It is passed between steps to enable iterative execution and adaptation.

> Sidenote:
>
> - Requires:
>   - :term[004: Agent/Call]{href="./004_agent_call.md"}
>   - :term[010: Agent/Loop]{href="./010_agent_loop.md"}
>   - :term[007: Agent/Variables]{href="./007_agent_variables.md"}
>   - :term[009: Agent/State]{href="./009_agent_state.md"}
>   - :term[013: Agent/Instancing]{href="./013_agent_instancing.md"}

The :term[Plan]{canonical="Plan"} message is the primary mechanism for enabling stateful, iterative execution. While the agent system is capable of handling simple, one-shot requests, providing a `Plan` message in the context signals a shift to a persistent workflow. Its presence instructs the :term[Execution Loop]{canonical="Execution Loop"} to retain the generated plan and the resulting :term[State]{canonical="State"} across multiple turns.

This persistence is the cornerstone of building adaptive agents. When the LLM receives the current :term[Plan]{canonical="Plan"} alongside the live :term[State]{canonical="State"} object, it gains complete situational awareness of its position in the workflow. This allows it to intelligently follow the existing plan, generate new :term[Tool Calls]{canonical="Tool Call"} to expand it, or discard it entirely and replan in response to unexpected outcomes. Without a `Plan` message, a request is treated as a stateless operation, and no state or strategy is carried over to subsequent steps.

## How a Plan is Formed

The connections in the graph are not created with explicit pointers, but through a data-flow convention using the :term[State]{canonical="State"} object.

- **Nodes (:term[Tool Calls]{canonical="Tool Call"}):** Each step in the workflow is a :term[Tool Call]{canonical="Tool Call"}, representing an action to be performed.
- **Edges (:term[State]{canonical="State"} Object):** The connections between steps are created by writing to and reading from the :term[State]{canonical="State"} object. One :term[Tool]{canonical="Tool"} writes its output to a specific path in the :term[State]{canonical="State"} using the :term[Output Path]{canonical="Output Path"} meta-property. A subsequent :term[Tool]{canonical="Tool"} can then use that output as an input by referencing the same path with a **:term[Variable Reference]{canonical="Variable Reference"}**.

  > Sidenote:
  >
  > - [008: Agent/Output](./008_agent_output.md)

This establishes a clear dependency: the second :term[Tool Call]{canonical="Tool Call"} cannot execute until the first has completed and populated the :term[State]{canonical="State"}.

For example, a :term[Plan]{canonical="Plan"} to fetch a user's profile and then summarize it would consist of two :term[Tool Calls]{canonical="Tool Call"}:

:::div{.limited-width}

```json
[
  {
    "_tool": "fetchUserProfile",
    "userName": "Alice",
    "_outputPath": "†state.userProfileData"
  },
  {
    "_tool": "summarizeProfile",
    "profile": "†state.userProfileData",
    "_outputPath": "†state.profileSummary"
  }
]
```

> Sidenote:
>
> ```mermaid
> graph TD
>     state_var{{"state.user.profile"}}
>
>     Call1["fetchUserProfile"]
>     Call2["summarizeProfile"]
>
>     Call1 -- writes to --> state_var
>     state_var -- read by --> Call2
> ```

:::

Here, the `summarizeProfile` call depends on the output of `fetchUserProfile`, creating a two-step plan. This relationship can be visualized as a simple graph.

## The Plan's Content: A Data-Flow Graph

The content of a :term[Plan]{canonical="Plan"} message is a data-flow graph. This structure is used to represent the agent's strategy as a sequence of interconnected :term[Tool Calls]{canonical="Tool Call"}. By representing the workflow as a graph, the system can clearly define the dependencies between steps, where the output of one :term[Tool Call]{canonical="Tool Call"} becomes the input for another. This graph-based format provides a clear, machine-readable structure that the agent's :term[Execution Loop]{canonical="Execution Loop"} can interpret and execute.

> Sidenote:
>
> A :term[Plan]{canonical="Plan"} is not limited to linear sequences. It can represent complex workflows with conditional logic, where the path of execution depends on the outcome of a previous step:
>
> ```mermaid
> graph TD
>     A[Get Weather] --> B{Is it Sunny?};
>     B -- state.sunny --> C[Find a Park];
>     B -- state.notSunny --> D[Find a Movie];
>     C -- state.suggestion --> E[Present Suggestion];
>     D -- state.suggestion --> E[Present Suggestion];
> ```

The underlying graph structure is not limited to just defining executable workflows. An agent can be prompted to generate a graph of :term[Tool Calls]{canonical="Tool Call"} that represents something else entirely—a visualization of a social network, a GitHub Actions workflow, or a database schema.

It is crucial to distinguish these outputs from a :term[Plan]{canonical="Plan"}. While they use the same graph structure, they are not "plans" in the architectural sense unless they are passed into a subsequent :term[Request]{canonical="Request"} as a :term[Plan]{canonical="Plan"} context message with the intent of being executed. This distinction prevents confusion between generating a representation of an existing system and creating an executable strategy.

While the content of a :term[Plan]{canonical="Plan"} can be a powerful tool for brainstorming, discussion, and "thinking out loud," its primary application in this system is to define executable workflows. For this purpose, we use a specific type of graph called a **Directed Acyclic Graph (DAG)**, where each node is a :term[Tool Call]{canonical="Tool Call"}.

A DAG has a few key properties that make it perfect for execution:

- **Graph:** The "graph" is the entire content of the :term[Plan]{canonical="Plan"} message—the collection of all :term[Tool Calls]{canonical="Tool Call"} (the nodes) and the data dependencies that connect them (the edges).
- **Directed:** The connections are one-way, determined by the flow of data. A step that creates data must come _before_ a step that uses it.
- **Acyclic:** The workflow cannot have circular dependencies, ensuring it has a clear beginning and end. This is a critical safety feature to prevent the LLM from generating a workflow with an infinite loop. The system validates that a :term[Plan]{canonical="Plan"} is acyclic before execution.

**A step is evaluated when its :term[Output Path]{canonical="Output Path"} has a value, and not before.** Nothing marks a step as done or deferred; the state is read off what has been written. That one rule is where the rest of this chapter's behaviour comes from — a branch, a step waiting on a result that does not exist yet, and the difference between the two.

> Sidenote:
>
> To implement iterative logic like a "for loop," a pattern of nested, delegated execution is used. An outer :term[Plan]{canonical="Plan"} manages the loop's state (e.g., an iteration counter), and for each iteration, it invokes a sub-request via a :term[Delegate]{canonical="Delegate"}. This sub-request contains its own separate, acyclic :term[Plan]{canonical="Plan"} that performs the logic for a single iteration. This ensures that loops are created explicitly and safely.

## Planning and Execution Are One Act

**Planning is execution.** There is no mode, no flag and no second evaluation strategy. When the LLM generates a `solution`, it is performing a single, continuous act of reasoning and execution for any step it can resolve.

- A latent tool (summarizing text, say) gets its `call` and its `_output` in the same thought.
- An explicit :term[Activity]{canonical="Activity"} gets its `call` dispatched by the :term[Execution Loop]{canonical="Execution Loop"} the moment that call's :term[Variable References]{canonical="Variable Reference" href="./007_agent_variables.md"} resolve.

That second clause is the whole of the scheduling rule, and it is why no mode is needed. **A call runs exactly when what it reads exists.** A step that should not run yet is a step reading something that does not exist yet.

So a turn is not obliged to finish the plan in one thought. It authors the entire graph at once and evaluates the steps it can: a latent step whose input has not been produced has nothing to compute from, so it leaves `_output` unanswered rather than inventing one. That step is not an error and is not dispatched — no value at its Output Path means it has not been evaluated. It stays in the plan, and a later turn, which can now see the value it was waiting for, evaluates it then.

A branching `_outputPath` (`†state.sunny <|> †state.rainy`) survives a turn for the same reason. An explicit call's result does not exist when the turn is written, so nothing forces the choice; the :term[Activity]{canonical="Activity"} makes it at run time. A **latent** call is the opposite case and always was: its `_output` is the model's own reasoning, so computing one *is* choosing, and a `<|>` on a latent call's Output Path is decided before the turn is even sent. The alternatives it did not take are then read back off the state — once the Expression resolves, every arm still without a value is one this run will never produce, and the calls waiting on those arms are carried rather than reported as failures.

### Proposing Rather Than Performing

A deliberate workflow — plan first, act after a review — needs no separate planning mode. It is the scheduling rule used on purpose: **a proposal is a call gated on a path nobody has written yet.**

Which paths those are is fixed by convention rather than declared. **Any path under `†state.approvals` is an approval gate**, and there is no other way to mark one: no flag on a call, no second channel, no list a host has to keep in step with the plan. A gate that has to be registered somewhere is a gate somebody can forget to register; the namespace cannot be forgotten because it is the path itself.

Give each alternative its own gate and let every step of that alternative read it. A plan A whose steps read `†state.approvals.planA`, a plan B whose steps read `†state.approvals.planB`. Neither path has a value, so neither plan starts, and both sit in the :term[Plan]{canonical="Plan"} as proposals — a pure data structure representing the whole strategy, which is exactly the checkpoint a deliberate workflow wants. It can be:

- **Validated:** the graph can be checked for circular dependencies or other structural errors.
- **Simulated:** a dry run can anticipate the workflow's behaviour.
- **Presented for Approval:** shown to a human for review, modification or approval before execution (:term[HITL]{canonical="HITL (Human-in-the-Loop)"}).

Approval is then an ordinary write to the chosen gate. Nothing new selects it; the steps of that plan become ready because what they read now exists, and the steps of the plan nobody chose keep waiting and are dropped on a later turn. Which alternative a step belongs to is carried by which path gates it — there is no proposal label and no second channel.

In practice only the FIRST step of an alternative names the gate; the rest read what that step would write. The whole subgraph hanging off a gate is held by it, and is waiting for the same reason, even though only one call in it mentions an approval path.

**Approval happens between runs, never inside one.** Nothing outside the loop can write :term[State]{canonical="State"} while the loop is turning, so a proposal is never dispatched in the run that authored it. The model proposes, the run ends, whoever decides writes the gate, and the next run begins with the value already in :term[State]{canonical="State"} — the waiting steps simply resolve. Nothing pauses, nothing is suspended, and the :term[Execution Loop]{canonical="Execution Loop"} gains no new state. That is why the whole mechanism costs a namespace and no machinery.

**A gated proposal is not a failure, and is not reported as one.** A call waiting on a gate is waiting; a call waiting on a path nothing will ever write is broken. Both are calls whose :term[Variable References]{canonical="Variable Reference"} did not resolve, so the namespace is the only thing that separates them. The wait is reported once, as the run ends, naming each gate and what it is holding — never once per turn, which would fill the context with notices and teach the model that gating is the thing being punished.

**Per-instance approval needs nothing added.** A gate is an ordinary :term[State]{canonical="State"} path, so it folds like one: written globally it resolves for every instance and all of them proceed; written inside one instance it resolves only there, and the same proposal stays held everywhere else. There is no instance-addressing syntax for approvals, because the specificity rule of :term[013: Agent/Instancing]{href="./013_agent_instancing.md"} already is one.

The cost of having no mode is named plainly, because nothing structural prevents it: **a proposal whose calls have no unmet dependency dispatches immediately.** A call that reads only values already in :term[State]{canonical="State"} is not a proposal, it is an action. And a gate has to be a :term[Variable Reference]{canonical="Variable Reference"}: `†state.approvals.planA` waits, the word `"approved"` sitting in the same parameter waits for nothing. If it is meant to be proposed, it must be gated.

## The Plan as an Evolving Strategy

A :term[Plan]{canonical="Plan"} is not static; it is a living strategy that can be adapted at each step of the execution loop. A key distinction is that a `Plan` is not just any output from the LLM. When an agent first generates a set of :term[Tool Calls]{canonical="Tool Call"}, this is simply a proposed sequence of actions. It becomes a true :term[Plan]{canonical="Plan"} only when it is passed as a context message into the _next_ request in the loop.

This cycle transforms a one-off output into an ongoing strategy:

- The **:term[context]{canonical="context"}** for a request contains the :term[State]{canonical="State"} object and the :term[Plan]{canonical="Plan"} message from the previous step.
- The **:term[solution]{canonical="Solution"}** generated by the LLM contains a new set of :term[Tool Calls]{canonical="Tool Call"} that becomes the **new :term[Plan]{canonical="Plan"}** for the next step.

This iterative process allows the agent to be both proactive and reactive. It can follow the existing :term[Plan]{canonical="Plan"}, but it can also modify it in response to the results of the previous step. For example, if a :term[Tool Call]{canonical="Tool Call"} fails, the agent can generate a new :term[Plan]{canonical="Plan"} that includes error-handling steps. This makes the system resilient and adaptable.

:::::details{title="Example: Planning one step ahead"}

This example demonstrates how a `Plan` provides a structural "happy path" that guides the agent, preventing it from deviating from a standard procedure even when other plausible actions are available.

**Scenario:** A customer support agent needs to process a refund. The standard procedure, triggered by the user's request, is to first check the billing history for context and then issue the refund.

**1. Initial Request**

The loop starts with the customer's request. Based on this `input`, the LLM formulates a standard two-step `Plan` to handle the refund. This represents the ideal, most common workflow.

::::columns
:::column{title="Context & Schema for Request"}

```json
// Agent.Request(config, schema, context)
{
  "schema": {
    "type": "object",
    "properties": {
      "calls": { "type": "array" },
      "output": {
        "type": "object",
        "nullable": true,
        "properties": {
          "confirmationId": { "type": "string" },
          "message": { "type": "string" }
        }
      }
    }
  },
  "context": [
    {
      "type": "input",
      "request": "I'd like a refund for my last order.",
      "customerId": "cust_123",
      "amount": 50.0
    }
  ]
}
```

:::
:::column{title="LLM's `solution`"}

```json
{
  "calls": [
    {
      "_tool": "checkBillingHistory",
      "customerId": "†input.customerId"
    },
    {
      "_tool": "issueRefund",
      "customerId": "†input.customerId",
      "amount": "†input.amount"
    }
  ],
  "output": null
}
```

:::
::::

**2. Next Request in the Loop**

The :term[Execution Loop]{canonical="Execution Loop"} runs the `checkBillingHistory` call and populates the :term[State]{canonical="State"}. The history reveals some complexity (e.g., a previous chargeback). At this point, an unguided agent might plausibly choose to use another available tool, `escalateToSupervisor`.

However, the `Plan` message in the context provides the necessary structure. By composing what it _knows_ (the complex `State`) with what it _should do_ (the `Plan`), the LLM understands its precise position in the workflow and sticks to the "happy path."

::::columns
:::column{title="Context"}

```json
[
  {
    "type": "state",
    "billingHistory": {
      "orders": 5,
      "lastChargeback": "2025-09-10"
    }
  },
  {
    "type": "plan",
    "plan": [
      {
        "_tool": "checkBillingHistory",
        "customerId": "†input.customerId"
      },
      {
        "_tool": "issueRefund",
        "customerId": "†input.customerId",
        "amount": "†input.amount"
      }
    ]
  }
]
```

:::
:::column{title="LLM's `solution`"}

```json
{
  "calls": [
    {
      "_tool": "issueRefund",
      "customerId": "†input.customerId",
      "amount": "†input.amount"
    }
  ],
  "output": {
    "confirmationId": "refund_xyz789",
    "message": "The refund has been processed successfully."
  }
}
```

:::
::::

The `Plan` ensures procedural consistency, preventing a premature escalation and keeping the agent on its intended track.

:::::

:::::details{title="Example: Adjusting a Plan"}

This example demonstrates how an agent can modify an existing :term[Plan]{canonical="Plan"} in response to new information by choosing a different tool.

::::columns
:::column{title="Context"}

The agent is given an existing "happy path" `Plan` and a new `Input` from the user that introduces a new constraint.

```ts
[
  { type: 'tool', tool: Tool.bookFlight },
  { type: 'tool', tool: Tool.bookHotel },
  { type: 'tool', tool: Tool.findPetFriendlyHotel },
  {
    type: 'plan',
    plan: [
      {
        _tool: 'bookFlight',
        destination: '†input.destination',
      },
      {
        _tool: 'bookHotel',
        destination: '†input.destination',
      },
    ],
  },
  {
    type: 'input',
    destination: 'Berlin',
    instruction: "Actually, I'll be traveling with my dog.",
  },
];
```

:::
:::column{title="LLM's `solution`"}

The LLM recognizes that the original plan is no longer suitable. It discards the old plan and generates a new one, replacing `bookHotel` with a more specialized tool.

```json
{
  "calls": [
    {
      "_tool": "bookFlight",
      "destination": "†input.destination"
    },
    {
      "_tool": "findPetFriendlyHotel",
      "destination": "†input.destination"
    }
  ],
  "output": null
}
```

:::
::::

The agent doesn't just change a parameter; it fundamentally alters its strategy by selecting a more appropriate tool (`findPetFriendlyHotel`) based on the new requirements. This new set of `Tool Calls` becomes the `Plan` for the next step in the :term[Execution Loop]{canonical="Execution Loop"}.

:::::

:::::details{title="Example: Handling Failure"}

This example demonstrates how an agent can deviate from a "happy path" :term[Plan]{canonical="Plan"} when it encounters an unexpected failure. The process is shown in two stages: the initial "happy path" plan, and the replanning that occurs after a tool fails.

**1. The Initial Plan**

The agent is given a set of tools and a user input. It generates an optimistic, two-step "happy path" plan that does not account for failure.

::::columns
:::column{title="Initial Context"}

```ts
Agent.Request(config, {
  schema: {
    type: 'object',
    properties: {
      calls: { type: 'array' },
      output: {
        type: 'object',
        nullable: true,
        properties: {
          status: {
            type: 'string',
            enum: ['Success', 'Failed'],
          },
        },
      },
    },
  },
  context: [
    { type: 'tool', tool: 'Tool.processPayment' },
    { type: 'tool', tool: 'Tool.confirmOrder' },
    { type: 'tool', tool: 'Tool.reportFailure' },
    { type: 'input', amount: 50.0 },
  ],
});
```

:::
:::column{title="Initial Solution"}

```json
{
  "calls": [
    {
      "_tool": "processPayment",
      "amount": "†input.amount",
      "_outputPath": "†state.receipt <|> †state.error"
    },
    {
      "_tool": "confirmOrder",
      "receipt": "†state.receipt"
    }
  ],
  "output": null
}
```

:::
::::

**2. Failure and Replanning**

The :term[Execution Loop]{canonical="Execution Loop"} attempts to run `processPayment`, but the tool fails. The engine populates `†state.error`. In the next iteration, the LLM sees this new error state alongside the original (now obsolete) plan and generates a new solution to handle the failure.

::::columns
:::column{title="Context for Next Request"}

```json
[
  {
    "type": "state",
    "error": { "code": "card_declined", "message": "Your card was declined." }
  },
  // The original, now-obsolete plan is still in the context
  {
    "type": "plan",
    "plan": [
      { "_tool": "processPayment", "_outputPath": "†state.receipt <|> †state.error" },
      { "_tool": "confirmOrder", "receipt": "†state.receipt" }
    ]
  }
]
```

:::
:::column{title="New Solution (Replanned)"}

```json
{
  "calls": [
    {
      "_tool": "reportFailure",
      "error": "†state.error"
    }
  ],
  "output": { "status": "Failed" }
}
```

:::
::::

The agent recognized the `error` in the `State`, ignored the obsolete "happy path" `Plan`, and generated a new, single-step plan to `reportFailure`. This demonstrates the agent's ability to reactively handle unexpected outcomes.

:::::

:::::details{title="Example: Schema-Guided Planning"}

This example demonstrates how providing a `schema` for the :term[State]{canonical="State"} object acts as a blueprint, guiding the LLM to generate a structurally correct :term[Plan]{canonical="Plan"}.

::::columns
:::column{title="Context with State Schema"}

The caller provides an `Input` and a `State` message that contains only a `schema`. This schema defines the intended data flow by specifying the "variables" the plan should use.

```json
[
  { "type": "tool", "tool": "Tool.detectLanguage" },
  { "type": "tool", "tool": "Tool.isEnglish" },
  { "type": "tool", "tool": "Tool.translateText" },
  {
    "type": "input",
    "text": "Bonjour le monde"
  },
  {
    "type": "state",
    "schema": {
      "type": "object",
      "properties": {
        "language": { "type": "string" },
        "isEnglish": { "type": "boolean" },
        "translatedText": { "type": "string" }
      }
    }
  }
]
```

:::
:::column{title="LLM's `solution`"}

The LLM uses the `State` schema as a guide to construct a valid plan, correctly wiring the :term[Output Path]{canonical="Output Path"} of one tool to the input of the next.

```json
{
  "calls": [
    {
      "_tool": "detectLanguage",
      "text": "†input.text",
      // The LLM knows to use this path from the schema.
      "_outputPath": "†state.language"
    },
    {
      "_tool": "isEnglish",
      // It correctly references the output of the previous step.
      "language": "†state.language",
      "_outputPath": "†state.isEnglish"
    },
    {
      "_tool": "translateText",
      "text": "†input.text",
      "isEnglish": "†state.isEnglish",
      "_outputPath": "†state.translatedText"
    }
  ],
  "output": null
}
```

:::
::::

By providing the `schema`, the caller gives the LLM a clear blueprint for the data flow. The LLM doesn't have to guess the variable names or the sequence; it simply populates the pre-defined slots, resulting in a more reliable and predictable `Plan`.

:::::

This iterative cycle of planning and execution is the core of a :term[Process]{href="./203_idea_process.md"}. It is is a self-contained snapshot of a workflow, capturing the :term[Tools]{canonical="Tool"} available, the live :term[State]{canonical="State"}, and the :term[Plan]{canonical="Plan"} itself.

## From Single Plan to Reusable Workflows

A :term[Plan]{canonical="Plan"} message defines a sequence of actions for a specific task. To make these workflows reusable, we need a way to encapsulate them into components that can be called from other :term[Plans]{canonical="Plan"}.

:term[013: Agent/Instancing]{href="./013_agent_instancing.md"} describes the protocol for this parallel execution.
