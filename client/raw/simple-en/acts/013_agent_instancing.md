# 012: Agent/Instancing

> [!DEFINITION] [Instancing](./000_glossary.md)
> Think of it like giving a special nametag to a group of messages. This lets you create separate work zones, called :term[Instances]{canonical="Instance"}, inside a single request. It’s like running many different conversations at the same time in one chat window, without them getting mixed up.

> Sidenote:
>
> - Needs: :term[009: Agent/State]{href="./009_agent_state.md"}
> - Works with:
>   - :term[006: Agent/Input]{href="./006_agent_input.md"}
>   - :term[015: Agent/Scopes]{href="./015_agent_scopes.md"}
>   - :term[012: Agent/Plan]{href="./012_agent_plan.md"}

The **Instancing Protocol** is a clever way to help an AI agent do many things at once. It lets the agent work on several separate tasks at the same time, all within a single command, without needing to change how the agent thinks.

How does it do this? By putting a special label on related messages, like the agent's memory (:term[State]{canonical="State"}) or its instructions (:term[Input]{canonical="Input"}). This label is a unique ID for each task, or “instance.” This way, a single recipe, or :term[Plan]{canonical="Plan"}, can be used to cook many different meals all at once, in parallel. This makes the agent much faster and more consistent.

## How Instancing Works

Instancing lets you process many separate jobs by adding a layer on top of messages that carry information, like the :term[State]{href="./009_agent_state.md"} message. Instead of sending one :term[State]{canonical="State"} with a single job, you can send a whole list of them. Each one is a separate :term[Instance]{canonical="Instance"} of a task.

To keep track of all these parallel jobs, each :term[State]{canonical="State"} message gets a **unique ID** using a special property called `_instance`. These IDs are like little stickers (e.g., `①`, `②`) that help the AI connect actions to the correct job.

Using Instancing is optional. You turn it on by adding the `_instance` sticker to a message like :term[State]{canonical="State"} or :term[Input]{canonical="Input"}. This tells the system that this message is part of its own separate, safe work area. If a message doesn't have an `_instance` sticker, it's treated like a shared tool that all the jobs can use.

This method has big advantages:

- **Efficiency**: It's like having ten chefs follow the same recipe at once instead of one chef making ten dishes one after another. You get a lot more done in one go.
- **Consistency**: The AI can see all the similar jobs at the same time. This helps it make smarter, more consistent decisions, just like a judge would make fairer rulings by seeing several similar cases together.

## Working with Other Messages

The real power of Instancing comes from how the `_instance` sticker changes the way different messages behave.

- **:term[State]{canonical="State"}:** The :term[State]{canonical="State"} message is the heart of the system. Each :term[Instance]{canonical="Instance"} is its own :term[State]{canonical="State"}, with its own unique `_instance` ID. Think of it as a separate workbench for each project, so tools and materials don't get mixed up.

  > Sidenote:
  >
  > - :term[009: Agent/State]{href="./009_agent_state.md"}

- **:term[Input]{canonical="Input"}:** An :term[Input]{canonical="Input"} message, or instruction, can be used in two ways. A shared :term[Input]{canonical="Input"} (with no `_instance` sticker) gives the same instructions to _all_ jobs. A targeted :term[Input]{canonical="Input"} (with an `_instance` sticker) gives a special instruction to just _one_ specific job, overriding the shared instructions.

  > Sidenote:
  >
  > - :term[006: Agent/Input]{href="./006_agent_input.md"}

- **:term[Scopes]{canonical="Scope"}:** The `_instance` sticker is very important for keeping work private. It acts like a wall around each workbench. When a command (:term[Call]{canonical="Call"}) is sent to a specific instance, its tools and information (`_scopes`) are also locked to that instance. This ensures that a helper agent (:term[Delegate]{canonical="Delegate"}) only sees the information relevant to its task, even if it's one of many helpers working on a big project.

  > Sidenote:
  >
  > - :term[015: Agent/Scopes]{href="./015_agent_scopes.md"}

:::details{title="Example: Checking Many Comments at Once"}

Instancing can turn an agent that does one task at a time into a powerhouse that handles many. Imagine an agent that checks user comments to see if they follow the rules.

Without instancing, the agent would check comments one by one. To check 100 comments, it would need to send 100 separate requests. This is slow, and the agent can't see the big picture. It might approve a comment that, when seen with others, is actually part of a spam attack.

With the **Instancing Protocol**, all 100 comments can be checked in one single request.

1.  **Group the Inputs**: The agent gets a list of 100 :term[Input]{canonical="Input"} messages. Each message has a unique `_instance` sticker and contains one user comment.

    ```json
    // A group of Input messages sent to the AI
    [
      {
        "type": "input",
        "_instance": "①",
        "comment": "This is a great post!"
      },
      {
        "type": "input",
        "_instance": "②",
        "comment": "I disagree with this..."
      },
      {
        "type": "input",
        "_instance": "③",
        "comment": "This is spam."
      }
      // ... and 97 more comments
    ]
    ```

2.  **Run the Plan in Parallel**: The AI sees all the comments at once. It can use a single :term[Plan]{canonical="Plan"} for all of them, like: first, check if the comment is positive or negative, then check for bad words. This single plan is applied to all 100 comments at the same time.

3.  **Get Targeted Results**: The agent's final answer (:term[solution]{canonical="Solution"}) will be a single list of actions (:term[Calls]{canonical="Call"}), but each action is aimed at a specific comment using its `_instance` sticker.

    ```json
    // The solution created by the AI
    {
      "calls": [
        {
          "_tool": "moderateComment",
          "_instance": "①",
          "decision": "approve"
        },
        {
          "_tool": "moderateComment",
          "_instance": "②",
          "decision": "approve"
        },
        {
          "_tool": "moderateComment",
          "_instance": "③",
          "decision": "reject"
        }
        // ... and 97 more decisions
      ]
    }
    ```

This approach gives you two huge benefits:

- **Speed**: What used to take 100 back-and-forth trips to the AI now happens in just one.
- **Consistency**: By seeing all the comments together, the AI gets more context. It can make better judgments and even spot patterns, like an organized spam attack, that it would miss if it only saw one comment at a time.

:::

:::::details{title="Giving One Instruction to a Group"}

This example shows how a single, general instruction (`Input`) can be used to control one specific worker in a group. It's like a manager telling a whole team, "Everyone keep working, but you, Bob, start a new high-priority task." This turns a simple batch job into a smart way to manage many tasks at once.

::::columns
:::column{title="The Setup (Context)"}

Imagine a manager giving instructions to employees. A single `input` is the manager's command. The different `state` messages are the employees, each with their own task. This shows how an agent built to manage one person can easily manage a whole team without being redesigned.

```json
[
  // A general instruction for one employee
  {
    "type": "input",
    "instruction": "Give employee B a new, high-priority task to 'Finalize the quarterly report'."
  },

  // What all the employees are currently doing
  {
    "type": "state",
    "_instance": "employee_A",
    "task": "Draft initial proposal",
    "status": "In Progress"
  },
  {
    "type": "state",
    "_instance": "employee_B",
    "task": "Review team submissions",
    "status": "Blocked"
  }
]
```

:::
:::column{title="The Result (Targeted Solution)"}

Even though the instruction was part of a general message, the AI understood who it was for. It creates a command (`Call`) aimed directly at `employee_B` using the `_instance` property. The other employee is not affected at all.

```json
{
  "calls": [
    {
      "_tool": "updateTask",
      "_instance": "employee_B",
      "newTask": "Finalize the quarterly report",
      "newStatus": "High Priority",
      "output": "†state"
    }
  ]
}
```

:::
::::
:::::

## How it Connects to Other Systems

Instancing works with other parts of the system to control how things get done.

- **:term[Calls]{canonical="Call"}:** Putting an `_instance` sticker on a :term[Call]{canonical="Call"} is the main way you direct an action. It makes sure that any changes—like saving a result to an :term[Output Path]{canonical="Output Path"} or reading information to use as an input—happen in the correct work area (:term[Instance]{canonical="Instance"}).

  > Sidenote:
  >
  > - [008: Agent/Output](./008_agent_output.md)

- **:term[Plan]{canonical="Plan"}:** A :term[Plan]{canonical="Plan"} is like a master blueprint that isn't tied to any single instance. It can be used with many different workbenches (:term[State]{canonical="State"} messages), allowing the same set of steps to run on many separate jobs at the same time.

  > Sidenote:
  >
  > - :term[012: Agent/Plan]{href="./012_agent_plan.md"}

## From a Plan to a Process

If a :term[Plan]{canonical="Plan"} is the reusable recipe for a task, and :term[Instancing]{canonical="Instancing"} is the kitchen that lets you cook many dishes at once, then a **:term[Process Idea]{href="./203_idea_process.md"}** is the final cookbook. It contains the complete record of the recipe and the current status of all the dishes being made.
