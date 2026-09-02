# 019: Agent/Intent

> [!DEFINITION] [Intent](./000_glossary.md)
> Think of an Intent like a "Help Wanted" sign in a plan. Instead of pointing to exactly *which tool* to use, it simply points to *what work needs to be done*. It fits perfectly into the map of a plan like any normal instruction, reading data in and writing answers out. But the agent that writes it down doesn't actually do the work. It leaves the step there until someone else who *does* hold the right tools comes along to finish the job.

> Sidenote:
> - Requires:
>   - :term[004: Agent/Call]{canonical="004: Agent/Call" href="./004_agent_call.md"}
>   - :term[008: Agent/Output]{canonical="008: Agent/Output" href="./008_agent_output.md"}
>   - :term[012: Agent/Plan]{canonical="012: Agent/Plan" href="./012_agent_plan.md"}
> - Enhances:
>   - :term[014: Agent/Delegate]{canonical="014: Agent/Delegate" href="./014_agent_delegate.md"}
>   - :term[015: Agent/Scopes]{canonical="015: Agent/Scopes" href="./015_agent_scopes.md"}

Imagine you are building a toy robot. You can only plan out tasks if you actually own the tools to do them (like a screwdriver or wrench). An AI agent works the same way. If it tries to write down a plan using a tool it doesn't have, the system stops it because it breaks the rules. **Because of this, an agent gets stuck only planning chores it can finish completely alone, which makes teamwork impossible.**

There are two ways to fix this, and one of them is terrible. The bad way is handing the planner every tool in the world, which crowds the system and makes every request heavy and slow. The better way is an **Intent**: a step where the planner simply writes down the *chore*, rather than the *tool* missing from its toolbox.

## It Plugs Right Into the Plan, and That Is the Clever Part

A :term[Plan]{canonical="Plan"} is like a map where lines connect different stops. A normal step tells the map where it gets its materials from (the data it reads) and where it leaves its finished product (the data it writes). An Intent does exactly the same thing. It announces, "I need to read this file, and when I'm done, I will drop the answer into that folder," exactly like a normal :term[Tool Call]{canonical="Tool Call"} does. Every step waiting downstream sits patiently for it to finish, so the map stays perfectly intact.

The system didn't even need special rules built for this. If the system encounters a step it can't figure out yet, it simply pauses and carries it forward un-run. An Intent is just a task the current agent can't figure out yet. Since the system already knows it's okay to wait for un-run tasks, it handles the Intent flawlessly.

```json
{
  "_tool": "intent",
  "title": "choose a row",
  "description": "pick a window seat from the manifest for a party of three",
  "with": ["†state.manifest"],
  "_outputPath": "†state.seats.row"
}
```

## Writing One Is an Ask, Not an Action

An Intent doesn't pretend to be taking action immediately. In code, it declares no inner `_activity`. Imagine shouting, "This room needs cleaning!" instead of saying, "I am turning on the vacuum." Because of this, the planner writes the step in the plan without being blocked or breaking down when it can't clean the room itself. It just places the promise there.

It isn't the Intent's job to pick *who* cleans the room. When another agent (a helper) receives this step, it finishes it using whatever tools it brought along. The Intent doesn't name a specific helper because doing so would lock everyone else out of pitching in.

## The Title Is Its Name Tag

The `title` tells everyone what the chore is, and it serves as the only identification badge. It works like matching puzzle pieces: when a helper brings back an answer, it sticks it to the title of the Intent, just like matching an answer to a test question. If you write two Intents with different titles for the exact same job, the system spins up two different steps. If you use the identical title twice, it treats it as the exact same step.

## Getting Help Depends on How You Pair Up

Sharing work is a bit like granting someone access to specific rooms in your house. When an agent asks for help, it gives the helper permission to write answers into certain spots on the plan. But handing over a room key only decides *where* they can work—it guarantees nothing about whether they have the right *tools*. A system that only checks room permissions might accidentally hand a job to an agent with empty hands.

**A job successfully moves to a helper when two facts line up: they are allowed to drop answers in the destination, AND they actively possess the needed tool.** Both boxes must be checked. Neither belongs in the actual file's data, because we don't want agents guessing about toolboxes. Only the final partnership knows if it works.

An Intent is magically exempt from the second rule, which is the whole point. Because it commands no specific tools, literally no toolbox is too small for it. A helper could currently hold nothing at all and still grab the Intent, dealing with how to actually do it when the time comes.

## Instructions Are Just Information

A map that says "do this, then tell that guy" doesn't need entirely new command signals. Doing the work is just a step. Handing it off is just a call. The confusing part—knowing *what* the job is for and *when* it counts as finished—is written plainly inside the Intent's `description`. The instructions pack right into the step's backpack.

When these instructions change on the fly rather than being planned out entirely in advance, they flow down as a step dropping safely into the helper's permitted zone. They travel using the simplest route possible: standard data rules. No secret routing codes, no secondary networks.

## Adding More Details

The basic shape of an Intent is just a starting blueprint. If an agent wants an ironclad guarantee—such as an absolute deadline, specific checkboxes to hit, or a certain shape for the final answer—they can declare it up front. All those extra rules just get mashed right into the task given to the helper model. We employ this same trick constantly to make tools bigger, keeping the extra rules highly visible exactly where they were asked for.

What an Intent absolutely shouldn't become is a sneaky secondary mini-plan. An Intent claims *what should happen* and *where the result goes*. Details regarding what the result looks like belong stuck to the result itself.

## It Is Free Until You Ask for It

No master database wastes time keeping track of all Intents globally. Even completely unused tools still burden the network by crowding up the menu space of every request. But an Intent stays completely invisible until an agent specifically needs one. If the context calls for an Intent, the system simply asks for it.