# 010: Agent/Loop

> [!DEFINITION] [Loop](./000_glossary.md)
> An iterative sequence of :term[Request]{canonical="Request"}s aimed at achieving a goal. The agent continues to invoke :term[Request]{canonical="Request"}s, process the resulting :term[Call]{canonical="Call"}s, and feed the output back into the context until the LLM generates a :term[Final Output]{canonical="Final Output"}, signaling the task is complete.

> Sidenote:
>
> - Requires:
>   - :term[001: Agent/Request]{href="./001_agent_request.md"}
>   - :term[008: Agent/Output]{href="./008_agent_output.md"}

The :term[Execution Loop]{canonical="Execution Loop"} is the engine that orchestrates all the message types and protocols from previous chapters. It enables an agent to perform multi-step tasks by iteratively making :term[Requests]{canonical="Request"}. This iterative process of context assembly, tool use, and feedback is what is commonly meant when referring to an "agent."

## The Execution Loop

The execution loop is the primary mechanism for autonomous, multi-step execution. It operates via a nested loop structure:

::::columns
:::column

1.  **Outer Loop (Request Generation):** The agent's lifecycle is a sequence of :term[Request]{canonical="Request"}s. It starts with an initial context and enters a loop.
2.  **Request & Call Streaming:** Inside the loop, it invokes a single :term[Request]{canonical="Request"}. The :term[Request]{canonical="Request"} streams back :term[Calls]{canonical="Call"} as they are generated, which are collected into a pending queue.
3.  **Inner Loop (Call Orchestration):** For each :term[Request]{canonical="Request"}, an inner orchestration loop is responsible for executing its associated :term[Calls]{canonical="Call"}. This process is a reactive, event-driven loop that is triggered by two events: a new :term[Call]{canonical="Call"} is streamed from the LLM, or a previously running :term[Call]{canonical="Call"} completes. This process is highly concurrent:
    - The orchestrator continuously scans the queue of pending :term[Calls]{canonical="Call"} to find all that are currently unblocked (i.e., their dependencies are met).
    - All unblocked :term[Calls]{canonical="Call"} can be presented for confirmation and then executed in parallel. This allows for high throughput, but requires careful management of state. If multiple concurrent :term[Calls]{canonical="Call"} write to the same path in the :term[State]{canonical="State"}, the final value will be determined by the last call to complete, which may lead to non-deterministic outcomes. The system relies on a "last-write-wins" approach for resolving these conflicts.

    - As each :term[Call]{canonical="Call"} completes, its output updates the shared context, potentially unblocking other pending :term[Calls]{canonical="Call"}.
    - This reactive, parallel execution continues until the stream for the current :term[Request]{canonical="Request"} is closed and all of its pending :term[Calls]{canonical="Call"} have been drained. This model significantly reduces latency, as the agent can start working on multiple independent steps simultaneously, even before the full plan is known.

4.  **Termination & Continuation:** Once the inner loop completes for a given :term[Request]{canonical="Request"}, the agent inspects the final :term[Solution]{canonical="Solution"}. The decision to continue is based on the `output` field:
    - **If `output` is `null`**, the agent determines that its task is not yet complete. It loops back to step 2, invoking a new :term[Request]{canonical="Request"} with the enriched context that now contains the results of the executed :term[Calls]{canonical="Call"}.
    - **If `output` is not `null`**, the agent's goal is considered achieved. The outer loop terminates, and the `output` value, which conforms to the user-defined output schema, is returned as the final result. An agent can produce both `calls` and a final `output` in a single step; the presence of the `output` is the definitive signal to stop.

      > Sidenote:
      >
      > - [008: Agent/Output](./008_agent_output.md)

:::
:::column

```mermaid
graph TD
    Start[/"Agent(...)"/] --> Request["Agent.Request(...)"];
    Request -- streams calls --> FindUnblocked{"Find all unblocked calls"};

    FindUnblocked -- "some found" --> ConfirmAndLaunch["Confirm & launch all (parallel)"];
    ConfirmAndLaunch -- "any completes" --> UpdateContext["Update context with result"];
    UpdateContext --> FindUnblocked;

    FindUnblocked -- "none found" --> CheckStreamAndPending{"Stream active or calls pending?"};
    CheckStreamAndPending -- Yes --> Wait["Wait for new calls or completions"];
    Wait --> FindUnblocked;

    CheckStreamAndPending -- No --> CheckTermination{"Solution has output?"};
    CheckTermination -- No --> Request;
    CheckTermination -- Yes --> End[\"Return context/solution"\];
```

:::
::::

## :term[Human-in-the-Loop]{canonical="HITL"}

The new :term[Execution Loop]{canonical="Execution Loop"} provides support for human oversight by placing the confirmation step just before execution. This ensures the user is only prompted to act on calls that are ready to run:

- **Approval:** Before an unblocked :term[Call]{canonical="Call"} is executed, the system can present it to a user for approval. This is an efficient approach, as it prevents the user from having to review and confirm calls that might be blocked by dependencies and never run.
- **Correction:** The user can modify the parameters of a :term[Call]{canonical="Call"} or even replace it with a different one

It is important to note that these specific :term[HITL]{canonical="HITL (Human-in-the-Loop)"} mechanisms are not part of the core protocol. The architecture simply provides the necessary separation between proposing actions and executing them, giving developers the flexibility to implement any kind of intervention, from a simple manual approval to a complex, automated system with timeouts.

This capability is critical for safety and for collaborative tasks where the agent acts as an assistant. User adjustments and feedback can be leveraged by the :term[Plan]{canonical="Plan"}, allowing the agent to refine its strategy based on human input.

Approval can also be taken a step earlier, in the :term[Plan]{canonical="Plan"} rather than at the moment of dispatch. A step gated on a path under `†state.approvals` is never unblocked, so it is never presented for confirmation at all until somebody writes that path — the review happens over a strategy instead of over a call about to run. See [Waiting for a Decision Is Not a Failure](#waiting-for-a-decision-is-not-a-failure) below, and :term[012: Agent/Plan]{href="./012_agent_plan.md"}.

## Error Handling and Self-Correction

The Execution Loop is designed to be resilient, providing a reactive safety net that allows the agent to handle and recover from both its own planning mistakes and unexpected runtime failures. This is achieved by catching errors and feeding them back to the LLM as structured, contextual information.

### The Error Message

To facilitate this, the system uses a specialized **`Error Message`**, which is based on the `Data Message`. When an error is caught, the loop generates an `Error Message` and appends it to the context for the next iteration. Its `data` property contains two critical pieces of information:

- **`call`**: The full, serialized `call` object that failed.
- **`error`**: An object or string detailing the reason for failure.

### The Self-Correction Loop

The loop catches two primary types of unhandled failures:

1.  **Structural Errors**: Caught _before_ execution. This includes schema validation failures on a call's parameters, mismatched types, or invalid :term[Variable References]{canonical="Variable Reference"}.
2.  **Runtime Errors**: Caught _during_ execution. This occurs when an explicit :term[Activity]{canonical="Activity"} throws an unhandled exception (e.g., due to an API failure or internal logic error).

When either error type is caught, the loop generates the `Error Message` and adds it to a temporary list. At the end of the execution tick (when no more calls are unblocked), all `Error Messages` collected during that turn are appended to the context.

This provides the LLM with direct, actionable feedback. On its next turn, it sees the failed call and the specific error it produced, allowing it to debug and generate a corrected plan.

### Waiting for a Decision Is Not a Failure

A structural error is a :term[Call]{canonical="Call"} waiting on a :term[Variable Reference]{canonical="Variable Reference"} nothing will ever write. A proposal awaiting approval is a :term[Call]{canonical="Call"} waiting on a :term[Variable Reference]{canonical="Variable Reference"} nobody has written **yet**. Read off the :term[State]{canonical="State"} alone those are the same shape, and reporting the second as the first teaches the model that gating a proposal is punished — so it stops gating, and the proposal becomes an action.

One reserved namespace separates them. **Any path under `†state.approvals` is an approval gate.** There is nothing to declare, no flag on a call and no registry to keep: the subtree is the whole convention. A :term[Call]{canonical="Call"} held only by paths under it — either because it reads one, or because it reads what such a call would write — is waiting. Anything else unresolved is still the structural error above, and one reference nobody owes is enough to make a call broken however it is gated.

Approval arrives from outside, and nothing outside can write :term[State]{canonical="State"} while the loop is turning. So a proposal is never dispatched in the run that authored it: it is written in one run, decided between runs, and in the next run `Call.next` simply finds its references satisfied. Nothing suspends and nothing is held open — the execution model above is unchanged.

Because the wait spans runs rather than turns, the loop reports it **once, when the run ends**, naming each gate and the tools it is holding. It is not an `Error Message`. A notice repeated every turn would accumulate in a context that only grows, which is the same pressure the false error applied.

> [!HEADSUP] Proactive vs. Reactive Error Handling
>
> The loop's ability to catch errors and report them back to the LLM is a **reactive** safety net for unplanned failures. However, the most robust workflows are designed with failure in mind from the start.
>
> Using the :term[Plan]{href="./012_agent_plan.md"} system with branching :term[Expressions]{href="./011_agent_expressions.md"}, you can create explicit "happy" and "unhappy" paths for your tools. An :term[Activity]{canonical="Activity"} can handle its own errors internally and, instead of throwing an exception, return a specific `Data Message` that directs the workflow down a pre-planned error-handling branch. This **proactive** approach is detailed further in the Plan chapter.

## From Simple Loops to Strategic Plans

The :term[Execution Loop]{canonical="Execution Loop"} provides a dynamic structure for agent behavior, but its power is in executing tactical, single-shot :term[Requests]{canonical="Request"}. To manage complex, multi-step workflows with dependencies, a more advanced system of strategic planning is required.

:term[011: Agent/Expressions]{href="./011_agent_expressions.md"} introduces the protocol for defining these long-term strategies as a graph of interconnected :term[Tool Calls]{canonical="Call"}.
