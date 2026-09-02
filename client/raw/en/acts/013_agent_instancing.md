# 012: Agent/Instancing

> [!DEFINITION] [Instancing](./000_glossary.md)
> A protocol for grouping context messages by assigning them a shared, unique identifier. This creates distinct :term[Instances]{canonical="Instance"} within a single request, enabling parallel, multiplexed computation.

> Sidenote:
>
> - Requires: :term[009: Agent/State]{href="./009_agent_state.md"}
> - Compatible:
>   - :term[006: Agent/Input]{href="./006_agent_input.md"}
>   - :term[015: Agent/Scopes]{href="./015_agent_scopes.md"}
>   - :term[012: Agent/Plan]{href="./012_agent_plan.md"}

The **Instancing Protocol** is a pattern for scaling agentic workflows by enabling multiple threads of computation within a single request. It allows any agent to operate on multiple, independent data contexts concurrently without requiring any changes to its core logic. This is achieved by associating context messages, such as :term[State]{canonical="State"} or :term[Input]{canonical="Input"}, with unique instance identifiers. As a result, a single, reusable :term[Plan]{canonical="Plan"} can be executed across many instances in parallel, but the protocol is generic and supports any combination of messages, improving throughput and consistency.

## The Instancing Mechanism

Instancing provides a way to process multiple, independent data contexts by layering on top of data-carrying messages like :term[State]{href="./009_agent_state.md"}. Instead of providing a single :term[State]{canonical="State"} message, a request can include an array of them, each representing a distinct :term[Instance]{canonical="Instance"} of a task.

To manage these concurrent contexts, each :term[State]{canonical="State"} message is assigned a **unique identifier** via a special `_instance` property. These identifiers are short, unique tokens (e.g., `①`, `②`) that allow the LLM to associate operations with a specific :term[Instance]{canonical="Instance"}.

Instancing is an opt-in feature enabled by the caller on a per-message basis. By adding the `_instance` property to a data message like :term[State]{canonical="State"} or :term[Input]{canonical="Input"}, the caller signals that this message should be treated as a distinct, thread-safe execution context. If the `_instance` property is omitted, the message is treated as a global resource available to all instances.

This approach provides benefits:

- **Efficiency**: It multiplies the throughput of the system by processing many instances in a single LLM request.
- **Consistency**: By allowing the LLM to see multiple related instances in a single context, it can generate more consistent and higher-quality plans.

## Composition with Context Messages

The protocol relies on how the `_instance` identifier scopes the behavior of different context message types.

- **:term[State]{canonical="State"}:** The :term[State]{canonical="State"} message is the core of the protocol. Each :term[Instance]{canonical="Instance"} is a distinct :term[State]{canonical="State"} message, uniquely identified by the `_instance` property. This provides an isolated canvas for a sequence of operations, ensuring that parallel workflows do not interfere with one another.

  > Sidenote:
  >
  > - :term[009: Agent/State]{href="./009_agent_state.md"}

- **:term[Input]{canonical="Input"}:** An :term[Input]{canonical="Input"} message can be used in two ways. A global :term[Input]{canonical="Input"} message (without an `_instance` identifier) provides configuration for all instances in a batch. A targeted :term[Input]{canonical="Input"} message (with an `_instance` identifier) provides data for a specific :term[State]{canonical="State"} message, overriding any global input.

  > Sidenote:
  >
  > - :term[006: Agent/Input]{href="./006_agent_input.md"}

- **:term[Scopes]{canonical="Scope"}:** The `_instance` identifier provides critical data isolation for :term[Scopes]{canonical="Scope"}. When a :term[Call]{canonical="Call"} targets a specific instance, its `_scopes` are also scoped to that instance's context. This is what allows a :term[Delegate]{canonical="Delegate"} to see only the data relevant to its specific unit of work, even when it is being orchestrated as one of many within a larger, multi-instance request.

  > Sidenote:
  >
  > - :term[015: Agent/Scopes]{href="./015_agent_scopes.md"}

:::details{title="Example: Content Moderation at Scale"}

Instancing transforms single-task agents into batch processors. Consider an agent designed to moderate user comments against a set of community guidelines.

Without instancing, the agent would operate sequentially. To review 100 comments, it would require 100 separate :term[Requests]{canonical="Request"}. This is not only slow but also context-blind; the agent judges each comment in complete isolation, which can lead to inconsistent rulings on similar content.

With the **Instancing Protocol**, the workflow is parallelized within a single :term[Request]{canonical="Request"}.

1.  **Batch Input**: The agent receives a `context` containing an array of 100 :term[Input]{canonical="Input"} messages. Each message is assigned a unique `_instance` identifier and contains a different user comment.

    ```json
    // Batch of Input messages provided to the LLM
    [
      {
        "type": "input",
        "_instance": "①",
        "comment": "This is a great post!"
      },
      {
        "type": "input",
        "_instance": "②",
        "comment": "I disagree with this..."
      },
      {
        "type": "input",
        "_instance": "③",
        "comment": "This is spam."
      }
      // ... 97 more comment instances
    ]
    ```

2.  **Parallel Plan Execution**: The LLM can now see the entire batch. It might use a single :term[Plan]{canonical="Plan"} that defines a two-step process: first, analyze the sentiment, then check against a list of forbidden keywords. This same plan is applied to all instances.

3.  **Targeted Output**: The agent's :term[solution]{canonical="Solution"} will contain a flat list of :term[Calls]{canonical="Call"}, but each `Call` is directed at a specific comment via the `_instance` property.

    ```json
    // Solution generated by the LLM
    {
      "calls": [
        {
          "_tool": "moderateComment",
          "_instance": "①",
          "decision": "approve"
        },
        {
          "_tool": "moderateComment",
          "_instance": "②",
          "decision": "approve"
        },
        {
          "_tool": "moderateComment",
          "_instance": "③",
          "decision": "reject"
        }
        // ... 97 more moderation calls
      ]
    }
    ```

This approach provides two major advantages:

- **Throughput**: What previously took 100 round-trips to the LLM is now accomplished in one.
- **Consistency**: By viewing the entire batch, the LLM gains a broader context. It can make more consistent judgments and even identify patterns (like a coordinated spam attack) that would be invisible when processing comments one by one.

:::

:::::details{title="Global Input with Instanced State"}

This example shows how a global `Input` message can direct the agent's focus to a specific instance within a larger batch. This allows for dynamic, targeted control over parallel workflows, turning a simple batch processor into a more sophisticated orchestrator, like a manager directing individual employees.

::::columns
:::column{title="Context"}

This scenario uses the metaphor of a manager directing employees. A single, global `input` acts as a high-level command, while multiple, instanced `state` messages represent employees with their own tasks. This setup demonstrates how an agent designed to manage a single entity can seamlessly scale to orchestrate many, without any change in its core design.

```json
[
  // A global instruction for a specific employee
  {
    "type": "input",
    "instruction": "Give employee B them a new, high-priority task to 'Finalize the quarterly report'."
  },

  // The current state of all employees
  {
    "type": "state",
    "_instance": "employee_A",
    "task": "Draft initial proposal",
    "status": "In Progress"
  },
  {
    "type": "state",
    "_instance": "employee_B",
    "task": "Review team submissions",
    "status": "Blocked"
  }
]
```

:::
:::column{title="Targeted Solution"}

Despite the `Input` being global, the LLM correctly interprets the natural language instruction. It generates a `Call` that is precisely targeted at `employee_B` using the `_instance` property, showing its ability to intelligently route commands. The other 'employee' instance is entirely unaffected.

```json
{
  "calls": [
    {
      "_tool": "updateTask",
      "_instance": "employee_B",
      "newTask": "Finalize the quarterly report",
      "newStatus": "High Priority",
      "output": "†state"
    }
  ]
}
```

:::
::::
:::::

## Interactions with other systems

Instancing integrates with higher-level protocols to manage execution flow.

- **:term[Calls]{canonical="Call"}:** The `_instance` property on a :term[Call]{canonical="Call"} is the core mechanism that directs its execution. It ensures that all state manipulations—whether writing to an :term[Output Path]{canonical="Output Path"} or reading a value from the state to use as an input—are correctly scoped to the intended :term[Instance]{canonical="Instance"}.

  > Sidenote:
  >
  > - [008: Agent/Output](./008_agent_output.md)

- **:term[Plan]{canonical="Plan"}:** The :term[Plan]{canonical="Plan"} message is not subject to instancing; it acts as a single, global template for a workflow. It can be combined with multiple, instanced :term[State]{canonical="State"} messages, allowing the same plan to be executed concurrently across many independent data contexts in a single request.

  > Sidenote:
  >
  > - :term[012: Agent/Plan]{href="./012_agent_plan.md"}

## From Planning to Process

Where a :term[Plan]{canonical="Plan"} provides the reusable template for a workflow, and :term[Instancing]{canonical="Instancing"} provides the mechanism to execute it at scale, a **:term[Process Idea]{href="./203_idea_process.md"}** is the artifact that captures the result. It is the complete, self-contained record of a strategic plan and its live execution state across all instances.
