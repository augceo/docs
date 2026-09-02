# 019: Agent/Intent

> [!DEFINITION] [Intent](./000_glossary.md)
> A plan step that names the work rather than a tool. It reads and writes like any other step, so it sits in the dependency graph unchanged — but it declares no activity and is never run by the party that wrote it. Whoever holds a tool that can do the work replaces it with a real call.

> Sidenote:
>
> - Requires:
>   - :term[004: Agent/Call]{href="./004_agent_call.md"}
>   - :term[008: Agent/Output]{href="./008_agent_output.md"}
>   - :term[012: Agent/Plan]{href="./012_agent_plan.md"}
> - Enhances:
>   - :term[014: Agent/Delegate]{href="./014_agent_delegate.md"}
>   - :term[015: Agent/Scopes]{href="./015_agent_scopes.md"}

An agent can only plan what it can name, and it can only name the tools it holds. That is not a convention — a request's response schema is composed from the tools in its own context, so a step naming anything else is a response the schema forbids. **A planner is therefore confined to planning work it could do itself, which is the opposite of delegating.**

There are two ways out and one of them is bad. Give the planner every delegate's tools, and the whole roster is composed into the schema of every request it ever makes, whether or not it delegates. An **Intent** is the other way: a step that names the work instead of the tool.

## It Is an Ordinary Node, and That Is the Whole Trick

A :term[Plan]{canonical="Plan"}'s edges are its Output Paths and the Variable References its steps read — nothing else records the graph. An Intent carries both. It reads `†state.manifest` and writes `†state.seats.row` exactly as a :term[Tool Call]{canonical="Tool Call"} would, so every step that depends on it waits for it, every cycle check sees it, and the graph is recovered from it without knowing what it is.

Nothing in the loop had to learn about it. A step the turn cannot evaluate is already carried into the plan unrun, and a step waiting on a path that unrun step promises is already a legitimate wait rather than a structural error. An Intent is simply a step nobody here can evaluate, and both of those rules already say what to do about it.

```json
{
  "_tool": "intent",
  "title": "choose a row",
  "description": "pick a window seat from the manifest for a party of three",
  "with": ["†state.manifest"],
  "_outputPath": "†state.seats.row"
}
```

## Authoring One Is a Statement, Not an Attempt

An Intent declares no `_activity`. A tool with one is dispatched; this one cannot be, so writing it says *this should happen* rather than *I am doing this*. The planner is not blocked by it and does not fail on it. It simply stands in the plan, owed.

Who fills it is not the Intent's business. A peer that receives the step realizes it with whatever tool it holds; a later turn of the same agent may realize it once something else is known. The Intent does not name a recipient, because a step that named one could not be given to anybody else.

## Identity Is the Title

`title` names the work, and there is no separate identifier. This is act 018's rule unchanged: a realization attaches by title the way an answer attaches to a question. Two Intents doing the same work under different titles are two steps; the same title twice is the same step.

## Delegability Is a Fact About the Pairing

A grant of writable paths cuts a plan, because the paths already are the edges — the steps whose every destination a grant covers are the steps that travel. But a grant says where a peer may **write** and says nothing about what it can **do**, so a cut checked by path alone can hand over a call naming a tool nobody over there holds.

**A step is the delegate's when the grant covers everywhere it writes AND the delegate has its tool.** Both halves are facts about the pairing. Neither belongs on the call: a field declaring a step delegatable is the author guessing at an answer only the pairing knows, it states one fact in two places, and it lets a plan contradict a grant.

An Intent is exempt from the second half, and that is its point. It names no tool, so no tool set can make it unrunnable — a delegate holding nothing at all can receive one and realize it with whatever it does hold.

## Guidance Is Data, Not a Mechanism

A plan that says *do this, then tell the other one* needs no vocabulary beyond what is already here. The doing is a step. The telling is a call. And the vague part — what the work is *for*, what would count as done — is an Intent's own `description`, which travels with it because it is part of the step.

Where the guidance should be **authored** rather than written in advance, it is a step whose Output Path lies inside the recipient's grant. It then travels by the same rule as every other value: no new content kind, no second routing rule, nothing that has to agree with the first one.

## Extending It

An Intent's shape is a floor rather than a ceiling. A caller that wants more of a contract — acceptance criteria, a deadline, a required form for the result — declares it as a schema in context, and the extra properties are merged into the tool the model is offered. That is the same mechanism every other kind already uses to widen a tool, and it keeps the addition where the caller can see it instead of in the library.

What it must not become is a second plan. An Intent says what should happen and where the answer goes. A contract about the answer's shape belongs on the answer.

## It Costs Nothing Until Named

Nothing registers an Intent globally. A tool that no request named still costs every request that composes it, and the schema is already the larger half of most payloads. A context that wants Intents asks for one.
