# 007: Agent/Variables

> [!DEFINITION] [Variable Reference](./000_glossary.md)
> A string with a special syntax (`†<kind>.<path>`) used in a :term[Tool Call]{canonical="Call"}'s parameters to dynamically reference a value from the agent's context.

:term[Variables]{canonical="Variable"} build upon the [Data](./005_agent_data.md) system to enable dynamic data flows. They make structured information interactive by allowing :term[Tool Calls]{canonical="Tool Call"} to reference values that exist within the agent's context (such as an :term[Input]{canonical="Input"} or :term[State]{canonical="State"} message) without needing to copy the data directly into their parameters.

## Reading from Context

A parameter in a :term[Tool Call]{canonical="Call"} can hold a **Variable Reference**—a special string that points to a value elsewhere in the context—instead of the value itself. This prevents the inefficient and error-prone process of having an LLM copy large data objects from its context into the parameters of a :term[Tool Call]{canonical="Call"}. Using a reference is faster, cheaper, and more reliable, as it eliminates the risk of the LLM altering the data during reproduction.

The reference is a simple string syntax prefixed with a dagger (`†`). The syntax is `†<kind>.<path>`, where `<kind>` is the type of :term[Data]{canonical="Data"} message (e.g., `state`, `input`) and `<path>` is the dot-notation path to the desired value

> Sidenote:
>
> - Wikipedia: [Dagger (mark)](<https://en.wikipedia.org/wiki/Dagger_(mark)>)

::::columns
:::column{title="Tool parameters using varaibles"}

```json
{
  "_tool": "greetUser",
  "userName": "†input.userName"
}
```

:::
:::column{title="Equivalent Typescript code"}

```typescript
greetUser({
  userName: input.userName,
});
```

:::
::::

## Interactions with other systems

- **:term[Data]{canonical="Data"}:** Variables are the mechanism that makes :term[Data]{canonical="Data"} messages interactive. :term[Variable References]{canonical="Variable Reference"} read from :term[Data]{canonical="Data"} messages (like :term[Input]{canonical="Input"} or :term[State]{canonical="State"}), and :term[Output Paths]{canonical="Output Path"} write back to them, creating a dynamic loop where data can be accessed and modified.

  > Sidenote:
  >
  > - [005: Agent/Data](./005_agent_data.md)

- **:term[Input]{canonical="Input"}:** :term[Variable References]{canonical="Variable Reference"} can target :term[Input]{canonical="Input"} messages, allowing a workflow to be parameterized. This is essential for creating reproducible plans where the overall structure is defined, but the specific data it operates on is provided at runtime.

  > Sidenote:
  >
  > - [006: Agent/Input](./006_agent_input.md)

- **:term[State]{canonical="State"}:** The :term[State]{canonical="State"} object is the primary scratchpad for variables in multi-step workflows. It's the most common target for :term[Output Paths]{canonical="Output Path"} because it persists between the ticks of an agent's execution loop, allowing one step to leave a result that a future step can read.

  > Sidenote:
  >
  > - [009: Agent/State](./009_agent_state.md)

- **:term[Plan]{canonical="Plan"}:** Variables are the fundamental technology that powers the :term[Plan]{canonical="Plan"} system. A :term[Plan]{canonical="Plan"} message contains a graph of :term[Tool Calls]{canonical="Call"} where the connections (edges) are formed by :term[Variable References]{canonical="Variable Reference"} pointing to :term[Output Paths]{canonical="Output Path"}. This allows an agent to define a complete, executable workflow as a single, declarative data structure.

  > Sidenote:
  >
  > - :term[012: Agent/Plan]{href="./012_agent_plan.md"}

## From Ephemeral Connections to Persistent Memory

:term[Variables]{canonical="Variable"} provide the mechanism for wiring data to tools within a single, atomic request. However, to build complex agents that execute tasks over multiple steps, a more persistent form of memory is required—a "scratchpad" where results can be stored and accessed across multiple, independent requests in an execution loop.

[008: Agent/Output](./008_agent_output.md) describes the mechanics of how results are written to the context.
