# 008: Agent/Output

> [!DEFINITION] [Output Path](./000_glossary.md)
> The `_outputPath` setting on a :term[Call]{canonical="Call"} acts like a delivery address. It tells the execution engine exactly where to drop off the results of a tool's work, saving it so other steps can use it later.

> Sidenote:
> - Requires:
>   - :term[004: Agent/Call]{href="./004_agent_call.md"}
>   - :term[007: Agent/Variables]{href="./007_agent_variables.md"}
> - Enables:
>   - :term[010: Agent/Loop]{href="./010_agent_loop.md"}

The agent system produces two kinds of results: the stepping stones (the intermediate results of :term[Tool Calls]{canonical="Tool Call"} saved along the way) and the :term[Final Output]{canonical="Final Output"} delivered at the end of the entire process.

## Writing to Context with :term[Output Path]{canonical="Output Path"}

Think of a workflow as having a shared notepad. While any :term[Data Message]{canonical="Data Message" href="./005_agent_data.md"} can act as that notepad, the :term[Output Path]{canonical="Output Path"} property on a :term[Call]{canonical="Call"} is how a tool actually writes its answer down. When a :term[Tool Call]{canonical="Tool Call"} finishes its job, the system attaches the result to the end of the context as a brand-new message.

> Sidenote:
> While any :term[Data Message]{canonical="Data Message"} can be a target, you'll usually write to a :term[State Message]{canonical="State Message" href="./009_agent_state.md"} to save information safely across multiple steps.

This new message is a standard :term[Data Message]{canonical="Data Message"}, but it carries a few invisible sticky notes attached to it for tracking:

- **`_call`**: Which exact :term[Tool Call]{canonical="Tool Call"} created this answer.
- **`_date`**: The exact timestamp of when it was written down.
- **`_outputMethod`**: Instructions on how to mix this new information with older information.

This gives us a clear, step-by-step history of everything the agent did, which is incredibly helpful when you need to debug or figure out how the AI made a decision.

### Defining the Output Path

You can set the :term[Output Path]{canonical="Output Path"} in two ways, giving you different levels of control over what the tool does.

::::columns{.examples}
:::column{title="Dynamic Path (LLM-Decided)"}

Here, the AI brain (LLM) gets to choose where to save the result on the fly. This makes the tool very flexible, like letting a worker pick the best filing cabinet drawer.

```json
// Tool schema allows any string for _outputPath
{
  "_outputPath": {
    "type": "string",
    "description": "Path to store the user summary.",
    "pattern": "^†"
  }
}
```

:::
:::column{title="Prescribed Path (Hard-Coded)"}

This locks the tool into one specific behavior. It guarantees the tool will always drop its output into exactly the same pre-chosen spot every single time.

```json
// Tool schema locks _outputPath to a specific value
{
  "_outputPath": {
    "type": "string",
    "const": "†data.user.summary"
  }
}
```

:::
::::

### Dynamic Variable Resolution

Importantly, when the system writes these results, it doesn't immediately overwrite the old data. It just slaps a new note on top of the pile. The final answer is only pieced together later, right when you actually need to read it.

When something needs to read a :term[Variable Reference]{canonical="Variable Reference" href="./007_agent_variables.md"} like `†data.user.name`, the engine looks through the context messages from **newest to oldest**.

- If the engine finds a note with a **`set`** rule (or no rule, since `set` is the default), it stops looking. That latest note is the final truth, instantly overriding anything written before it.
- If it finds notes with rules like **`merge`** (combine details), **`assign`** (shallow combine), **`push`**, or **`concat`** (add to a list), it keeps digging backward. It gathers all related notes until it hits a `set` instruction, then carefully applies the updates one by one from oldest to newest to build the final answer.
- If it finds an **`erase`** note, it stops hunting. It treats that spot as if nothing was ever written there, making it completely blank again.

This "time machine" design keeps the history of the data crystal clear and prevents messy errors when changing information.

### Erasing a Path, and Why It Is the Only Undo

A :term[Tool Call]{canonical="Call"} is considered "done" the moment its :term[Output Path]{canonical="Output Path"} contains a value. There is no hidden checklist of completed tasks. **The data itself is the only record.** The system pauses because a spot is empty, and it moves forward because something filled it.

If you want the agent to redo a step, you have to take away its previous answer. By erasing the output path, you make the spot empty again, which forces the system to owe that call once more. This is what `erase` is for.

The rule here is simple: **a call that writes nothing doesn't exist to the system.** If a :term[Call]{canonical="Call"} doesn't have an `_outputPath` — like the fire-and-forget tools mentioned below — the system won't know if it ran once, twice, or never. If an action absolutely must happen exactly one time, it must leave a mark in the data to prove it finished. Once it leaves that mark, it follows the same rules as every other step.

:::::details{title="Example: Appending and Resolving"}

**1. Initial State**

The context starts with some basic data.

```json
[
  {
    "type": "data",
    "data": { "user": { "name": "Alex", "status": "active" } }
  }
]
```

**2. Tool Call Execution**

A tool is triggered to change the user's status.

```json
// Call being executed
{
  "_tool": "updateUserStatus",
  "newStatus": "inactive",
  "_outputPath": "†data.user.status"
}
```

**3. Context After Execution**

The system pastes a new note with the updated status at the end of the history. It includes metadata proving which call created it.

```json
[
  // Original data message
  {
    "type": "data",
    "data": { "user": { "name": "Alex", "status": "active" } }
  },
  // Appended output message from the call
  {
    "type": "data",
    "data": { "user": { "status": "inactive" } },
    "_call": {
      "_tool": "updateUserStatus",
      "newStatus": "inactive",
      "_outputPath": "†data.user.status"
    },
    "_date": "2025-10-26T12:00:00Z"
  }
]
```

**4. Variable Resolution**

- To figure out `†data.user.status`, the engine checks the newest note first. It sees `user.status` is `"inactive"` and stops there.
- To figure out `†data.user.name`, it looks at the newest note, doesn't find a name, so it checks the older note. It finds `"Alex"` there and uses that.

:::::

The real magic happens when you pair :term[Variable References]{canonical="Variable Reference"} with :term[Output Paths]{canonical="Output Path"}. You can tell a tool to use data that hasn’t even been generated yet. You set up the instructions, and the system easily maps out flexible toolchains.

You can even daisy-chain :term[Calls]{canonical="Call"}. You can set a tool's input to trigger off the exact :term[Output Path]{canonical="Output Path"} of a previous call in the same series. This creates an automated assembly line where the outcome of one job feeds directly into the start of the next.

## Calls Without an Output Path

Not every :term[Tool Call]{canonical="Call"} needs to drop a record in the notepad. Skipping the `_outputPath` is a built-in feature that changes how the tool behaves depending on the type of call.

### Ephemeral Reasoning for Latent Calls

For internal AI brainwork (latent calls), skipping the output path creates a private thought bubble. It’s a temporary reasoning step that helps the AI figure out what to do right now, but it isn’t saved to the permanent :term[State]{canonical="State"}.

Think of it like sketching on scrap paper. The agent might use a hidden `think` tool to quickly map out a plan. The sketch gets thrown away, but doing the math helps the LLM easily pick the best, concrete :term[Calls]{canonical="Call"} in the very next step because its internal memory was just refreshed.

### Fire-and-Forget for Explicit Calls

If you skip the output path on an explicit call to an :term[Activity]{canonical="Activity"}, it becomes a "fire-and-forget" action. The :term[Execution Loop]{canonical="Execution Loop"} will throw the command over the fence and immediately move on, without waiting around for a result or writing anything down.

This is perfect for background tasks where you don't need a receipt to keep working. Common examples include:

- Logging user clicks to an analytics server.
- Pushing a quick popup notification to a phone.
- Starting a massive background download that shouldn't freeze up your main program.

Remember the rule: if it leaves no record, the system is totally blind to whether it happened or not. If running it twice is dangerous, give it an `_outputPath`. That way, the system records that it finished, and you can purposely `erase` it if you need it to run again.

## Interactions with other systems

- **:term[Data Message]{canonical="Data Message"}:** The `_outputPath` is how workflows create and change :term[Data Messages]{canonical="Data Message"}. It turns a forgetful :term[Tool Call]{canonical="Tool Call"} into a tracked process by stamping its result into the history.

  > Sidenote:
  > - :term[005: Agent/Data]{href="./005_agent_data.md"}

- **:term[State Message]{canonical="State Message"}:** Usually, you will write these results straight into a :term[State Message]{canonical="State Message"}. This makes the :term[State]{canonical="State"} act like the main clipboard of the system, letting different tools effortlessly share bits of information as the :term[Execution Loop]{canonical="Execution Loop"} ticks forward.

  > Sidenote:
  > - :term[009: Agent/State]{href="./009_agent_state.md"}

- **:term[Variable Reference]{canonical="Variable Reference"}:** This is the direct partner to the :term[Output Path]{canonical="Output Path"}. The output path pushes data in, and the variable reference pulls data out. Working together, they map out exactly how information travels across your workflow.

  > Sidenote:
  > - :term[007: Agent/Variables]{href="./007_agent_variables.md"}

- **:term[Expressions]{canonical="Expression"}:** Expressions put logic inside these data pipes. By plugging special symbols like `<|>` and `*>` into an :term[Output Path]{canonical="Output Path"}, a :term[Tool Call]{canonical="Tool Call"} can route its answers to multiple places or change destinations on the fly. It upgrades rigid pipelines into smart intersections.

  > Sidenote:
  > - :term[011: Agent/Expressions]{href="./011_agent_expressions.md"}

- **:term[Plan]{canonical="Plan"}:** Inside a :term[Plan]{canonical="Plan"}, output paths act as the actual wiring snapping different :term[Tool Calls]{canonical="Tool Call"} together into one big machine. The brain can lay out a massive multi-step puzzle and execute it perfectly using these connections.

  > Sidenote:
  > - :term[012: Agent/Plan]{href="./012_agent_plan.md"}

- **:term[Instancing]{canonical="Instancing"}:** If a :term[Tool Call]{canonical="Tool Call"} includes an `_instance` tag, any output path it uses is boxed off into that specific instance’s private workspace. This keeps data totally safe when running many tasks at once, ensuring one tool’s output doesn't accidentally overwrite your other data.

  > Sidenote:
  > - :term[013: Agent/Instancing]{href="./013_agent_instancing.md"}

## From Ephemeral Outputs to Persistent State

Passing outputs between tools works great in the short term. However, when you want a smart agent that learns and adapts for days or weeks, it needs permanent memory.

:term[009: Agent/State]{href="./009_agent_state.md"} breaks down how this long-term storage actually works.
