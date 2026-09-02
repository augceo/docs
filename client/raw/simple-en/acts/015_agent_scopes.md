# 014: Agent/Scopes

> [!DEFINITION] [Scope](./000_glossary.md)
> Think of a scope as putting blinders on an AI. It's a way to let a specific task see only a small, controlled part of all the available information. This acts like an "allow list," giving the task a focused and secure peek at the data it needs to do its job, and nothing more.

> Sidenote:
> - You should know about:
>   - :term[002: Agent/Tool]{href="./002_agent_tool.md"}
>   - :term[004: Agent/Call]{href="./004_agent_call.md"}

The **:term[Scoped context]{canonical="scope"}** pattern is a very important tool for managing what information a task, or :term[Call]{canonical="Call"}, can access. In a big AI system, a task doesn't just happen on its own; it often needs information from its surroundings, like what a user just typed, what happened in previous steps, or the current situation.

Scopes provide a safe and clear way to control this flow of information. By limiting what the AI sees, we make the system more secure, stop it from accidentally seeing private data, and help the AI focus. This makes its actions more predictable and saves money. This controlled view is also the secret to making different AI parts, like :term[Ideas]{canonical="Idea"} and :term[Activities]{canonical="Activity"}, like building blocks that can be reused anywhere without causing problems.

## Giving vs. Asking for Information

How we set up the `_scopes` property decides if information is automatically **given** to a task or if the task has to **ask** for it when it runs.

> Sidenote:
> ```mermaid
> graph TD
>     subgraph All Available Info
>         direction LR
>         input("input")
>         state("state")
>     end
>
>     subgraph The Task Being Called
>         direction LR
>         filter{{"Scope: Only allow 'input'"}}
>     end
>
>     input --> filter
>     state -.-> filter
>
>     subgraph Information Passed to Tool
>         Execute(Run the Tool)
>     end
>
>     filter --> HITL{{A human can approve}}
>     HITL --> Execute
>
>     classDef unused stroke-dasharray: 5, 5, stroke:#aaa, color:#aaa
>     class state unused
>     classDef optional stroke-dasharray: 5, 5
>     class HITL optional
> ```

- **Static Scopes (Giving Information)**: If the `_scopes` rule is set to a fixed value, it means the information is **given** automatically. The person who designed the tool has already decided exactly what information it's allowed to see, and it can't be changed.

  ```json
  {
    "_scopes": {
      "const": ["input"]
    }
  }
  ```

- **Dynamic Scopes (Asking for Information)**: The `_scopes` rule can also be flexible, allowing the information to be **asked for**. The AI looks at the situation and decides which pieces of available information it needs to complete the task.

  ```json
  {
    "_scopes": {
      "type": "array",
      "items": {
        "enum": ["state", "input"]
      }
    }
  }
  ```

  This is especially useful when a person is in the loop to approve the AI's request, adding an important layer of safety and control.

## How Scopes Work with Different Tasks

The `_scopes` property is the main way to control the information a :term[Call]{canonical="Call"} gets to see. It acts like a bouncer with a VIP list, filtering the environment to give the task a limited and focused view. This is key to how tasks are handled, and it works in different ways depending on what the task is trying to do.

- **Normal Tasks (Latent Execution)**: In a standard task, `_scopes` act like a gentle hint to the AI. It tells the AI, "Hey, you should probably pay attention to this specific information." It’s not a strict rule, but it helps the AI avoid getting distracted by useless data, making it more reliable and cheaper to run. See the [Helping the AI Choose with Scopes](#example-disambiguation-with-scopes) example.

  > Sidenote:
  > - See :term[002: Agent/Tool]{href="./002_agent_tool.md"}.

- **Code-Based Tasks (`_activity`)**: When a task is powered by a piece of code (an :term[Activity]{canonical="Activity"}), scopes play a more direct role. All the information allowed by the scope is handed directly to the code. This gives the :term[Activity]{canonical="Activity"} all the details it needs to work, even if the AI didn't use those details to come up with the main instructions for the task. See the [Giving Information to an Activity](#example-providing-context-to-an-activity) example.

  > Sidenote:
  > - See :term[003: Agent/Activity]{href="./003_agent_activity.md"}.

- **Batch Tasks (`_instance`)**: Imagine you need to do the same task on a big batch of items, like translating 100 different sentences at once. Scopes make sure that when the AI works on one item, it only sees the information for *that specific item*. This is crucial to keep data separate and prevent information from one task from spilling over into another.

  > Sidenote:
  > - See :term[013: Agent/Instancing]{href="./013_agent_instancing.md"}

- **Delegated Tasks (`_delegate`)**: When a task is handed off to another specialized AI agent (a :term[Delegate]{canonical="Delegate"}), scopes act as gatekeepers. They decide what information from the main agent's environment gets added to the specialized agent's world. The specialized agent can't see anything from the main agent unless the scope explicitly allows it, which keeps its work completely separate. See the [Scoping a Delegate's Information](#example-scoping-a-delegates-context) and [Working on Batches with Scoped Delegates](#example-instancing-with-scoped-delegates) examples.

  > Sidenote:
  > - See :term[014: Agent/Delegate]{href="./014_agent_delegate.md"}

The `_scopes` property is the bridge that connects a task to the information it needs. We've seen how this one simple idea can adapt to different situations—from gently guiding an AI to strictly defining everything it can see for a delegated task. This flexibility makes it a foundational part of building powerful, secure, and well-organized AI systems.

## Examples

:::::details{title="Example: Helping the AI Choose with Scopes"}

::::columns
:::column{title="Confusing Situation"}

In this example, the AI's environment is confusing. There are two different people: `currentUser` (Alice) and `mentionedUser` (Bob). The AI is told to send a message, but it's not clear who should get it.

```json
[
  {
    "type": "state",
    "currentUser": { "id": "user_A", "name": "Alice" }
  },
  {
    "type": "input",
    "mentionedUser": { "id": "user_B", "name": "Bob" },
    "instruction": "Send a welcome message to the user mentioned above."
  }
]
```

:::
:::column{title="Task with a Scope"}

By adding `_scopes: ["input"]`, we give the AI a big hint. We're telling it to focus only on the `input` information. This clears up the confusion and makes sure the message is sent to the right person, Bob.

```json
{
  "_tool": "sendMessage",
  "_scopes": ["input"],
  "recipientId": "user_B",
  "message": "Welcome, Bob!"
}
```

:::
::::
:::::

:::::details{title="Example: Giving Information to an Activity"}

Here, a coded task (an :term[Activity]{canonical="Activity"}) needs to know something that isn't part of its main instructions. The `logEvent` tool only asks for an `eventName`, but the code behind it also needs the `userId` to work properly.

::::columns
:::column{title="Task with a Scope"}

The task instruction is simple, just giving the `eventName`. But the `_scopes: ["state"]` part tells the system to also hand the entire `state` object (which contains the `userId`) over to the code.

```json
{
  "_tool": "logEvent",
  "_scopes": ["state"],
  "eventName": "user_login"
}
```

:::
:::column{title="How the Activity's Code Works (TypeScript)"}

The code for the :term[Activity]{canonical="Activity"} is set up to receive both the main instruction (`eventName`) and the extra scoped information (`state`). This gives it clean and direct access to both the event name and the user's ID.

```typescript
// The code is set up to grab 'eventName' from the main instructions
// and 'state' from the extra scoped information.
Activity.register('logEvent', async ({ eventName }, { state }) => {
  const userId = state.userId;
  await analytics.track(eventName, { userId });
});
```

:::
::::
:::::

:::::details{title="Example: Scoping a Delegate's Information"}

When you delegate a job, `_scopes` decide what information from the main agent gets *added* to the specialized agent's world. Here, a main `Orchestrator` agent passes the `state` object to a specialized `Summarizer` agent so it can do its job.

::::columns
:::column{title="Main Agent's Task with Scope"}

The main `Orchestrator` agent has its own information. It creates a task that uses a scope to include the `state` object when it delegates the work.

```json
// The Orchestrator's Information and Task
[
  {
    "type": "state",
    "articleText": "A long and complex article..."
  },
  {
    "_tool": "summarizeArticle",
    "_delegate": "SummarizerAgent",
    "_scopes": ["state"]
  }
]
```

:::
:::column{title="Delegate's Information for the Job"}

The `Summarizer`'s working environment is made up of its own built-in knowledge plus the extra information passed to it from the main agent. This lets it be a separate, reusable tool while still getting the info it needs.

```json
// The information given to the Summarizer for this specific job
[
  // The Summarizer's own built-in instructions
  {
    "type": "system",
    "message": "You are an expert summarizer."
  },
  // The scoped information from the main agent is added
  {
    "type": "state",
    "articleText": "A long and complex article..."
  }
]
```

:::
::::
:::::

:::::details{title="Example: Working on Batches with Scoped Delegates"}

This example shows how `_scopes` and `_instance` work together. A single main agent can manage multiple, separate delegated jobs at the same time.

::::columns
:::column{title="Main Agent's Info & Solution"}

The main agent has two different pieces of text to translate, marked as instance `①` and `②`. It creates a solution with two tasks, one for each instance. Each task correctly scopes only the `state` for its own instance.

```json
// PARENT CONTEXT
[
  { "type": "state", "_instance": "①", "text": "Hello" },
  { "type": "state", "_instance": "②", "text": "Bonjour" }
]

// LLM'S SOLUTION
{
  "calls": [
    {
      "_tool": "translate",
      "_delegate": "translatorDelegate",
      "_instance": "①",
      "_scopes": ["state"]
    },
    {
      "_tool": "translate",
      "_delegate": "translatorDelegate",
      "_instance": "②",
      "_scopes": ["state"]
    }
  ]
}
```

:::
:::column{title="Information for Each Delegate's Job"}

The system creates two separate jobs. The `_instance` and `_scopes` properties work together to make sure that each delegate gets only its own piece of text to work on, with no mix-ups.

```json
// CONTEXT FOR DELEGATE ①
[
  { "type": "system", "message": "You are a translator." },
  { "type": "state", "text": "Hello" }
]

// CONTEXT FOR DELEGATE ②
[
  { "type": "system", "message": "You are a translator." },
  { "type": "state", "text": "Bonjour" }
]
```

:::
::::
:::::
