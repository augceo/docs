# 104: Idea/Latent

> Sidenote:
> - Needs to know:
>   - [101: Concept/Idea](/101_concept_idea.md)
> - Helps with:
>   - [001: Agent/Request](/001_agent_request.md)

> [!DEFINITION] [Latent](/000_glossary.md)
> Using a big AI brain's (like a Large Language Model) own knowledge and thinking power to figure things out, instead of giving it exact, step-by-step code.

**Latent** is a big idea in how this system works. Think of it like a person's imagination or intuition. It lets the system use an AI's giant 'brain' to connect the dots between a problem (the `input`) and an answer (the `output`), without needing to be told exactly how to do it with code.

This is the system's go-to way of doing things. It's what makes the system so flexible, allowing it to connect different ideas and tools together to create new things, even if no one has written specific code for those steps yet.

## The Latent Space: An Ocean of Knowledge

Imagine the AI's brain is like a giant, magical ocean. This is the **Latent Space**. It’s not a list of facts like in an encyclopedia. Instead, everything the AI learned from reading the internet is connected in a huge web of ideas. Things that are similar, like "king" and "queen," are close together in this ocean. Things that are different are far apart.

The trick isn't just knowing the ocean exists, but knowing how to navigate it. The system is designed to be a good captain, asking the right questions and giving the AI the right maps (the 'context') to find the treasures of knowledge hidden inside.

## Latent Execution: Sailing the Ocean

**Latent Execution** is the act of actually using that ocean of knowledge to get something done. This happens automatically whenever the system gets asked to do something, but there isn't a specific coded instruction (an `Activity`) for it.

Here’s how it works:

1.  A request, or a `Call`, is made for a `Tool` or `Idea` to do something.
2.  The system looks for a set of pre-written instructions, but finds none.
3.  So, the system gives the AI all the information it has about the problem (the `context`) and a blueprint for what the final answer (`_output`) should look like (the `schema`).
4.  The AI's job is to dive into its ocean of knowledge, use the information as a compass, and come back with an answer that fits the blueprint perfectly.

This turns the AI from something that just writes text into a smart engine that can figure out how to solve new problems on the spot.

## Context Management: Giving the AI a Map and Compass

The most important job the system has is to prepare the AI for its journey. It needs to arrange all the information perfectly so the AI doesn't get lost in its own knowledge. It uses a few tricks to help the AI focus:

- **Rich Information**: It gives the AI a clear story of what's happened so far—past messages, important facts, and what the user wants. This is like giving a detective all the clues before they start investigating.
- **A Blueprint for the Answer**: By giving the AI a `JSON Schema`, the system isn't just checking the final answer. It's giving the AI a template to fill out. This forces the AI to think in a structured way, like building with a guide instead of just piling bricks randomly.
- **Instructions and Tools**: These act like special lenses, telling the AI to focus on one specific part of its knowledge. If the task is about math, it tells the AI to use its 'math brain'.

Giving the AI a good map and compass is what makes this whole process work. It’s how the system turns a huge, messy ocean of ideas into a powerful and precise tool.

## The Default Way of Doing Things

Using the AI's brain this way isn't a backup plan; it's the main plan. The system is 'optimistic'—it believes that any task can be done just by asking the AI in the right way. Writing specific, step-by-step code is seen as an upgrade you add later, maybe for tasks that need to be super fast or reliable.

This way of thinking changes everything for people building things:

- **Build Things Super Fast**: You can design a whole plan with many steps, connecting different `Tools` and `Ideas`, without writing any real code. The AI just figures out how to do it all.
- **Mix and Match Easily**: You can combine tools made by different people, and the system can make them work together because the AI acts as the universal translator.
- > Sidenote: We talk more about this in [Act 303: Ideator/Reactor](/acts/303_ideator_reactor.md).

  **Improve Over Time**: You can start with a project that runs entirely on the AI's 'imagination'. Later, if you find a step that's slow or important, you can replace just that part with fast, specific code, without changing anything else.

By making this the default, the system lets people create and test complex ideas incredibly quickly.

## The Engine of AI-Powered Systems

This whole idea is what powers a new kind of 'AI-Native' system. The ability to dream up, connect, and run complex tasks without writing code first is what makes these systems so smart and flexible. It allows the AI to be its own planner, designer, and worker.

This is how we move from systems that just *use* AI as a tool to systems that are actually *built out of* AI.

> Sidenote:
> - [105: Concept/AI-Native](/105_concept_ai_native.md)

The next document, [105: Concept/AI-Native](/105_concept_ai_native.md), dives deeper into what it means to build things this way.
