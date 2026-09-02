# 004: Agent/Call

> [!DEFINITION] [Call](./000_glossary.md)
> A :term[Tool]{canonical="Tool"} is like a recipe for a task. A **:term[Call]{canonical="Call"}** is when you actually follow that recipe with specific ingredients. It's an instruction to _do something now_.

> Sidenote:
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

The :term[Tool]{canonical="Tool"} system sets up the list of things an AI agent knows how to do. The **:term[Call]{canonical="Call"}** system is all about how the agent actually does them.

A :term[Call]{canonical="Call"} is like taking a general :term[Tool]{canonical="Tool"} (like `searchTheWeb`) and turning it into a specific command (`searchTheWeb for 'pictures of kittens'`). While :term[Tools]{canonical="Tool"} describe _what can be done_, :term[Calls]{canonical="Call"} are the commands that say _how to do it right now_.

> [!HEADSUP] Heads up
> When you ask the agent to do something (a :term[Request]{href="./001_agent_request.md"}), and it decides to use several of its :term[Tools]{canonical="Tool"} to get the job done, that bundle of commands (:term[Calls]{canonical="Call"}) is called a :term[Vessel]{href="./202_idea_vessel.md"}. A :term[Vessel]{canonical="Vessel"} is like the agent's immediate plan of action.
>
> > Sidenote:
> >
> > - [001: Agent/Request](./001_agent_request.md)
> > - [202: Idea/Vessel](./202_idea_vessel.md)

## Call Results and Final Output

> [!DEFINITION] [Final Output](./000_glossary.md)
> The final answer the agent gives you once it has completely finished your entire request. It's the end result you were waiting for.

When you make a :term[Request]{canonical="Request"}, the goal is to get back a **:term[Final Output]{canonical="Final Output"}**. The AI brain (LLM) looks at all the information it has and makes a choice:

- If it has enough information, it creates the :term[Final Output]{canonical="Final Output"} right away, and the job is done.
- If it doesn't have enough information, it realizes it needs to learn more. So, instead of a final answer, it creates a set of :term[Calls]{canonical="Call"} to go find the missing pieces.

After these :term[Calls]{canonical="Call"} are finished, their results are added to the agent's memory. Then, the agent tries again with this new, better information, giving it another chance to produce the :term[Final Output]{canonical="Final Output"}. This cycle of asking questions and making :term[Calls]{canonical="Call"} keeps going until the main goal is reached.

> Sidenote:
> - [010: Agent/Loop](./010_agent_loop.md)

This separation between tiny steps (getting results from :term[Calls]{canonical="Call"}) and the big final answer is what lets us build powerful agents that can solve complex, multi-step problems.

## Interactions with other systems

A :term[Call]{canonical="Call"} is just a simple instruction on its own. It becomes powerful when it works with other systems that control how it runs. These other systems are activated by special properties in a :term[Tool]{canonical="Tool"}'s blueprint (these properties start with `_`). This allows a single :term[Call]{canonical="Call"} to behave in many different ways.

Think of it like adding special modifiers to a video game character's ability. By understanding what these special properties mean, the AI can creatively combine them to build complex plans. It goes from just picking a :term[Tool]{canonical="Tool"} to inventing whole workflows on the fly. This teamwork between :term[Calls]{canonical="Call"} and these other systems is the secret to how the agent tackles big, tricky tasks.

> [!TIP]
> The next few points explain how :term[Calls]{canonical="Call"} connect to other parts of the system we'll discuss later. You don't need to understand them perfectly right now. You can always come back here later.

- **Doing something specific (`_activity`)**: The most basic upgrade is linking a :term[Call]{canonical="Call"} to a piece of computer code that always does the same thing. The `_activity` property tells the system, "Don't just think about this; run this specific code to get the answer."

  > Sidenote:
  > - [003: Agent/Activity](./003_agent_activity.md).

- **Asking for help (`_delegate`)**: A :term[Call]{canonical="Call"} can pass a job to another expert agent, called a :term[Delegate]{canonical="Delegate"}. It's like a manager giving a task to a specialist. The `_delegate` property usually points to a saved :term[Request]{canonical="Request"}, letting the agent reuse complex tasks easily. This keeps the work separate and organized.

  > Sidenote:
  > - [014: Agent/Delegate](./014_agent_delegate.md).

- **Focusing its attention (`_scopes`)**: The :term[Scopes]{canonical="Scope"} system tells a :term[Call]{canonical="Call"} exactly what information it's allowed to look at. This is like giving someone only the documents they need for a specific task instead of the whole library. It helps the AI focus and produce better answers. When used with a :term[Delegate]{canonical="Delegate"}, it's even stricter, defining the *only* information the helper agent can see.

  > Sidenote:
  > - [015: Agent/Scopes](./015_agent_scopes.md).

- **Saving the result (`_outputPath`)**: A :term[Call]{canonical="Call"} can be told where to save its result. It's like telling someone to put a finished document in a specific folder. The :term[Output Path]{canonical="Output Path"} property points to a spot in the agent's memory (:term[State]{canonical="State"}). This lets you chain tasks together, where the result of one :term[Call]{canonical="Call"} becomes the starting point for the next one.

  > Sidenote:
  > - [009: Agent/State](./009_agent_state.md).
  > - [008: Agent/Output](./008_agent_output.md).

- **Working on many things at once (`_instance`)**: A :term[Call]{canonical="Call"} can be told to work on one specific item in a big batch of items. The `_instance` property gives each item a unique ID, so the agent can run the same plan on many different things at the same time without getting confused. It’s like having an assembly line where each worker focuses only on the product right in front of them.
  > Sidenote:
  > - [013: Agent/Instancing](./013_agent_instancing.md).

## From Actions to Data

While :term[Calls]{canonical="Call"} are about what an agent *does*, that's only half the story. The other half is the organized information, or data, that flows in and out of those actions. A :term[Call]{canonical="Call"} needs clean data to work with, and it needs to store its results in a clean way so the next :term[Call]{canonical="Call"} can use them.

The next document, :term[005: Agent/Data]{href="./005_agent_data.md"}, explains how we organize and manage all this information inside the agent.
