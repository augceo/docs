# 012: Agent/Plan

> [!DEFINITION] [Plan](./000_glossary.md)
> A context message holding a map of :term[Tool Calls]{canonical="Tool Call"} that represents an agent's strategy. It passes forward between steps so the agent remembers what it is doing and adapts along the way.

> Sidenote:
> - Requires:
>   - :term[004: Agent/Call]{href="./004_agent_call.md"}
>   - :term[010: Agent/Loop]{href="./010_agent_loop.md"}
>   - :term[007: Agent/Variables]{href="./007_agent_variables.md"}
>   - :term[009: Agent/State]{href="./009_agent_state.md"}
>   - :term[013: Agent/Instancing]{href="./013_agent_instancing.md"}

The :term[Plan]{canonical="Plan"} is how the agent avoids amnesia. While the system can easily handle single, quick requests, putting a `Plan` message into the mix turns the interaction into an ongoing workspace. It tells the :term[Execution Loop]{canonical="Execution Loop"} to hold onto the strategy and the working :term[State]{canonical="State"} over multiple turns.

Think of it like giving a chef a recipe card. When the AI gets the current :term[Plan]{canonical="Plan"} along with everything sitting on the counter (the live :term[State]{canonical="State"}), it knows exactly where it is in the process. It can smoothly follow the recipe, pencil in new :term[Tool Calls]{canonical="Tool Call"} if needed, or throw the card away and write a new one if dinner burns. Without a `Plan` message, every request is a brand-new, isolated event where no progress carries over.

## How a Plan is Formed

The steps in the map are not tied together with rigid commands. They connect by passing materials across the :term[State]{canonical="State"} workspace.

- **Nodes (:term[Tool Calls]{canonical="Tool Call"}):** Every step is a :term[Tool Call]{canonical="Tool Call"}, representing a specific action to take.
- **Edges (:term[State]{canonical="State"} Object):** Steps link up by placing items on the counter and picking them up. One :term[Tool]{canonical="Tool"} finishes its job and drops the result in a specific spot in the :term[State]{canonical="State"} using the :term[Output Path]{canonical="Output Path"} property. The next :term[Tool]{canonical="Tool"} picks up that item by pointing to the same spot with a **:term[Variable Reference]{canonical="Variable Reference"}**.

  > Sidenote:
  > - [008: Agent/Output](./008_agent_output.md)

This sets a permanent rule: the second step cannot possibly start until the first step has actually finished and placed its item on the counter.

For example, a :term[Plan]{canonical="Plan"} to grab a user's profile and summarize it looks like this:

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

Here, the `summarizeProfile` action waits for the output of `fetchUserProfile`. This creates a two-step plan, behaving just like a simple map.

## The Plan's Content: A Data-Flow Graph

The actual content inside a :term[Plan]{canonical="Plan"} message is a data-flow graph. This is simply a map showing the agent's strategy as a chain of connected tasks. Drawing the workflow this way clearly marks how one :term[Tool Call]{canonical="Tool Call"}'s output feeds directly into another's input. It builds a perfect, machine-readable track for the :term[Execution Loop]{canonical="Execution Loop"} to drive along.

> Sidenote:
> A :term[Plan]{canonical="Plan"} effortlessly handles branching tracks, steering execution down different lines based on what happened earlier:
>
> ```mermaid
> graph TD
>     A[Get Weather] --> B{Is it Sunny?};
>     B -- state.sunny --> C[Find a Park];
>     B -- state.notSunny --> D[Find a Movie];
>     C -- state.suggestion --> E[Present Suggestion];
>     D -- state.suggestion --> E[Present Suggestion];
> ```

This map structure is flexible. You can ask an agent to draw a map representing a social network, a GitHub Actions workflow, or a database layout.

It is vital to tell these pictures apart from a real :term[Plan]{canonical="Plan"}. Just because something is drawn like a map does not mean the agent will immediately try to run it. A drawing only becomes an active plan when it is pushed into a :term[Request]{canonical="Request"} with instructions to execute it. This prevents the system from accidentally launching a database schema as if it were a list of chores.

While an agent can use this map format to brainstorm out loud, its real job here is running live workflows. We use a specific type of map called a **Directed Acyclic Graph (DAG)**.

This map has strict rules that make it safe to run automatically:

- **Graph:** The full package—the complete list of tasks (nodes) and the flow of data connecting them (edges).
- **Directed:** Traffic only flows one way. An action that creates data must physically happen _before_ an action that uses that data.
- **Acyclic:** The tracks never loop back on themselves. It has a clear start and finish. This strictly stops the AI from getting entirely trapped in an endless loop. The system checks the map for loops before it ever starts the engine.

**A step only triggers when its :term[Output Path]{canonical="Output Path"} actually receives a value.** There are no checkboxes or finish lines. The engine simply looks at what is sitting on the counter. Everything else in this chapter—branching paths, waiting for answers—comes directly from this single physical rule.

> Sidenote:
> To handle repeating tasks—like a continuous looping checklist—the system uses a separate mini-plan. A larger :term[Plan]{canonical="Plan"} keeps the master count, and triggers a clean, isolated sub-task for each round via a :term[Delegate]{canonical="Delegate"}. This strictly guarantees all repetitive actions remain safe and won't spin out of control.

## Planning and Execution Are One Act

**Planning is execution.** There is no "thinking mode" and "doing mode." When the AI writes a `solution`, it instantly reasons through and executes any step it possibly can.

- A mental task (like summarizing text) writes its command and its final answer at the exact same moment.
- An external action (an :term[Activity]{canonical="Activity"}) is fired off by the :term[Execution Loop]{canonical="Execution Loop"} the absolute second all of its required :term[Variable References]{canonical="Variable Reference" href="./007_agent_variables.md"} land on the counter.

That simple timing rule eliminates the need for separate modes. **A task wakes up exactly when its ingredients exist.** If a step shouldn't run yet, it simply means its ingredients aren't there yet.

An agent does not have to finish the whole plan in one breath. It writes down the entire map at once and handles what it can. If a mental step depends on an ingredient that hasn't arrived, it leaves the answer blank instead of making something up. It just waits in the plan for the next loop, when the missing piece finally arrives and wakes it up.

Branching paths (`†state.sunny <|> †state.rainy`) survive loops for the exact same reason. The external action decides the path when it actually runs, so nothing forces a fake choice early. A **latent** (mental) call is chosen immediately because thinking *is* choosing. The paths the AI didn't pick are left empty, and any tasks waiting down those empty paths safely coast along without throwing false errors.

### Proposing Rather Than Performing

Sometimes you want a deliberate pause—sketch the map and wait for human review. You don't need a special feature for this. You just use the basic rule on purpose: **a proposal is just a task waiting behind a locked gate that nobody has opened yet.**

We label these gates purely by putting them in a specific spot: **Any path inside `†state.approvals` is an approval gate.** There is no unique setting to toggle or extra checklist to manage. The lock is just the path itself.

Give every choice its own gate and let the tasks read from them. A Plan A waits on `†state.approvals.planA`, a Plan B waits on `†state.approvals.planB`. Neither gate is open, so neither plan starts. They sit there purely as proposals—a complete map waiting for a green light. This pause allows you to:

- **Validate:** check the map for broken links.
- **Simulate:** test-drive how the workflow will behave.
- **Present for Approval:** show it to a person for review before taking action (:term[HITL]{canonical="HITL (Human-in-the-Loop)"}).

Approval is simply someone dropping a value into the right gate. The waiting tasks see the value, wake up, and start running. The ignored alternatives stay asleep and eventually drop off. Which choice a task belongs to is determined entirely by which gate it watches—there are no separate tracking lists.

In reality, only the VERY FIRST step of an alternative watches the gate; the rest just watch what that first step produces. The entire chain hangs securely off that single lock.

**Approval happens between runs, never during one.** The machine stops, the gate is opened, and the next run starts with the key already turned. Nothing has to awkwardly pause mid-run, and the :term[Execution Loop]{canonical="Execution Loop"} requires zero complicated machinery to handle it.

**A locked proposal is not a failure.** It is just happily waiting. A task waiting for an impossible item is broken. The system easily tells the difference because of where they wait. The system only reports the wait once when the active run ends, listing what is locked, rather than complaining repeatedly and tricking the AI into thinking waiting is a mistake.

**Per-instance approval needs nothing extra.** A gate is just a normal spot in the :term[State]{canonical="State"}. Opened globally, everybody moves forward; opened locally for only one instance, it moves forward while the others quietly stay locked. Specificity rules in :term[013: Agent/Instancing]{href="./013_agent_instancing.md"} already handle this perfectly.

Because there is no "propose mode," you must physically lock the gates: **if a proposed task already has all its ingredients, it will run instantly.** Reading an already-finished value is an action, not a proposal. A true proposal must be gated.

## The Plan as an Evolving Strategy

A :term[Plan]{canonical="Plan"} is fully alive. It adapts at every step. A simple list of commands only becomes a true :term[Plan]{canonical="Plan"} when it is actively fed back into the next loop.

This continuous cycling turns simple outputs into a living strategy:

- The **:term[context]{canonical="context"}** hands the AI its current :term[State]{canonical="State"} and the previous :term[Plan]{canonical="Plan"}.
- The **:term[solution]{canonical="Solution"}** adds new :term[Tool Calls]{canonical="Tool Call"} that become the **new :term[Plan]{canonical="Plan"}** for the next step.

This lets the agent follow the tracks or build brand new ones on the fly. If an action fails, it simply draws a new map to handle the error. This keeps everything highly resilient.

:::::details{title="Example: Planning one step ahead"}

This shows how a `Plan` keeps the AI on the straight and narrow, preventing it from getting distracted by shiny tools when trying to finish a standard job.

**1. Initial Request**

The customer asks for something. Based on the `input`, the AI formulates a standard two-step `Plan` to handle the refund. This maps out the ideal scenario.

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

The :term[Execution Loop]{canonical="Execution Loop"} checks the billing history and puts it on the counter. The history shows a messy chargeback. An unguided AI might panic and call a supervisor tool.

But the `Plan` sitting right there provides a track to follow. By blending what it _knows_ (the messy `State`) with what it _should do_ (the `Plan`), it sticks to the intended path and finishes the refund.

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

The `Plan` acts as an anchor, avoiding unnecessary escalations.

:::::

:::::details{title="Example: Adjusting a Plan"}

This shows how an AI adapts easily when new information changes the rules entirely.

::::columns
:::column{title="Context"}

It is handed a standard travel `Plan` and a sudden `Input` mentioning a pet.

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

The AI sees the hotel tool is useless now. It scraps the old step and maps out a brand new path using a specialized pet tool.

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

It fundamentally alters the journey based on new obstacles, sending a new `Plan` into the :term[Execution Loop]{canonical="Execution Loop"}.

:::::

:::::details{title="Example: Handling Failure"}

This shows how the AI rips up the map when things go wrong and draws a detour.

**1. The Initial Plan**

The AI draws up a rosy, two-step map assuming everything will go perfectly.

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

The `processPayment` tool completely fails. The system drops a `†state.error` on the counter. On the next loop, the AI sees the error alongside its useless happy-path plan. It ignores the useless plan and builds a fix.

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

The agent recognized the obstacle, dumped the old tracks, and laid down a single step to `reportFailure`.

:::::

:::::details{title="Example: Schema-Guided Planning"}

This shows how feeding a `schema` to the AI acts as a "fill-in-the-blanks" coloring book, effortlessly guiding it to wire things correctly.

::::columns
:::column{title="Context with State Schema"}

We hand the AI an `Input` and a `State` containing just a `schema`. This defines the exact shapes the data needs to fit into.

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

The AI uses the blueprint to wire everything perfectly, plugging outputs neatly into the preset blanks.

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

By providing the blueprint, the AI doesn't have to guess how to name things—it just fills in the slots.

:::::

This endless cycle of reading maps, hitting roadblocks, and drawing new maps is the beating heart of a :term[Process]{href="./203_idea_process.md"}. It takes a full snapshot of the available tools, the items on the counter, and the active plan itself.

## From Single Plan to Reusable Workflows

A :term[Plan]{canonical="Plan"} generally maps out a solitary task. To piece workflows together like standard parts, we let one map act as a trigger for a completely different map.

:term[013: Agent/Instancing]{href="./013_agent_instancing.md"} lays down the tracks for running these setups side-by-side.
