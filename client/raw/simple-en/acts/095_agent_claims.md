# 095: Agent/Claims

> [!DEFINITION] [Claim](./000_glossary.md)
> A named test declared once for a run. Its answer is kept at `†expr.<name>` and is read just like any other :term[Variable Reference]{canonical="Variable Reference"}. The :term[Call]{canonical="Call"} that needs it will pause and wait until that test becomes true.

> Sidenote:
> - Requires:
>   - :term[007: Agent/Variables]{href="./007_agent_variables.md"}
>   - :term[010: Agent/Loop]{href="./010_agent_loop.md"}
>   - :term[011: Agent/Expressions]{href="./011_agent_expressions.md"}
>   - :term[012: Agent/Plan]{href="./012_agent_plan.md"}
> - Works closely with:
>   - :term[013: Agent/Instancing]{href="./013_agent_instancing.md"}
>   - :term[009: Agent/State]{href="./009_agent_state.md"}

> [!HEADSUP] Heads up
> **One section here is a design for the future, not a current feature: freezing.** Right now, the system doesn't permanently save or "freeze" anything. If a claim and its final answer were frozen, there isn't actually a storage locker for them yet. That section just describes what that locker *would* look like. Everything else works today: evaluating the tests, checking the conditions, `_when` rules, holding things back as gates, and reporting errors.
>
> The number `095` is temporary, just like the chapters before it.

A run already knows *when* a step is ready: a :term[Call]{canonical="Call"} waits for its :term[Variable References]{canonical="Variable Reference"} to arrive. But it doesn't know *if the information that arrived is actually good or useful*.

Imagine an agent needs to do different things depending on a **value**—maybe a policy document goes down one path, and an update goes down another. Right now, it can't split roads on its own. It has to stop, hand the value to the AI, and waste a whole turn asking it to make a simple choice. The fork in the road isn't difficult; the agent just didn't have a way to talk about it.

## The Rule

**A claim changes the wait rule from *wait until the data arrives* to *wait until the data arrives AND proves to be true*.**

:term[011: Agent/Expressions]{href="./011_agent_expressions.md"} handles the basic spelling and grammar of these tests. What follows here is how they live longer than a single step: how we give an expression a name, keep it around, check it whenever someone asks, and report on it.

## What a Claim Is

It is simply a name and a test condition, passed along in a message:

```jsonc
{ "type": "expressions", "expressions": [
    { "name": "isPolicy",   "expression": "†state.kind === 'policy'" },
    { "name": "wellFormed",
      "expression": "†state.result?.effectiveDate != null && †state.result.limits > 0",
      "description": "a result carries an effective date and a positive limit" }
] }
```

Giving it a name gives us something powerful: **a failure with a clear identity.** Saying *`wellFormed` wasn't true for document ②* gives the AI a concrete fact it can see and fix. If a nameless test fails, it's just a blank failure that tells the AI nothing.

## Reading a Claim Is Naming It

When a claim finishes, its answer sits at `†expr.<name>`, which acts exactly like a normal Variable Reference. The system finds it the same way it finds `†state.total`. There is no special roster of names to register.

```jsonc
{ "_tool": "summarize", "of": "†state.result", "_when": "†expr.wellFormed" }
```

**Any step that looks at a claim is doing two things: grabbing its answer, and waiting for that answer to be true.** This is how two steps looking at opposite claims turn into a fork in the road.

**This is also why you can't just use a "NOT" to handle the other path.** If a step reads `_when: "!†expr.wellFormed"`, it is waiting for `wellFormed` to become true. So, a step designed to handle broken documents would be stuck holding out for the document to be perfect. **If you want a fork in the road, you need two different claims**, each becoming true in its own specific situation.

Claims aren't just for blocking steps. You can do math with them like `†expr.total * 10`, because a claim is just a piece of data first, and a gatekeeper second.

> Sidenote:
> The value and the gate happen in one motion, which is tricky: a claim that results in a value of `0` gives you a `0` but **holds the call back**, because zero is not 'true' in logic. A claim is for making decisions. If you just need to count something, keep it in :term[State]{canonical="State"}.

## `_when`, for a Condition Used Once

If only one step needs a specific rule, you don't have to name it. You can just write the test directly into the step using `_when`:

```jsonc
{ "_tool": "summarize", "of": "†state.result", "_when": "†state.result.limits > 0" }
```

**It checks if the rule is true, not just if the data exists.** This is the big difference between treating something as a check versus just a reference. If a `_when` rule is false, the step waits. It might run later if the data changes.

**There is another reason to write a rule directly inside the step: claims created right now don't work on steps running right now.** When the AI creates a named claim, it goes into the plan for *after* the current steps finish. If two steps try to branch using a brand new claim, the system hasn't officially registered the name yet, so it ignores the lock and lets **both** steps run without a warning.

**If you just made a value and need to branch on it right now, write the rule directly in the step.** Named claims are for rules built before the current turn started.

## An Array, Because a Model Cannot Author a Key

The obvious design would be a dictionary of names. We can't use that here because the AI has to follow strict formats, and it can't safely invent dictionary keys on the fly. So the name is just a regular text value inside a list.

**Who writes the data decides its shape.** Human builders can use dictionaries safely. AI models need lists so they can fill in the blanks predictably.

The downside of a list is that you could accidentally name two claims the same thing. The rule here is simple: **the newest claim replaces the older one**. If an AI writes the same name again with a new rule, it simply updates it.

**Unlike steps in a plan, claims don't disappear if you stop mentioning them.** If the AI leaves a claim out of its next turn, the claim stays alive. If it thinks leaving it out turns it off, it's making a mistake. The claim keeps blocking steps in the background.

**You can't delete a claim.** Once you declare a rule, the system tracks it. To "turn it off," you have to overwrite it with a simple rule that is always true.

## Global by Declaration, Per Instance by Evaluation

**You declare it once, but it is checked exactly where it's used.** If you write a rule once, every document (Instance) runs it against its own data. Document ② gets tested on Document ②'s data. Document ③ gets Document ③'s data. We don't have to copy the rule for every single item.

**There are no special mechanisms for this.** When the system checks a claim, it automatically gathers the data for that specific situation. If a specific instance has its own custom rule, it overrides the general one.

This mirrors how the :term[012: Agent/Plan]{href="./012_agent_plan.md"} works. The plan doesn't care about specific instances. It just aims the rule exactly when it's being read.

If a host wants a claim to only apply to a single specific instance, they pin it, and that pinned rule beats the global one.

> Sidenote:
> An **AI model** can't write that narrow rule right now. The output it creates holds a name, a test, and a description, so it applies to the whole run. This isn't actually a big deal: the automatic per-document test is what the AI wants almost every time, and pinning is just a backup tool for whoever hosts the system.

## One Format in Every Position

The AI's output, the system's storage, and the human's code all use the **same shape**. :term[012: Agent/Plan]{href="./012_agent_plan.md"} established this: steps come back completely unchanged, needing no translation.

Claims follow this too, which means the run can **hold onto what it learns**. A claim written on turn three carries forward perfectly and can be reused forever. There are no translation steps, so nothing gets lost in translation.

> Sidenote:
> The final block sits in the **derived** layer rather than the permanent frozen layer, and :term[091: Agent/Caching]{href="./091_agent_caching.md"} is the reason why. If an AI writes a new rule for an old claim, it overwrites it. If we kept it in the permanent cache, changing one little rule would break the cache for everything after it.

## How a Claim Is Evaluated

**The system gathers all the variables before checking the rule.** It doesn't sloppily paste raw data into text, which can cause breaks or security leaks. The variables are cleanly swapped in as background arguments.

Any waiting for network data happens right at the start instead of in the middle of a check. This keeps the code simple, avoiding messy setups where half of a complex formula is still waiting for internet results.

What you get is standard JavaScript. Regular list setups and comparisons just work out of the box, and tests happen in parallel.

> Sidenote:
> A claim can't read another claim. Every claim is checked against the raw data that arrived. If you try to use `†expr.other` inside a test's own code, it just returns nothing, and gives no warning. Stacking claims would require telling the system exactly what order to read them in, which isn't currently supported.

### What the Evaluator Owes

**A time limit.** We give it five seconds. If a test gets stuck waiting forever, it crashes the whole operation. Since it's only reading local data, a good test finishes in a microsecond. If a test hits the time limit, it is **abandoned, not canceled** — the system just gets an 'out-of-time' answer and moves on.

**A limit on doing everything at once.** We don't want a rogue code block launching a billion tests at once. The limit applies to the whole page, since an AI writes many tests but only reads a handful at a time.

**Accepting that answers might change.** If a rule checks a live website, the answer might be different tomorrow. We have to accept that when we freeze (save) the results.

### Failure Is Not a Verdict

**If a test crashes, takes too long, or looks for a variable that doesn't exist, it simply acts like it isn't there — it doesn't break everything — and tells you clearly.**

Breaking isn't a magical 'fatal error'. It just means *we don't have an answer*. Since we have no answer, we don't hold any step back. Only a test that successfully checked everything and turned out 'false' will lock a step.

This is just how the system already behaves. When it hits something weird, it skips it and leaves a note, instead of shutting down the whole building.

**A broken claim can never cause a permanent traffic jam.** The worst it does is let a step run through by failing to hold it back.

What gets reported is an :term[Error Message]{canonical="Error Message"} that the next :term[Request]{canonical="Request"} reads. You only see it once per run instead of flooding your screen. Keep in mind: steps that are just naturally waiting for good data don't report anything. So if a step gets stuck forever waiting on a bad condition, the run might quietly end without a warning.

### Referenced by Nothing, Still Checked

A claim is a setup rule, not an instruction. It gets checked at the end of the turn even if no step asks for it. **One rule does two jobs:** it can act like a universal truth for the system ("every policy must have a date") and, at the exact same time, act as a lock holding one specific step back.

## A Branch That Costs No Turn

```jsonc
{ "calls": [
  { "_tool": "extractCoverages",   "doc": "†state.raw",
    "_outputPath": "†state.result", "_when": "†expr.isPolicy" },
  { "_tool": "extractEndorsement", "doc": "†state.raw",
    "_outputPath": "†state.result", "_when": "†expr.isEndorsement" }
] }
```

Both of these branches use claims that were **already tested** when the turn started. Only one can be true. The system automatically lets one run and leaves the other waiting forever on a signal that's never coming — a simple fork in the road.

**Because this choice is built in right at the start, it costs zero extra time.** Deciding it any other way means stopping to look at `†state.kind` and wasting an entire turn for every single document.

## Speculation Belongs Inside One Expression

The way a test works changes based on how confident it is. Confident: one step goes when the value is good. Uncertain: two steps form a fork (like above). Balanced (where waiting is expensive): one step **races** two futures and takes whichever answers first.

If you want a race, how much code you need depends on where you put it.

**If we launched two separate steps and picked a winner later, we'd be writing two answers to the exact same place.** We can't hit 'undo' easily. Rolling back a failed step is messy and unpredictable.

A race inside a single expression skips all of that. **It’s one test, one answer, and one save.** So there is no complicated undo machinery. It is just a quick line of regular JavaScript picking a winner.

> Sidenote:
> A race doesn't cancel out the loser, and it shouldn't. **Only the flow of the answer stops traveling.** The losing side keeps running its code, meaning if it pinged an external tool, it still reaches out to it. This isn't a mistake; true cancellation requires an entirely different setup.

Right now, a race just looks at what the test itself is waiting on. Since all variables are gathered before the test even starts, racing two existing variables is instantaneously decided. A real race needs two things that arrive at totally different times, and the test has to construct that setup on its own.

## Freezing, and What Must Be Frozen With It

A regular plan is easy to freeze because the exact same steps will do the exact same thing later. **But a test checking the outside world isn't predictable**, because the outside world changes.

So, if we freeze a claim, we have to freeze **the final answer it came up with** alongside it. That way, a future run can dig it up and ask, *"Is this still true today?"* This turns a silent, unpredictable shift into a trackable difference.

The final result already has the shape we need: the name, the specific instance it was for, the exact test code, and the final answer. The only piece missing is a hard drive to store it all.

## Outro

A plan explains what the system wants to do. A claim explains what facts must be true to make doing it worthwhile — and because it checks the facts directly, the system doesn't have to pause and ask anyone.

This is the final piece the system needed to manage itself. Now, :term[101: Concept/Idea]{href="./101_concept_idea.md"} returns to the bigger picture of what this entirely self-managed run is actually for.
