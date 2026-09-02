# 801: Package/Agent

> [!DEFINITION] [Agent](./000_glossary.md)
> Think of this as the main engine for the **Acts of Emergence**. It is the core software that runs smart, step-by-step tasks, from simple questions to complex, memory-keeping Agents.

> Sidenote:
> - Implements:
>   - :term[001: Agent/Request]{canonical="Request", href="./001_agent_request.md"}
>   - :term[010: Agent/Loop]{canonical="Loop", href="./010_agent_loop.md"}
> - Links:
>   - [NPM: @idealic/agent](https://www.npmjs.com/package/@idealic/agent)

The `@idealic/agent` library is the official toolkit built around the rules defined in the Acts. Unlike other tools that just focus on talking to AI, this library focuses on following the rules perfectly. It acts like a strict traffic controller, ensuring every interaction has a clear shape, follows a specific blueprint, and works exactly predictably.

Two important things happen because of this design. First, every Act is a separate tool you can pick up and use on its own, so simple tasks never force you to use complex features you do not need. Second, every power this engine has was planned out before a single line of code was written. That is why the list below serves as a map of the entire package rather than just a summary.

If you want to know how to actually use this library or find a quick guide, you should read its own README file. This list here just tells you where the rules live. If you are looking for speed tests or numbers, those live in a separate place called [measurements](../measurements/README.md).

## What the Package Admits

A new feature is only added to this library if an Act asks for it. The rule comes first, and the code comes second. If a behavior does not have an Act backing it up, it does not belong here. This means the library is extremely focused: it does not include things like built-in databases or web hosting, because the Acts do not define those things.

## The Register

### Core Primitives

- :term[001: Agent/Request]{canonical="Request", href="./001_agent_request.md"} — the basic building block of processing. It takes a starting situation and a blueprint, and turns them into a structured `Solution`. It is the foundation everything else is built on.
- :term[002: Agent/Tool]{canonical="Tool", href="./002_agent_tool.md"} — a blueprint for a specific power. It explains what an action should look like without worrying about how it actually gets done.
- :term[003: Agent/Activity]{canonical="Activity", href="./003_agent_activity.md"} — the actual, predictable code behind a Tool. It connects the blueprint to real action. If a Tool lacks an Activity, the AI uses its own brain to guess the answer instead.
- :term[004: Agent/Call]{canonical="Call", href="./004_agent_call.md"} — a specific moment when a Tool is used, carrying information directly between what the AI wants to do and what the system actually does.

### Data & State

- :term[005: Agent/Data]{canonical="Data", href="./005_agent_data.md"} — the standard container we put all structured information into, along with the rules for how to merge two containers.
- :term[006: Agent/Input]{canonical="Input", href="./006_agent_input.md"} — strict instructions for what information can go into a Request, turning an open-ended task into a safe, reusable tool.
- :term[009: Agent/State]{canonical="State", href="./009_agent_state.md"} — the shared memory pad that survives across different steps. This makes it possible to pause a run and pick it back up later.
- :term[016: Agent/Meta]{canonical="Meta", href="./016_agent_meta.md"} — the Agent's identity and history. It tracks versions and origins, so an Agent can evolve while remembering its past.

### Wiring & Flow

- :term[007: Agent/Variables]{canonical="Variables", href="./007_agent_variables.md"} — special addresses (like `†kind.path`) that let a Call read information directly from where it lives, instead of copying it.
- :term[008: Agent/Output]{canonical="Output", href="./008_agent_output.md"} — the `_outputPath`, which tells the system exactly where to save a result so that other steps can read it later.
- :term[011: Agent/Expressions]{canonical="Expressions", href="./011_agent_expressions.md"} — a way to do math or logic using those special addresses. It uses three special symbols — `<|>` to choose, `*>` to block, `|>` to pass along — marking them clearly as separate from normal code.

### Orchestration

- :term[010: Agent/Loop]{canonical="Loop", href="./010_agent_loop.md"} — the main engine that keeps things moving: it sends out a request, runs everything that is ready, feeds the results back in, and repeats this cycle until the job is done.
- :term[012: Agent/Plan]{canonical="Plan", href="./012_agent_plan.md"} — a map of how different Calls connect to each other, separating the big idea of the workflow from the actual running of it.
- :term[013: Agent/Instancing]{canonical="Instancing", href="./013_agent_instancing.md"} — a feature that lets you point one single Plan at many different targets without copying the setup over and over again.
- :term[014: Agent/Delegate]{canonical="Delegate", href="./014_agent_delegate.md"} — tiny, guarded side-quests. This is how one Agent can shrink down to become a Tool used by another Agent.
- :term[015: Agent/Scopes]{canonical="Scopes", href="./015_agent_scopes.md"} — strict blinkers that ensure a Delegate or Activity can only see exactly what it needs to see, and nothing more.
- :term[017: Agent/Advisor]{canonical="Advisor", href="./017_agent_advisor.md"} — helpful sidekicks that think about the problem before the main Agent acts, casting votes on what should happen next.
- :term[018: Agent/Question]{canonical="Question", href="./018_agent_question.md"} — asking someone else to solve a problem. The answers pile up in one spot so you can track how an opinion changes over time and understand the reasoning behind it.

### Cross-Cutting Disciplines

These rules apply everywhere at all limits, rather than just at a single stage of the process.

- :term[090: Agent/Typing]{canonical="Typing", href="./090_agent_typing.md"} — figuring out a blueprint's exact data structure once and keeping it securely attached as it moves around the system.
- :term[091: Agent/Caching]{canonical="Caching", href="./091_agent_caching.md"} — organizing memory so the unchanging information always stays at the front, saving time for the AI's brain.
- :term[092: Agent/Limits]{canonical="Limits", href="./092_agent_limits.md"} — the budget for a run, checking things like time and computer power. It guides the AI like an advisor, but enforces the boundaries strictly like a judge.
- :term[093: Agent/Compaction]{canonical="Compaction", href="./093_agent_compaction.md"} — quietly packing away the story of what happened while the final results are still being written.
- :term[094: Agent/Config]{canonical="Config", href="./094_agent_config.md"} — the settings for the AI, like how hard it should think. These are shared directly with the model, which can even adjust them safely within its allowed limits.
- :term[095: Agent/Claims]{canonical="Claims", href="./095_agent_claims.md"} — a saved statement that a Call reads and waits for. If two Calls wait for things that contradict each other, the system knows one branch is a dead end instantly.
- :term[096: Agent/Hooks]{canonical="Hooks", href="./096_agent_hooks.md"} — a way to plug in extra behaviors. An 'interceptor' stands in the path and can change the flow, while an 'observer' just watches without interfering.

### The Concept Behind the Default

- :term[104: Concept/Latent]{canonical="Latent", href="./104_concept_latent.md"} — when a Tool has no code attached, the AI has to do the heavy lifting using just its brain. You can build an entire workflow out of blueprints first, and seamlessly swap in real code later without breaking the setup.

## Outro

This list is simply a map of doors. The library answers only to the actual rules inside those doors, not to the short summaries collected here.

The same goes for the visual layer where humans interact with these tools. :term[802: Package/UI]{canonical="Package/UI", href="./802_package_ui.md"} takes these blueprints and directly turns them into the visual interface you can look at and touch.
