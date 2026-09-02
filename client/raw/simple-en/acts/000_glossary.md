# A Dictionary of Big Words

This dictionary explains the important words used in the a system for sharing ideas called the Idea Protocol.

- :dfn[Advisor]{canonical="Advisor" href="./017*agent_advisor.md"}: An Advisor is like giving the AI a special role to play, for example, a “cautious expert” or a “creative writer.” It tells the AI to stop and give an “opinion” or a score of how sure it is *before* it picks a :term[Tool]{canonical="Tool"} or gives an :term[Output]{canonical="Output"}.

  > Sidenote:
  > - [017: Agent/Advisor](./017_agent_advisor.md)

- :dfn[Idea]{canonical="Idea" href="./101_concept_idea.md"}: An Idea is not just a passing thought. Think of it like a smart recipe that has three parts: the ingredients (`schema`), the final dish (`solution`), and notes about how you made it (`context`). It's a permanent building block of knowledge that you can come back to and improve, not just a temporary chat message.

  > Sidenote:
  > - [101: Concept/Idea](./101_concept_idea.md)

- :dfn[Ideator]{canonical="Ideator" href="./103_concept_ideator.md"}: An Ideator is a special type of :term[Idea]{canonical="Idea"} that can take an input and turn it into something new. If an Idea is a recipe, an Ideator is like a machine in the kitchen. You put ingredients in one end, and it makes the dish for you at the other end.

  > Sidenote:
  > - [103: Concept/Ideator](./103_concept_ideator.md)

- :dfn[Idea Transformer]{canonical="Idea Transformer" href="./103_concept_ideator.md"}: This is a very special :term[Ideator]{canonical="Ideator"} that takes a whole :term[Idea]{canonical="Idea"} as its input. It’s like a master chef who can take a finished dish, improve it, and turn it into something completely new.

  > Sidenote:
  > - [103: Concept/Ideator](./103_concept_ideator.md)

- :dfn[Input Message]{canonical="Input Message" href="./006_agent_input.md"}: This is like an order form you give to an AI. It clearly lists what kind of information it needs (`schema`) and the actual information you are giving it (`input`). This makes it easy to make the same kind of :term[Request]{canonical="Request"} over and over again.

  > Sidenote:
  > - [006: Agent/Input](./006_agent_input.md)

- :dfn[Tool]{canonical="Tool" href="./002_agent_tool.md"}: A Tool is like a special power you can give to an AI. It describes something the AI can do, like “search the internet” or “draw a picture.” The AI sees a list of its available tools and makes a `Call` to use one when it needs to. The actual work is then done either by the AI itself or by a separate piece of code (an `Activity`).

  > Sidenote:
  > - [002: Agent/Tool](./002_agent_tool.md)

- :dfn[Activity]{canonical="Activity" href="./003_agent_activity.md"}: An Activity is the real code that makes a `Tool` work. If the `Tool` is the “search the internet” button, the Activity is the program that actually connects to the internet and gets the results. It does the real work for things the AI can’t just think up on its own, like talking to other websites.

  > Sidenote:
  > - [003: Agent/Activity](./003_agent_activity.md)

- :dfn[AI-Native]{canonical="AI-Native" href="./105_concept_ai_native.md"}: This is a way of building things where the AI is the main builder, not just a helper. Imagine a house that was designed, built, and even repaired by smart robots. In an AI-Native system, the AI is the center of everything from the very beginning.

  > Sidenote:
  > - [105: Concept/AI-Native](./105_concept_ai_native.md)

- :dfn[Agency]{canonical="Agency" href="./111_concept_life.md"}: Agency is the ability to make your own choices and act on them. It’s about being able to see the world around you, think about it, and decide what to do next. It’s what makes you an active player, not just a piece on a board.

  > Sidenote:
  > - [111: Concept/Life](./111_concept_life.md)

- :dfn[Boundaries]{canonical="Boundaries" href="./111_concept_life.md"}: Boundaries mean that something is unique and self-contained. You have boundaries—there’s only one of you. A specific :term[Idea]{canonical="Idea"} also has boundaries. If you change its core definition, it becomes a new and different :term[Idea]{canonical="Idea"}.

  > Sidenote:
  > - [111: Concept/Life](./111_concept_life.md)

- :dfn[Life]{canonical="Life" href="./111_concept_life.md"}: In this system, “Life” isn't just for plants and animals. It's for anything that has three special abilities: :term[Agency]{canonical="Agency"} (it can make choices), :term[Boundaries]{canonical="Boundaries"} (it is unique), and :term[Scalability]{canonical="Scalability"} (it can be copied perfectly).

  > Sidenote:
  > - [111: Concept/Life](./111_concept_life.md)

- :dfn[Scalability]{canonical="Scalability" href="./111_concept_life.md"}: This is the superpower of being able to be copied perfectly, over and over again. It’s like a computer file—you can send it to a million people and every copy is exactly the same. This allows an idea to be in many places at once and last forever.

  > Sidenote:
  > - [111: Concept/Life](./111_concept_life.md)

- :dfn[Branch]{canonical="Branch" href="./108_concept_visibility.md#branches-publication-and-partitioning"}: Imagine a big project is like the trunk of a tree. A Branch is a new branch that grows off of it. It’s a safe, separate copy where you can try out new things without changing the main project. When your experiment is done, you can merge it back into the trunk.

  > Sidenote:
  > - [108: Concept/Visibility](./108_concept_visibility.md)

- :dfn[Call]{canonical="Call" href="./004_agent_call.md"}: If a `Tool` is a general recipe for “making lunch,” a Call is the specific command: “Make lunch with a turkey sandwich and an apple.” It’s a direct order for a `Tool` to do a specific job with all the details filled in.

  > Sidenote:
  > - [004: Agent/Call](./004_agent_call.md)

- :dfn[Cutoff Time]{canonical="Cutoff Time" href="./108_concept_visibility.md#the-cutoff-time-temporal-retrieval"}: This is like a time machine for ideas. When you ask for an :term[Idea]{canonical="Idea"}, you can add a Cutoff Time to say, “Show me the version of this Idea exactly as it was last Tuesday at 3:00 PM.” It lets you travel to the past to see a specific version.

  > Sidenote:
  > - [108: Concept/Visibility](./108_concept_visibility.md)

- :dfn[Method]{canonical="Method" href="./004_agent_call.md"}: This is simply the way a `Call` gets its job done.
  - :dfn[Explicit Execution]{canonical="Explicit Execution" href="./003_agent_activity.md"}: A real piece of code (called an `Activity`) does the work. This is predictable and reliable, like using a calculator.

    > Sidenote:
    > - [003: Agent/Activity](./003_agent_activity.md)

  - :dfn[Latent Execution]{canonical="Latent Execution" href="./104_concept_latent.md"}: The AI simply thinks of the answer. It uses its vast knowledge to create the result, which is more like using imagination or intuition.

    > Sidenote:
    > - [104: Concept/Latent](./104_concept_latent.md)

- :dfn[Delegate]{canonical="Delegate" href="./014_agent_delegate.md"}: Imagine you need a secret job done. You can `Delegate` it to a helper by putting them in a locked “clean room.” You give them only the exact information they need (`_scopes`) and nothing else. This keeps the job separate and safe from any mix-ups.

  > Sidenote:
  > - [014: Agent/Delegate](./014_agent_delegate.md)

- :dfn[Scope]{canonical="Scope" href="./015_agent_scopes.md"}: A Scope is like a keycard that gives an AI access to a specific piece of information it needs for a job. By providing only the right keycards (`_scopes`), you can focus the AI's attention on just the important details.

  > Sidenote:
  > - [015: Agent/Scopes](./015_agent_scopes.md)

- :dfn[Loop]{canonical="Loop" href="./010_agent_loop.md"}: This is when an AI solves a hard problem step-by-step. It makes one :term[Request]{canonical="Request"}, checks the result, and then decides what to do next. It keeps going around in this Loop until it has enough information to give the :term[Final Output]{canonical="Final Output"}.

  > Sidenote:
  > - [010: Agent/Loop](./010_agent_loop.md)

- :dfn[Output Path]{canonical="Output Path" href="./008_agent_output.md"}: This is like putting a label on a folder. When a :term[Tool]{canonical="Tool"} creates a result, the `_outputPath` tells the system exactly where to file it away so other steps can find and use it later.

  > Sidenote:
  > - [008: Agent/Output](./008_agent_output.md)

- :dfn[Final Output]{canonical="Final Output" href="./008_agent_output.md"}: This is the final answer! After an AI finishes all its thinking and step-by-step work in a :term[Loop]{canonical="Loop"}, this is the completed result that you originally asked for.

  > Sidenote:
  > - [008: Agent/Output](./008_agent_output.md)

- :dfn[Meta Message]{canonical="Meta Message" href="./016_agent_meta.md"}: This is like an ID card for an :term[Idea]{canonical="Idea"}. It’s a message that tells the AI precisely what it's working on, like, “This is the ‘Chocolate Cake Recipe,’ version 2.”

  > Sidenote:
  > - [016: Agent/Meta](./016_agent_meta.md)

- :dfn[Meta Properties]{canonical="Meta Properties" href="./016_agent_meta.md"}: This is the actual information written on the :term[Idea]{canonical="Idea"}'s ID card. It includes its official name, the family of ideas it belongs to, and its version number.

  > Sidenote:
  > - [016: Agent/Meta](./016_agent_meta.md)

- :dfn[HITL (Human-in-the-Loop)]{canonical="HITL (Human-in-the-Loop)" href="./010_agent_loop.md#human-in-the-loop-hitl"}: This means “pause and wait for a person.” It’s a checkpoint where an AI working on a task shows its plan to a human. The person can approve it, make changes, or cancel it before the AI continues. It’s like having an adult double-check your work.

  > Sidenote:
  > - [010: Agent/Loop](./010_agent_loop.md#human-in-the-loop-hitl)

- :dfn[Hierarchical Versioning]{canonical="Hierarchical Versioning" href="./108_concept_visibility.md#versioning-creating-the-states-to-be-seen"}: A smart way to name different versions of an :term[Idea]{canonical="Idea"}. Instead of just `v1` and `v2`, a name like `1.2.feature-x.3` can tell you a whole story, like “This is the third try of a new feature that was built on top of version 1.2.” It organizes everything like a family tree.

  > Sidenote:
  > - [108: Concept/Visibility](./108_concept_visibility.md)

- :dfn[Evolution]{canonical="Evolution" href="./106_concept_evolution.md"}:This is when a system built with AI can teach itself and get better on its own. It learns from what works and what doesn't, and can even change its own programming to improve. It’s like a character in a game who can redesign their own special powers.

  > Sidenote:
  > - [106: Concept/Evolution](./106_concept_evolution.md)

- :dfn[Emergent Identity]{canonical="Emergent Identity" href="./107_concept_identity.md"}: An :term[Idea]{canonical="Idea"}'s true meaning doesn’t come from the idea alone, but from how it’s connected to other ideas. Think of a single star in the sky. By itself it’s just a dot of light. But when you see it connected to other stars, they form a constellation, like the Big Dipper. The identity of the “Big Dipper” comes from that pattern.

  > Sidenote:
  > - [107: Concept/Identity](./107_concept_identity.md)

- :dfn[idea:]{canonical="idea:" href="./110_concept_addressing.md"}: Just like `http://` is the address for a website, `idea:` is the special address used to find any :term[Idea]{canonical="Idea"} in this entire system. It helps you ask for exactly the version you need.

  > Sidenote:
  > - [110: Concept/Addressing](./110_concept_addressing.md)

- :dfn[Identity Relationships]{canonical="Identity Relationships" href="./107_concept_identity.md#the-relational-fabric-of-identity"}: These are the connections that link different :term[Ideas]{canonical="Idea"} together, allowing a bigger pattern or :term[Emergent Identity]{canonical="Emergent Identity"} to appear.
  - :dfn[Lineage]{canonical="Lineage" href="./107_concept_identity.md#the-relational-fabric-of-identity"}: The family tree of an :term[Idea]{canonical="Idea"}, showing its history of past and future versions.
  - :dfn[Causality]{canonical="Causality" href="./107_concept_identity.md#the-relational-fabric-of-identity"}: A link showing cause-and-effect, where one :term[Idea]{canonical="Idea"} directly caused another one to be created.
  - :dfn[Grouping]{canonical="Grouping" href="./107_concept_identity.md#the-relational-fabric-of-identity"}: A way to bundle many related :term[Ideas]{canonical="Idea"} together, like putting them all into a single folder.

  > Sidenote:
  > - [107: Concept/Identity](./107_concept_identity.md)

- :dfn[State Message]{canonical="State Message" href="./009_agent_state.md"}:This is a message that carries the memory of an AI's work. As the AI works on a problem step-by-step, the State Message is updated to remember everything it has figured out so far.

  > Sidenote:
  > - [009: Agent/State](./009_agent_state.md)

- :dfn[Data Message]{canonical="Data Message" href="./005_agent_data.md"}: This is a note containing a key piece of information that stays the same throughout an AI's whole thinking process. It provides important background that the AI can always refer back to.

  > Sidenote:
  > - [005: Agent/Data](./005_agent_data.md)

- :dfn[Plan]{canonical="Plan" href="./012_agent_plan.md"}: A Plan is a map of what an AI intends to do. It shows all the steps and :term[Tool Calls]{canonical="Tool Call"} the AI will use to reach its goal. The AI can check and update this map as it works.

  > Sidenote:
  > - [012: Agent/Plan](./012_agent_plan.md)

- :dfn[Process Idea]{canonical="Process Idea" href="./012_agent_plan.md"}: This is a complete package that holds both the AI’s strategic [Plan](./012_agent_plan.md) and its current progress. Its recipe (`schema`) is the list of available [Tools](./002_agent_tool.md), its finished dish (`solution`) is the plan for the very next step, and its notes (`context`) contain a log of everything that has happened so far.

  > Sidenote:
  > - [012: Agent/Plan](./012_agent_plan.md)

- :dfn[Vessel Idea]{canonical="Vessel Idea" href="./202_idea_vessel.md"}: This is a special :term[Idea]{canonical="Idea"} that holds both a list of all possible actions and the one specific action that was chosen. Its recipe (`schema`) lists every `Tool` available, while its final dish (`solution`) records the exact `Calls` that were made when it had to react to something.

  > Sidenote:
  > - [202: Idea/Vessel](./202_idea_vessel.md)

- :dfn[Instancing]{canonical="Instancing" href="./013_agent_instancing.md"}: This is a way for an AI to handle many different jobs at the same time, without getting them mixed up. Each job is an independent `Instance` with its own unique ID and its own memory (`State` message).

  > Sidenote:
  > - [013: Agent/Instancing](./013_agent_instancing.md)

- :dfn[Request]{canonical="Request" href="./001_agent_request.md"}: A Request is a single, complete job for an AI. You give it some background information (`context`) and a goal (`schema`), and it gives you back a final answer (`solution`).

  > Sidenote:
  > - [001: Agent/Request](./001_agent_request.md)

- :dfn[Instance]{canonical="Instance" href="./013_agent_instancing.md"}: An Instance is one specific, unique job that an AI is working on as part of a larger batch. It has its own memory and information that keeps it separate from all the other jobs.

  > Sidenote:
  > - [013: Agent/Instancing](./013_agent_instancing.md)

- :dfn[Reactor]{canonical="Reactor" href="./303_ideator_reactor.md"}: This is a special `Idea Transformer` that acts like a game engine. It’s built to run any game or process that happens in turns. You give it the current state of a game (`Idea`), and it gives you back the very next state (`Idea`).

  > Sidenote:
  > - [303: Ideator/Reactor](./303_ideator_reactor.md)

- :dfn[Schemistry]{canonical="Schemistry" href="./800_package_schemistry.md"}: This is a tool that lets a single blueprint (the **Schema**) define all the important rules for a computer program. From that one blueprint, Schemistry automatically creates data checks, code types, and instructions for AIs, making sure everything in the program fits together perfectly.

- :dfn[Variable Reference]{canonical="Variable Reference" href="./007_agent_variables.md"}: This is a special shortcut. When telling a :term[Tool]{canonical="Call"} what to do, you can use a special code (like `†data.name`) to tell it: “Don’t use this exact text, go find the value from the AI’s memory and plug it in here.”

  > Sidenote:
  > - [007: Agent/Variables](./007_agent_variables.md)
