# 013: Agent/Delegate

> [!DEFINITION] [Delegate](./000_glossary.md)
> A rule for giving a task its own private workspace. You trigger it using a special `_delegate` property in a :term[Call]{canonical="Call"}. It runs a job (:term[Activity]{canonical="Activity"}) or a new :term[Request]{canonical="Request"} in this separate space. The `_scopes` property acts like a keycard, giving it limited access to information from the main project.

> Sidenote:
> - You should read first:
>   - :term[004: Agent/Call]{href="./004_agent_call.md"}
> - Related topics:
>   - :term[015: Agent/Scopes]{href="./015_agent_scopes.md"}

The **Delegation** pattern solves a big problem: how do you get lots of smart assistants (agents) to work together on big projects without getting in each other's way? It’s a powerful way to run :term[Tools]{canonical="Tool"} in their own “sandboxes.” This stops information from one task from messing up another and makes :term[Tools]{canonical="Tool"} easy to reuse.

Think of it like a head chef delegating a task. Instead of doing everything themselves, they can give a job (a :term[Call]{canonical="Call"}) to a sous chef (the delegate). This lets the system build very complex behaviors out of small, independent, and focused helpers.

## How it Works: It's just an Activity

Delegation isn't a brand new invention. It’s a clever use of the system we already have for running jobs, called an :term[Activity]{canonical="Activity"}. When the system sees a :term[Call]{canonical="Call"} with the `_delegate` property, a special, built-in "Delegate Activity" automatically takes over to handle it.

This `Delegate Activity` gets the usual three pieces of information:

- **`call`**: The full job order, including all the details for the task that's being handed off.
- **`tool`**: The tool's instruction manual, where the activity reads the `_delegate` property to know *who* to give the job to.
- **`context`**: Any relevant notes from the main project that the new helper will need to see.

The activity's job is simple: it uses these three things to put together and start a new, separate :term[Request]{canonical="Request"}. This shows how powerful the basic rules are. Because the :term[Activity]{canonical="Activity"} system is so well-designed, even complicated patterns like this can be built on top of it without needing to invent a new set of rules.

## The Problem: Messy Kitchens and Tangled Tools

As AI assistants get more capable, trying to keep all their :term[Tools]{canonical="Tool"} in one big toolbox becomes a huge mess.

1.  **Too Many Gadgets**: An AI can get confused if you show it hundreds of tools at once. Just like a chef with a cluttered countertop, it can't think clearly about which one to use.
2.  **Flavor Bleeding**: When all :term[Tools]{canonical="Tool"} are in the same workspace, information from one task can “bleed” into another, causing mistakes. You don’t want the smell of fish from one dish getting into the dessert you're making.
3.  **Hard to Reuse**: A tool designed for one kitchen is hard to move to another if it's tangled up with everything else.

Delegation solves this by creating **Delegated Isolation**—giving each special task its own clean and separate workstation.

## How to Delegate a Job

You signal a delegation by adding the `_delegate` property to a :term[Tool]{canonical="Tool"}'s description. This tells the system not to do the job right here, but to hand it off to an outside helper.

The `_delegate` property is a string that can be used in two ways:

- **Point to a saved recipe**: The string can be a link to a file that describes a complete, ready-to-go :term[Request]{canonical="Request"}. This lets a :term[Tool]{canonical="Tool"} hand its job over to a completely separate set of instructions.

  > Sidenote:
  > A saved, reusable set of instructions (a :term[Request]{href="./001_agent_request.md"}) is the most common form of what we call an :term[Idea]{canonical="Idea"}. The Delegation pattern is the main way we combine these :term[Ideas]{canonical="Idea"} to build bigger and more complex systems. See :term[101: Concept/Idea]{href="./101_concept_idea.md"} for more details.

- **Create a workspace from scratch**: The special word `'anonymous'` tells the system to create a fresh, clean workspace on the spot. This is for running a single :term[Tool Call]{canonical="Tool Call"} in isolation without needing a pre-written recipe file.

## Working in a Clean Room

A delegate provides a "clean room" for a job. Instead of running in the middle of the parent assistant's busy workspace, the :term[Call]{canonical="Call"} gets its own quiet, focused session. The information in this new session is carefully chosen, not just copied over.

This is where the :term[Scoped context]{canonical="scope" href="./015_agent_scopes.md"} is essential. The `_scopes` property on the :term[Tool]{canonical="Tool"}'s description acts as a bridge. It clearly states which pieces of information from the parent's workspace should be passed into the delegate's clean room. This gives the parent total control over what the delegate sees, preventing information leaks and keeping everything tidy.

> Sidenote:
> - :term[015: Agent/Scopes]{href="./015_agent_scopes.md"}

## Handling Huge Blueprints

Delegation also helps when a :term[Tool]{canonical="Tool"} produces something very big or complicated. Instead of cluttering up the main workspace with a giant blueprint (an `_output` schema), a :term[Tool]{canonical="Tool"} can be defined with just its starting instructions and a pointer to a `_delegate`.

The main AI can plan to use the :term[Tool]{canonical="Tool"} with just that basic information. The complicated result will be created inside the delegate's private workspace. This lets an assistant plan out a series of complex steps without needing to see every single tiny detail of every step all at once. The AI trusts that the delegate will do its job correctly and return the finished part when it's done.

## When to Figure Things Out: Two Strategies

A :term[Tool]{canonical="Tool"} becomes a :term[Delegate]{canonical="Delegate"} just by having the `_delegate` property. The big question is *when* the system figures out the details of that delegated job. There are two options, letting you choose between being super flexible or super safe.

### 1. Figure it Out When it Runs (Default)

The default way is the most flexible: figure out the delegate's job at the moment it needs to be **executed**.

This allows for something amazing that normal computer code can't do: **the AI acts as a smart glue.** An assistant can make a :term[Call]{canonical="Call"} with details that don't perfectly match what the delegate expects. When the job runs, the system gives the delegate both the main assistant's request and its own instruction manual, and the delegate's AI is smart enough to bridge the gap.

This is a huge advantage. It means delegate tools can be updated and changed on their own. Even if a delegate's instructions change, other assistants using it won't instantly break. The AI will try to adapt the old request to the new instructions, making the whole system more resilient and loosely connected.

The process works like this:

1.  An assistant decides to use the delegate :term[Tool]{canonical="Tool"}.
2.  The system sees the `_delegate` property and starts the hand-off process.
3.  **Prepare the Workspace**: The system gets the delegate's recipe (if it has one) and uses `_scopes` to add the necessary notes from the parent.
4.  **Map the Request**: The details from the :term[Call]{canonical="Call"} are packaged up as an input message. This is where the AI's "glue" magic happens—it will use this input to do the job, even if the names and formats don't perfectly match.
5.  **Run the Job**: A new, isolated :term[Request]{canonical="Request"} is started with the prepared workspace. The result is sent back as the output of the original :term[Call]{canonical="Call"}.

### 2. Figure it Out Upfront (Optional)

For situations where you need to be very strict and safe, you can have the system figure everything out **upfront**, before the first :term[Request]{canonical="Request"} is even sent to the assistant.

In this mode, the system grabs the delegate's recipe and merges its instructions with the :term[Tool]{canonical="Tool"}'s description. This lets the main AI see the delegate's exact requirements from the very beginning, making sure the :term[Call]{canonical="Call"} it creates is perfectly correct. This can also include the delegate's expected output, creating a strict contract for what will happen.

This method gives you the safety of a traditional contract, where you know exactly what goes in and what comes out. You lose the flexibility of the "smart glue" approach, so it's best for important jobs where you don't want any surprises.

:::::details{title="Example: Flexible Input Mapping at Execution Time"}

This example shows the "AI as glue" idea. A delegate can work correctly even if the :term[Call]{canonical="Call"} from the main assistant doesn't perfectly match its own instructions. This is the default, flexible way it works.

::::columns
:::column{title="What the Main Assistant Does"}

A main assistant needs to send a message. It knows about a `sendMessage` :term[Tool]{canonical="Tool"} that hands the job off to a specialist. It creates a :term[Call]{canonical="Call"} with `userId` and `text`, based on the information it has.

```json
// THE CALL FROM THE MAIN ASSISTANT
{
  "_tool": "sendMessage",
  "_delegate": "http://example.com/agents/speaker_EN",
  "userId": "u_123",
  "text": "Hello, world!"
}
```

:::
:::column{title="What the Delegate Sees & Does"}

The `speaker_EN` delegate is a separate specialist with its own instruction manual. When it gets the job, the system packages the main assistant's request into an `input` message. The delegate's own instructions (`schema`) expect different names: `recipientId` and `messageBody`. But because the delegate is also an AI, it's smart enough to understand that `userId` means `recipientId` and `text` means `messageBody`. It translates between them automatically.

```json
// THE FINAL WORKSPACE FOR THE DELEGATE
[
  {
    "type": "system",
    "message": "You are an expert in messaging in English."
  },
  {
    "type": 'input',
    // This is what the main assistant sent.
    "input": {
      "userId": "u_123",
      "text": "Hello, world!"
    },
    // This is what the delegate usually expects.
    "schema": {
      "type": "object",
      "properties": {
        "recipientId": { "type": "string" },
        "messageBody": { "type": "string" }
      }
    }
  }
]
```

:::
::::
:::::

:::details{title="Example: Music Makers"}

Delegates let you build amazing things by letting different specialists work together. This creates a clear way to organize work: high-level assistants can focus on the big picture while giving specific tasks to low-level, reusable experts.

Imagine you have two specialists: a **`Composer`** and a **`Sound-Designer`**.

- The **`Sound-Designer`** is the expert on the nitty-gritty details. It's a focused helper that knows how to use synthesizers to create specific sounds.

- The **`Composer`** is a mid-level specialist. Its job is to create a song. It figures out the melody and structure. To turn its ideas into actual music, it makes :term[Calls]{canonical="Call"} to the `Sound-Designer` delegate.

This two-layer setup is very common. But the real magic of delegates is how they can be combined in different ways depending on the job.

Now, let's add a high-level **`Producer`** assistant. The `Producer`'s goal is to create a finished album. The `Producer` can use its specialists in different ways:

> Sidenote:
> This setup is very flexible. A high-level `Producer` can give a job to a `Composer`, who then uses a `Sound-Designer`. But the `Producer` can also skip the `Composer` and talk to the `Sound-Designer` directly for specific tasks.
>
> ```mermaid
> graph TD
>     Producer --> Composer
>     Producer --> SoundDesigner(Sound-Designer)
>     Composer --> SoundDesigner(Sound-Designer)
> ```

- **Giving one big job**: To create a song, the `Producer` might make a single :term[Call]{canonical="Call"} to the `Composer` delegate. The `Producer` gives a high-level goal ("I need a sad ballad"), and the `Composer` handles the entire process, including making its own calls to the `Sound-Designer`. The `Producer` doesn't even need to know the `Sound-Designer` exists.

- **Giving several small jobs**: If the `Producer` also needs a specific sound effect for the album (like a door slamming), it can make a :term[Call]{canonical="Call"} directly to the `Sound-Designer` for that one task, while the `Composer` is working on the song separately.

This shows the main idea: the way the specialists work together isn't fixed. The `Producer` can treat the `Composer` as a single tool that does everything, or it can interact with the individual specialists directly, depending on what it needs at that moment. This flexibility allows the same team of experts to be combined in endless ways to create new things.

:::

## From Delegating to Scoping

Delegation gives a helper a private room to work in, but for that to be useful, you need a way to pass information into that room. The `_scopes` property is the tool for that job, acting as a controlled bridge between workspaces. How that bridge works is explained in the :term[Scoped context]{canonical="scope" href="./015_agent_scopes.md"} pattern.
