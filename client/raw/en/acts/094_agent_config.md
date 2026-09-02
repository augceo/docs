# 094: Agent/Config

> [!DEFINITION] [Config](./000_glossary.md)
> The settings a run executes under — model, provider, thinking budget, sampling — shown to the model as a message, and changeable by it within a declared allowance.

> Sidenote:
>
> - Requires:
>   - :term[001: Agent/Request]{href="./001_agent_request.md"}
>   - :term[005: Agent/Data]{href="./005_agent_data.md"}
> - Complemented by:
>   - :term[092: Agent/Limits]{href="./092_agent_limits.md"}
>   - :term[091: Agent/Caching]{href="./091_agent_caching.md"}

> [!HEADSUP] Heads up
> **This chapter is a design, not a description.** There is no `config` content kind in the library. What it rests on ships: the settings are already a schema, the handler tuple already carries the config, and the loop already never rewrites it.
>
> The number `094` is provisional, on the same footing as the three chapters before it.

A run executes under settings it cannot see. Which model is answering, how much thinking it is funded for, how deterministic it is asked to be — all decided by the host before turn one, all invisible to the party they govern. So a model plans as though the budget were the one it imagines, and a model that has failed twice on a hard step has no way to say *this needs a stronger model* even when that is the correct next move.

The settings themselves are not the problem. They are already better shaped for this than anything else in the library.

## What Already Exists

**They are already a schema.** Each provider declares its own `ConfigProperties` — `model` with its enum of live ids, `thinkingBudget` with its range, `temperature`, `topP`, `topK`, `maxTokens` — with types, bounds and defaults. A subset of that table is a valid response-schema fragment as it stands. Nothing needs authoring to show a model what it may change; the description of the settings already exists, because the host needed one.

**The handler tuple already carries the config.** A content handler receives `[config, schema, messages, callback]` and returns the same shape — the tuple's own comment calls these *the arguments handlers get to mutate*. Every handler passes the config through untouched today, and none is obliged to.

**The loop never rewrites it.** The config is built once and handed to each request; the loop holds no mutable settings state to fight with.

So the mechanism is not the work. The **decisions** are.

## The Rule

**Opt-in by presence, and refusable by the host.**

A run with no :term[Config Message]{canonical="Config Message"} in its context behaves exactly as it does now: the model is shown nothing and can change nothing. Send one, and the feature exists — the same idiom by which a :term[Question]{href="./018_agent_question.md"} exists only when asked and a :term[Tool]{canonical="Tool"} exists only when declared. Nothing here is a global capability; it is a message a caller chose to include.

And a request is a request. The host grants, narrows, or refuses, and the model is told which — an unfulfilled change reported as a fact rather than silently dropped, because a run planning around a stronger model it did not get makes worse decisions than one told plainly it is still on the old one.

## Changing a Setting Is Calling a Tool

Act 018 already made this move and it is the right one here: a :term[Question]{href="./018_agent_question.md"} owns no root property, no carrier and no message shape of its own — its handler replaces its own message with a `tool` message, and answering is calling that tool. A Config Message does the same. It emits two things and keeps nothing:

- a **rendered block** of the settings as they stand, and
- a **Tool** whose schema is exactly the settings this run may change.

The consequences are all in the model's favour. A change becomes a discrete, auditable event rather than a field the model must fill on every turn. It can be planned, gated behind an approval, or made conditional, because a :term[Call]{canonical="Call"} already composes with all of those. And it costs nothing on turns where nothing changes — where a root property would occupy a named slot in **every** response schema, and named slots are the one thing measured to displace a cached prefix.

**The tool is activity-backed, not latent, and that is the load-bearing choice.** A latent call would mean the model computes the new value and the value simply becomes true — no seam at which anyone could say no. Routing it through an :term[Activity]{canonical="Activity"} under a reserved name, as :term[014: Agent/Delegate]{href="./014_agent_delegate.md"} does for delegation, puts the host at the decision. What the activity returns is what was **granted**, which is not the same object as what was asked, and filing it at the call's :term[Output Path]{canonical="Output Path"} makes the difference between the two the audit trail. No second mechanism records it.

## Three Tiers of Visibility, and Only One of Them Is Marked

The settings table does not divide itself. It holds the credential beside the sampling temperature, and a design that says *show the config* without saying which config puts an API key into a schema — on a provider where schema prose is free, unbounded, and read end to end.

**Hidden.** `apiKey`, `endpoint`. Absent from the rendered block and absent from the tool: not shown, not mentioned, not named. They are withheld not because the model would misuse them but because a secret's blast radius is every place the schema travels, including the logs of whoever is debugging it.

**Read-only.** Rendered in the block, absent from the tool. `stream`, `callbackPath`, `n` — transport shape the model has no useful view on, but knowing it is not streaming is worth something to a model deciding how to answer. **Nothing marks these read-only.** They are read-only because they are not in the writable schema, which is the only kind of read-only worth having: a lever that is described as forbidden is a lever, and a model under pressure will try it.

**Writable.** In the block and in the tool. `model`, `provider`, `thinkingBudget`, `temperature`, `topP`, `topK`, `maxTokens` — each changes something the model is positioned to judge: how hard the current step is, how much determinism it wants, whether it is on the right engine for what it just failed at.

## The Caller Names Them, in the Shape Every Other Kind Uses

Membership of the writable tier is not a property of the property. A caller enumerates what **this run** may touch, and the enumeration becomes the tool's schema — so the same `thinkingBudget` is writable in one run and read-only in the next, with no flag anywhere claiming otherwise.

A list would be the wrong shape, and the reason is worth stating because it is the same reason the tool exists. `checklist` and `validations` take a list because their members are independent items whose names are incidental — a `$id` bolted on when one is wanted. A setting's name is not incidental: it **is** the thing being addressed, and it has to match the provider's own key exactly or nothing resolves. So the allowance is a map, keyed by the name, and it happens to be the same shape as the `properties` object it becomes:

```typescript
{ type: 'config', config: {
    thinkingBudget: true,                    // the provider's own entry, unchanged
    model: true,
    temperature: { maximum: 0.3 },           // the provider's entry, narrowed
    verbosity: {                             // a setting the provider never had
      type: 'string', enum: ['terse', 'full'],
      description: 'How much explanation to include.',
    },
} }
```

**`true` costs nothing to expose, because the schema for it already exists.** Each provider's `ConfigProperties` declares `model` with its enum of live ids, `thinkingBudget` with its range, `temperature` with its bounds — types, limits and defaults, authored because the host needed them. Naming a property pulls that entry through as it stands, so the model is offered the real range rather than a hand-copied one that drifts out of date.

**A partial schema narrows what the provider allows.** `temperature: { maximum: 0.3 }` merges over the provider's entry rather than replacing it: the type, the minimum and the default all survive, and the ceiling comes down. This is the capability a list cannot express cleanly, and it is the one a cautious caller reaches for first — *you may tune this, within these bounds*, rather than the all-or-nothing a name alone offers.

**A full schema under an unknown name adds a setting the provider never had.** A host with its own runtime knobs — a verbosity mode, a house style, a feature flag — exposes them through the same map, in the same tool, under the same refusal. Nothing about this mechanism is provider-specific except the properties that happen to come from a provider.

**The map form also gives the types away for free.** `keyof` the map is the set of writable names, so a call's shape follows from the allowance itself rather than being restated: a mapped type over those keys, resolving each through `FromSchema` against the provider's entry, yields a call typed to the real values — `model` narrowed to the live-id union, `thinkingBudget` to a number. Measured on the shipped tables: an id outside the enum fails to compile.

Two edges worth stating rather than discovering. A name that no provider arm declares is an authoring error and should fail where it is written, not silently vanish from the tool. And because `provider` decides which other properties exist, a run that makes `provider` writable is offering a tool whose *other* fields are conditional on it — which is an argument for keeping `provider` out of the writable tier unless a caller has a specific reason, rather than a reason it cannot be there.

## Provider Is Not Just Another Property

`Provider.Schema` is a discriminated union keyed on `provider`, and each arm brings its own `ConfigProperties`. So the provider is the property that decides **which other properties exist** — change it and the valid settings change with it, along with the schema dialect, the rates, the thinking convention, and the cache.

That has three consequences worth stating rather than discovering:

- **A settings change spanning providers must be atomic.** Half-applying it leaves a config describing one provider with properties from another.
- **A provider switch is always a cold cache write**, whatever the prefix looked like — a different endpoint has never seen it. :term[092: Agent/Limits]{href="./092_agent_limits.md"} is the component that can price that before the model commits to it.
- **What the model may change is provider-discriminated too.** The offered schema is not a fixed list; it is that provider's arm, narrowed by the caller's allowance.

## Read Before Write

The two halves have different risk and different value, and they do not have to ship together.

**Reading is nearly free and useful alone.** A Config Message with an empty map emits the block and no tool: the settings become visible and nothing becomes changeable. Read-only needs no vocabulary of its own — a property is read-only exactly when the map does not name it. That block is static, so it belongs in the frozen tier and costs no cache. A model that knows its thinking budget is 128 rather than 32,000 plans differently, and better, without being able to change a thing.

**Writing is the design.** It needs the allowance, the refusal, an audit of what was asked against what was granted, and a ceiling — because `thinkingBudget` and `model` are both spend levers, and a model that can raise its own is a model that can spend without bound. The channel and the cap are two halves of one conversation: a budget over settings the model cannot influence is a cap on nothing it does, and a channel with no budget is an open tab.

## Where the Change Lives

Not in a variable. The loop's config is built once and never reassigned, and that is worth preserving rather than working around.

A granted change lands as a message, and every subsequent turn's handler applies it from there. The settings then live where :term[State]{canonical="State"} lives — in the append-only list, resolved by walking it — so a run's settings history is readable after the fact by the same means as everything else it did, and the audit of what was asked and granted is not a second mechanism.

## What This Absorbs

Two things that looked like features are instances of it.

**A cache hold across a wait** is a settings request: the model declares it is about to be idle and asks for its prefix to be retained. What it costs and whether it pays is arithmetic :term[092: Agent/Limits]{href="./092_agent_limits.md"} already owns.

**The reactive half of retry.** A standing rule — *retry this tool twice with backoff* — is the author's, and belongs where the tool is declared. But *this failed twice, put me on a stronger model* is a judgment about a situation the model can see and the author could not, and it is an ordinary settings request.

The half that does not belong here is standing policy. A model handed a lever over behaviour whose consequences it cannot observe will pull it, and the results will be attributed to the wrong thing.

## Outro

Every other chapter gives the model a way to say something about the work. This one gives it a way to say something about the conditions the work runs under — and it is the one place where the answer is legitimately *no*, because the host is paying.

That the settings turn out to be a schema, the tuple turns out to be mutable, and the loop turns out to hold nothing it would have to give up is not luck. It is what a design looks like when the pieces were built to compose, and :term[101: Concept/Idea]{href="./101_concept_idea.md"} returns to what they are composing.
