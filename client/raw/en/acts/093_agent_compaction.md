# 093: Agent/Compaction

> [!DEFINITION] [Compaction](./000_glossary.md)
> Releasing the account a run gives of its work while keeping the work itself — the narration stops being rendered, the writes it narrated still resolve.

> Sidenote:
>
> - Requires:
>   - :term[005: Agent/Data]{href="./005_agent_data.md"}
>   - :term[009: Agent/State]{href="./009_agent_state.md"}
>   - :term[091: Agent/Caching]{href="./091_agent_caching.md"}
> - Complemented by:
>   - :term[092: Agent/Limits]{href="./092_agent_limits.md"}
>   - :term[014: Agent/Delegate]{href="./014_agent_delegate.md"}

> [!HEADSUP] Heads up
> **This chapter is a design, not a description.** Nothing here ships: there is no compaction kind in the library and no `†recall`. What it rests on does ship — the append-only message list and the render tier of :term[091: Agent/Caching]{href="./091_agent_caching.md"} — and that separation is the point of the chapter.
>
> The number `093` is provisional, on the same footing as the two chapters before it.

Start with what does *not* grow, because it is more than a reader expects.

This agent keeps no conversation. :term[091: Agent/Caching]{href="./091_agent_caching.md"} opens on exactly that: a chat agent appends to a history, and this one renders current state instead. There is no assistant turn in the list, no reasoning from turn three sitting in turn nine, none of the chatter a chat transcript accumulates. :term[State]{canonical="State"} does not grow either — it folds. Ten writes to `†state.findings` render as one block holding the standing value.

Two things do grow, and they are narrower and more specific than "the account of the work".

**The failure log.** `Content` names the tier honestly — *grows, never mutates* — and that is what it does: an :term[Error Message]{canonical="Error Message"} for a call that threw, one for a call never unblocked, one for a spent budget, each appended and none ever removed. A failure resolved on the next turn is still sent on every turn after it.

**Accumulating writes.** Where :term[Output]{canonical="Output"} was filed with `push`, every write stays in the list because the sequence *is* the value. A question answered nine times carries nine answers.

So the thing to release is not "the telling" in general. It is a resolved failure nobody will read again, and it is far more specific than a first look suggests.

## The Rule

**Release the telling. Keep what was told.**

A compacted span stops being rendered. It is not deleted, and nothing about it is rewritten.

## Why That Is Safe

Because there are two walks over the list, and they are not the same walk.

:term[091: Agent/Caching]{href="./091_agent_caching.md"} established the half that matters: the underlying message list is append-only, and only the rendering folds. The loop holds one canonical list; every `Var.resolve` runs against it; rendering is a transformation that produces what goes on the wire and leaves the list alone.

So a message the renderer declines to emit is still a message the resolver walks. **The model stops reading a write; the run does not stop knowing it.**

And the withholding needs no new machinery, though it is not where a reader would first look for it. The render **tier** cannot do it: `render` is a sort key, and sorting a block to the end is not the same as leaving it out. What can do it is a **content handler**, which already returns the message list it wants rendered — `Plan` withholds every superseded plan exactly this way, by filtering them out of the list it returns. Compaction is that move made general, and it belongs at the same seam: **over kind-bearing content, before the fold**, where a message still knows what it is.

Deleting instead would be a different act with a different name, and a worse one. A run that forgets what it did cannot answer for it.

## What Releases Freely

**Superseded plans — and they already do.** Worth stating plainly rather than claiming as a saving: the `plan` handler drops every plan a later turn replaced, today, before this chapter existed. Only the standing plan renders. It is named here because it is the **worked example of the whole mechanism** — a handler withholding a message from the render while the message stays in the list — not because compaction adds it.

**A resolved failure, past a given age.** A call that threw on turn two and succeeded on turn three leaves an :term[Error Message]{canonical="Error Message"} that renders forever. Age is the property the loop can evaluate without asking anyone: `turns` is in scope at every point the loop appends, so a message can be stamped with the turn that produced it and released some distance behind the front.

## Two Different Acts, and Only One Of Them Is Safe

Everything above is **demotion**: the message stays in the list, a handler stops emitting it, and `Var.resolve` never notices. The run's knowledge is safe by construction, because the resolver reads the list rather than the render.

**What is not safe is demoting a :term[Data]{canonical="Data"} message, and the reason is worth stating exactly.** :term[005: Agent/Data]{href="./005_agent_data.md"} folds every message of one `kind` into a single block and gives that block the *minimum* tier over its members. So the tier is an **output** of the fold, not a filter on its input. Lower one member's tier and two things happen, neither of them the intended one: the fold still consumes the message, so its value renders anyway — and the whole folded block drops into the volatile tail, which costs cache on everything folded beside it. **Demoting state saves nothing and makes caching worse.**

The rule that follows: **demotion applies to messages that render on their own, never to a member of a fold.** A plan, a failure, a standing instruction — yes. A `data` write — no, and the way to shrink a fold is to remove from it, which is the other act.

**Removal** is the other act, and it is not a more aggressive setting of the first one. Taking a message out of the list changes what resolves — and it does so in **three** ways, not one, which matters because only the first is obvious enough to guard against:

- **An accumulating write.** Where :term[Output]{canonical="Output"} was filed with `push`, the sequence *is* the value: `Var.resolve` replays oldest-first, so dropping an older write does not shorten a history, it **rewrites** one. The standing position on a :term[Question]{href="./018_agent_question.md"} is the clearest case — lose the third of five answers and what remains is coherent and wrong.
- **An erase.** Dropping the write that cleared a path **resurrects** the value it cleared. The run reads a fact it had explicitly retracted.
- **A `set`.** Dropping one does not leave the path empty; resolution simply continues to the write beneath, so the path **re-grounds on an older value**. And where the dropped `set` belonged to an :term[Instance]{canonical="Instance"}, the walk falls through to the shared layer and the **global value is promoted into the instance** — the instance silently inherits something it had overridden.

All three are silent, and nothing downstream can tell a rewritten history from a real one.

So the two must not share a name or a setting. Demotion is the default and needs no ceremony. Removal is rare, deliberate, and owes the account below.

## When The Run Must Choose

Demotion is bounded. Eventually a run is large because its **state** is large, and demoting state saves nothing a model needs — it only blinds the model to what it knows. Then the only remaining move is removal: deciding what the run no longer needs to remember at all.

It is model-authored, because nothing else knows what the run is for. It is lossy in a direction nothing can undo. And it is the one place in the library where a mechanism can quietly destroy a fact and leave no evidence — which is why the discipline is not *may it drop things* but **what does it owe when it does**:

> A release names what it let go and why, and the naming survives the release.

The summary the model writes accrues at `†recall`, and so does the account of what it dropped to write it. A compaction that leaves no trace of its own losses is indistinguishable from a bug, and the first time a run gives a confidently wrong answer nobody will be able to tell which of the two it was.

The distinction is the same one that runs through :term[092: Agent/Limits]{href="./092_agent_limits.md"}: a thing the system computes is a fact, and a thing the model reports is a claim. A released span is a claim about what mattered.

## What Sets It Off

Context is the one budgeted dimension that falls as well as rises. Turns, clock and tokens only accumulate, and a threshold on them can only mean *you are running out*. A threshold on context means *there is something you can do*, and this is the something.

So the trigger belongs to the budget and the response belongs here. Neither is useful alone — a gauge nobody acts on, or a tool nobody calls.

> Sidenote:
>
> Compaction and delegate resumption are one mechanism pointed at two lists. :term[014: Agent/Delegate]{href="./014_agent_delegate.md"} builds a clean room from its definition, the scoped parent slice and the call's parameters, then discards it: `Delegate.context` composes an opening context, never a saved one. A resumable delegate is a list that survives instead of being discarded — and *what* survives is exactly the question answered above. Whatever the answer, the scope is re-granted on every resumption rather than persisted, or the clean room accumulates the parent across calls and the isolation it exists for is gone.

## Outro

The fold made this agent legible; the tiering made it affordable to send. Compaction is the third of the same idea: the list a run keeps and the context a model reads were never obliged to be the same thing, and once they are allowed to differ, a run can work for a long time without its prompt growing in proportion to how long it has worked.

What it may not do is forget quietly. Everything released leaves a record of having been released, which is the only property that keeps a shrinking context distinguishable from a failing one.

That a run can now work far longer than its context would once have allowed raises the question of what it is working *toward*, and :term[101: Concept/Idea]{href="./101_concept_idea.md"} takes it up.
