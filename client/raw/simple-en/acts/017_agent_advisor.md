# 017: Agent/Advisor

> [!DEFINITION] [Advisor](./000_glossary.md)
> A specific instruction (labeled as `kind: "advisor"`) that acts like a council member or expert. It tells the agent to pause, share a structured opinion, or give a confidence score *before* it picks a :term[Tool]{canonical="Tool"} or creates an :term[Output]{canonical="Output"}.

> Sidenote:
> - Requires:
>   - :term[001: Agent/Request]{href="./001_agent_request.md"}
> - Enhances:
>   - :term[002: Agent/Tool]{href="./002_agent_tool.md"}
>   - :term[010: Agent/Loop]{href="./010_agent_loop.md"}

The **Advisor Protocol** forces the system to "think before it acts." By setting up **Advisors**, the agent looks at the situation through the eyes of different experts—like a "Security Architect" looking for flaws, or a "Product Manager" focusing on users—before making a final move.

## Undirected Thought

When an AI tries to reason without clear rules, it often gets messy. Imagine someone wandering through a forest without a map, juggling clues, thoughts, and decisions all at once. The result is just a noisy dump of text.

This lack of structure fills the system's memory with too much rambling. Without a guide, the agent changes its mind constantly, leading to shaky choices and more mistakes. It makes the system slower and harder to trust. Plus, it is really hard to measure or compare a giant paragraph of rambling thoughts when trying to balance competing priorities.

The Advisor Protocol fixes this by giving the thinking process a clear blueprint. Instead of a messy block of text, the system gives you a neatly organized list of opinions, scores, and recommendations. This makes it instantly clear where the experts disagree or what they strongly prefer, making the final decision much easier to trust and track.

This protocol does not stop the agent from thinking; it just organizes the most important parts. It turns abstract thoughts into solid, measurable artifacts—like a scored voting ballot—that directly shape the final move.

## Divergent Perspectives & Tool Weighting

By creating multiple Advisors, the system forces the agent to look at the problem from opposing sides (for example, comparing "Risk" versus "Opportunity"). We capture these different views using **Weighted Tool Selection**, where Advisors cast actual votes or confidence scores for specific :term[Tools]{canonical="Tool"}.

The Agent gathers all these weighted votes from its council of experts. It does not just randomly "pick" a tool. Instead, it looks at the different opinions, weighs them against each other, and calculates the smartest path forward. Using a tool becomes a carefully mapped out decision based on expert advice.

## The Advisor Message

The **Advisor Message** is the setup that defines how this expert sees the problem.

- **`id`**: A unique name for the advisor (like `"securityArchitect"`).
- **`persona`**: What this advisor actually does. It sets the expert's background and what they care about. We call it `persona` instead of `role` because `role` is a system term for the message wrapper itself, and we want to keep things distinct.
- **`answer`**: A specific format (JSON Schema) that tells the advisor exactly how to reply. Instead of just talking, they can give precise numbers (like risk scores) or budgets. We call it `answer` because it shapes what the advisor gives back to us.
- **`on`**: Tells the system exactly *when* this advisor should speak up.
- **`scopes`**: A list of focus areas (like `["input", "state"]`) that tells the advisor exactly what clues to look at.
- **`isInstanced`**: A simple true/false switch (defaulting to `false`). If you turn it `true`, the advisor looks at pieces of data individually, giving unique advice for every single item in a large batch.

Both `persona` and `answer` are optional. If you tell the `answer` to be a strict yes-or-no format, the system guarantees you get exactly that format back, rather than a loose guess.

## One Advisor, Many Messages

An Advisor is not just a single message; it is a living profile that updates whenever a new message shares its `id`. If a newer message comes in, it adjusts the advisor's settings piece by piece. This means a request can easily tweak an Advisor that was set up earlier:

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

Notice that the message above does not list a `persona`, because it does not have to. The newest update always wins. If you use an `id` that no one has seen before, you just accidentally created a brand new advisor out of thin air. It is important to spell things correctly: a typo in the `id` will quietly create a blank, useless advisor instead of updating the one you actually wanted to change.

The `on` rule is the one exception here. It is **set once**. It decides how the system handles the Advisor—whether it runs quietly in the background or uses the `ConsultAdvisor` tool. If you try to change this setting later on, things break, because the system wouldn't know when to expect the answer. If a newer message leaves it blank, it just keeps using the original setting.

## Execution Strategies

The `on` property controls when the advisor steps in:

- **`start` (Initial)**: Speaks up only at the very beginning of a task to help steer the ship. Once the model actually starts working, this advisor goes quiet.
- **`finish` (Stopping)**: Asked for an opinion right at the end, usually to check if the job is truly done.
- **`request` (Continuous)**: A constant companion that shares an opinion at the start, the end, and during every single step in between. Perfect for safety guards or big-picture strategists.
- **`null` / `undefined` (On-Demand)**: Stays quiet in the background unless specifically asked. The system provides a special lifeline, `ConsultAdvisor`, so the agent can tap these experts on the shoulder whenever things get confusing.

## Advisory Order

When Advisors are active, the system builds their structured opinions _first_, before doing anything else. The agent puts on the hats of these different personas to score the current situation and cast votes. The Agent then grabs those results immediately to make the final choice about which :term[Tool]{canonical="Tool"} to use.

Hard numbers, like tool "calls", are packed tightly into **Inline JSON** within a simple text string. This keeps the data clean and strict without making the whole system too complicated.

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

- **:term[Tool]{canonical="Tool"}:** Advisors do not actually push buttons or change files; they just vote on what to do. The `advisors` section in the output might look like a tool call, but it is purely a thinking step. It proves that the agent gathered expert advice before making a real move with a :term[Tool]{canonical="Tool"}.

  > Sidenote:
  > - :term[002: Agent/Tool]{href="./002_agent_tool.md"}

- **:term[Loop]{canonical="Loop"}:** The `on` setting wires the advisor right into the agent's heartbeat. Advisors marked `on: request` act like security cameras, checking in on every single loop. Those marked `on: null` are emergency levers, waiting to be pulled when the agent gets stuck during execution.

  > Sidenote:
  > - :term[010: Agent/Loop]{href="./010_agent_loop.md"}

- **:term[Plan]{canonical="Plan"}:** Getting advice is incredibly valuable when setting up a plan. By talking to advisors marked `on: start` or `on: request`, the agent makes sure its first steps line up with expert consensus, building a :term[Plan]{canonical="Plan"} that is solid from day one.

  > Sidenote:
  > - :term[012: Agent/Plan]{href="./012_agent_plan.md"}

- **:term[Instancing]{canonical="Instancing"}:** Advisors handle large groups of tasks well. By flipping `isInstanced: true`, the advisor knows to look at items individually. The system will create targeted advice for each specific piece of data, making sure no item gets skipped or rubber-stamped.

  > Sidenote:
  > - :term[013: Agent/Instancing]{href="./013_agent_instancing.md"}

- **:term[Scopes]{canonical="Scopes"}:** The `scopes` setting is a way to point a flashlight for the advisor. Instead of locking away data completely, it acts as **soft guidance**. It hints to the advisor exactly where to look (like `["†state.deploymentHistory"]`), keeping everything organized without hiding the rest of the puzzle.

  > Sidenote:
  > - :term[015: Agent/Scopes]{href="./015_agent_scopes.md"}
