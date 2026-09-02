# 011: Agent/Expressions

> [!DEFINITION] [Expression](./000_glossary.md)
> A way of writing instructions that combines different data paths in a :term[Variable Reference]{canonical="Variable Reference"} or :term[Output Path]{canonical="Output Path"}. Think of it like a smart train system that can switch tracks, merge trains together, or send cargo down multiple paths at once.

> Sidenote:
> - Requires:
>   - :term[007: Agent/Variables]{href="./007_agent_variables.md"}
>   - :term[008: Agent/Output]{href="./008_agent_output.md"}
> - Enables:
>   - :term[012: Agent/Plan]{href="./012_agent_plan.md"}

While :term[Variable References]{canonical="Variable Reference"} and :term[Output Paths]{canonical="Output Path"} are like simple pipes that move data from A to B, **Expressions** add smart switches to those pipes. They let an agent make choices and handle multiple tasks at once. Expressions use `<|>` (meaning OR) to make choices, and `*>` (meaning AND) to force things to wait for each other. You can use them to gather ingredients before starting a task, or to route the final products after a task is done.

## Input Expressions (Many-to-One)

When you use expressions for a :term[Tool Call]{canonical="Tool Call"}'s inputs, you can pull information from several places to fill one single slot. This is like a chef having backup ingredients ready just in case the main one is missing. It creates a **many-to-one (`M:1`)** data flow.

> Sidenote:
> ```mermaid
> graph TD
>     SourceA["†state.userInput"] --> ToolInput["processData(data)"]
>     SourceB["†state.default"] --> ToolInput
> ```

This makes tools much more reliable. Instead of breaking because it needs data in one strict location, a tool can adapt and look around for what it needs based on the context.

::::columns
:::column{title="Fallback Logic with <|>"}

Using `<|>` in a :term[Variable Reference]{canonical="Variable Reference"} acts like a safety net. The expression `†state.userInput <|> †state.default` tells the system: "Look for what the user typed. If they did not type anything, use the default text instead." It gracefully handles optional or missing information.

```json
{
  "_tool": "processData",
  "data": "†input.optionalData <|> †state.defaultData"
}
```

:::
:::column{title="Concurrent Dependency with *>"}

Using `*>` in a :term[Variable Reference]{canonical="Variable Reference"} acts like a security checkpoint. It waits until all the required pieces of data have safely arrived before opening the gate. This guarantees a tool only begins working when all of its necessary supplies are ready.

```json
// This tool will only run if both a user profile and their permissions are loaded.
// The `permissions` parameter will receive the value of `†state.permissions`.
{
  "_tool": "renderDashboard",
  "permissions": "†state.userProfile *> †state.permissions"
}
```

:::
::::

## Output Expressions (One-to-Many)

When used in an :term[Output Path]{canonical="Output Path"}, expressions work in reverse. A single tool's finished work can be sent to multiple destinations. This creates a **one-to-many (`1:M`)** flow, like a lawn sprinkler shooting water in different directions based on what zone needs it.

> Sidenote:
> ```mermaid
> graph TD
>     ToolOutput["verifyUser()"] --> DestA["†data.user.verified"]
>     ToolOutput --> DestB["†data.user.failed"]
> ```

This allows one tool to trigger completely different outcomes depending on what happened. For example, a login tool might send data down a "success" path or a "failure" path, allowing the agent to react intelligently to the situation.

::::columns
:::column{title="Alternative Paths (Branching) with <|>"}
Using `<|>` in an :term[Output Path]{canonical="Output Path"} sets up different possible tracks for the data to travel down. The pathway naturally defaults to the **first** track. If something goes differently, the :term[Activity]{canonical="Activity"} can pack the data into a special message and send it down the alternative track instead. The tool itself decides which way the data goes, just like a train conductor pulling a lever.

The symbol is exactly three characters: `<|>`. Missing the angle brackets and accidentally using basic symbols like `|` or `||` will cause the data to land nowhere and stall the process. Always rely on the complete `<|>` shape.

```json
// If `verifyUser` succeeds, the result is written to `data.user.verified`;
// otherwise, it's written to `data.user.failed`
{
  "_tool": "verifyUser",
  "userId": "perfect-stranger",
  "_outputPath": "†data.user.verified <|> †data.user.failed"
}
```

:::
:::column{title="Concurrent Paths (Fan-out) with *>"}
Using `*>` in an :term[Output Path]{canonical="Output Path"} acts like a megaphone. It takes the exact same result and shouts it to multiple places at the exact same time. This is incredibly helpful for saving a user's data while simultaneously copying it to a secret backup record.

```json
// Writing to `user` and `audit` objects in the state simultaneously.
{
  "_tool": "generateSummary",
  "text": "Long body of text here...",
  "_outputPath": "†data.user.summary *> †data.audit.summary"
}
```

:::
::::

## An Input Expression Is Any Computation, Not Only a Choice

The input shortcuts we just explored are handy, but they are just part of the story. **An input expression can actually be any kind of standard code math or formula you already know.** So, picking a backup (`†state.a <|> †state.b`) and calculating a total (`†state.items.filter(i => i.paid).length`) are handled exactly the same way.

This smart math setup helps in three major ways:

- **Parameters**: A tool gets exactly the finished answer it needs, like a final number, rather than receiving raw data and instructions on how to figure it out.
- **Conditions**: A step can wait until the math gives a specific answer (like waiting for a counter to hit a certain number) before moving forward.
- **Named rules**: You can write a formula, save the result under a specific name, and let everything else use it easily as a normal :term[Variable Reference]{canonical="Variable Reference"} without recalculating it.

**The data you calculate on does not even have to require a long background process.** Sometimes, a simple math question can be answered effortlessly by the system on the very same request. The core mechanics are handled by :term[003: Agent/Activity]{href="./003_agent_activity.md"}; the important piece here is that input fields fully support this math.

**However, outputs do not have this flexibility.** An :term[Output Path]{canonical="Output Path"} must stay purely simple—it is just the destination address on an envelope. If we let destinations do complex coding math, data could secretly get written to the wrong places, which is a confusing and terrible bug to fix.

## Why the Operators Are Not the Language's Own

If you know coding languages like JavaScript, you might wonder why we don't just use their standard symbols, like `||` and `&&`. **They mean something slightly different, and mixing them up causes invisible mistakes.**

Our expressions are designed to check if a piece of data **exists yet**. Standard code symbols like JavaScript's check if a value is **"true"**. If you have a counter sitting at zero, JavaScript might mistakenly think that zero means "nothing" and apply a backup number instead. That would secretly break your data flow.

A :term[Variable Reference]{canonical="Variable Reference"} is really a **future promise**. It declares "I will need this eventually" even if the data does not exist at all yet. Standard programming math cannot handle things that are still pending; it needs the answer right now.

To solve this, we rely on specific symbols built precisely for handling missing futures:

- **`<|>`** — `Alternative` (Backup plan). It looks at the left path. If data eventually arrives there, it takes it. If that path comes up empty, it switches to the right path instead.
- **`*>`** — `Sequence` (Checkpoint gate). It waits patiently for the left path, ignores whatever data arrived, and then lets the right path proceed. It guarantees step-by-step order.

**These two operators handle two different jobs.** The `<|>` decides *which data to use*, but it also decides *when the process is ready to go*. If a tool has a fast path and a slow path connected by `<|>`, it simply moves forward the instant either side completes.

We add a third symbol when you want to pass data through an assembly line of steps:

- **`|>`** — `Flow` (Pipeline). This takes whatever results on the left and feeds it smoothly into the function on the right, like a conveyor belt sliding a product to the next work station.

```
†state.items |> map(enrich, 4) |> filter(i => i.valid) |> reduce(sum, 0)
```

Since none of these three symbols (`<|>`, `*>`, `|>`) work in standard JavaScript, they stand out safely as our own syntax. You use regular symbols for simple math, and these special brackets specifically for managing the future.

**Order of operations (tightest to loosest): `<|>`, then `|>`, then `*>`.** The system picks a backup, pushes it through the conveyor belt, and lastly enforces the checkpoints. Always include parentheses when mixing them up so there is zero confusion about what happens first.

### A Pipeline Is the Whole Parameter, or It Is Not a Pipeline

**These three pipeline symbols only work if they sit at the very outer edge of your instruction.** If you try to bury them deep inside a normal text string, the system ignores them entirely. For example, if you write `'total: ' + (†state.x |> f)`, the `|>` rests harmlessly inside text formatting. The system will process it as normal, jagged code rather than a smart pipeline, creating immediate headaches without any warning sirens.

To avoid this, **keep the pipeline out in the open.** If you need to mix it with text or change small elements, place the varying part inside a little function block where regular :term[Variable References]{canonical="Variable Reference"} are allowed:

```
†state.passages |> filter(p => †state.chosen.includes(p.id)) |> map(p => p.words, 4)
```

Always position choices and backups at the absolute start of your logic. It keeps your intent crystal clear.

> Sidenote:
> `|>` carries one small risk that the other two do not. Right now, there is an ongoing debate about how to officially add a pipeline symbol into standard JavaScript. If they ever officially decide to use `|>`, our special symbol will suddenly conflict with it. However, if that happens, the system will break loudly with massive red flags. We drastically prefer loud, obvious failures over the silent, invisible mistakes common with symbols like `||`.

## Where the Stages Come From

The tools inside that pipeline, like `map`, `filter`, and `reduce`, are not standard built-in functions. They come from a custom toolkit called `@idealic/iterators`. They are designed like perfectly fitted puzzle pieces that only need an `input` and immediately produce an `output`.

This specific pipeline toolkit does three powerful things that standard code cannot do:

- **They can multitask.** Sending `map(enrich, 4)` tells the toolkit to process four items exactly simultaneously. Standard arrays only process one stubborn item strictly after the other.
- **They are lazy in a good way.** A tool stops the conveyor belt the moment its job is done. If you say `take(2)`, the system processes exactly two and completely stops doing the expensive chore. Standard arrays exhaustively process every single item before throwing away the extras.
- **They can be cancelled.** If a process gets stuck or runs out of budget, every single tool in our toolkit carries an emergency stop button. Standard code has virtually no way to slam the brakes.

If your setup finishes with a streaming tool, it pleasantly hands you an organized list of items. If it finishes with a tool that calculates a final answer, like `reduce`, you get that exact final answer immediately.

> Sidenote:
> When you race two processes against each other, you expect the champion to win. However, you cannot properly stage this race in the middle of a regular math expression. A standard expression stubbornly forces everything to fully complete before it will move forward. The true race lives solely in the `<|>` symbol, which smoothly unlocks tracking the very second a piece of data successfully arrives. A key feature of our design: the "losing" slower trace is absolutely not discarded or punished; it simply stops pushing its data forward while finishing its task in the background safely.

## From Expressions to Strategies

Expressions represent the intelligent connective tissue between your basic :term[Tool Calls]{canonical="Tool Call"}. Once mastered, they let your agent step beyond rigid, predictable checklists to instead command dynamic, branching workflows.

See :term[012: Agent/Plan]{href="./012_agent_plan.md"} to learn exactly how you combine these expressions to orchestrate an entire, masterful :term[Plan]{canonical="Plan"}.
