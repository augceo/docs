# 007: Agent/Variables

> [!DEFINITION] [Variable Reference](./000_glossary.md)
> A special piece of text that looks like `†<type>.<location>`. It's a shortcut that tells a :term[Tool Call]{canonical="Call"} where to find a piece of information the agent already knows, instead of making the agent write out the whole thing.

:term[Variables]{canonical="Variable"} are what make an agent’s information useful and interactive. Think of them as shortcuts that let an agent’s tools grab information from its memory (like a user's :term[Input]{canonical="Input"} or the agent's current :term[State]{canonical="State"}) without having to copy-paste it.

## Looking Up Information

When a tool needs information, instead of giving it the actual piece of data, we can give it a **Variable Reference**—a special shortcut that points to where the data is stored. This avoids making the AI do a lot of slow, clumsy, and error-prone copy-pasting of large amounts of information.

Using a shortcut is faster, cheaper, and safer. It's like giving someone a library card number instead of asking them to copy an entire book by hand. The library card is small, easy to share, and guarantees they get the exact right book without any typos.

The shortcut has a simple format that starts with a dagger symbol (`†`). It looks like this: `†<type>.<location>`, where `<type>` is the kind of memory to look in (like `state` or `input`) and `<location>` is the specific address to find the data you want.

> Sidenote:
> - Wikipedia: [Dagger (mark)](<https://en.wikipedia.org/wiki/Dagger_(mark)>)

::::columns
:::column{title="Tool instructions using variables"}

```json
{
  "_tool": "greetUser",
  "userName": "†input.userName"
}
```

:::
:::column{title="What this means in code"}

```typescript
greetUser({
  userName: input.userName,
});
```

:::
::::

## How Variables Connect Everything

- **:term[Data]{canonical="Data"}:** Variables are what bring the agent's data to life. **Variable References** read information from :term[Data]{canonical="Data"} messages (like :term[Input]{canonical="Input"} or :term[State]{canonical="State"}), and **Output Paths** write new information back. This creates a cycle where data is constantly being used and updated.

  > Sidenote:
  > - [005: Agent/Data](./005_agent_data.md)

- **:term[Input]{canonical="Input"}:** You can use shortcuts to point to information from the user's initial :term[Input]{canonical="Input"}. This lets you create reusable game plans, where the steps are always the same, but the specific details change depending on what the user provides.

  > Sidenote:
  > - [006: Agent/Input](./006_agent_input.md)

- **:term[State]{canonical="State"}:** The agent's :term[State]{canonical="State"} is like its short-term memory or a notepad. It's the main place where tools write their results, so that other tools later in the process can pick up where they left off and use that information.

  > Sidenote:
  > - [009: Agent/State](./009_agent_state.md)

- **:term[Plan]{canonical="Plan"}:** A :term[Plan]{canonical="Plan"} is basically a flowchart of tool actions connected by variables. Each step uses a shortcut to grab the result from a previous step. This is how an agent can create a complete, ready-to-run workflow all at once.

  > Sidenote:
  > - [012: Agent/Plan](./012_agent_plan.md)

## From Quick Connections to Lasting Memory

:term[Variables]{canonical="Variable"} are great for connecting tools together in a single step. But for longer tasks that take multiple steps, the agent needs a more lasting memory—a "notepad" where results can be saved and looked up again and again.

The next document, [008: Agent/Output](./008_agent_output.md), explains how tools write their results down in the agent's memory.
