# 009: Agent/State

> [!DEFINITION] [State Message](./000_glossary.md)
> A special message that works like a workflow's live memory. Think of it as a set of sticky notes that the agent can read from and write on, letting it remember things across many steps.

> Sidenote:
> - Needs These Ideas First:
>   - [004: Agent/Call](./004_agent_call.md)
>   - [005: Agent/Data](./005_agent_data.md)
> - Makes These Ideas Possible:
>   - [012: Agent/Plan](./012_agent_plan.md)
> - Works Together With:
>   - [013: Agent/Instancing](./013_agent_instancing.md)
>   - [014: Agent/Delegate](./014_agent_delegate.md)
>   - [015: Agent/Scopes](./015_agent_scopes.md)

The **State message** is a special kind of :term[Data]{canonical="Data"} message that gives an agent a working memory during its :term[Execution Loop]{canonical="Execution Loop"}. If you think of :term[Variables]{canonical="Variable"} as the cables that connect different tools, then the :term[State]{canonical="State"} is the **whiteboard** where the agent jots down the results. It’s where information is stored and updated as the agent works through a task.

The :term[State]{canonical="State"} is the single source of truth for what's happening. It's the key to making the agent's work robust. Because it saves a complete snapshot of the workflow, a task can be paused and started again later right where it left off. When the agent starts a new cycle, it looks at the :term[State]{canonical="State"} from the last cycle to understand what's already been done and what to do next.

## Guiding the Workflow with a Schema

You can give the :term[State]{canonical="State"} a `schema`, which is like a template or a set of rules for what the agent's memory should look like. This is optional but very helpful. A schema outlines what kind of information should be stored, which gives the AI hints about how the different :term[Tools]{canonical="Tool"} are supposed to work together.

This creates a helpful guide for the AI. When it knows what kind of information belongs in the :term[State]{canonical="State"}, it's more likely to call the right :term[Tools]{canonical="Tool"} and tell them to save their results in the right place (using an :term[Output Path]{canonical="Output Path"}). This helps make sure the agent's actions are logical and follow the plan.

> Sidenote:
> - [007: Agent/Variables](./007_agent_variables.md)
> - [008: Agent/Output](./008_agent_output.md)

## Multi-Step Tools

The main job of the :term[State]{canonical="State"} message is to let different :term[Tools]{canonical="Tool"} share information as part of one continuous process. It acts as a shared whiteboard where tools can post their results for others to see.

This works with a simple read-and-write system. One :term[Tool]{canonical="Tool"} writes its result to the :term[State]{canonical="State"}, and a later :term[Tool]{canonical="Tool"} can read that information and use it as its starting point. This lets you build chains of tools where the output of one becomes the input for the next, all without losing track of what’s going on.

## Planning vs. Execution

Using the :term[Output Path]{canonical="Output Path"} to write to the state and :term[Variable References]{canonical="Variable Reference"} to read from it is the secret to how an agent can separate planning from doing. It allows the agent to map out a full workflow—a chain of :term[Tool Calls]{canonical="Call"} connected by a flow of information—*before* a single tool is actually run.

This map of how data will move can be checked for errors, saved for later, or even simulated. It works perfectly with how AIs think, which is to figure out the whole plan first. The system is flexible because a designer can either let the AI decide where to get its inputs and save its outputs, or they can set those paths in stone to create a very predictable and reliable process.

> [!HEADSUP] Heads up
> When you create a set of :term[Tool Calls]{canonical="Call"} that are linked together through the :term[State]{canonical="State"}, you are creating a plan. This system provides all the pieces needed for that: a lasting :term[State]{canonical="State"} to act as the whiteboard, :term[Variable References]{canonical="Variable Reference"} and the :term[Output Path]{canonical="Output Path"} to act as the connectors, and the agent's :term[Loop]{canonical="Loop"} to provide the engine that runs it all. Together, these allow the agent to build a complete map of how data will flow, which is the heart of a :term[Plan]{canonical="Plan"}.
>
> > Sidenote:
> >
> > - [010: Agent/Loop](./010_agent_loop.md)
> > - [012: Agent/Plan](./012_agent_plan.md)

## How It Works with Other Parts

- **:term[Call]{canonical="Call"}:** A :term[Tool Call]{canonical="Call"} is deeply connected to the :term[State]{canonical="State"} through its :term[Output Path]{canonical="Output Path"} property. This property turns a simple action (like using a tool) into an action that changes the agent's memory. By telling the :term[Call]{canonical="Call"} where to save its result, the agent can record what happened. This allows a series of :term[Calls]{canonical="Call"} to build on each other's results, creating a chain of actions recorded in the :term[State]{canonical="State"}.

  > Sidenote:
  > - [004: Agent/Call](./004_agent_call.md)

- **:term[Data]{canonical="Data"}:** The :term[State]{canonical="State"} is just a special version of a :term[Data]{canonical="Data"} message (specifically one with `kind: "state"`). It uses all the core features of :term[Data]{canonical="Data"} messages to give the agent a memory. The `schema` property is used to define the structure of this memory, giving the AI a blueprint to follow. The way :term[Data]{canonical="Data"} messages can be merged is also very important, as it allows the :term[State]{canonical="State"} to be updated in small pieces, which the system then combines into one clear picture.

  > Sidenote:
  > - [005: Agent/Data](./005_agent_data.md)

- **:term[Scopes]{canonical="Scope"}:** :term[Scopes]{canonical="Scope"} are how a tool running in an isolated space (like a :term[Delegate]{canonical="Delegate"}) can be given access to the main workflow's memory. When an agent hands off a task, the `_scopes` property can tell it to include the :term[State]{canonical="State"} in the tool's private workspace. This lets tools that are sealed off for safety still read from the main :term[State]{canonical="State"} in a controlled way.

  > Sidenote:
  > - [015: Agent/Scopes](./015_agent_scopes.md)

- **:term[Instancing]{canonical="Instancing"}:** The :term[State]{canonical="State"} works perfectly with the :term[Instancing]{canonical="Instancing"} system. If a task needs to be run on many different items at once, each item gets its own separate :term[State]{canonical="State"} (its own memory). When a tool needs to get some information (e.g., `†state.currentUser.id`), the system automatically knows which item's memory to look in. This lets a single, general-purpose Plan run on many different tasks at the same time without them getting mixed up.

  > Sidenote:
  > - [013: Agent/Instancing](./013_agent_instancing.md)

- **:term[Plan]{canonical="Plan"}:** While the :term[State]{canonical="State"} is useful for simple sequences of tools, it truly shines as the foundation of the :term[Plan]{canonical="Plan"} system. A :term[Plan]{canonical="Plan"} describes a workflow as a map where the :term[Tool Calls]{canonical="Call"} are the locations. The :term[State]{canonical="State"} provides the roads connecting these locations. It allows one tool to leave a result and others to pick it up, which makes complex workflows with choices (if/then) or parallel tasks possible.

  > Sidenote:
  > - [012: Agent/Plan](./012_agent_plan.md)

## From Single State to Orchestrated Workflows

The :term[State]{canonical="State"} message gives us a way to manage the memory of a single workflow. With a whiteboard for remembering things and variables to connect tools, we can now build and run complex, multi-step projects.

The next document, :term[012: Agent/Plan]{href="./012_agent_plan.md"}, explains how we can organize these workflows into a map of :term[Tool Calls]{canonical="Call"}.
