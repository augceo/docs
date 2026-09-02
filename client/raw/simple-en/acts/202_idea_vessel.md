# 202: Idea/Vessel

> [!DEFINITION] [Vessel Idea](./000_glossary.md)
> Imagine an Idea that's a complete package. It doesn't just describe a tool; it describes *all the possible tools* for a situation and also records *which one was picked*. The `schema` part is like a menu of all possible choices, and the `solution` part is the one item you ordered.

> Sidenote:
> - You'll want to know about:
>   - [101: Concept/Idea](./101_concept_idea.md)
>   - [002: Agent/Tool](./002_agent_tool.md)
>   - [004: Agent/Call](./004_agent_call.md)

A **Vessel Idea** is like a perfect snapshot of an A.I.'s decision at a specific moment. It’s more than just a log of what happened. It’s a complete picture that includes all the surrounding details of the choice, which makes it easy to check, change, and reuse later on.

## 1. The Anatomy of a Reaction

A `Vessel Idea` uses the three main parts of an `Idea` to show both what could have happened and what actually did happen:

> Sidenote:
> - [101: Concept/Idea](./101_concept_idea.md)

- **`context` (The "Why"):** This is the information or problem that made the A.I. need to act. It's the reason for the decision.
- **`schema` (The "What Ifs"):** This describes every single tool or action the A.I. *could have* used at that moment. Think of it as a complete playbook of all possible moves.
- **`solution` (The "What Happened"):** This is the record of what the A.I. *actually did*. It's a list of the specific `Calls` (the chosen tools from the playbook) that were used.

Because a `Vessel` holds both the universe of possibilities (`schema`) and the final choice (`solution`), it gives you a complete story of a single decision.

## 2. The Power of Self-Contained Decisions

Having everything bundled together in one package is what makes the system so smart. By including the playbook with the final play, a `Vessel Idea` automatically allows for:

- **Human Check-ups and Changes:** A person can look at a `Vessel` and see not only what the A.I. did, but all the other things it could have done. They can then approve the action or even change the `solution` by picking a different `Call` from the A.I.'s playbook (`schema`).
- **Making a New Plan:** If a chosen action (`Call`) doesn't work out, the A.I. can easily try something else. It already has the original problem (`context`) and its full list of options (`schema`), so it can quickly pick a different `Call` and create a new `Vessel`.
- **Reusing Past Decisions:** A `Vessel` can be used as a an example to help make similar decisions in the future.

### 2.1. Interactive Time Travel

Because a `Vessel Idea` is a complete, unchangeable snapshot, it lets you do something like interactive time travel. You can load a `Vessel` from the past and see the A.I. exactly as it was at that very moment.

The `schema` part guarantees that the A.I.'s abilities are frozen in that instant. Even if the modern A.I. has learned new tricks and gained new `Tools`, the old `Vessel` will only show the options that were available when it was created. This lets you:

- **Replay History:** See exactly how a past decision was made.
- **Explore "What-Ifs":** You can give a different problem (`context`) to that old `Vessel` to see how the past version of the A.I. would have handled it.

This is a super powerful way to fix bugs, check the A.I.'s work, and understand how it gets smarter over time.

## 3. The Vessel in the Action Loop

A `Vessel` is like a single beat in a rhythm. It’s one decision made and recorded in a single step. The results of its actions (`Calls`) are then fed back as new information (`context`) for the next beat, which might create a new `Vessel`.

> Sidenote:
> - [010: Agent/Loop](./010_agent_loop.md)

This call-and-response pattern is great for building A.I.s that can react to things happening in real time. However, it's not built for planning out big, complicated projects that require memory and a long-term strategy.

## 4. From Quick Reactions to Big Plans

While a `Vessel` is a perfect record of a single reaction, reaching bigger goals requires a way to plan things out step-by-step. This is where a **Process Idea** comes in. It uses the same all-in-one-package idea but applies it to a long-term, strategic plan.

> Sidenote:
> - [203: Idea/Process](./203_idea_process.md)
