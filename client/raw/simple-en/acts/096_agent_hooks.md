# 096: Agent/Hooks

> [!DEFINITION] [Hook](./000_glossary.md)
> A tool that connects to a specific point in a program. It comes in two types: an **interceptor** stands in the way and can change what passes through, so if it breaks, the whole process stops. An **observer** stands on the side and just watches, so if it breaks, the main process keeps going.

> Sidenote:
> - Requires:
>   - :term[001: Agent/Request]{href="./001_agent_request.md"}
>   - :term[010: Agent/Loop]{href="./010_agent_loop.md"}
> - Complemented by:
>   - :term[011: Agent/Expressions]{href="./011_agent_expressions.md"}
>   - :term[092: Agent/Limits]{href="./092_agent_limits.md"}
>   - :term[108: Concept/Visibility]{href="./108_concept_visibility.md"}

> [!HEADSUP] Heads up
> The number `096` is temporary. Files starting with `09` cover mixed topics and might be reorganized later. Treat the filename as a placeholder, but the information inside is final. The reading order might change since the previous chapter also connects to :term[101: Concept/Idea]{href="./101_concept_idea.md"}.

A system touches a running program for two different reasons, and only one is dangerous.

Sometimes, it steps in to **change** things. It might add a secret password before sending a message, or hide private details so they do not leak. This action must be able to fail loudly—if it cannot attach the password, it has to stop the message entirely rather than send it unprotected. Think of it like a security guard holding up a stop sign.

Other times, the system just wants to **watch**. It might track how much memory is used or log what decisions were made. People add these watchers everywhere. Because they only watch, a broken watcher should never crash the main program. Think of it like a security camera on the wall.

If we force both behaviors into one tool, it breaks. Either the camera accidentally gets the power to block the door, or the security guard loses the ability to stop anyone.

## The Rule

**Two actions, one setup method.** You add, remove, and check these tools the exact same way, but the system treats them differently.

- An **interceptor** gets the data, holds it, and hands it back. It can change the data. **If it breaks, the error spreads** and stops everything.
- An **observer** gets a copy of the data and gives nothing back. It cannot change anything. **If it breaks, the error is ignored** and the system keeps running smoothly.

This difference is the whole point of the design. You should see right away if a tool has the power to change things or just watch.

Both return a receipt when you attach them, so you can easily remove them later.

> Sidenote:
> Creating these connections was written three different times before we combined it into one clear rule. There was one setup for interceptors, one for usage trackers, and one for anything else. We combined them to fix a repeating bug: trying to remove a tool that wasn't there would accidentally delete a different one instead.

## The Seams

- **The outgoing message** uses interceptors. This is a place where you must *change* things—like adding a password or fixing a text before it leaves.
- **Measuring usage** uses observers. Counting how much work a step took is just recording facts. A broken meter should not ruin the work itself.
- **The main system loop** uses observers. Every turn, it reports which path a decision took, which options were locked out for good, and which :term[Calls]{canonical="Call"} are still waiting on paths nobody took.
- **The expression evaluator** uses observers. This tool reports exactly how choices were made and why a :term[Call]{canonical="Call"} decided to run. The system already knew this information, but now it actually tells you instead of keeping it a secret.

## What the Evaluator May Report, and What It May Not

An observer helps you understand the system, so its reports must be perfectly true to how things work:

**`<|>` means preferring the left side, not racing.** The system tells you *which side provided the answer*, not which one finished first. There is no timer checking speed, so words like *winner* or *race* belong nowhere in the report.

**Nothing gets canceled.** If the system picks the left side, it simply ignores the right side. We call this *unread*. Nothing is permanently destroyed or thrown out. If the unread side was already running, it finishes its work, but its final answer is just left alone.

**Being ready is separate from the final answer.** A :term[Call]{canonical="Call"} jumps into action as soon as either option is available. What it actually reads is decided in a separate step. If we mixed these two facts together, the report would be confusing.

**Names stay exactly as you wrote them.** By the time the code runs, the system has changed all your specific :term[Variable References]{canonical="Variable Reference"} into random computer IDs. Because a person cannot read those IDs, the system turns them back into your original words before showing you the report.

> Sidenote:
> The system reports its readiness **every time you ask, not just when it changes.** The system checks over and over again and gets the same answer while things stay the same. It does not have a memory of its own past tracking, so it cannot tell you if something is new; keeping track of the last answer is the job of whatever tool is drawing the screen.

## Watching Must Be Free When Nobody Watches

Observers checking the math fire off hundreds of times a second. **If a tracking tool slows the system down when nobody is looking, people will turn it off**, making it useless.

So, if no one is watching, the system does zero extra work. It does not gather data, create reports, or waste memory. It only starts taking notes when it knows an observer is actually listening. Part of taking notes involves checking options the system would normally skip, so it fiercely avoids this work unless asked.

You can test this easily: if you attach an observer *after* the system has already run, your log will be completely empty. The system did not secretly write things down in the background.

## Why a Side Channel Rather Than a Stamp on the Value

The easy way to do this would be stamping the information directly onto the :term[Call]{canonical="Call"}—marking its answer and letting anyone read it.

But that way is expensive. A stamp takes up space, needs careful formatting, and changes the actual item. It breaks a core rule of how this system is built: making a choice should not permanently alter the original items.

**An observer never touches the value itself.** The result you get is exactly the same whether someone is watching or not. It hides no extra data. If you remove the observer, the system returns immediately to its normal, undisturbed state.

The idea behind stamping was right—we needed the information. The way we deliver it is just better now.

## Outro

When you can watch a system run without messing it up, you can finally show it to people. You can draw it on a screen in real-time, measure its costs, and explain exactly why it made its decisions.

This was the final piece the system needed to connect with the outside world. Now, :term[101: Concept/Idea]{href="./101_concept_idea.md"} returns to explain what all this running is actually for.
