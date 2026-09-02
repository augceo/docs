# 014: Agent/Scopes

> [!DEFINITION] [Scope](./000_glossary.md)
> A mechanism for making a controlled subset of context from a parent environment available to an execution. The `_scopes` property acts as an allow-list, defining a focused and secure view of the data a :term[Call]{canonical="Call"} can access.

> Sidenote:
>
> - Requires:
>   - :term[002: Agent/Tool]{href="./002_agent_tool.md"}
>   - :term[004: Agent/Call]{href="./004_agent_call.md"}

The **:term[Scoped context]{canonical="scope"}** pattern is a mechanism for managing the context available to a :term[Call]{canonical="Call"}. In a complex agentic system, a :term[Call]{canonical="Call"} rarely executes in a vacuum; it often needs access to information from its parent environment, such as user input, current state, or the results of previous steps. Scoped context provides a secure and explicit way to control this flow of information.

By restricting the context, scopes enhance security, prevent accidental data leakage, and focus the LLM, leading to more predictable and cost-effective executions. This controlled context is also the key to modularity, allowing components like :term[Ideas]{canonical="Idea"} and :term[Activities]{canonical="Activity"} to be truly self-contained and reusable. This document explains how this pattern works and how it composes with other agent capabilities.

## Provisioning vs. Requesting Context

The schema for the `_scopes` property determines whether context is statically **provisioned** or dynamically **requested** at runtime.

> Sidenote:
>
> ```mermaid
> graph TD
>     subgraph Parent Context
>         direction LR
>         input("input")
>         state("state")
>     end
>
>     subgraph Tool Call
>         direction LR
>         filter{{"_scopes: ['input']"}}
>     end
>
>     input --> filter
>     state -.-> filter
>
>     subgraph Provisioned Context
>         Execute(Execute Tool)
>     end
>
>     filter --> HITL{{Human approval}}
>     HITL --> Execute
>
>     classDef unused stroke-dasharray: 5, 5, stroke:#aaa, color:#aaa
>     class state unused
>     classDef optional stroke-dasharray: 5, 5
>     class HITL optional
> ```

- **Static Scopes (Context Provisioning)**: The `_scopes` schema can be a `const` value, which means the context is **provisioned**. The designer has hard-coded the exact context the tool is allowed to see.

  ```json
  {
    "_scopes": {
      "const": ["input"]
    }
  }
  ```

- **Dynamic Scopes (Context Requesting)**: The `_scopes` schema can be more flexible, allowing the context to be **requested**. The LLM decides which of the available scopes it needs to generate the :term[Call]{canonical="Call"}.

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

  This dynamic pattern is effective when combined with a human-in-the-loop approval system, providing a critical layer of transparency and control.

## The Role of Scopes in Call Composition

The `_scopes` property is the mechanism for controlling the context available to a :term[Call]{canonical="Call"}. It acts as an allow-list, filtering the parent environment to provide a focused, limited field of view for the execution. This controlled context is fundamental to how a :term[Call]{canonical="Call"} is processed, and its role adapts to support a compositional model of execution where different capabilities like explicit logic, instancing, and modularity can be combined.

- **Latent Execution**: In the default latent execution, `_scopes` serve as a "nudge" to focus the LLM's attention on relevant parts of the parent context. This is a best-effort guide, not a strict filter, but it is crucial for improving the reliability and cost-effectiveness of LLM-driven reasoning by reducing noise from irrelevant data. See the [Disambiguation with Scopes](#example-disambiguation-with-scopes) example.

  > Sidenote:
  >
  > - :term[002: Agent/Tool]{href="./002_agent_tool.md"}.

- **Explicit Execution (`_activity`)**: When a :term[Call]{canonical="Call"} is backed by a deterministic :term[Activity]{canonical="Activity"}, the role of scopes becomes more direct. The scoped context is passed wholesale to the :term[Activity]{canonical="Activity"} function as an additional parameter. This gives the :term[Activity]{canonical="Activity"} full access to the necessary contextual data, even if that data wasn't directly used by the LLM to generate the :term[Call]{canonical="Call"}'s primary parameters. See the [Providing Context to an Activity](#example-providing-context-to-an-activity) example.

  > Sidenote:
  >
  > - :term[003: Agent/Activity]{href="./003_agent_activity.md"}.

- **Instancing (`_instance`)**: In a multi-instance request where the agent processes a batch of similar data objects, scopes become instance-aware. The protocol ensures that a :term[Call]{canonical="Call"} targeting a specific instance receives only the context for _that_ instance. This is critical for maintaining data integrity and preventing context from "leaking" between parallel executions.

  > Sidenote:
  >
  > - :term[013: Agent/Instancing]{href="./013_agent_instancing.md"}

- **Delegated Isolation (`_delegate`)**: When a :term[Call]{canonical="Call"} is delegated to an external :term[Delegate]{canonical="Delegate"}, scopes act as the gatekeepers of context. They define what from the parent's environment gets _appended_ to the delegate's own internal context, creating the final context for the isolated sub-request. Nothing from the parent is available unless explicitly scoped, ensuring true encapsulation. See the [Scoping a Delegate's Context](#example-scoping-a-delegates-context) and [Instancing with Scoped Delegates](#example-instancing-with-scoped-delegates) examples.

  > Sidenote:
  >
  > - :term[014: Agent/Delegate]{href="./014_agent_delegate.md"}

The `_scopes` property is the bridge that allows a :term[Call]{canonical="Call"} to receive context. This chapter has shown how this single mechanism adapts to different execution modes—from gently guiding a latent call to strictly defining the entire context for a delegated one. This flexibility supports building complex, secure, and modular agentic systems.

## Examples

:::::details{title="Example: Disambiguation with Scopes"}

::::columns
:::column{title="Ambiguous Context"}

In this example, the context is ambiguous. It contains two different users: the `currentUser` in the `state` and a `mentionedUser` in the `input`. The agent needs to send a message, but it's unclear to whom.

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
:::column{title="`Call` with Scope"}

By adding `_scopes: ["input"]`, the caller provides a crucial hint. It tells the LLM to focus on the `input` message, effectively resolving the ambiguity and ensuring the message is sent to the correct recipient, Bob.

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

:::::details{title="Example: Providing Context to an Activity"}

Here, an :term[Activity]{canonical="Activity"} needs access to contextual information that isn't a direct parameter of the tool. The `logEvent` tool only takes an `eventName`, but the underlying activity also needs to know the `userId` to function correctly.

::::columns
:::column{title="`Call` with Scope"}

The :term[Call]{canonical="Call"} is simple, only providing the `eventName`. However, the `_scopes: ["state"]` property tells the execution engine to pass the `state` object to the activity.

```json
{
  "_tool": "logEvent",
  "_scopes": ["state"],
  "eventName": "user_login"
}
```

:::
:::column{title="Activity Implementation (TypeScript)"}

The :term[Activity]{canonical="Activity"} is registered with a function that destructures its parameters directly from the call and the scoped context. This provides clean, direct access to both the `eventName` and the `userId`.

```typescript
// The parameters are destructured to directly access 'eventName'
// from the call and 'state' from the scoped context.
Activity.register('logEvent', async ({ eventName }, { state }) => {
  const userId = state.userId;
  await analytics.track(eventName, { userId });
});
```

:::
::::
:::::

:::::details{title="Example: Scoping a Delegate's Context"}

When delegating, `_scopes` define what from the parent context gets _appended_ to the delegate's own internal context. Here, a high-level `Orchestrator` delegates a task, passing the entire `state` object to a specialized `Summarizer` delegate.

::::columns
:::column{title="Parent `Call` with Scope"}

The `Orchestrator` agent has its own context. It makes a `Call` that scopes the `state` to be included with the delegate's context.

```json
// Orchestrator's Context and Call
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
:::column{title="Delegate's Sub-Request Context"}

The `Summarizer`'s execution environment is composed of its own internal context plus the data scoped from the parent. This allows for modularity while still providing necessary information.

```json
// Context constructed for the Summarizer's sub-request
[
  // The Summarizer's internal context
  {
    "type": "system",
    "message": "You are an expert summarizer."
  },
  // The scoped context from the parent is appended
  {
    "type": "state",
    "articleText": "A long and complex article..."
  }
]
```

:::
::::
:::::

:::::details{title="Example: Instancing with Scoped Delegates"}

This example shows how `_scopes` and `_instance` compose to allow a single agent to orchestrate multiple, isolated delegate calls in parallel.

::::columns
:::column{title="Parent Context & Solution"}

The parent agent has two distinct `State` instances. It generates a `solution` with two `Calls` to a `translatorDelegate`. Each call targets a different instance and correctly scopes only the `state` for that instance.

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
:::column{title="Delegate Sub-Request Contexts"}

The execution engine creates two independent sub-requests. The `_instance` and `_scopes` properties work together to ensure that each delegate receives only its own, correctly-scoped `state`.

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
