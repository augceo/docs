# 095: Agent/Claims

> [!DEFINITION] [Claim](./000_glossary.md)
> A named expression declared once for a run: its value lands at `†expr.<name>` and is read like any other :term[Variable Reference]{canonical="Variable Reference"}, and the :term[Call]{canonical="Call"} that reads it waits until that value is true.

> Sidenote:
>
> - Requires:
>   - :term[007: Agent/Variables]{href="./007_agent_variables.md"}
>   - :term[010: Agent/Loop]{href="./010_agent_loop.md"}
>   - :term[011: Agent/Expressions]{href="./011_agent_expressions.md"}
>   - :term[012: Agent/Plan]{href="./012_agent_plan.md"}
> - Complemented by:
>   - :term[013: Agent/Instancing]{href="./013_agent_instancing.md"}
>   - :term[009: Agent/State]{href="./009_agent_state.md"}

> [!HEADSUP] Heads up
> **One section here is a design rather than a description: freezing.** Nothing in the library freezes anything, so a claim frozen together with the verdict it produced has nowhere to be kept, and that section says what such a store would have to hold rather than what it holds. Everything else runs: the `expressions` content kind, the declaration walk, `†expr.<name>` through the ordinary resolver, `_when`, the gate, the reporting, and the retrospective check.
>
> The number `095` is provisional, on the same footing as the chapters before it.

A run can already say *when* a step may go: a :term[Call]{canonical="Call"} waits until its :term[Variable References]{canonical="Variable Reference"} resolve. What it cannot say is *whether what arrived is any good*.

So a plan that needs to branch on a **value** — this document is a policy, that one is an endorsement — cannot branch at all. It has to stop, hand the value to the model, and spend a turn on a decision a machine could have taken. The branch is not hard. It is simply unsayable.

## The Rule

**A claim extends readiness from *resolves* to *resolves and is true*.**

:term[011: Agent/Expressions]{href="./011_agent_expressions.md"} owns the grammar — what an expression is, the positions that admit one, and why its operators are not the language's own. What follows is the half that outlives a single call: an expression given a name, kept for the length of the run, evaluated wherever it is read, and checked whether or not anybody asked.

## What a Claim Is

A name and an expression, in a message the run carries:

```jsonc
{ "type": "expressions", "expressions": [
    { "name": "isPolicy",   "expression": "†state.kind === 'policy'" },
    { "name": "wellFormed",
      "expression": "†state.result?.effectiveDate != null && †state.result.limits > 0",
      "description": "a result carries an effective date and a positive limit" }
] }
```

Naming buys the thing an inline condition cannot have: **a failure with an identity.** *`wellFormed` did not hold for instance ②* is a fact a model can act on. An anonymous condition that did not hold is a fact about nothing.

## Reading a Claim Is Naming It

A claim's value lands at `†expr.<name>`, and that is an ordinary Variable Reference — the resolver answers it exactly as it answers `†state.total`. There is no registry, no roster of names on the call, and no second lookup path to keep in step with the first.

```jsonc
{ "_tool": "summarize", "of": "†state.result", "_when": "†expr.wellFormed" }
```

**A call that reads a claim both takes its value and waits for it to be true.** That is what makes two calls reading opposite claims a branch, and it is why nothing on the call enumerates them: reading is naming.

**Which is also why a negation cannot be the other arm.** `_when: "!†expr.wellFormed"` reads `wellFormed`, and reading it means waiting for it to become true — so the arm meant to run when the result is malformed is held by exactly the condition that should release it, and never runs. Nothing reports this: the claim was read, so it is not an unreferenced claim, and the call is waiting rather than failing. **Two calls branching on one claim want two claims**, each true in its own case, each arm reading the one that names it.

It also composes past the gate. `†expr.total * 10` is a parameter computing on a claim's value, because a claim is a value first and a gate second.

> Sidenote:
>
> The value and the gate are one reading, which has an edge worth knowing before it bites: a claim whose value is `0` supplies `0` to the parameter **and holds the call**, because zero is not true. A claim is how a run decides. A count belongs in :term[State]{canonical="State"}.

## `_when`, for a Condition Used Once

A condition exactly one call needs does not need a name. `_when` carries it inline:

```jsonc
{ "_tool": "summarize", "of": "†state.result", "_when": "†state.result.limits > 0" }
```

**It tests truthiness, not existence** — the whole difference between a condition and a reference. A `_when` resolving to `false` holds the call rather than letting it through, and a later turn may let that same call go once the value moves.

**There is a second reason to write a condition inline, and it has nothing to do with reuse: a claim a turn declares does not gate that turn's own calls.** Declarations reach the context the way a plan does — through the carrier, which runs after the turn's calls have drained. So while those calls are being considered the name has never been heard, and a name nothing declared behaves as absent, which is deliberately permissive. Two arms branching on a claim declared in the same breath **both run**, and nothing reports it.

**A branch on a value produced this turn is written inline.** A named claim is for a condition already standing when the turn begins — declared by the host, or by a turn before this one.

## An Array, Because a Model Cannot Author a Key

The obvious shape is a map keyed by name. It cannot be used, for a reason that has nothing to do with taste: **under strict structured output every property is declared in advance and no others are admitted**, so a key the model invents cannot be emitted at all. The name therefore lives in value position, where a model may write it.

This is worth stating as a general rule rather than a local workaround, because the same question gets the opposite answer elsewhere in the library: a settings allowance authored by a **host** is rightly a map, since its keys are known before the schema is built. **Map or array is decided by who authors it.**

The cost of an array is that uniqueness stops being structural, so a rule is owed — and the library has already made it twice. **A later declaration of a name replaces the earlier one**, exactly as a :term[Plan]{canonical="Plan"} persists until a turn replaces it and a `set` supersedes the write beneath it. So a model redeclaring a name with a different expression changes that claim rather than colliding with itself.

**Where a claim parts company with a plan is the omission.** A call left out of a turn is dropped from the plan. A claim left out is dropped from nothing: declarations accumulate, oldest to newest, and a name nobody restated keeps the expression it already had. So a turn emits only what is new or changed, and re-emitting the whole list every turn buys nothing but payload.

**Nothing retires a claim.** Once declared, a name is checked for the rest of the run, and the nearest thing to withdrawing one is redeclaring it with an expression that always holds. Said plainly because the absence is the trap: a model that believed omission retired a claim would leave it standing, still gating whatever reads it, with nothing anywhere reporting the mistake.

## Global by Declaration, Per Instance by Evaluation

**One declaration, evaluated where it is used.** `wellFormed` is written once for the run, and a claim reading `†state.result` reads *each instance's own* result — instance ②'s when instance ②'s call is being considered, instance ③'s when it is ③'s turn. The declaration is not copied per :term[Instance]{canonical="Instance"} and does not have to be. It is aimed at the moment it is read.

**This needs no new scoping**, because the aiming is the resolver's ordinary walk. A check carries the instance it is being made for, the declarations in force are gathered through the same layer walk every read uses, and the expression is then computed against that instance's values. Layers go most specific first, so an instance's own declaration of a name replaces the shared one however old that one is.

The shape is the one :term[012: Agent/Plan]{href="./012_agent_plan.md"} already established, one level up. A plan says nothing about instances — the word does not appear in it — and it still fans out, because the aiming rides on each **call**. A claim block is instance-blind in exactly the same way, and the aiming rides on each **reading**. Neither declaration knows which instance it serves, and that is why one of each serves all of them.

A host that wants a claim to mean one instance and no other pins the block to it, and that pinned declaration beats the global of the same name by the same specificity walk.

> Sidenote:
>
> A **model** cannot write that narrowing today. The response property it fills carries a name, an expression and a description, so what comes back is filed as a declaration of the whole run. This is a smaller gap than it sounds: the per-instance reading is the behaviour a model wants in nearly every case, and pinning is a host's tool for the exception.

## One Format in Every Position

What a model emits, what renders back to it, and what an author writes by hand are the **same shape**. :term[012: Agent/Plan]{href="./012_agent_plan.md"} established this: a turn's calls come back as the next turn's plan unchanged, with nothing in between to translate them.

Claims inherit it, and the payoff is that a run can **keep what it learned**. A claim a model wrote in turn three is carried forward like a plan, renders every turn after, and can be lifted verbatim into a later run's authored context. There is no conversion step, so there is no place for the forms to drift apart.

> Sidenote:
>
> The rendered block sits in the **derived** tier rather than the frozen one, and :term[091: Agent/Caching]{href="./091_agent_caching.md"} is the reason. A model re-emitting a claim with a new expression rewrites that block in place; seated ahead of the plan, one edited expression would invalidate the cached prefix of everything after it.

## How a Claim Is Evaluated

**References are resolved before the expression runs, and the expression never sees one.** Each `†path` is rewritten to a generated identifier and the resolved values are passed as arguments. Not substituted into the text — a value spliced into source breaks on objects and quoting, and is an injection surface besides.

Each reference arrives as a promise and is awaited once, in a generated prologue, rather than at each place it is used. An `await` spliced into a nested callback — `†state.items.filter(i => i.id === †state.target)` — is a syntax error, and awaiting at the top is also what keeps `†state.kind === 'policy'` comparing the value rather than silently comparing a promise.

What is left is ordinary JavaScript, so array methods, optional chaining and comparison work without having been designed for. Claims are independent, so they are evaluated in parallel.

> Sidenote:
>
> A claim cannot read another claim. Every one is evaluated against the context beneath the block their values land in, so `†expr.other` inside a claim's own expression resolves to nothing, and nothing warns. Nesting would need an evaluation order nothing declares.

### What the Evaluator Owes

**A timeout**, five seconds by default and overridable per evaluation. An expression may `await`, and one waiting on something that never settles would otherwise hold the loop open — the single outcome the failure rule exists to prevent. The ceiling is chosen against what a claim *is* rather than against what a network call costs: it reads values the run already holds, so a well-behaved one finishes in microseconds and the bound never binds. A claim that hits it is **abandoned rather than cancelled** — the verdict comes back on time and the work goes on running.

**A ceiling on concurrency**, likewise overridable. Evaluating in parallel is right; evaluating without a bound hands a model-authored expression a fan-out nobody declared. The bound is across the whole scan rather than per call, because a turn authors many calls and each reads few.

**An acknowledgement that determinism is gone.** An expression that reaches outside the run answers differently at different times. That is a real cost of the expressiveness, and it lands on freezing.

### Failure Is Not a Verdict

**A claim that throws, times out, will not compile, or names something nothing declared behaves as though it were absent — the call is not held — and says so explicitly.**

Broken is not a third kind of failure. It is the *absence of an answer*: an expression nobody could evaluate says nothing about the value, so it holds nothing back. Only a claim that was checked and did not hold gates anything.

This is the library's existing posture rather than a new leniency. A carrier handed a value it cannot read returns nothing rather than erroring; a malformed output method is read as the default; an unrecognised format is an annotation rather than a failure. In each case the machinery declines and reports instead of ruling.

It also settles the failure mode that would otherwise make this whole chapter dangerous: **a broken claim can never deadlock a graph.** The worst it can do is fail to hold something back, which is where the run already was.

The rule is doing more work than it appears, because a name a call reads **cannot be constrained to the claims that exist** — they are authored in the same turn as the call that reads them. A read of a claim nobody declared is not an error condition to design around; it is the same case as one that threw.

What is reported is an :term[Error Message]{canonical="Error Message"} the next :term[Request]{canonical="Request"} reads, once per run rather than once per scan, so a claim that keeps failing states its fact once instead of filling the context with it. A call a claim is *holding* is not reported at all: it is waiting, exactly as the arm of a branch the run did not take is waiting. The loop cannot yet tell waiting from impossible, so a claim that can never hold strands its call and the run ends without saying so.

### Referenced by Nothing, Still Checked

A claim is a declaration rather than an instruction, so one no call read is evaluated anyway at the end of the turn, and one that stopped holding is reported. **One declaration, two uses** — *every result carries an effective date* can be a standing invariant of the run and, in the same breath, the thing holding one step back.

## A Branch That Costs No Turn

```jsonc
{ "calls": [
  { "_tool": "extractCoverages",   "doc": "†state.raw",
    "_outputPath": "†state.result", "_when": "†expr.isPolicy" },
  { "_tool": "extractEndorsement", "doc": "†state.raw",
    "_outputPath": "†state.result", "_when": "†expr.isEndorsement" }
] }
```

Both arms are authored in one turn, over claims **already in force** when that turn began. They cannot both hold, so the loop dispatches one and leaves the other waiting on a value that will never arrive — the same shape as the untaken arm of an alternative, reached through the value instead of through the :term[Output Path]{canonical="Output Path"}.

**The decision was authored in the turn that authored the plan**, so taking it costs nothing. Deciding it any other way costs a full turn per document, because somebody has to look at `†state.kind` to choose.

## Speculation Belongs Inside One Expression

The same declaration does a different job as confidence drops. Confident: one call, one claim, and it goes when the value is good. Uncertain: two calls whose claims cannot both hold, which is the branch above. Genuinely balanced, where waiting costs more than tokens: one call whose expression **races** two futures and takes whichever answers first.

The last of those is speculation, and where it is written decides how much machinery it needs.

**Dispatching two calls and selecting a winner afterwards would mean two writes to one path.** There is no cancellation, so both arms land, and removing the loser's write is not the mirror of making it: dropping a write can resurrect a value that was cleared, re-ground a path on an older base, or promote a shared value into an instance that had overridden it. Each of those is non-monotone, and none is visible at the point the write is dropped.

A race inside a single expression has none of that shape. **One expression, one value, one write** — so there is no `erase`, no rescinding machinery, and nothing to un-write. It is a line of ordinary JavaScript rather than a mechanism.

> Sidenote:
>
> A race does not cancel what loses, and should not: that is what a race means in the language this is written in, so it composes with what an author already knows. **Only the value stops travelling.** A losing arm keeps running, and one that awaited a side-effecting tool has still called it. That is inherent to the choice rather than an oversight, and a cancelling form is a separate operator for whoever needs one.

What a race can discriminate between today is whatever the expression itself awaits — not its references. A run's own values are read from one snapshot before the expression body runs, so a race over two references is already settled the moment it is written. That is still one value and one write, which is the shape the argument is about; it is not yet a contest. A contest needs two things that settle at different times, and the expression has to bring those itself.

## Freezing, and What Must Be Frozen With It

A plan is frozen meaningfully: the same plan means the same thing later. **A claim that reaches outside the run is not**, because the world it asked about has moved.

So a claim would have to be frozen **together with the verdict it produced**. A later run then compares rather than merely re-running: *does this still hold?* A claim that answers differently between runs becomes a signal instead of a silent difference — and a run's checks become measurable rather than merely present.

The verdict already carries the shape such a store would keep: the name, the instance it was evaluated for, the expression as written, and what it answered. What is missing is anywhere to put it.

## Outro

A plan says what a run intends to do. A claim says what has to be true for it to be worth doing — and because it says so in the run's own values, the loop can read it without asking anybody.

That is the last thing the run needed to describe itself to itself, and :term[101: Concept/Idea]{href="./101_concept_idea.md"} returns to what such a run is for.
