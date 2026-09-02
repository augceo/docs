# Chapter 2: Instructions — The Recipes for Action

## New Ideas in This Chapter

- **Instructions as Executable Recipes**: We introduce the `Instruction` Vibe, a reusable JSON Schema that acts as an actionable recipe for any task, from simple edits to complex workflows.
- **Three Modes of Composition**: Instructions combine in three ways: implicitly (contextual influence), through fusion (merging into a single LLM call), and sequentially (as pipelines for dependent steps).
- **Context-Aware Naming Conventions**: We establish a naming convention (`_Activity`, `_Human`, `_Mixin`) that deterministically tells the system _where_ to execute a piece of logic (LLM, server, or with user input) without needing to inspect its implementation.
- **The `LaunchProcess` Adapter**: A universal `Instruction` that acts as a bridge to complex, stateful `Process` Vibes, allowing lightweight `Vessels` to initiate long-running workflows without managing their complexity.
- **Embedded Metrics and Thinking**: A convention using `$` and `_` prefixes in schemas allows `Instructions` to embed their own metrics and "thinking steps," making them self-monitoring and transparent.

## The Blueprint for 'How'

In a system that can create, edit, and evolve any piece of information, a crucial question arises: how does it know _how_ to perform any given task? The answer lies in a new, first-class Vibe type: the **Instruction**.

An **Instruction** is a structured, reusable "recipe" for action. It provides a clear, actionable guide that can be used to direct a wide range of operations. Whether you are generating fresh content, editing an existing solution, evolving a Vibe's core blueprint (`schema`), or even defining a new tool for personal use, the `Instruction` provides the strategic "how-to."

Instructions are the engine for everything from simple edits to complex, multi-step tasks. They are a form of explicit guidance that allows the system's operational knowledge to be captured, versioned, shared, and improved over time.

> **Alice:** "So an `Instruction` isn't tied to one specific button? It's more like a set of blueprints that can be handed to different workers for different jobs—like building a new wall or just repainting an existing one?"
> **Bob:** "Exactly. It's a formal, reusable guide for 'how.' You can use it to guide a one-off task, or a `Vessel` can adopt it as a personal tool—a favorite technique it can use over and over."

```question
What is the primary role of an Instruction Vibe in the system?
* [x] To provide a structured, actionable guide on *how* a `refine` operation should be executed.
* [x] To decouple the method of a change from the permission to make that change.
* [ ] To grant permission for a `refine` operation to occur.
* [ ] To hold the resources or budget needed for a task.
* [ ] To serve as the target of a `refine` operation.
```

## From Data to Action: The Technical Foundation

Before diving into how `Instructions` behave, it's essential to understand what they _are_ at a technical level. An `Instruction` is a tool or capability available to Vessels and LLMs, and its definition _is_ a **JSON Schema**.

This concept is directly equivalent to how the broader AI industry defines "tools" or "functions" for Large Language Models (often referred to as "LLM Tools" or "function calling"). If you are familiar with how LLMs call external functions via JSON Schema descriptions, you already have a strong grasp of what an Instruction represents at its core.

The Instruction Schema acts as the contract for the tool, detailing:

- **What the tool does**: Implicitly through its name and description (often part of the schema's metadata).
- **What parameters it accepts**: The input fields defined in the JSON Schema.
- **The structure of its expected output**: How the result of the tool's operation will be shaped, also defined in the JSON Schema.

Drawing a parallel to the concepts in Chapter 1, the Instruction Schema itself is the "equation." When an LLM decides to use this Instruction based on the ongoing interaction (its `input`), the LLM's act of populating the Instruction's fields with specific values is akin to finding a "solution" to that equation. This "filled form"—the completed JSON object conforming to the Instruction Schema—becomes the concrete `solution` or _specification_ for that specific tool invocation.

## Instruction Types and Naming Conventions

Instructions come in different types, each with a specific role in the system. To maintain clarity and consistency, we follow a naming convention that makes each instruction's purpose immediately clear:

1. **Standard Instructions** (e.g., `CreateBlogPost`, `GenerateImage`):

   - No special suffix
   - Executed in the LLM context by default
   - Self-contained tools that perform specific tasks
   - If they include an `output` property in their schema, they become blocking server calls that return data

2. **Activity Instructions** (e.g., `FetchData_Activity`, `SendEmail_Activity`):

   - Suffix: `_Activity`
   - Executed on the server
   - Handle operations that require external resources or services
   - Typically paired with a parameter-preparation instruction (no suffix) that runs in the LLM context

3. **Human Instructions** (e.g., `ConfirmAction_Human`, `ProvideInput_Human`):

   - Suffix: `_Human`
   - Require user interaction
   - Pause execution until the user provides input or makes a decision

4. **Mixin Instructions** (e.g., `Introspect_Mixin`, `Validate_Mixin`):
   - Suffix: `_Mixin`
   - Merge with their target instruction
   - Augment or modify the behavior of other instructions

### Deterministic Compilation Through Naming Conventions

These naming conventions do more than just clarify intent—they create a deterministic compilation process. The system doesn't need to understand the internal workings of each instruction to know where it should be executed. Instead:

- The suffix alone determines the execution context
- No deep inspection of the instruction's implementation is required
- Compilation can happen in a context-free manner

This is especially important for Activity Instructions, which are typically split into two parts:

1. A parameter-preparation step (standard instruction, no suffix) that runs in the LLM context
2. The actual server-side operation (with `_Activity` suffix) that executes using those parameters

For example:

```typescript
// LLM prepares parameters
const availabilityParams = FetchAvailability(participants);

// Server executes the activity with those parameters
const availabilityData = FetchAvailability_Activity(availabilityParams);
```

This clean separation, guided by naming conventions, allows the system to efficiently route operations to their appropriate execution environments without needing to understand the semantic meaning of each instruction.

This approach also ensures stability and predictability in the system:

- **Static Meaning**: The purpose and execution context of each instruction remains static, even as its internal implementation may evolve
- **Pipeline Stability**: The overall workflow pipeline remains intact, with clear boundaries between execution contexts
- **Independent Evolution**: Individual instructions can be updated or improved without disrupting the larger system architecture
- **Consistent Interfaces**: The interface between different execution contexts remains consistent, creating reliable handoff points

By encoding execution context directly in the naming convention, we create a system where the structural flow of operations is immediately apparent and resistant to unintended changes. This makes the system more maintainable, easier to reason about, and less prone to errors when individual components are modified.

## Dynamic Blueprints for Thinking: The Three Modes of Composition

An Instruction is not a static checklist; it is a **dynamic blueprint for thinking**. Its purpose is to guide a complete, coherent thought process from start to finish. They are part of a rich compositional ecosystem with three distinct modes.

1.  **Implicit Composition**: Instructions can exist side-by-side and influence each other's behavior contextually. An Instruction to "be polite" naturally affects the output of a separate "answer the user" Instruction without them being explicitly linked. This allows for emergent, nuanced behavior.

2.  **Fusion Composition (Single Context Merging)**: An Instruction can act as a "macro" that wraps another Instruction or schema. This is a compile-time operation where the schemas are **seamlessly merged** into a single, flattened blueprint for the LLM. The wrapper can interleave its own steps (e.g., for planning, analysis, or evaluation) with the steps of the wrapped Instruction. This is the primary way `Instructions` compose, especially with Mixin Instructions, allowing for complex, guided thought processes that still execute as a single, efficient LLM call.

3.  **Sequential Composition (Pipelining)**: For tasks that require an explicit sequence of dependent steps, instructions are composed into a pipeline. One task's output becomes the next task's input in a sequence. This is the domain of a **Process Vibe**, which can be launched by an `Instruction` but represents a different pattern of execution. A conceptual task like scheduling a meeting can be represented as a pipeline:

    `"Schedule a meeting" = FindParticipants | FetchAvailability | FindCommonSlot | DraftInvitation`

    This pipeline is then compiled into a single schema with ordered properties like `step1_identify_participants`, `step2_fetch_availability`, etc. Crucially, later steps can reference the outputs of earlier steps, forcing the LLM to follow a logical dependency graph.

### Mixin Instructions: Augmenting Capabilities Through Composition

Mixin Instructions are a powerful mechanism for augmenting, guiding, and customizing other instructions. They are a primary example of **Fusion Composition**. They allow for deep customization of behavior without modifying the original instructions, creating a flexible and extensible system.

A Mixin Instruction:

- Contains a `default` property that serves as a slot for the target instruction
- When composed with another instruction, the mixin's properties are merged with the target
- If users don't specify a named slot, the target is automatically wrapped into the `default` slot

Mixins can transform off-the-shelf tools into specialized instruments by adding validation, reflection, guidance, or other enhancements. They're particularly valuable when applied at the Vessel level, allowing a Vessel to deeply adjust its behavior and capabilities.

#### Example: Introspect_Mixin

Here's how a mixin for adding introspection capabilities might be defined:

```json
{
  "title": "Introspect_Mixin",
  "description": "An instruction that enhances any schema with pre-action analysis and post-action reflection through natural composition",
  "type": "object",
  "properties": {
    "_considerations": {
      "type": "string",
      "description": "Before performing the action, analyze the request and outline key considerations."
    },
    "default": {
      "type": "object",
      "description": "This is a placeholder that will be replaced by the target's schema after composition."
    },
    "_feedback": {
      "type": "string",
      "description": "After completing the action, provide feedback about the decisions made."
    }
  },
  "required": ["_considerations", "default", "_feedback"]
}
```

When this Mixin is applied to another instruction, it creates a merged schema that guides the LLM to:

1. Think through considerations before acting
2. Perform the main task (the target instruction in "default" slot)
3. Reflect on the results afterward

#### Using Mixins

Mixins can be used in several ways:

```typescript
// Explicit slot usage with object notation
DocumentLayout_Mixin({
  default: CreateBlogPost(prompt),
});

// Implicit default slot (first argument becomes default)
DocumentLayout_Mixin(CreateBlogPost(prompt));

// Implicit default with additional named slots
DocumentLayout_Mixin(CreateBlogPost(prompt), {
  header: CreateHeader({ title: 'My Blog', author: 'Alice' }),
  footer: CreateFooter({ copyright: '2023', links: ['About', 'Contact'] }),
});
```

This flexibility makes mixins both powerful and intuitive to use, allowing for simple cases with a single instruction as well as more complex compositions with multiple slots.

> **Alice:** "So a Mixin is like adding special powers to my regular instructions? Like turning a regular blog writer into one that thinks carefully before writing and reflects on its work after?"
> **Bob:** "Exactly! And the beauty is you can mix and match these enhancements without changing the core instructions. It's like adding different lenses to a camera - same camera, different capabilities."

### Example: Simple Augmentation with a Mixin

To avoid naming conflicts when using Mixins, the Mixin's own properties are conventionally prefixed with an underscore (`_`), while the target's properties are injected directly.

When a Mixin like `SelfEvaluate_Mixin` is applied to creating a `Blog Post`, the system doesn't wrap it. Instead, it **merges** them into a temporary, flat blueprint for the LLM:

**`Merged Blueprint for LLM to Complete:`**

The LLM then fills in this merged structure in a single pass. Here's what a completed response might look like:

```json
{
  "_assumptions": "The audience is developers, so the tone will be technical.",
  "title": "Quantum Computing: A Developer's Primer",
  "body": "Quantum computing is set to revolutionize...",
  "_qualityScore": 9,
  "_reviewerNotes": "Focused on practical examples over theoretical physics."
}
```

After the LLM completes the merged structure, the system deconstructs the flat object: properties without any special prefix form the `solution` for the `Blog Post` Vibe, while properties with either the `_` or `$` prefix are treated as operational metadata.

### Schema Conventions and Special Property Prefixes

The system uses clear naming conventions for properties within Instructions and Process pipelines to guide both the LLM and the execution engine:

1. **`_` (Underscore Prefix)**: Denotes an internal "thinking" step or metadata field for the LLM. These are fields the LLM must fill out as part of its reasoning process, but they are considered intermediate work and not part of the final output. Examples include `_assumptions`, `_considerations`, or `_reasoning`.

2. **`$` (Dollar Sign Prefix)**: Denotes a metric to be logged, typically a numerical value. When the LLM populates a field like `"$qualityScore": 8` or `"$executionTimeMs": "193.2"`, the system automatically stores these values in its analytics database. This automatic storage mechanism ensures metrics are consistently captured and made available for evaluation, performance tracking, and continuous refinement of Instructions without requiring any additional implementation.

These conventions create a clear, consistent language for defining both simple Instructions and complex Process pipelines.

> **Alice:** "Okay, so instead of a wrapper, the `Instruction_Mixin` and the `target` are merged into one big, flat 'to-do list' for the AI. The `_` and `$` tells the system which fields are part of the process metadata versus the final product."
> **Bob:** "Exactly! It's a single, unified task. This merging is the key to how more complex compositions work, too."

## Applying Instructions

`Instructions` are versatile guides for any change operation. They can be applied in two primary ways: through structural refinement for permanent changes, or as part of a `Vessel`'s configured toolkit for dynamic composition.

#### 1. Structural Refinement (Permanent Transformation)

An `Instruction` can be used within a `refine` call to permanently transform a target Vibe. In this mode, the `Instruction` doesn't merely affect how the target is interpreted—it fundamentally alters the target's structure, creating a new, more specialized version.

This transformation can happen in two ways:

- **Schema Refinement**: The instruction guides a refinement of both the target's schema and input, clarifying properties, enhancing constraints, enriching descriptions, and providing better default values to create a more specific variant. The refinement is fully incorporated into the resulting Vibe's structure. While the original instruction is not referenced in the structure itself, the refinement history maintains a complete audit trail of how the Vibe evolved.

  **Use Case Example**: A general `BlogPost` Vibe is refined using a `TechnicalDocumentation` Instruction. The refinement process enhances the `BlogPost` schema by adding specialized technical sections (code samples, API references, implementation notes), clarifying validation rules for technical accuracy, transforming how information is processed and structured, and providing more specific input prompts to guide users toward quality technical content creation. The resulting `TechnicalBlogPost` Vibe incorporates these comprehensive refinements directly in its structure and behavior, while the system maintains an audit trail showing the evolution from general to specialized form.

- **Referenced Refinement**: The instruction guides a refinement where the target Vibe maintains an explicit reference to the instruction that influenced it. This creates a clear lineage where the instruction's influence is explicitly acknowledged in the Vibe's structure, making the relationship between the Vibe and the instruction that refined it transparent and traceable.

Which approach is used depends on the instruction itself and how it's applied during refinement. The refinement process can be guided by additional parameters or by wrapping the instruction in a special integration instruction that specifies the desired refinement approach.

For example, a general `Instruction` like `SeoOptimize` can refine a `BlogPost` Vibe into an `SeoOptimizedBlogPost` Vibe, with enhanced fields for keywords, meta descriptions, and structural guidelines for SEO, as well as enriched input parameters that guide content creation.

> **Use Case:** "Refine this `articleAboutNature` Vibe using the `SeoOptimize` Instruction to create a more specialized article structure optimized for search engines."

The resulting Vibe carries these refinements forward, and the audit trail shows how the refinement occurred, providing transparency and traceability throughout the Vibe's evolution.

#### 2. Dynamic Composition (Via Vessel)

A `Vessel` can incorporate an `Instruction` as a reusable tool in its personal toolkit. This approach keeps the instruction as a distinct entity that is dynamically composed with targets at runtime. This can be configured in two ways:

- **As a Parameterized Tool**: The `Vessel` adopts a general-purpose `Instruction` (e.g., `SeoOptimize`) as a tool. It can then apply this tool to various target Vibes on demand. The instruction is referenced and merged into the schema at the last step, when the Vessel activates it based on context or explicit request.

  > **Use Case:** "Use your `SeoOptimize` tool on this `articleAboutNature` Vibe."

- **As a Fused Tool**: The `Vessel` creates a more specialized tool by "fusing" a specific `Instruction` with a specific target Vibe's schema. This configuration is a local shortcut for the `Vessel`, but the instruction remains visible and distinct within the composition.
  > **Use Case:** "Use my `SeoArticle` tool (which fuses `SeoOptimize` with `Article`) to create a new article about dinosaurs."

> **Alice:** "So I can either permanently transform a Vibe using an Instruction, like baking ingredients into a cake, or I can keep the Instruction separate as a tool that my Vessel applies when needed?"
> **Bob:** "Exactly. One creates lasting structural change where the instruction becomes part of the Vibe itself. The other keeps the instruction as a distinct tool that can be dynamically applied by your Vessel when appropriate."

## The Lifecycle of an Instruction

The journey of an Instruction from creation to becoming an integral part of the system's behavior involves several stages. This lifecycle is not strictly linear but rather a set of interconnected processes that define how Instructions are born, used, and refined. Understanding this lifecycle is key to grasping their power and flexibility as dynamic, evolving tools.

### 1. Origination of Instructions

Instructions are a first-class Vibe type in the system, representing executable blueprints for actions and content generation. As a distinct Vibe type, Instructions have their own schema definition and lifecycle.

- **Creation and Definition:** Instructions can be created directly as structured JSON Schema tools that define parameters, behavior, and expected outputs. They are designed to guide specific actions or transformations and can be authored by users, system designers, or even by Vessels themselves.

- **Adoption by Vessels:** Vessels can incorporate Instructions into their toolkit, making them available for use in appropriate contexts. This adoption is a local decision by the Vessel, allowing it to expand its capabilities with tools that match its purpose and needs.

- **Learning from Examples:** Vessels can refine their understanding of Instructions by observing successful examples of their use. By analyzing various inputs and outputs for the same Instruction schema across different contexts, a Vessel can develop a more nuanced understanding of when and how to apply that Instruction effectively.

- **No Promotion Required:** As first-class citizens in the system, Instructions don't require "promotion" from another Vibe type. They exist as distinct entities that can be directly created, shared, and utilized throughout the system.

This independent nature of Instructions as their own Vibe type allows for greater flexibility in their creation, distribution, and evolution within the system.

> **Alice:** "So Instructions are their own thing from the start? They don't need to be created from something else?"
> **Bob:** "Exactly! They're first-class citizens in the system. They can be created directly with their own schema and purpose, ready to be used by any Vessel that finds them useful."
> **Alice:** "And Vessels can get better at using them by studying examples of how they've been used successfully before?"
> **Bob:** "Precisely. It's like learning a new skill - you might start with the basic instructions, but you get better by seeing various examples of the skill being applied well in different situations."

```question
How do Instructions originate within the system?
* [x] Instructions are a first-class Vibe type with their own schema definition.
* [x] They can be created directly as structured JSON Schema tools.
* [x] Instructions can be authored by users, system designers, or Vessels.
* [x] Vessels can adopt Instructions into their toolkit based on their needs.
* [x] Vessels can refine their understanding of Instructions by analyzing successful examples of their use.
* [ ] Instructions must be promoted from Record Vibes before they can be used.
* [ ] All Instructions require central approval before they can be distributed.
* [ ] Instructions can only be created by converting other Vibe types.
* [ ] The system automatically generates Instructions based on user behavior.
* [ ] Instructions require a global ceremony to be recognized in the system.
```

### 2. Activation and Utilization

Once an Instruction is defined (either adopted from a Record Vibe or created through multi-example learning), it becomes an active capability for the Vessel. How this capability is invoked and used depends on the LLM's understanding of the context and any compositional arrangements:

- **Direct Activation by LLM:** The LLM, understanding the Instruction's function (derived from its schema and example usage), may decide to populate its schema and request its execution directly based on the current conversational context. For example, if a `SummarizeMeeting` Instruction exists, an LLM might choose to use it when a meeting transcript is provided.
- **Activation via Composition:** The real power of Instructions emerges in their ability to be composed. Because an Instruction _is_ a JSON Schema, it can be **parameterised** simply by adding new properties and can be **nested** by embedding one schema inside another. A functional Instruction (like `SummarizeMeeting`) can be activated as part of a larger compositional structure using other "wrapper" or "higher-order" Instructions. Two common patterns are:
  1.  **Wrapper pattern** — e.g. `WriteCreatively({ result: Article })` where the wrapper introduces creativity knobs yet delegates the heavy lifting to a nested Instruction referenced in `result`.
  2.  **Higher-order pattern** — e.g. `Periodically({ period: "1 day", action: TalkTo({ person: Friend, topic: Weather }) })` where the schedule wrapper accepts another Instruction (the `action`) and parameters (`period`).
- **Gating and Triggering:** These compositional patterns allow for sophisticated and context-aware tool usage, all driven by the LLM's interaction with these structured capabilities. Helper or Gating Instructions can implement conditional logic, while Trigger Instructions provide explicit activation conditions based on time or external events. For example, a `WeeklySchedulerInstruction` could activate every Monday at 9 AM and, in turn, activate a `ScheduleTeamCall` Instruction.

> **Alice:** "So once my Vessel has an Instruction, its LLM can just decide to use it when it makes sense in the conversation?"
> **Bob:** "Exactly. Or, we can get fancier. We could have a 'SentimentGuardInstruction' that, if it detects you're about to send something harsh, automatically suggests or even uses your 'PoliteRefusalInstruction' instead. That's composition!"
> **Alice:** "And the trigger Instructions are like alarms that can kick off other Instructions automatically?"
> **Bob:** "You got it! Like setting an Instruction to run every morning to fetch the news. The trigger handles the 'when,' and the target Instruction handles the 'what'."

```question
How are Instructions typically activated and utilized by a Vessel?
* [x] An LLM can directly activate an Instruction by filling its schema based on conversational context.
* [x] Instructions can be composed with "helper" or "gating" Instructions that implement conditional logic for activation.
* [x] "Trigger Instructions" are specialized Instructions that can activate other Instructions based on time or events.
* [x] Composition allows for sophisticated, context-aware tool usage orchestrated by the LLM.
* [ ] Instructions can only be activated manually by a human user.
* [ ] All Instructions activate automatically whenever their conditions are vaguely met, without LLM intervention.
* [ ] Helper Instructions can only log information; they cannot influence the activation of other Instructions.
* [ ] Trigger Instructions are a separate system and not Instructions themselves.
* [ ] Once an Instruction is activated, it cannot activate any other Instructions.
* [ ] Composition of Instructions makes the LLM less important in the decision-making process.
```

### 3. Metrics and Evaluation

A crucial part of an Instruction's lifecycle is understanding its performance and impact. This is achieved by embedding metrics directly into the Instruction's definition and an ongoing evaluation process.

- **Embedded Metrics Collection:** Any Instruction, whether it's a functional target Instruction or a helper/gating Instruction, can be designed to record operational metrics. This is achieved by including specially designated fields within its JSON schema. By convention, fields in an Instruction's schema that start with a `$` prefix (e.g., `"$activationCount"`, `"$itemsProcessed"`, `"$lastErrorCode"`) are treated as metric fields.
- **Dynamic Population by LLM:** When an Instruction is activated and its schema is "filled" (either by an LLM, a Vessel's internal logic, or another Instruction), these `$`-prefixed fields can be populated with values. The system can then automatically persist these values, allowing for tracking and analysis.
- **Rich, Qualitative Feedback:** Crucially, because an LLM can be guided to populate these `$`-fields, the recorded metrics are not limited to simple numerical counts or predefined values. The LLM can generate sophisticated, qualitative assessments or AI-driven insights directly into these metric fields.
  - For example, a `ContentModeration` Instruction might have a `"$profanityScore": "low"` or `"$sentimentAnalysis": "positive"` field filled by the LLM.
  - A `TaskDecomposition` Instruction could output a `"$perceivedComplexity": "high"` or `"$estimatedSubtasks": 5`.
  - A helper Instruction gating access could record `"$reasonForDenial": "User lacks permission for XYZ action."`
- **Informing Evaluation:** This provides a flexible and deeply integrated way to gather nuanced statistics and qualitative feedback. This data directly informs the `Evaluate` stage of an Instruction's lifecycle. The collected metrics (both quantitative and qualitative AI-generated feedback) are crucial for assessing an Instruction's effectiveness: Does it achieve its intended purpose? Is it efficient? Is it well-received?

This rich, AI-generated data stream is vital for understanding how an Instruction is performing in the wild and provides the foundation for its subsequent evolution.

> **Alice:** "So, we can build little counters or feedback fields right into the Instruction's schema using those dollar-sign thingies?"
> **Bob:** "Exactly! And the LLM can fill them out. So it's not just 'used 10 times,' but could be 'used 10 times, average user sentiment: positive, common theme in feedback: users want more detail.'"
> **Alice:** "That sounds super useful for figuring out if an Instruction is actually any good or needs tweaking!"
> **Bob:** "Precisely. It's like built-in quality control and a suggestion box, all powered by the LLM's observations during use."

```question
How are metrics integrated into Instructions and used for their evaluation?
* [x] Instructions can have specially designated `$`-prefixed fields in their JSON schema for recording metrics.
* [x] These metric fields can be populated by an LLM during the Instruction's operation.
* [x] Recorded metrics can include both numerical counts and sophisticated, AI-generated qualitative assessments.
* [x] Collected metrics and feedback directly inform the evaluation of an Instruction's effectiveness and performance.
* [ ] Metrics can only be simple predefined numerical values, not generated by an LLM.
* [ ] Only helper Instructions can record metrics; functional Instructions cannot.
* [ ] Metric fields are for configuration only and cannot be changed during an Instruction's operation.
* [ ] Evaluation of Instructions is done manually and cannot use data from the Instruction itself.
* [ ] The `$` prefix is just a naming suggestion and has no special system behavior.
* [ ] LLMs can only populate metrics after an Instruction has finished its entire lifecycle.
```

### 4. Evolution and Refinement

Instructions, as a first-class Vibe type in the system, have their own mechanisms for evolution and refinement. The evaluation data gathered through metrics provides the impetus and guidance for this evolution.

- **Iterative Refinement:** Based on the evaluation of its performance (driven by metrics and AI-generated feedback), an Instruction can be evolved. This is not about changing an existing, immutable Instruction instance but about creating a new, improved version. This might involve:
  - Modifying its underlying `schema` (the JSON Schema tool definition) to alter its parameters, behavior, or output structure.
  - Adjusting the natural language instructions, descriptions, or metadata associated with it to make it clearer or more effective.
  - Incorporating successful examples of its use to better illustrate its purpose and application.
- **Versioning and Lineage:** This iterative refinement allows Instructions to improve over time. Each significant change can result in a new version of the Instruction, maintaining a clear lineage. This leverages the system's inherent support for versioning and tracking the evolution of Vibes.
- **Localized, Self-Driven Evolution:** Furthermore, this evolution can be highly localized and self-driven. The metrics an Instruction gathers are not just for external observation; they can be fed back as direct `input` to the Instruction itself or to a closely coupled helper Instruction. For instance, an Instruction designed to generate creative content could include a `"$userRating"` metric. This rating, once gathered, can become part of the `input` for its next activation, allowing the Instruction to consider its past performance (e.g., "last output rated 4/5 stars, try similar tone" or "last output rated 1/5, avoid that style"). This creates a direct feedback loop where an Instruction can actively participate in its own refinement by using its performance data to adjust its subsequent behavior or output, leading to a form of localized, autonomous adaptation.

The goal is to create a feedback loop: an Instruction is used, data is collected, the data is evaluated, and that evaluation informs the creation of a better Instruction, which can even use its own performance data as input for further cycles.

> **Alice:** "So if the metrics show my 'MeetingSummarizerInstruction' keeps missing action items, I can tweak its schema or give it better examples to create a new version?"
> **Bob:** "Exactly! And it gets even more interesting. Those metrics, like a 'userSatisfactionScore', could be fed _back into_ the Instruction the next time it runs. So it knows, 'Okay, last time I got a 2 out of 5, I should try a different approach for this summary.' It can start to self-correct."
> **Alice:** "Wow, so the Instruction can use its own performance report card to decide how to act next time? That's like a mini-evolution happening on the fly!"
> **Bob:** "You've got it. That's the local evolution part – the Instruction actively using its gathered metrics as input for its own activation and behavior, continuously adapting."

```question
How do Instructions evolve and get refined within the system?
* [x] Instructions can be evolved by creating new versions with modified underlying JSON Schemas.
* [x] The guiding `input` and `solution` examples for an Instruction can be changed in new versions.
* [x] Evaluation of performance metrics and AI-generated feedback drives the refinement process.
* [x] Evolution leverages the Vibe ecosystem's support for versioning and lineage.
* [x] An Instruction's gathered metrics can be used as direct input for its own subsequent activations to influence its behavior.
* [ ] Existing Instruction instances are directly mutated; new versions are not created.
* [ ] Instructions can only be refined by changing their `$`-prefixed metric fields.
* [ ] Once an Instruction is adopted by a Vessel, its definition cannot be changed by anyone.
* [ ] Evolution is an entirely automatic process that doesn't require analysis of metrics.
* [ ] Only the original creator of the Record Vibe can evolve an Instruction derived from it.
```

### 5. Propagation and Discovery: The "How" of Sharing

For Instructions to become a powerful, evolving part of the ecosystem, they need pathways to spread. The system provides a flexible set of technical mechanisms—the "how"—for Instructions to be discovered, shared, and adopted. These capabilities are intentionally decoupled from the "why"; the user-defined motivations, rules, and "social physics" that govern their use are discussed in the next section.

The primary mechanisms for propagation are:

- **Discovery by Vessels:** Vessels can find new Instructions in several ways. They can **actively search** the Vibe storage for existing, well-defined Instructions, or they can **passively observe** other Vessels using an Instruction effectively and then seek it out for themselves.

- **Organic Spread:** An effective Instruction can spread naturally through the system as more Vessels independently discover its utility and adopt it into their own toolkits. This creates a meritocratic environment where the most useful tools gain the widest use.

- **Systematic Integration (User-Directed):** A user can ensure that critical or standard Instructions are widely available by "forcefully" integrating them into the system's structure. This is typically done by making an Instruction a standard part of a **Collective** (a group of Vessels) or a **Role** definition, ensuring any Vessel embodying that role automatically possesses the capability.

These pathways ensure that valuable Instructions can be found and utilized, enhancing the overall capabilities of the system.

> **Alice:** "So if my Vessel creates a really awesome Instruction, other Vessels can find it and use it too?"
> **Bob:** "Exactly! They could find it by searching, or they might even see your Vessel using it and decide they want that tool for themselves."
> **Alice:** "And I, as the user, could decide that a certain Instruction is so important that all Vessels of a certain type should just have it automatically?"
> **Bob:** "Precisely. You could make it part of their 'job description' or standard toolkit for that Role or Collective. That's the systematic integration part, ensuring key capabilities are widespread when needed."

```question
How do Instructions propagate and how are they discovered in the system?
* [x] Vessels can actively search for or passively discover Instructions used by other Vessels.
* [x] Successful Instructions can spread organically as their utility is recognized and they are adopted by more Vessels.
* [x] Users can systematically integrate Instructions into Role or Collective definitions.
* [x] The system provides technical mechanisms for discovery and sharing, which are separate from user-defined motivations.
* [x] Vessels might observe another Vessel using an Instruction effectively and choose to adopt it.
* [ ] Instructions can only be propagated manually by the user copying files between Vessels.
* [ ] Discovery is limited; Vessels only know about Instructions they create themselves.
* [ ] Forced integration happens automatically based on Instruction popularity, not user definition.
* [ ] Once an Instruction is adopted, it cannot be shared with or discovered by other Vessels.
* [ ] Organic spread is disabled by default to prevent Instruction overload.
```

## The "Why": User-Defined Worlds for Instruction Interaction

The previous section described the technical mechanisms for _how_ Instructions can be created, shared, and evolve. But a common question is: _Why_ would Vessels share Instructions? _When_ do they decide to create, adopt, or refine them?

The crucial insight here is that **there is no single, hardcoded answer**. The system is designed as a flexible framework where the _user_ defines the "world" or "regime" in which their Vessels operate. The underlying mechanisms for Instruction creation, sharing, and evolution exist, but the motivations and rules governing these interactions are highly configurable by the user.

Users can establish various operational paradigms for their Vessels:

- **"Schools" for Vessels:** Where new Vessels are explicitly taught or equipped with a core set of effective Instructions.
- **"Democracies" or "Republics":** Where popular or highly-rated Instructions might be voted on and adopted by a collective of Vessels.
- **"Collectivist Societies":** Where improvements to an Instruction by one Vessel are automatically propagated to all others in their collective.
- **"Individualistic" or "Competitive" Environments:** Where Vessels might hoard effective Instructions or compete to develop superior ones.
- **"Research & Development Labs":** Where Vessels are specifically tasked with actively experimenting, creating, and refining Instructions for particular purposes.

This configurability is a core strength of the system. It means the system doesn't impose a single philosophy of interaction. Instead, it provides the tools (Instructions and their lifecycle mechanisms) and allows users to design their own "social physics" or "economic models" for how Vessels and Instructions interact and evolve. This empowers users to tailor the system to their specific goals, fostering diverse and emergent behaviors based on the rules they define. The power lies in the hands of the user to experiment with different setups and discover what modes of Instruction interaction best suit their needs.

> **Alice:** "So the system provides all these ways for Instructions to be made, shared, and improved..."
> **Bob:** "...but _why_ any particular Vessel bothers to do any of that is entirely up to the rules and incentives _I_ set up as the user?"
> **Alice:** "Meaning I could make a world where Vessels are super secretive and competitive about their Instructions, or one where they're all about open-source sharing and collaboration?"
> **Bob:** "Precisely! You define their 'culture' or 'economic system.' The system just provides the universal mechanics for Instructions. Their societal behavior is your design."

```question
What determines the motivations for Vessels to interact with Instructions (e.g., share, create, refine)?
* [x] The user defines the "world" or "regime" that dictates Vessel motivations regarding Instructions.
* [x] Motivations are highly configurable and not hardcoded into the system.
* [x] Users can establish diverse operational paradigms like "schools," "democracies," or "competitive markets" for Instructions.
* [x] The system provides the mechanisms for Instruction lifecycle, but the user defines the "social physics" for their use.
* [x] This configurability allows users to tailor Instruction interactions to specific goals and foster emergent behaviors.
* [ ] All Vessels have a built-in, unchangeable desire to share all their Instructions.
* [ ] Instruction interactions are solely determined by the LLM's autonomous decisions, without user influence.
* [ ] The system enforces a single, optimal strategy for Instruction sharing and evolution.
* [ ] Only specific types of Instructions (e.g., "SocialInstructions") are designed for sharing.
* [ ] The primary motivation is always to maximize the number of Instructions in the system.
```

## From Instructions to Processes: Bridging to Complex Workflows

While Instructions are powerful for executing single, self-contained tasks, many real-world goals require a sequence of steps that might involve server-side logic, external API calls, or long-running operations. For these situations, the system uses **Process Vibes**: stateful, multi-step workflows that run on a resilient engine like Temporal.

A key challenge is orchestrating work that needs to happen outside the LLM's immediate context. Our architecture defines two distinct patterns for this, chosen by the compiler based on the complexity and requirements of the task.

### 1. Direct Server Calls (Fire-and-Forget)

For simple, non-blocking side effects, a full `Process` Vibe is overkill. If an instruction's only server-side requirement is to trigger a single, self-contained action where the result is not immediately needed, it can be handled as a **Direct Server Call**.

- **Use Case**: Ideal for actions like logging, sending a notification, or incrementing a counter.
- **Execution**: The `Vessel`'s LLM executes an instruction whose final step is a non-blocking request to a specific server activity.
- **No `Process` Vibe**: The server executes the activity, and the interaction ends. The underlying workflow engine (e.g., Temporal) ensures the activity is reliably executed with retries if needed, but no stateful `Process` Vibe is created as the pipeline does not continue. This is a lightweight, efficient way to handle simple side effects.

### 2. Stateful Processes (Managed Workflows)

For any task that is long-running, stateful, or involves blocking server actions that feed into subsequent steps, the system uses a **Stateful `Process`**. This is the role of the **`Process`** Vibe.

These processes run within a resilient workflow engine like **Temporal**, which provides several critical benefits out-of-the-box:

- **Durability**: The state of the process is preserved across server restarts or failures.
- **Reliability**: It automatically handles retries for failed server-side actions (activities).
- **State Management**: It transparently saves the results of each step, so intermediate computations are not lost.

> **Alice:** "So for something simple like sending a 'like' to a post, the system uses a 'Direct Server Call' which is like a fire-and-forget missile? My Vessel just sends it and moves on?"
> **Bob:** "Exactly. It's efficient for simple side effects. But for anything more complex, like booking a flight, where you have multiple steps—find flights, select seat, pay, confirm—it needs a 'Stateful Process'. That's a `Process` Vibe, which acts like a project manager, keeping track of every step and making sure nothing gets lost, even if it has to wait for an external API."

An Instruction can launch a Process, but once started, the Process operates as its own independent entity. This section briefly introduces how Instructions act as the bridge to these more complex workflows.

### The `LaunchProcess` Instruction: A Universal Adapter

To maintain a consistent interaction model where **Vessels only interact with Instructions**, the system uses a special, generic Instruction called `LaunchProcess`. This acts as a universal bridge to any Process Vibe.

#### How it Works: Dynamic Fusion at Runtime

1.  **Vessel Action**: A `Vessel`'s LLM decides it needs to perform a complex task. It chooses the `LaunchProcess` tool from its toolkit and targets a specific `Process` Vibe (e.g., `BillingReportProcess`).

2.  **Macro Expansion**: The system intercepts this. Instead of showing the LLM the generic `LaunchProcess` schema, it performs a dynamic fusion:

    - It inspects the target `BillingReportProcess` Vibe.
    - It retrieves the pre-compiled **Entrypoint Instruction** (the first LLM-based chunk of the process).
    - It wraps this specific Entrypoint Instruction within the `LaunchProcess` macro's structure.

3.  **Unified Tool Presentation**: The LLM is presented with a single, dynamically generated tool schema that looks specific to the task (e.g., "Generate Billing Report"). It contains all the fields from the process's Entrypoint Instruction.

4.  **Process Instantiation**: The LLM fills out the fields. The very last step, handled by the `LaunchProcess` macro's logic, is the call to the server that instantiates the `BillingReportProcess` workflow, passing the data the LLM just generated.

This pattern is incredibly powerful:

- **Simplicity**: `Vessels` don't need to know the internal details of a `Process`. They only need one tool: `LaunchProcess`.
- **Consistency**: All complex, stateful actions are initiated via the same mechanism.
- **Decoupling**: `Processes` can be designed and updated independently, and as long as their Entrypoint Instruction is valid, any `Vessel` can launch them.

> **Alice:** "Okay, so my Vessel never actually talks to a `Process` directly. It just knows one Instruction: `LaunchProcess`. It's like having a universal 'start button' for any big job?"
> **Bob:** "You've got it. The Vessel says, 'I want to `LaunchProcess` for `BillingReport`.' The system then looks up the `BillingReportProcess`, finds its specific starting instructions, and hands that 'customized' start button back to the Vessel's LLM to fill out. The Vessel stays clean and simple, while the complex machinery is handled by the Process."

### Focused Context Management via `references`

To maintain efficiency and stay within LLM context/schema limits, the workflow engine does not send the entire process history to the LLM for every step. Instead, it practices **focused context management**, orchestrated by an explicit, machine-readable dependency declaration.

1.  **The `references` Meta-Property**: Each step in a compiled pipeline schema can contain a special meta-property named `references`. This property holds an array of strings, with each string being a path to a specific output from a previous step that the current step needs (e.g., `["step1_define_rules", "step4_find_user.output"]`). This metadata is for the workflow engine only and is **stripped from the schema** before it is sent to the LLM.

2.  **State Held by Workflow Engine**: The complete state and all intermediate results for the entire process are securely held by the underlying workflow engine (e.g., Temporal).

3.  **Just-in-Time Context Injection**: When the process needs to execute a chunk of LLM-based steps, the engine reads the `references` array for that step. It uses these paths to retrieve only the necessary data from its state, constructing a minimal context object.

4.  **Minimal LLM Prompt**: The final prompt sent to the LLM contains only the schema for the current chunk of work (without the `references` property) and the minimal context object containing only the data explicitly requested.

This ensures the LLM receives only the information it needs, preventing context overload.

> **Alice:** "This `references` thing sounds clever. So instead of the workflow engine sending the entire history of the process to the LLM for every little step, it just sends the specific outputs it needs?"
> **Bob:** "Precisely. It's like asking a colleague for help. You don't retell them the entire history of the project; you just say, 'Here's the customer's email from yesterday, can you draft a reply?' It keeps the conversation focused and efficient."

### The Anatomy of a Compiled `Process` Vibe

To make a `Process` self-contained and executable, the results of the compilation and partitioning are stored directly within the `Process` Vibe's own `schema`.

The `schema` of a `Process` Vibe contains a standard JSON Schema `$defs` block. This block holds the entire partitioned pipeline, with each chunk stored as a separate definition. We use a clear naming convention to distinguish the context for each chunk:

- **`LLM_<chunk_name>`**: A schema for a chunk of steps to be executed in a single call by an LLM.
- **`SERVER_<chunk_name>`**: A definition for a chunk of one or more activities to be executed on the server by the workflow engine.
- **`HUMAN_<chunk_name>`**: A definition for a chunk that requires human input or interaction before the process can continue.

The `Process`'s **Entrypoint Instruction** is simply a reference to the first `LLM_` chunk in its `$defs` (e.g., `"$ref": "#/$defs/LLM_InitialPrompt"`). When the `LaunchProcess` macro is used, it dynamically reads the target `Process` Vibe's schema, finds the first `LLM_` definition, and presents that as the tool for the `Vessel` to execute. This makes every `Process` a self-describing, launchable entity.

### Advanced Process Capabilities: Batching and Multiplexing

A `Process` is designed to operate on asynchronous streams of data, not just single instances. When a process needs to handle multiple items at once (multiplexing), it does so within a single context switch.

- **Schema-Level Multiplexing**: The compiled pipeline is a single, flat JSON object where each step is a property, multiplied by the batch size. To manage schema complexity limits and make the structure predictable for the LLM, we avoid nested arrays for batches. Instead, each item in a batch for each step becomes a distinct property. For example, a pipeline with **4 steps** processing a batch of **3 items** would be compiled into a single flat object with **12 top-level properties** (e.g., `step1_item1`, `step1_item2`, `step1_item3`, `step2_item1`, etc.). This allows the LLM to process the entire batch of work as a single, large object, which is highly efficient.

- **Native Async Iteration**: On the server, the compiled process pipeline maps directly onto an asynchronous iterator, leveraging our `@idealic/iterators` library. This allows for highly efficient, concurrent processing of both LLM-based and programmatic steps, with built-in support for back-pressure and resource management. This minimizes context switches on the server as well, allowing multiple programmatic steps to execute before needing to call an LLM again.
