# 091: Agent/Caching

> [!DEFINITION] [Caching](./000_glossary.md)
> The discipline by which a context is ordered so that the part of it that does not change between turns comes first — because a provider caches a prompt by its prefix, and a prefix ends at the first byte that moved.

> Sidenote:
>
> - Requires:
>   - :term[005: Agent/Data]{href="./005_agent_data.md"}
>   - :term[009: Agent/State]{href="./009_agent_state.md"}
>   - :term[010: Agent/Loop]{href="./010_agent_loop.md"}
> - Complemented by:
>   - :term[090: Agent/Typing]{href="./090_agent_typing.md"}

> [!HEADSUP] Heads up
> The number `091` is provisional. The `09x` range holds cross-cutting concerns and the author reindexes; treat the file name as a placeholder and the content as settled.

A chat agent never has this problem. It appends to its history, so the bytes it sent last turn are a prefix of the bytes it sends this turn, and every provider's cache is free. This agent does not carry a history. It renders **current state**: :term[Data]{canonical="Data"} folds by `kind` into one block, `Var.resolve` folds a run of accumulating writes into one value, and the model reads what is true now rather than the sequence that made it true.

That fold is the best property the design has, and it is the one that defeats caching. A fold mutates in place. Add a `¶findings` block on turn two and it does not land after `¶state` — it lands *before* it, and everything downstream shifts by the width of the new block. A provider looking for a prefix finds one that ends at the insertion point.

The resolving observation is that **the underlying message list is append-only; only the rendering folds.** A :term[Data]{canonical="Data"} message, once written, is never rewritten. There is immutable substrate under a mutable presentation, and the presentation is free to order itself however it likes.

## The Rule

**Render in order of how often a block changes, and never the other way.**

Three tiers, and every rendered block belongs to exactly one:

- **Frozen** — instructions, :term[Tool]{canonical="Tool"} declarations, the :term[Input Message]{canonical="Input Message"}, the opening prompt. Settled before turn one and unchanged after it.
- **Append-only** — the failure log. It grows; it never rewrites what it already holds.
- **Derived** — the :term[State]{canonical="State"} fold, the standing :term[Plan]{canonical="Plan"}, an :term[Advisor]{canonical="Advisor"}'s advice. Recomputed from everything above it, every turn.

Frozen, then append-only, then derived. The prefix is then stable through the first two tiers, and the volatile suffix is small — a fold is smaller than the history it folds.

This is not a house convention. It is what the vendors ask for in their own words. Anthropic: *"Place static content (tool definitions, system instructions, context, examples) at the beginning of your prompt."* OpenAI, more pointedly: *"Put stable developer instructions and shared reference material first. If developer instructions or shared material contain timestamps, user-specific content, or other dynamic content, place those at the end rather than the beginning."*

## Priority Was Never Render Order

The obstacle is that until now there was nothing to order.

`Content.HandlerOptions` carried one number, `priority`, and it means *processing* order: which handler runs first. It is honest about that, and it has to be — the `state` handler must run before `data` so there is a :term[Data]{canonical="Data"} message to fold by the time the fold happens.

But a handler injects messages while it runs. `Message.replace` filters the original out and concatenates the replacement onto the **end** of the list; the `data` handler pushes its blocks after everything it did not touch. The list is re-sorted by priority after every transformation — and the thing a handler produces is `type: 'text'`, whose registered priority is `0`. A rendered message has no priority left to be sorted by. Its position freezes wherever the append put it.

So the order blocks reach the model in is the order handlers **finished** in. Nobody chose it. Traced on a context of a prompt, a tool, a state, an input and one `data` block left by an earlier :term[Output Path]{canonical="Output Path"}:

```
handle tool  (p90)  →  state, input, data:findings, ¶prompt, ¶TOOL_EXECUTION_SYSTEM
handle state (p5)   →  input, data:findings, ¶prompt, ¶TOOL…, data:state
handle input (p4)   →  data:findings, data:state, ¶prompt, ¶TOOL…, data:input
handle data  (p1)   →  ¶prompt, ¶TOOL…, ¶findings, ¶state, ¶input
```

`¶findings` renders ahead of `¶state` for a reason with nothing to do with either: `state` and `input` were *converted into* :term[Data]{canonical="Data"} messages and appended to the end, while a message that was already `data` was never moved, so the fold met it first. The heading order is a side effect of a `concat`.

Ordering by mutability therefore requires render order to exist as a concept, and the whole of making it exist is a second number:

```typescript
export type HandlerOptions = {
  /** PROCESSING order. Higher runs earlier. Says nothing about where output lands. */
  priority: number;
  /** RENDER order: which tier the messages this handler emits belong to. */
  render: number;
};

export const Render = { frozen: 300, appended: 200, derived: 100 } as const;
```

`render` is required, not optional. A kind that does not say which tier it renders into does not compile.

## The Tier Rides on the Message

It cannot live on the handler alone, and the reason is worth stating because it is the same shape as the problem it solves. What a handler emits is `text`. Ask `text` how often it changes and it has nothing to say — a prompt and a state fold are both `text` by the time anyone can sort them.

So the tier is stamped onto the message envelope as it is produced, once, in the one place that knows both the message and the kind that made it:

> A message a handler just produced belongs to the tier of the kind that produced it, unless the handler stamped a tier of its own.

The stamp sits beside `role` and `content`, and `Message.List.group` — which builds what the provider actually receives — reads `role` and `content` and nothing else. The tier never reaches the wire. `Request` then sorts once, stably, immediately before grouping, so blocks inside a tier keep the order their own handler chose.

### The fold takes its most volatile member

A :term[Data]{canonical="Data"} fold merges many messages of one `kind`, and they need not share a tier. `¶input` is folded from :term[Input Messages]{canonical="Input Message"}, which never change. `¶state` is folded from writes that change every turn. Rendering both as one homogeneous run would drag the stable one into the volatile tail, so a folded block takes the **minimum** tier over its members:

**No fold is more stable than its most volatile input.**

This needs no registry of which kinds are stable, because the tier rode in on the messages the fold consumed.

### A kind may emit into more than one tier

The `plan` handler emits `## Plan (eager)` — the strategy this run is following, different every turn — and `# ¶PLAN`, a static explanation of what a plan is. The kind's declared tier is a default, not a verdict. A block that is prose about the protocol is frozen no matter which handler happened to write it, and says so:

```typescript
const planInstructionMessage = {
  role: 'system',
  render: Content.Render.frozen,
  content: { type: 'text', text: `\n# ¶PLAN\n…` },
} as const satisfies Message;
```

The same is true of the question a :term[Question]{href="./018_agent_question.md"} puts: the question as asked is part of the brief and never changes. Only the answers accruing under it at `†question` do, and those are a fold.

## What a Kind's Author Has To Do

**Declare a tier, and make sure it is true.**

The declaration is one line, and the type checker demands it. Truth is the author's problem, and this is where the discipline earns its keep, because **a mis-declared tier fails silently and in one direction it fails expensively.**

Declaring a block **too volatile** costs cache hits. The block is correct, the model reads the right bytes, and you pay full price for text that never moved. Money, quietly.

Declaring a block **too stable** is worse than not caching at all. The block still renders correctly — nothing errors, no test goes red, the model reads exactly what it should. But the block sits in front of a cache breakpoint and changes anyway, so every turn writes a new cache entry that the next turn will not match. Anthropic prices a cache write at 1.25× base input, and a read at 0.1×; a write that is never read is a **25% surcharge on the entire frozen tier, every turn, forever.** Caching turns from a saving into a tax and the only evidence is the bill.

There is a live example of exactly this in the library, and it is instructive because the mistake is so small.

:term[016: Agent/Meta]{href="./016_agent_meta.md"} renders the run's identity, which by its own account does not evolve inside a loop. Perfectly frozen — except that it appends the wall clock to the same block:

```typescript
text: `## Meta:\n${JSON.stringify(finalMeta, null, 2)}\n\n§CURRENT_TIME: ${new Date().toISOString()}`
```

Measured across consecutive turns, the divergence signature between two prompts was `15Z"}]}]` against `30Z"}]}]`. Fifteen seconds of wall clock, and the block moves. In a context where `¶Meta` happens to render last that costs almost nothing; anywhere else it invalidates every byte after it. OpenAI's guidance names this case specifically — *"timestamps, user-specific content, or other dynamic content"* — which is a fair indication of how ordinary the mistake is.

The honest fix is to split the clock out of the identity so the identity can be frozen again. The cheap fix is to declare the whole block derived. Either is fine; declaring it frozen and leaving the timestamp in is not.

**Because the failure is silent, the tier wants a check rather than a convention.** The check is mechanical: run a loop, capture the rendered blocks on consecutive turns, and fail if a block declared frozen is not byte-identical to itself.

## Two Mechanisms, and Only One of Them Reaches Everything

It matters not to conflate these, because they buy different things and reach different tiers.

**Prefix caching.** The bytes are still sent. The provider recognises that it has processed this prefix before and charges less for it — an order of magnitude less, at all three vendors. It needs nothing from you but ordering discipline, it costs nothing to adopt, and **it applies to the whole prompt up to the first change.** This is what the tiering is for.

**Server-side references.** The bytes are genuinely not sent; you name a handle the provider already holds. This works only on content that cannot change, because a handle is a promise that what it names is still what it named. **It reaches the frozen tier and nothing else.** Google offers it as `CachedContent`, and charges storage per token per hour for the privilege; OpenAI reaches for the same durability from the other direction, extending an ordinary prefix entry to `24h` rather than naming a handle. OpenAI's stored responses look like this and are not: reusing one saves you the upload, but *"all previous input tokens for responses in the chain are billed as input tokens."*

The practical shape of the design follows from a coincidence worth leaning on. Anthropic allows four cache breakpoints; OpenAI allows four cache writes. Four is exactly: tools, frozen, append-only — and derived, uncached, past the last breakpoint where its volatility costs nothing. Three tiers plus tool declarations fits the budget with nothing left over and nothing wasted.

## Prefix Caching Is Not Purely Implicit

The two mechanisms above were described as though the first were automatic and beyond influence — order well and hope. That is no longer the whole picture, and the correction matters because it converts a heuristic into a declaration.

OpenAI's prompt cache takes arguments. `prompt_cache_retention` accepts `in_memory` or `24h`; `prompt_cache_options.ttl` accepts `30m`, *"the only supported value"* and also the default; `prompt_cache_key` influences which machine a request lands on, though the guide is careful that keys *"do not pin requests to a machine or guarantee a cache read hit."* Most consequentially, `prompt_cache_options.mode` accepts `explicit`, and a `prompt_cache_breakpoint` may then be placed on a content block to mark where a reusable prefix ends.

**A tier boundary and a cache breakpoint are the same line drawn by two parties.** The tiering exists because a provider caches by prefix and a prefix ends at the first byte that moved, so the discipline was to arrange the blocks and let the provider infer where the stable part stopped. Where breakpoints can be placed, the boundary is no longer inferred — the same three tiers that decide render order say directly, on the wire, where the reusable prefix ends. The ordering discipline does not change; it stops being a hint.

Google's `CachedContent` remains the stronger form on that side, and it is genuinely controllable rather than merely durable: a cache is created with a `ttl`, and `caches.update` takes a new `ttl` or an `expire_time` — *"Changing anything else about the cache isn't supported."* So a run that knows it is about to wait can hold its frozen tier and extend the hold when it wakes.

What none of the vendor pages state is whether a response schema is part of the cached prefix. OpenAI's names *"instructions, developer messages, tool definitions, and conversation history"* and stops; Google's says nothing about `responseSchema` either. Measured against live calls, it is: on `gpt-4o-mini` a payload that was almost entirely schema reported **11,008 of 11,029 prompt tokens cached** on repeat, and on `gpt-5.4-mini` a different one reported **5,376 of 5,970**. Two models and two probes, quoted as such — they are not one series and the pair must not be averaged.

Gemini reaches the same conclusion from the opposite direction, and more sharply. Its schema is not billed as a prompt token at all — and changing it still breaks the cache. Holding the prompt byte-identical at 22,560 tokens and varying only the schema, the entry dies when the schema changes and **returns when the schema returns**, twice over; a single edited description, eight bytes, is enough to lose the hit. So the rule for an author is not "keep the schema small" but **keep it still**: a schema whose descriptions are tuned per turn costs exactly what one whose shape changes costs, and the bytes it costs are invisible in the token count. Tool declarations being cached is stated; the schema being cached had to be found out.

## A Cache Has A Floor And A Lifetime

Two limits sit underneath all of this, and neither is visible in a response that simply reports fewer cached tokens than expected.

**A byte-identical repeated prefix is not sufficient. The prompt has to be big enough first.** OpenAI documents a minimum of 1,024 visible input tokens on its newer models and 2,048 on earlier ones, hedged with *"some models may cache shorter prefixes"*.

The measurements do not simply confirm that, and the disagreement is the useful part. `gpt-4o-mini` cached nothing at 1,013 prompt tokens and 1,024 of 1,083 — under the documented 2,048, which is the hedge doing its work. But `gpt-5.4-mini` cached nothing at 1,289, **above** the 1,024 figure, and first cached at 1,429. So the documented number is neither a floor that binds nor one that can be relied on per model, and the honest reading is that a floor exists, sits in the low thousands, and has to be measured for the model you are actually calling.

Google's documented minimum — 4,096 tokens on its recent Flash models — fails harder, and in both directions at once. `gemini-3.5-flash` was measured reporting cached tokens on a **1,916**-token turn, under half the stated figure; `gemini-3.7-flash` was measured caching **nothing at 4,674**, above it, across twenty consecutive calls. A number that mispredicts the boundary on two models in opposite directions is not a threshold to design against, whatever it is. The likely reconciliation is that a minimum governs the reusable **prefix** rather than the prompt, so overlap between turns decides it — offered as a fitted explanation, and supported by the cached fraction being non-monotonic in prompt size rather than by anything anyone has confirmed. Below the floor a prompt repeated verbatim, all day, caches nothing at all — which means a small tool room can be billed at full rate forever while a large one is discounted, and nothing in either response explains why.

**An entry expires on two clocks at once.** OpenAI's in-memory entries survive *"around 5 to 10 minutes of inactivity, up to one hour"* — an idle timer and an absolute cap, and the absolute one is not refreshed by use. A probe run against one prefix touched repeatedly agreed: the cache held at every interval out to sixty minutes and was cold at ninety, and it was the absolute cap rather than the idle timer that ended it, since the touch before and the touch after were the same thirty minutes apart. That probe's readings are recorded in the session log rather than in the measurements set, and its model was never established — take it as corroboration of the documented behaviour, not as an independent measurement of it.

The consequence for a design is larger than the numbers. **Cadence decides whether any of this pays, and size does not.** A run whose turns are seconds apart collects the whole discount. A run that wakes hourly writes a cold entry every time, however large its frozen tier and however carefully ordered — and the only lever it has is an explicit hold, bought at a storage price, for which :term[092: Agent/Limits]{href="./092_agent_limits.md"} is the component that can say whether it is worth it.

## The Loop Never Moves the Schema

One guarantee is worth stating as a rule, because a great deal rests on it and it is cheap to keep.

**A run's response schema is a function of its authored context. The loop's own additions never change it.**

What the loop appends between turns is an :term[Error Message]{canonical="Error Message"} for a call that failed, the standing :term[Plan]{canonical="Plan"} carried forward, and an advisor's :term[Data]{canonical="Data"} — kinds `error`, `plan` and `data`. **None of the three touches the response schema.** Every kind that does — the ones declaring a tool, a checklist, validations, an instruction wrapper, or the variable injection — is driven by a message an author put in the context, and the loop produces none of them.

So the schema is settled at turn one and holds for the run. Something outside the loop can still move it — a host appending a tool mid-run, a :term[Question]{href="./018_agent_question.md"} raised after the fact, a settings change that switches provider — and each of those is a deliberate act with a visible cause, not a surprise the loop sprang.

**Why it matters more on Gemini than the ordering discipline does.** A response schema there is billed at nothing and still participates in the cache key: change one description by eight bytes and the hit is gone. If the loop could quietly reshape the schema between turns, every run would cold-write its cache on every turn and no amount of block ordering would help. It does not, so the schema half of the prefix is stable by construction and the ordering discipline gets to work on the half that actually moves.

**It is worth a test rather than a promise**, and the test needs no provider: compose the wire schema on two consecutive turns of a run and assert the two are byte-identical. The pieces are already exported — the content pass composes a schema from messages, and the provider transform turns that into what goes on the wire — so the assertion is offline, keyless, and fast. Until it exists this is a property of the code rather than a guarantee of it.

## What Ordering Cannot Buy

The measured effect of putting this in place, across several hundred consecutive-turn transitions on the library's own offline runs: the byte-weighted common prefix went from **57.5% to 70.0%**, with no transition getting worse. An oracle allowed to reorder whole blocks freely reaches **77.7%**.

It does not reach 95%, and the gap is not a failure of ordering. Roughly a quarter of these prompts is genuinely different every turn — the plan is recomputed, the state fold absorbs new writes, the failure log grows. **Ordering moves what changes behind everything that does not. It cannot make a thing that changes stop changing.** A design that promises more than the stable fraction of its own prompt is promising something arithmetic forbids.

Two things follow. The ceiling is a property of the workload, so the way to raise it is to make more of the prompt frozen — not to sort harder. And the byte-weighted figure is the one that matters: averaging per turn flatters the result badly, because the turns that cache worst are the large ones, and a large prompt's miss costs more than a small prompt's hit saves.

> Sidenote:
>
> One block resists the whole scheme, and honestly. :term[003: Agent/Activity]{href="./003_agent_activity.md"} lets an :term[Activity]{canonical="Activity"} return a Message to be placed in the context **verbatim** — the activity speaking rather than reporting. Such a block is `text`, so it inherits the frozen tier, but it arrives on turn two and shoves the tier it joined. Stamping it would rewrite a message the library has promised to pass through untouched. **The tier of a raw block is decided by its provenance, not its kind**, and provenance has to be tracked beside the message rather than on it.

## Outro

The fold is what makes this agent legible to a model, and it is what makes it expensive to a provider. Neither has to give way. The message list underneath was always append-only; all that was missing was a way for the presentation to say which of its blocks were standing still — one number beside `priority`, one stamp as a message is made, one sort before it is sent.

With the context now ordered by what it costs to send, :term[101: Concept/Idea]{href="./101_concept_idea.md"} returns to what the system is composing with it.
