# 103: Concept/Ideator

> [!DEFINITION] [Ideator](./000_glossary.md)
> An :term[Idea]{canonical="Idea"} that's designed to accept information (input). Think of it like a function: it takes something in, processes it, and gives something new back (output).

> Sidenote:
> - Builds On:
>   - [101: Concept/Idea](./101_concept_idea.md)
>   - [006: Agent/Input](./006_agent_input.md)
> - Makes Possible:
>   - [004: Agent/Call](./004_agent_call.md)
>   - [102: Concept/Sovereignty](./102_concept_sovereignty.md)

This document explains how we make :term[Ideators]{canonical="Ideator"} and :term[Idea Transformers]{canonical="Idea Transformer"} into active, usable services that can perform tasks. It builds on the basic structure of an :term[Idea]{canonical="Idea"} (from [101: Concept/Idea](./101_concept_idea.md)) and shows how we turn that data into something that can be run like a program.

To learn about the different ways these can be hosted and run, see [102: Concept/Sovereignty](./102_concept_sovereignty.md).

## From Idea to Ideator

An **:term[Ideator]{canonical="Ideator"}** isn't a totally new thing, but rather a job that any :term[Idea]{canonical="Idea"} can perform. You can think of it as a tool that does work by turning an input into an output. This work happens in a :term[latent space]{canonical="Latent Execution"}, which is a fancy way of saying its logic doesn't come from traditional code. Instead, an AI (like a large language model) reads the :term[Idea]{canonical="Idea"}'s `context`—its rules, examples, and plain English instructions—to figure out what to do.

The single clue that tells you an :term[Idea]{canonical="Idea"} is an :term[Ideator]{canonical="Ideator"} is a message in its `context` that says `type: "input"`. This message describes the kind of data the :term[Ideator]{canonical="Ideator"} is expecting. An :term[Ideator]{canonical="Ideator"} that can be run might also include a `context` message with `type: "code"`, which points to a specific piece of code to run.

### The Idea Transformer: A Special Case

A very useful and common pattern is an :term[Ideator]{canonical="Ideator"} that takes another :term[Idea]{canonical="Idea"} as its input. We call this special kind of :term[Ideator]{canonical="Ideator"} an **:term[Idea Transformer]{canonical="Idea Transformer"}**. This is the key to creating assembly lines where :term[Ideas]{canonical="Idea"} are passed from one tool to the next to be changed and improved.

## Implementations and Composition

The rules described here create a **behavioral contract**, which is like a promise for how any :term[Ideator]{canonical="Ideator"} service should behave. This isn't for just one program, but a common standard that allows many different versions to exist and work together smoothly.

### A Plurality of Implementations

An :term[Ideator]{canonical="Ideator"} service's promise is kept as long as it follows its public rules (it accepts an :term[Idea]{canonical="Idea"} and returns another one). This allows for different ways to build them, each good for different situations:

- **Managed Services**: A company can offer to run your :term[Ideators]{canonical="Ideator"} for you on their powerful computers in the cloud. This way, you don't have to worry about the technical setup, as explained in the [:term[Sovereignty]{href="./102_concept_sovereignty.md"} Protocol](./102_concept_sovereignty.md).
- **Self-Hosted Instances**: A developer can run the service on their own computer or server, giving them complete control.
- **In-Memory Implementations**: For practice and testing on a local machine, an :term[Ideator]{canonical="Ideator"}'s logic can be run directly inside a program without needing a network connection, while still following the same basic rules.

### Composition and Higher-Order Systems

In this system, there are no secret or "private" connections. All services are designed to talk to each other through their public, agreed-upon rules.

More advanced services, which you can think of as **Higher-Order Systems**, are built by combining several simpler :term[Ideators]{canonical="Ideator"} together. The inner workings of a higher-order service involve it calling out to other public :term[Ideators]{canonical="Ideator"} to get parts of its job done.

For example, the **:term[Reactor]{canonical="Reactor"}** system is a higher-order :term[Ideator]{canonical="Ideator"}. To run a game, it might:

1.  Receive the current game's status as an :term[Idea]{canonical="Idea"} from the outside world.
2.  Internally, it calls a public `Player` service to create and keep track of players.
3.  It then calls a public `Storage` service to save a record of what happened in the game.
4.  Finally, it returns the newly updated game status :term[Idea]{canonical="Idea"} back to the outside world.

From an outsider's perspective, the :term[Reactor]{canonical="Reactor"} just looks like any other :term[Ideator]{canonical="Ideator"}. Its complexity is hidden inside, managed by using other separate, public services.

## The Refiner: An Ideator for Evolution

While most :term[Ideators]{canonical="Ideator"} work with the rules (`schema`) they're given to create a :term[Solution]{canonical="Solution"}, a special type of :term[Idea Transformer]{canonical="Idea Transformer"} exists to change the rules themselves. This tool is called the **:term[Refiner]{canonical="Refiner"}**.

A :term[Refiner]{canonical="Refiner"} is a tool for evolving the core structure of an :term[Idea]{canonical="Idea"}. It takes an existing :term[Idea]{canonical="Idea"} and a command (like, "Add a spot for the author's name to this article") as its input, and produces a brand *new* :term[Idea]{canonical="Idea"} as its output.

This new :term[Idea]{canonical="Idea"} has:

- An **updated `schema`** (the new set of rules).
- An **updated :term[Solution]{canonical="Solution"}** that follows the new rules.
- **Migrated data** from the old :term[Solution]{canonical="Solution"}. The AI, because it understands both the old and new rules, intelligently tries to move the information over to the new structure.

The :term[Refiner]{canonical="Refiner"} is the main way an :term[Idea]{canonical="Idea"}'s family tree, or **:term[lineage]{canonical="Lineage"}**, grows. If the rule change breaks compatibility with the old version, the new :term[Idea]{canonical="Idea"} gets a new major version number. This allows the system's fundamental building blocks to evolve safely and clearly, all handled by a dedicated, reusable :term[Ideator]{canonical="Ideator"}.

Now that we have the :term[Ideator]{canonical="Ideator"} as a way to get things done, the next question is how that work actually happens. This leads us to the concept of :term[Latent Execution]{canonical="Latent Execution"}, where an AI uses an :term[Idea]{canonical="Idea"}'s context to produce a result without needing to be given step-by-step code.
