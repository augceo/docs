# 096: Agent/Hooks

> [!DEFINITION] [Hook](./000_glossary.md)
> A registered function the runtime calls at a named seam. Two verbs: an **interceptor** sits in the path and may replace what flows through it, so its failure is the operation's failure; an **observer** sits beside the path and may change nothing, so its failure is its own.

> Sidenote:
>
> - Requires:
>   - :term[001: Agent/Request]{href="./001_agent_request.md"}
>   - :term[010: Agent/Loop]{href="./010_agent_loop.md"}
> - Complemented by:
>   - :term[011: Agent/Expressions]{href="./011_agent_expressions.md"}
>   - :term[092: Agent/Limits]{href="./092_agent_limits.md"}
>   - :term[108: Concept/Visibility]{href="./108_concept_visibility.md"}

> [!HEADSUP] Heads up
> The number `096` is provisional. The `09x` range holds cross-cutting concerns and the author reindexes; treat the file name as a placeholder and the content as settled. Where this sits in the reading order is a reindexing call, since the chapter before it also hands off to :term[101: Concept/Idea]{href="./101_concept_idea.md"}.

A host has two entirely different reasons to reach into a run, and only one of them is dangerous.

It reaches in to **change** something: put the API key on the outgoing request, rewrite a setting, redact a payload before it leaves the process. That reach has to be able to fail loudly — a key that failed to arrive must stop the request rather than send it unauthenticated.

It also reaches in to **watch**: draw the run, meter what it spent, log which branch was taken. That reach is added liberally, from code that has no business affecting what it is watching, and a bug in it must not be able to end a run.

Collapsing those into one mechanism costs one of the two properties. Either the watcher gains the power to corrupt what it watches, or the changer loses the power to change — which is the one thing it exists for.

## The Rule

**Two verbs over one registration.** Registering, removing, and reading the live list are identical for both; the difference is what the runtime does with what comes back.

- An **interceptor** receives the arguments and returns them. It may replace any of them. **A throw propagates** and fails the operation.
- An **observer** receives the arguments and returns nothing. There is no channel a replacement could come back on. **A throw is swallowed** and the operation continues.

The asymmetry is the whole design, and it has to be legible at the call site rather than in a comment. A reader who sees the verb knows the power.

Both return the function they registered, so a hook written inline is still removable.

> Sidenote:
>
> Registration is the part that was written three times before there was one primitive — once for request interception, once for usage reporting, once again for anything else that wanted a list of callbacks. The bug that motivates a shared implementation is small and repeatable: removing a hook that is not registered splices the last one instead.

## The Seams

- **The outgoing request** takes interceptors, because that is the seam where something must be *changed* — a credential injected, a body rewritten, a setting overridden for one call.
- **Usage reporting** takes observers. What a turn cost is a fact about the turn; a meter cannot be permitted to alter it, and a meter that throws must not take the run with it.
- **The loop** takes observers. Once per tick that ruled anything out, it says which arms of a disjunctive :term[Output Path]{canonical="Output Path"} received a value, which a sibling's write ruled out for good, and which :term[Calls]{canonical="Call"} are waiting on an arm nobody took.
- **The expression evaluator** takes observers, and this is the seam that did not exist before. It reports which arm of a `<|>` supplied the value, and — separately — why a :term[Call]{canonical="Call"} became ready when it did.

The loop's reading is not new information the runtime went and computed. It has always derived which arm was ruled out, because that is how a :term[Call]{canonical="Call"} stranded on an untaken arm is excused rather than reported as a failure. What changed is that the reading is said out loud instead of dropped on the floor.

## What the Evaluator May Report, and What It May Not

An observer's payload is a vocabulary, and a vocabulary teaches. So this one is constrained by what the operators actually mean:

**`<|>` is left-preferred, not a race.** The reported fact is therefore *which arm supplied the value* — never which arrived first, because nothing records an arrival order and the operator would not read it if it did. The words *winner*, *race*, *first*, *loser* and *discard* are absent from the payload by design, not by omission.

**There is no cancellation.** When the left answers, the right is not evaluated at all, and the honest word for that is *unread*: nothing was rescinded and nothing was thrown away. Where a losing arm did run, its work continues and only its value stops travelling.

**The speculation lives in readiness**, which is why readiness is reported as its own thing. A :term[Call]{canonical="Call"} goes as soon as either side of an alternative has landed; the value it then reads is decided separately. Those are two questions and a payload that fused them would teach one wrong.

**Arm labels are in the author's own words.** By the time an expression runs, every :term[Variable Reference]{canonical="Variable Reference"} in it has been rewritten to a generated identifier. A report naming those identifies nothing a person can find, so the original text is restored before anyone reads it.

> Sidenote:
>
> Readiness fires **per ask, not per transition.** The drain asks the same question many times a turn and gets the same answer while the values stand. The evaluator holds no per-call memory, so it cannot tell a repeat from a change; remembering the last answer belongs to whoever is drawing.

## Watching Must Be Free When Nobody Watches

An observer at the request seam fires once a turn. An observer at the evaluator fires per expression resolved and per readiness asked, which is orders of magnitude more. **A visibility channel that costs something when unused is a channel people turn off**, and then it is not a channel.

So: the notification returns before allocating anything when nothing is registered, and every payload is built only after the registry says somebody is listening. Some of what a report needs is work the runtime otherwise skips — deciding *which* side of an alternative is satisfied means asking both, which is exactly what a short-circuit avoids — and that work is done only under observation.

The falsifier for the claim is worth stating because it is the only one that actually bites: register an observer *after* a batch of evaluations and the log must be empty. That is only true if nothing was constructed and queued while unobserved.

## Why a Side Channel Rather Than a Stamp on the Value

The obvious alternative is to record provenance on the :term[Call]{canonical="Call"} itself — mark which arm answered, and let anyone who wants it read the object.

It is the expensive one. A stamp is a schema slot that has to be serialized, versioned, and explained to a model reading its own context; it changes what a caller receives; and it reopens a decision this architecture has already taken deliberately, which is that resolution mutates nothing and stores nothing.

**An observer never touches the value.** The object a caller receives is identical whether or not anyone is watching. Nothing new is written into the run's own data, no return contract moves, and the whole channel is removable: unregister the observer and the runtime is exactly what it was.

The instinct behind the stamp is right — the information exists and was being discarded. Only the delivery was wrong.

## Outro

A run that can be watched without being disturbed is a run that can be shown to somebody: drawn as it happens, metered as it spends, and explained after the fact in terms of the decisions it actually took rather than the ones it appeared to.

That is the last thing the runtime owed the outside, and :term[101: Concept/Idea]{href="./101_concept_idea.md"} returns to what such a run is for.
