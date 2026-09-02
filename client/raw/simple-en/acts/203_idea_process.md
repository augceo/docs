# 203: Idea/Process

> [!DEFINITION] [Process Idea](./000_glossary.md)
> An [Idea](./101_concept_idea.md) that holds a strategic [Plan](./012_agent_plan.md) and keeps track of its progress. Think of it as a complete recipe book for a task. The `schema` is its list of all possible cooking techniques ([Tools](./002_agent_tool.md)), the `solution` is the *next* step in the recipe (`Plan`), and the `context` is everything it needs to know right now, like the original food request (`Input`), the current state of the dish (`State`), and the last recipe step it finished.

> Sidenote:
> - You should know about:
>   - [101: Concept/Idea](./101_concept_idea.md)
>   - [012: Agent/Plan](./012_agent_plan.md)
>   - [009: Agent/State](./009_agent_state.md)
>   - [006: Agent/Input](./006_agent_input.md)
> - Works well with:
>   - [013: Agent/Instancing](./013_agent_instancing.md)

While a [Vessel Idea](./202_idea_vessel.md) is for a single, one-off decision, a **Process Idea** is for a whole journey—a project with multiple steps. It’s the official record created by an AI as it works through a complicated task from start to finish. This makes it a complete log of what the AI *could* do and what it *chose* to do.

## How a Plan is Built

A `Process Idea` uses its three parts to give a complete picture of a project at any given moment. This picture shows not just the agent's strategy (the `Plan`), but also all the tools it has and the current status of the work.

- **`schema` (The Toolbox):** This is the full collection of all available [Tools](./002_agent_tool.md) that an agent can use to build its `Plan`. It’s like a toolbox containing every screwdriver, hammer, and wrench the agent might need.

- **`context` (What's Happening Now):** This holds all the information the agent needs to figure out the next step.
  - **Input Message:** The original request or problem that started the whole process. This usually doesn't change.

    > Sidenote:
    > - [006: Agent/Input](./006_agent_input.md)

  - **State Message:** This is the project's live data that changes and grows as work gets done. It holds all the information that connects one step of the plan to the next.

    > Sidenote:
    > - [009: Agent/State](./009_agent_state.md)

  - **Plan Message:** This message contains the confirmed plan from the *last* step. The agent can look at it and decide whether to stick with it or change course.

    > Sidenote:
    > - [012: Agent/Plan](./012_agent_plan.md)

- **`solution` (The Next Move):** This is the agent's strategy for the *current* step, laid out as a series of actions (`Calls`). This `solution` becomes the `Plan Message` for the very next step in the process.

### Planning and Doing are the Same Thing

A key idea here is that planning and doing happen at the same time. In many systems, you first make a big plan, and then you follow it. Here, they are merged into one. The agent doesn't create a blueprint and then start building. The act of figuring out the next `solution` (the new `Plan`) *is* the work for that moment.

This makes the system very quick and flexible. The `Plan` is like a living map that is being drawn as the journey happens. It doesn't have to be complete to be useful. When the AI generates the next step, it's not just planning; it's also checking its previous work, adjusting based on new information, and taking the next action, all in one go. There's no separate, slow step for re-planning if something changes.

> Sidenote:
> - [004: Agent/Call](./004_agent_call.md)

This all-in-one approach gives us two big advantages. First, you can test out a `Plan` just by looking at its toolbox (`schemas`), even before you have any real data. The AI can pretend to do each step, letting you design and check complex projects from the very beginning. Second, the AI acts like a smart adapter between steps. If one tool produces an output that isn't exactly what the next tool needs, the AI can smooth things over and make it fit. This makes the whole system very stable and prevents the kind of chain-reaction failures you see in rigid, old-fashioned systems.

> Sidenote:
> - [105: Concept/AI-Native](./105_concept_ai_native.md)

### How a Plan Can Grow Up

The `State` object allows a `Plan` to grow and mature through three stages, starting as a flexible sketch and becoming a reliable procedure:

> Sidenote:
> - [009: Agent/State](./009_agent_state.md)

1.  **Freeform Plan:** At the start, the `State` might not have a strict structure. This gives the agent total freedom to explore and figure out a `Plan`, creating new data pathways as it goes. Think of it as brainstorming.

2.  **Specified Connections:** The next step is to lock down the main data pathways by giving the `State` a `schema` (a structure). You can do this in two ways:
    - **Open Connections:** You can set the main pathways but still allow the agent to create new, minor ones if needed. It's like having a recipe but giving the chef freedom to add an extra spice.
    - **Closed Connections:** You can lock the structure down completely, forcing the agent to follow the exact recipe with no changes.

3.  **Resolved Plan:** This is the final stage, where a plan is considered fully grown and predictable. A `Resolved Plan` is the trusted plan that gets passed along at each step. It can be:
    - **Happy Plan:** A plan that works perfectly for the ideal, best-case scenario. It's fast but might need a change if something unexpected happens.
    - **Complete Plan:** A plan that has been tested and updated over time to handle common problems and different situations, making it extremely reliable.

This growth process allows a `Process Idea` to change from a fluid sketch into a rock-solid, reusable tool.

## Unchangeable Workflows: Running, Analyzing, and Growing

The real power of a `Process Idea` is that it's unchangeable. Every step in a project creates a new, complete snapshot of the `Plan` and `State`, keeping a perfect history of everything that happened. This idea of creating a chain of unchangeable records is what makes advanced features like managing, analyzing, and scaling up the work possible.

- **Reliable, Flexible Work:** The step-by-step process is naturally reliable because each new step starts with a complete picture of the last one. This prevents confusion and makes sure the agent's decision to stick to the `Plan` or change it is based on a clear and stable view of the world. The result of each step is a new `Process Idea`, moving the project forward safely.

  > Sidenote:
  > - [010: Agent/Loop](./010_agent_loop.md)
  > - [012: Agent/Plan](./012_agent_plan.md)

- **Super-Stable Management:** Because a `Process Idea` is a self-contained package, managing projects becomes very sturdy. You can pause a project just by saving its latest snapshot. Then, other systems—or a human—can look at that frozen state to make decisions. Any changes, like re-planning or tweaking the data, create a *new* `Process Idea`. This lets you safely restart the project without messing up the historical record.

  > Sidenote:
  > - [009: Agent/State](./009_agent_state.md)

- **High-Quality Time Travel:** This system is what makes interactive time travel possible. Every historical `Process Idea` is a perfect snapshot of the project at a specific moment. This lets you load any point in the project's past to check what happened, figure out problems, or run "what-if" scenarios by creating a new branch of the project from that point.

  > Sidenote:
  > - [202: Idea/Vessel](./202_idea_vessel.md)

- **Doing Many Things at Once:** The snapshot system allows for massive growth through a method called Instancing. This treats each task in a big batch as its own separate project, each with its own private `State` snapshot. A single `Plan` acts as the master strategy for all of them. In one single step, the agent plans for the entire batch at once, creating a unique set of actions for each task. This ensures all the tasks are handled in a consistent way while being incredibly efficient.

  > Sidenote:
  > - [013: Agent/Instancing](./013_agent_instancing.md)

## From a Predictable Plan to Smart Guidance

A `Process Idea` can become a **Resolved Plan**—a very predictable and reliable workflow. But being too rigid can be a bad thing. The system needs a way to add smart, situation-specific advice without breaking the trustworthy plan.

This is where an **[Instruction Idea](./204_idea_instruction.md)** comes in. An `Instruction` is like a recipe for thinking that gives higher-level advice. It acts as a "mental checklist" that helps the workflow in two ways: it guides the agent in picking or creating the best `Plan` for the situation, and it gives specific tips during the plan's execution. In short, an `Instruction` makes the project's *thinking* better, ensuring that both the big-picture strategy and the small, moment-to-moment actions are aligned with the right approach.

> Sidenote:
> - [204: Idea/Instruction](./204_idea_instruction.md)
