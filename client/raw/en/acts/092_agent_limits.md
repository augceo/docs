# 092: Agent/Limits

> [!DEFINITION] [Limits](./000_glossary.md)
> The resources a single run may consume — turns, wall clock, tokens, money, and cache held against a future wake — declared before it starts, shown to the model as advice, and enforced by the loop as law.

> Sidenote:
>
> - Requires:
>   - :term[010: Agent/Loop]{href="./010_agent_loop.md"}
>   - :term[001: Agent/Request]{href="./001_agent_request.md"}
> - Complemented by:
>   - :term[091: Agent/Caching]{href="./091_agent_caching.md"}
>   - :term[093: Agent/Compaction]{href="./093_agent_compaction.md"}

> [!HEADSUP] Heads up
> **This chapter is a design, not a description.** `maxTurns` and its exhaustion notice ship today; nothing else here does. There is no budget kind in the library, and the dimensions, thresholds, rates and hold arithmetic below are what is intended rather than what can be called.
>
> The number `092` is provisional, on the same footing as the chapter before it. The `09x` range holds cross-cutting concerns and the author reindexes.

The loop is already bounded, and it already does the hard part correctly.

`maxTurns` caps the run. One turn before the cap the loop appends an exhaustion notice to the context, so the last request carries the news that it is the last — and when the cap arrives the run returns nothing rather than throwing. The reasoning is written beside it: act 010's idiom for a structural failure is an Error Message the next :term[Request]{canonical="Request"} reads, not a throw, so a spent budget is announced the same way, and the turn that reads it is the one still able to act on it.

That is one dimension budgeted properly. **The gap is the other four, and the fact that even the budgeted one is invisible until it is nearly gone.** A run cannot see how many turns it has left, what it has spent, or what the next call will cost — so it cannot spend deliberately. It can only be told, once, that it is nearly over.

A limit the model cannot see is a limit it walks into and hits with nothing to show. A limit the model can see is a fact it can plan against, and planning against it is the whole difference between a cap and a budget.

## The Rule

**Advisory to the model, binding in the loop.**

The model reads what remains and decides how to spend it. The loop enforces the cap whatever the model decided. Neither half works alone: enforcement without visibility produces runs that die mid-thought, and visibility without enforcement is a suggestion, which is what an unbounded loop already ignores.

## What Is Counted

Five dimensions, and they do not all have the same shape:

- **Turns** — how many times the model is asked.
- **Wall clock** — how long the run has been alive, including time spent inside an :term[Activity]{canonical="Activity"}.
- **Tokens** — prompt, answer, and thinking, kept apart rather than summed.
- **Money** — an estimate derived from the tokens and the provider's rates.
- **Held cache** — tokens parked against a future wake, charged for storage while nothing runs.

The first four accrue on activity. The last accrues on **idleness**, which is new, and is the reason it needs saying out loud.

### Context is a level, not a total

Turns, clock, tokens and money only ever go up. Context does not: it rises as the run works and falls when the run compacts. A threshold on a cumulative dimension means *you are running out*. A threshold on context means *do something about it* — and there is something to do, which is the whole of :term[093: Agent/Compaction]{href="./093_agent_compaction.md"}. It is the one dimension the model can answer rather than merely obey.

### Thinking is counted differently by each family, and the provider must say which

Google reports thinking as a third sibling beside prompt and answer. A measured turn on `gemini-pro-latest`: prompt 986, answer 651, **thinking 777**, total 2,414. The thinking cost more than the answer, and a reader watching only the answer count sees 651 output tokens for a turn that produced 1,428.

The OpenAI shape nests reasoning *inside* the completion count. So billed output is

```
completionTokens + (completionIncludesReasoning ? 0 : reasoningTokens)
```

and **it cannot be computed without knowing which convention the provider used.** A budget that normalises both families into one convention silently misreports one of them.

**This half already exists and is the model to follow rather than to restate.** The library's usage record carries every count as *a number or `null`* — never a defaulted zero — and carries the convention flag beside them, under a comment saying the arithmetic above cannot be computed without it. So the provider declares its convention and the reader never infers it, today.

The rule underneath is what generalises to the other four dimensions: **an absent field is not a zero.** A turn spent entirely on thinking carries no answer-token field at all, and rendering that absence as zero converts *we did not measure this* into *we measured none* — the more expensive error, because it looks like data. Every dimension a budget counts inherits that distinction or reports confident fictions.

## Rates Belong to the Provider, and They Carry a Date

Money is the dimension a person actually asks about, and it is the only one that is an estimate. Token counts come from the API. Prices come from a table someone typed, and a price table without a date is a promise to keep it current that nobody keeps.

Two rules keep it honest:

**A rate the table does not have is uncosted, never free.** A model whose price is unknown reports its token totals and a flag saying the money figure excludes it. Bucketing an unknown as zero produces a total that is confidently wrong, and nothing downstream can tell it from a cheap run.

**Cached and uncached tokens are different prices, and the difference is most of the bill.** OpenAI's prompt-caching guide prices a cache write at 1.25× the input rate and a read at **0.1×** on its newer models — the same pair :term[091: Agent/Caching]{href="./091_agent_caching.md"} records for Anthropic, which is either convergence or a figure that wants re-reading before either is trusted as a rate rather than a shape.

The shape is what matters here, and it is not in doubt: a schema-heavy prompt caches almost entirely. On `gpt-4o-mini`, 11,008 of 11,029 prompt tokens came back cached on a payload that was almost all schema; on `gpt-5.4-mini`, 5,376 of 5,970 on a different one. Those are two models and two probes and must not be read as one series. A budget carrying a single blended rate is wrong by close to an order of magnitude on exactly the runs that cost the most.

### The budget is the only thing that knows whether the next call will be cached

Because it holds the clock and the token count together. A provider's cache has a lifetime — OpenAI's in-memory entries stay warm through *"around 5 to 10 minutes of inactivity, up to one hour"* — so **a run that wakes after a long wait pays the uncached rate on a prefix it thinks it still owns.** Nothing else in the system can see that coming; the budget can, and can price a resumption before the model commits to sleeping through one.

### Holding a cache has a break-even, and it can be derived

Where a provider offers an explicit hold — Google's `CachedContent`, created with a `ttl` and extended through `caches.update`; OpenAI's `prompt_cache_retention` of `24h` — the hold is charged for storage over time while the rebuild is charged once on return. Those cross. Per token, holding pays while

```
storage_rate × hours  <  input_rate − cached_read_rate
```

which yields a **maximum profitable hold duration** that falls out of the rates rather than being configured. Past it, holding costs more than rebuilding.

A configured maximum is still worth having, but it belongs on top as a policy ceiling — *I do not care if it is profitable, do not park more than an hour* — not as the primary control. The derived number answers the question the model is actually asking, which is not *may I hold this* but **should I**.

## Warnings Are Prose, and Prose Has a Tier

A threshold that fires produces a sentence in the context: at three quarters spent, that the run should be converging; at the last turn, that it is the last turn. This is the right shape — the model already reads prose and obeys it, and a number in a corner it must interpret is worse than a sentence that says what to do.

But a threshold notice **changes**, so it renders in the derived tier and nowhere else. Injected into a frozen block it becomes the mistake :term[091: Agent/Caching]{href="./091_agent_caching.md"} documents at length: a moving byte inside a block declared still, invalidating everything behind it, silently, and doing so at exactly the moment the run is already expensive.

## The Last Turn Narrows the Room

The exhaustion notice already gets the announcement right, and generalising it to five dimensions is most of the work: the same message, naming which cap is closing rather than only the turn count.

What the notice cannot do on its own is stop the model spending the turn it is warning about. A model told *this is your last turn* and handed the same room it had before may spend that turn on a tool call, and then the run ends with nothing. So on the final turn the room narrows: **the tools are withdrawn and `output` is the only thing the model can answer with.** The instruction and the schema then say the same thing, which is the only arrangement that reliably survives contact with a model under pressure.

**And this breaks a rule stated two sections above, deliberately.** :term[Tool]{canonical="Tool"} declarations render in the frozen tier — they are the archetype of settled-before-turn-one — so withdrawing them rewrites the cached prefix on the run's largest prompt, which is exactly the mistake :term[091: Agent/Caching]{href="./091_agent_caching.md"} spends a chapter naming. The trade is stated rather than hidden: the cost is **one cache write on one request, the last one**, against a run that would otherwise return nothing at all. A rule worth breaking once, at a point where there is no next turn to pay for it, is not the same as a rule that does not apply.

A run that ends spent has still done work, and what it returns carries the ledger: which cap closed, on which dimension, and what was consumed reaching it. An exhausted run that reports only *limit exceeded* has thrown away the one artifact that would let a caller raise the right number.

## What a Provider's Author Has To Do

Declare the rates with a date, declare which thinking convention the API reports, and declare whether the provider can hold a cache and on what terms. Every one of these is a fact about a vendor rather than a preference, and each has been observed to change under a library that assumed otherwise.

The trap is the same one the feature maps already fell into once: a declaration nothing reads rots quietly, and a declaration that is silently ignored is worse than one that is refused. A budget asked to hold a cache on a provider that cannot must **say so, to the model**, rather than dropping the request. A run planning around a hold it does not have makes a worse decision than one told plainly it has none.

## Outro

A budget makes a run's cost legible to the run itself, and legibility is what turns a limit from an execution into a plan. Most of its dimensions can only be obeyed — a turn spent is spent, an hour gone is gone. Context is the exception: it is a level, and a run that is running out of it has something it can do about that, which is to decide what it still needs to remember.

That decision is :term[093: Agent/Compaction]{href="./093_agent_compaction.md"}.
