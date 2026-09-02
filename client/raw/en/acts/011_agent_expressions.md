# 011: Agent/Expressions

> [!DEFINITION] [Expression](./000_glossary.md)
> A syntax for combining multiple paths in a :term[Variable Reference]{canonical="Variable Reference"} or :term[Output Path]{canonical="Output Path"} to create conditional or parallel data flows.

> Sidenote:
>
> - Requires:
>   - :term[007: Agent/Variables]{href="./007_agent_variables.md"}
>   - :term[008: Agent/Output]{href="./008_agent_output.md"}
> - Enables:
>   - :term[012: Agent/Plan]{href="./012_agent_plan.md"}

While :term[Variable References]{canonical="Variable Reference"} and :term[Output Paths]{canonical="Output Path"} provide the basic wiring for data flow, **Expressions** introduce logic into this wiring. They are a syntax for creating conditional branches and parallel data flows, allowing an agent to define more complex and adaptive behaviors declaratively.

Expressions use `<|>` (OR) for conditional logic and `*>` (AND) for concurrent operations. They can be applied in two primary ways: gathering inputs for a tool and distributing outputs from a tool.

## Input Expressions (Many-to-One)

When used in a :term[Tool Call]{canonical="Tool Call"}'s input parameters, expressions allow a single parameter to draw data from multiple sources. This creates a **many-to-one (`M:1`)** data flow, enabling flexible data aggregation patterns like fallbacks and merges.

> Sidenote:
>
> ```mermaid
> graph TD
>     SourceA["†state.userInput"] --> ToolInput["processData(data)"]
>     SourceB["†state.default"] --> ToolInput
> ```

This capability is key to creating robust, reusable tools. Instead of requiring a rigid, hard-coded data structure for its inputs, a tool can be designed to flexibly adapt to the available context, sourcing its data from different places based on what is present at runtime.

::::columns
:::column{title="Fallback Logic with <|>"}

When used in a :term[Variable Reference]{canonical="Variable Reference"}, `<|>` acts as a fallback. The expression `†state.userInput <|> †state.default` instructs the engine to first look for a value at `state.userInput`. If it's not found, it will use the value from `state.default` instead. This is useful for providing default values or handling optional inputs gracefully.

```json
{
  "_tool": "processData",
  "data": "†input.optionalData <|> †state.defaultData"
}
```

:::
:::column{title="Concurrent Dependency with *>"}

When used in a :term[Variable Reference]{canonical="Variable Reference"}, `*>` acts as a gate, ensuring that all specified data paths exist in the context before proceeding. If all paths are present, the expression resolves to the value of the _last_ path in the sequence. This enforces dependencies, ensuring a tool only runs after all its prerequisite data is available.

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

When used in an :term[Output Path]{canonical="Output Path"}, expressions allow a single tool's result to be written to multiple destinations. This creates a **one-to-many (`1:M`)** data flow, enabling patterns like conditional branching and fan-out.

> Sidenote:
>
> ```mermaid
> graph TD
>     ToolOutput["verifyUser()"] --> DestA["†data.user.verified"]
>     ToolOutput --> DestB["†data.user.failed"]
> ```

This mechanism allows a single tool to have multiple, context-dependent outcomes. It is the foundation for creating dynamic agents that can react intelligently to situations, choosing the appropriate path of execution based on the results of their actions.

::::columns
:::column{title="Alternative Paths (Branching) with <|>"}
Using `<|>` in an :term[Output Path]{canonical="Output Path"} allows a tool to declare its possible outcomes; exactly one of them is written. The Path on its own writes the **first** alternative, which is the outcome a tool declares as its ordinary one. To write any other, the :term[Activity]{canonical="Activity"} returns a ready-made `Data Message` naming that path instead of returning a value — a message no Output Path governs. That is the whole of "the tool's internal logic decides": there is no separate protocol, and nothing records which alternative was meant.

The operator is three characters with nothing between them: a less-than, a pipe, a greater-than. `|` and `||` are near misses rather than the operator: one is read as part of the path, so the write lands somewhere nothing reads and every call waiting on either outcome stalls. `|>` is not a near miss but a different operator — it pipes a value through a function and belongs in a parameter, and an Output Path carrying one is refused rather than written, which is the one failure here that says so out loud.

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
Using `*>` in an :term[Output Path]{canonical="Output Path"} directs the engine to perform a fan-out, writing the same output to multiple paths at once. This is useful for broadcasting a result to multiple parts of the state or for auditing purposes.

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

The input side above combines several paths into one value, and it has exactly one combinator to do it with. That is the poorest possible many-to-one. **The general form is an expression in the language the reader already knows** — so `†state.a <|> †state.b` and `†state.items.filter(i => i.paid).length` are the same kind of thing, one of them just picks rather than computes.

This lands in three places at once, and none of them is new machinery:

- **A parameter** holds a value it computed rather than one it looked up. A tool that wants a count is handed a count, not an array and an instruction.
- **A condition** holds a call back until what arrived is worth acting on. A branch on a *value* — not merely on a value's presence — becomes something the loop can take by itself.
- **A named one** is written once and used by everything: its result lands at `†expr.<name>`, so referencing it is an ordinary :term[Variable Reference]{canonical="Variable Reference"} and needs no second lookup.

**What it computes over need not have been computed by anything.** A :term[Call]{canonical="Call"} with no :term[Activity]{canonical="Activity"} behind it is answered by the model in the same response that authors it, and that write lands inside the same drain — so an expression reading its :term[Output Path]{canonical="Output Path"} becomes ready on that turn rather than the next. A judgement nobody wrote code for, and the arithmetic over it, settle in one request. :term[003: Agent/Activity]{href="./003_agent_activity.md"} owns the mechanism; what matters here is that a computed parameter may read one.

**The output side does not follow.** A :term[Output Path]{canonical="Output Path"} must remain a path — one-to-many stays what it is, and an expression is admitted in input positions only. Blurring that would let a computed string be read as a multi-target write, which is a silent and expensive way to be wrong.

## Why the Operators Are Not the Language's Own

The temptation is to keep `||` and `&&`, since the general form is now JavaScript and JavaScript has both. **They mean something else here, and the difference is invisible.**

These operators test **existence**: a path either resolved or it did not. JavaScript tests **truthiness**. So `†state.count || 5` yields the count under one reading and `5` under the other whenever the count is zero — the same characters, a different answer, and nothing to notice.

The deeper reason is that a :term[Variable Reference]{canonical="Variable Reference"} is a **future**. It names a value that will exist, addressed before it does — which is what the loop's whole notion of readiness is about. And **JavaScript's operators work on values.** A reference that has not resolved is not `null` and not `undefined`; it is a thing still pending, so `??` cannot help and would silently take the left side every time.

So the two operators are borrowed from the tradition that has thought hardest about values that may not be there:

- **`<|>`** — `Alternative`. *The first alternative that has a value.* Over futures, left-preferred: the value read is the left whenever the left has one, and the right is reached only when it does not.
- **`*>`** — sequence-right. *Wait for the left, discard it, take the right.* The gate, exactly as it already behaves.

**`<|>` answers two different questions and they are worth keeping apart.** *Which value* is the left-preference above. *When the call may go* is the speculative half: readiness is satisfied as soon as **either** side has landed, so a call whose slow arm is still outstanding goes on the fast one. Neither question is answered by "whichever arrived first" — nothing records an arrival order, and the operator would not read it if it did.

A third joins them once an expression can carry a pipeline rather than a single term:

- **`|>`** — flow. `x |> f` is `f(x)`, so a chain reads in the order it happens instead of inside-out. It is the pipe of F#, OCaml, Elixir and Julia, and the one JavaScript's own pipeline proposal reaches for.

```
†state.items |> map(enrich, 4) |> filter(i => i.valid) |> reduce(sum, 0)
```

None of the three is writable in JavaScript — all are syntax errors — so none can ever be quietly read as the language's own. The grammar then divides by what it operates on rather than by convention: **`||` and `??` for values, `<|>` `*>` `|>` for futures.**

**Precedence, tightest to loosest: `<|>`, then `|>`, then `*>`.** A source is chosen before it is piped, and a gate governs everything after it — so `†a <|> †b |> f` takes whichever source answered and pipes that, while `†gate *> †docs |> f` holds the whole pipeline until the gate resolves. **Parenthesise whenever the three are mixed.** The middle case reads two ways to a careful reader, and the model that had to be told a single `|` is not the operator is not the reader to leave it to.

### A Pipeline Is the Whole Parameter, or It Is Not a Pipeline

**The three operators are read only at the top level of an expression**, and one welded into a larger expression is silently not read at all. In `'total: ' + (†state.x |> f)` the `|>` sits at depth one, where nothing cuts it — so the string does not compile as an expression, falls through to its last available reading as an ordinary literal, and **the :term[Activity]{canonical="Activity"} is handed the dagger text itself.** Nothing errors, at any layer, and the wrong value is a plausible one.

The way around it is not an escape but a placement. **Keep the pipeline as the whole parameter and let the varying part ride inside a callback**, where a :term[Variable Reference]{canonical="Variable Reference"} is legal:

```
†state.passages |> filter(p => †state.chosen.includes(p.id)) |> map(p => p.words, 4)
```

The same rule cuts the other way for an author reaching for concurrency: a sigil inside an argument list — `take(†a <|> †b)` — is never rewritten either. An alternative belongs at the head of the pipeline, which is also where it says what the author meant.

> Sidenote:
>
> `|>` carries one risk the other two do not, and it is worth naming because of how it would fail rather than whether it will. JavaScript's pipeline proposal is stalled between two spellings, and if the language ever ships the other one, `|>` stops being a syntax error. That failure is **loud** — a tokenizer meeting valid JavaScript where it expected its own operator — and loud is the whole reason `||` had to be given up. The direction agrees either way; only the placeholder differs.

## Where the Stages Come From

`map`, `filter` and `reduce` in that chain are not the array methods. They are `@idealic/iterators`, and an expression is handed them already bound — which is what makes the chain read as written, because every one of those operators has a curried form and a curried operator **is** the `input → output` a stage has to be.

Three things follow that an `Array.prototype` chain cannot do, and they are the reason the pipeline is worth having at all:

- **Concurrency is an argument.** `map(enrich, 4)` enriches four at a time. An array chain has one item at a time and no way to say otherwise.
- **A pipeline is lazy, so `take` stops the work rather than discarding it.** `|> map(expensive) |> take(2)` runs `expensive` twice. The same array chain runs it on everything and then throws most of it away.
- **The run can cancel it.** Every operator carries the evaluation's own abort signal, so when an expression exceeds its budget the work is told to stop instead of running on unobserved. `Promise.race` deliberately has no way to say that, which is why a raced arm keeps going and a piped one does not.

A pipeline that ends in a streaming operator answers with the items collected: `†state.items |> map(enrich, 4)` reaches the parameter as an array. A pipeline ending in `reduce`, `find` or `every` answers with that value directly.

> Sidenote:
>
> A future that can be raced is a future that can be speculated on — but **a race over two references is not where that happens.** An expression waits for every reference it reads before its body runs, so both values are already in hand and a `Promise.race` between them is settled by list order. Speculation lives in `<|>`, which lets the call go as soon as either side has landed. A genuine contest needs two things that settle at different times, and the expression has to bring those itself.
>
> Wherever one does happen, what it buys is that it needs no machinery: **one expression, one value, one write**, so there is nothing to rescind and no way to unwrite a losing branch. The loser is not cancelled, in keeping with what a race means everywhere else; only its value stops travelling. An arm that reached a tool has still reached it.

## From Expressions to Strategies

Expressions provide the logical glue to connect individual :term[Tool Calls]{canonical="Tool Call"}. With this capability, an agent can move beyond simple, linear sequences of actions and begin to construct sophisticated, branching workflows.

:term[012: Agent/Plan]{href="./012_agent_plan.md"} describes how these expressive connections are used to build a complete, strategic :term[Plan]{canonical="Plan"}.
