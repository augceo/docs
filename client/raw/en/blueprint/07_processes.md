# Chapter 8: Processes and Compiled Instructions

## New Ideas in This Chapter

- **Instruction Composition Modes**: We define three ways Instructions combine: implicitly (contextual influence), fusion (compile-time merging into a single LLM call), and sequential (pipelines for dependent steps).
- **Compiled & Partitioned Pipelines**: Complex workflows are not interpreted on the fly. They are designed as pipelines that are **compiled** and **partitioned** into context-aware chunks (LLM vs. Server), creating durable, efficient, and self-describing `Process` Vibes.
- **The `LaunchProcess` Adapter**: We introduce a universal `Instruction` that acts as a macro. It provides a consistent interface for `Vessels` to launch any stateful `Process`, decoupling the lightweight agent from the complex, long-running workflow.
- **Focused Context via `references`**: To manage LLM context windows, `Process` steps explicitly declare their data dependencies via a `references` meta-property. The workflow engine uses this to inject only the necessary data from the process's state into the LLM prompt, ensuring efficiency and scalability.

---

Based on our discussion, this chapter outlines a powerful architecture for defining and executing complex, multi-step workflows that seamlessly blend LLM-native cognition with robust, server-side computation. The core challenge is to create a system that is expressive, composable, and highly efficient, primarily by minimizing context-switching and intelligently handling stateful, long-running, and data-intensive tasks like batching and multiplexing.

The vision is to move beyond simple, blocking "tool calls." We aim for a model where tasks can be defined as dynamic, compilable pipelines that are intelligently partitioned to run in the most appropriate context—either as a single, consolidated request to an LLM or as a stateful, server-managed process for more complex jobs.

## The Foundation: Instructions as Composable Tools

At the heart of the system is the **Instruction**: a JSON Schema that acts as a tool or "thinking primitive" for an LLM. However, unlike traditional LLM tools, these are not just simple triggers for external actions. They are part of a rich compositional ecosystem with three distinct modes.

### The Three Modes of Instruction Composition

1.  **Implicit Composition**: Instructions can exist side-by-side and influence each other's behavior contextually. An Instruction to "be polite" naturally affects the output of a separate "answer the user" Instruction without them being explicitly linked. This allows for emergent, nuanced behavior.

2.  **Fusion Composition (Macros)**: An Instruction can act as a "macro" that wraps another Instruction or schema. This is a compile-time operation where the schemas are fused into a single, flattened blueprint for the LLM. The wrapper can interleave its own steps (e.g., for planning, analysis, or evaluation) with the steps of the wrapped Instruction. By convention, these "thinking" or metadata steps are prefixed with an underscore (`_`) to distinguish them from the final output properties. This allows for creating complex, guided thought processes that still execute as a single, efficient LLM call.

3.  **Sequential Composition (Pipelines)**: For tasks that require an explicit sequence of dependent steps, instructions are composed into a pipeline. A conceptual task like scheduling a meeting, which might be thought of as `Schedule(FindSlot(FetchAvailability(FindParticipants(p))))`, can be represented more clearly as a pipeline:

    `"Schedule a meeting" | FindParticipants | FetchAvailability | FindCommonSlot | DraftInvitation`

    This pipeline is then compiled into a single schema with ordered properties like `step1_identify_participants`, `step2_fetch_availability`, etc. Crucially, later steps can reference the outputs of earlier steps, forcing the LLM to follow a logical dependency graph.

## Executing Server-Side Logic

A key challenge is orchestrating work that needs to happen outside the LLM's immediate context. Our architecture defines two distinct patterns for this, chosen by the compiler based on the complexity and requirements of the task.

### 1. Direct Server Calls (Fire-and-Forget)

For simple, non-blocking side effects, a full `Process` Vibe is overkill. If an instruction's only server-side requirement is to trigger a single, self-contained action where the result is not immediately needed, it can be handled as a **Direct Server Call**.

- **Use Case**: Ideal for actions like logging, sending a notification, or incrementing a counter.
- **Execution**: The `Vessel`'s LLM executes an instruction whose final step is a non-blocking request to a specific server activity.
- **No `Process` Vibe**: The server executes the activity, and the interaction ends. The underlying workflow engine (e.g., Temporal) ensures the activity is reliably executed with retries if needed, but no stateful `Process` Vibe is created as the pipeline does not continue. This is a lightweight, efficient way to handle simple side effects.

> **Alice:** "So for something simple like sending a 'like' to a post, the system uses a 'Direct Server Call' which is like a fire-and-forget missile? My Vessel just sends it and moves on?"
> **Bob:** "Exactly. It's efficient for simple side effects. But for anything more complex, like booking a flight, where you have multiple steps—find flights, select seat, pay, confirm—it needs a 'Stateful Process'. That's a `Process` Vibe, which acts like a project manager, keeping track of every step and making sure nothing gets lost, even if it has to wait for an external API."

### 2. Stateful Processes (Managed Workflows)

For any task that is long-running, stateful, or involves blocking server actions that feed into subsequent steps, the system uses a **Stateful `Process`**. This is the role of the **`Process`** Vibe.

These processes run within a resilient workflow engine like **Temporal**, which provides several critical benefits out-of-the-box:

- **Durability**: The state of the process is preserved across server restarts or failures.
- **Reliability**: It automatically handles retries for failed server-side actions (activities).
- **State Management**: It transparently saves the results of each step, so intermediate computations are never lost.

#### The Process Lifecycle: Design, Compilation, and Execution

The journey from a complex idea to an executable `Process` is not an on-demand event but a structured design lifecycle.

1.  **Design & Analysis**: A new `Process` is designed as a pipeline. During this design stage, the pipeline is compiled. The compiler analyzes the entire dependency graph to determine the execution pattern.

    - If the pipeline resolves to a single, terminal, non-blocking server action, it generates a "Direct Server Call Instruction".
    - For all other cases, it partitions the pipeline into **context-aware chunks** (LLM vs. Server) and proceeds with generating a `Process` entrypoint.

2.  **Entrypoint Generation**: (For Stateful Processes) The compiler extracts the first purely LLM-based chunk of the pipeline. This becomes the `Process`'s **Entrypoint Instruction**—a compact, self-contained schema that can be integrated directly into a `Vessel`'s toolkit.

3.  **Vessel-Initiated Execution & Process Instantiation**: A `Vessel`'s LLM executes the `Entrypoint Instruction`. The very last step of this instruction is a request to the server to start the full workflow. The server-side workflow engine receives this request and **immediately instantiates the `Process` Vibe**.

4.  **Process-Led Execution**: From this moment on, the `Process` Vibe exists and controls the entire execution flow. Its first action is to make the first blocking server call (e.g., the database query). Because the process is durable, it will wait as long as needed for the result. When the server action completes, the result is transparently passed back to the running process. The process can also execute non-blocking, fire-and-forget steps (like the Direct Server Calls described above) as part of its flow without waiting for their completion. The `Vessel` is no longer involved.

This model allows `Vessels` to remain lightweight and responsive, initiating complex processes without being burdened by their full complexity. The state, intermediate results, and long-running logic are reliably managed by the dedicated `Process` Vibe and its underlying workflow engine.

#### Focused Context Management via `references`

To maintain efficiency and stay within LLM context/schema limits, the workflow engine does not send the entire process history to the LLM for every step. Instead, it practices **focused context management**, orchestrated by an explicit, machine-readable dependency declaration.

1.  **The `references` Meta-Property**: Each step in a compiled pipeline schema can contain a special meta-property named `references`. This property holds an array of strings, with each string being a path to a specific output from a previous step that the current step needs (e.g., `["step1_define_rules", "step4_find_user.output"]`). This metadata is for the workflow engine only and is **stripped from the schema** before it is sent to the LLM.

2.  **State Held by Workflow Engine**: The complete state and all intermediate results for the entire process are securely held by the underlying workflow engine (e.g., Temporal).

3.  **Just-in-Time Context Injection**: When the process needs to execute a chunk of LLM-based steps, the engine reads the `references` array for that step. It uses these paths to retrieve only the necessary data from its state, constructing a minimal context object.

4.  **Minimal LLM Prompt**: The final prompt sent to the LLM contains only the schema for the current chunk of work (without the `references` property) and the minimal context object containing only the data explicitly requested.

This ensures the LLM receives only the information it needs, preventing context overload. Let's see it in action.

##### Example: Data Flow with `references`

After a blocking step `step4_find_user` completes, the next step `step5_draft_email` is defined with its data dependencies in the `references` array:

```json
"step5_draft_email": {
    "description": "Draft a personalized email to the user found in the previous step.",
    "references": [ "step4_find_user.output" ],
    "type": "object",
    "properties": {
        "recipientName": {
            "description": "Use the 'userName' field from the 'step4_find_user.output' object provided in the prompt context.",
            "type": "string"
        },
        "emailBody": { "type": "string" }
    }
}
```

The workflow engine processes this:

1. It sees `references: [ "step4_find_user.output" ]`.
2. It retrieves the entire output object `{ "userId": "...", "userName": "..." }` from its state.
3. It constructs a minimal context and injects it into the prompt for the LLM.

```
--- Context from previous steps ---
{
    "step4_find_user": {
        "output": {
            "userId": "u-12345",
            "userName": "Jane Doe"
        }
    }
}
--- End Context ---

Please generate the JSON for the `step5_draft_email` step using the information provided above.
```

4. It sends the schema for `step5_draft_email` to the LLM, but **without the `references` property**. The LLM then uses the provided context to follow the instructions in the `description` fields. This provides a clean and powerful mechanism for piping data from server-side actions back into the LLM's reasoning process.

> **Alice:** "This `references` thing sounds clever. So instead of the workflow engine sending the entire history of the process to the LLM for every little step, it just sends the specific outputs it needs?"
> **Bob:** "Precisely. It's like asking a colleague for help. You don't retell them the entire history of the project; you just say, 'Here's the customer's email from yesterday, can you draft a reply?' It keeps the conversation focused and efficient."

## Integrating Processes with Vessels via a Launcher Macro

We've established that `Vessels` operate using a toolkit of `Instructions`, and that `Processes` are complex, stateful workflows. The final step is to elegantly connect them. We do this by maintaining the rule that **Vessels only interact with Instructions**, and introducing a generic, reusable macro for kicking off workflows.

### The `LaunchProcess` Instruction

Instead of having `Vessels` use `Processes` directly, they use a special, generic `Instruction` called `LaunchProcess`. This `Instruction` functions as a **macro** that takes a `Process` Vibe as its main argument.

### How it Works: Dynamic Fusion at Runtime

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

By using a macro `Instruction` as the universal adapter, we create a clean, robust, and scalable bridge between our immediate-execution `Vessels` and our stateful `Processes`.

> **Alice:** "Okay, so my Vessel never actually talks to a `Process` directly. It just knows one Instruction: `LaunchProcess`. It's like having a universal 'start button' for any big job?"
> **Bob:** "You've got it. The Vessel says, 'I want to `LaunchProcess` for `BillingReport`.' The system then looks up the `BillingReportProcess`, finds its specific starting instructions, and hands that 'customized' start button back to the Vessel's LLM to fill out. The Vessel stays clean and simple, while the complex machinery is handled by the Process."

### The Anatomy of a Compiled `Process` Vibe

To make a `Process` self-contained and executable, the results of the compilation and partitioning are stored directly within the `Process` Vibe's own `schema`.

The `schema` of a `Process` Vibe contains a standard JSON Schema `$defs` block. This block holds the entire partitioned pipeline, with each chunk stored as a separate definition. We use a clear naming convention to distinguish the context for each chunk:

- **`LLM_<chunk_name>`**: A schema for a chunk of steps to be executed in a single call by an LLM.
- **`SERVER_<chunk_name>`**: A definition for a chunk of one or more activities to be executed on the server by the workflow engine.

The `Process`'s **Entrypoint Instruction** is simply a reference to the first `LLM_` chunk in its `$defs` (e.g., `"$ref": "#/$defs/LLM_InitialPrompt"`).

When the `LaunchProcess` macro is used, it dynamically reads the target `Process` Vibe's schema, finds the first `LLM_` definition, and presents that as the tool for the `Vessel` to execute. This makes every `Process` a self-describing, launchable entity.

## Advanced Process Capabilities

The `Process` architecture is built for industrial-scale work, with batching, multiplexing, and concurrency as first-class citizens.

### Batching, Multiplexing, and Server-Side Iteration

A `Process` is designed to operate on asynchronous streams of data, not just single instances. When a process needs to handle multiple items at once (multiplexing), it does so within a single context switch.

- **Schema-Level Multiplexing**: The compiled pipeline is a single, flat JSON object where each step is a property, multiplied by the batch size. To manage schema complexity limits and make the structure predictable for the LLM, we avoid nested arrays for batches. Instead, each item in a batch for each step becomes a distinct property. For example, a pipeline with **4 steps** processing a batch of **3 items** would be compiled into a single flat object with **12 top-level properties** (e.g., `step1_item1`, `step1_item2`, `step1_item3`, `step2_item1`, etc.). This allows the LLM to process the entire batch of work as a single, large object, which is highly efficient.

- **Native Async Iteration**: On the server, the compiled process pipeline maps directly onto an asynchronous iterator, leveraging our `@idealic/iterators` library. This allows for highly efficient, concurrent processing of both LLM-based and programmatic steps, with built-in support for back-pressure and resource management. This minimizes context switches on the server as well, allowing multiple programmatic steps to execute before needing to call an LLM again.

By combining a powerful, composable `Instruction` system with a robust, stateful, and batch-oriented `Process` architecture, we can create a highly scalable and efficient system. `Vessels` use `Instructions` for immediate, single-shot tasks, and they act as the initiators for complex `Processes`, which handle the heavy lifting of stateful, long-running, and data-intensive workflows.

## Schema Conventions for Compiled Pipelines

To make the compiled pipelines unambiguous for both the LLM and the server-side workflow engine, we use a set of clear conventions directly within the JSON schema.

### 1. Prefixes for Special Properties

- **`_` (Underscore Prefix)**: Denotes an internal "thinking" step for the LLM. These are fields the LLM must fill out as part of its reasoning process, but they are considered intermediate work and are not part of the final, primary output.
- **`$` (Dollar Sign Prefix)**: Denotes a metric to be logged. When the LLM populates a field like `"$qualityScore": 8`, the system automatically captures this as a metric associated with the task, without it cluttering the primary output.

### 2. Defining Pipeline Steps: LLM vs. Server Context

Any property within the pipeline schema represents a step. The schema defines how to distinguish between a simple, non-blocking step executed within the LLM's context and a blocking step that requires a server-side round trip.

#### LLM-Context Steps (Non-Blocking)

By default, every step in a pipeline chunk being executed by the LLM is **non-blocking**. The LLM simply fills out the properties of the step's schema and moves to the next.

From the LLM's perspective, there is little difference between a "thinking" step and a step that triggers a fire-and-forget server action. The server-side interpreter is responsible for noticing if the step name (e.g., `step3_log_event`) matches a registered server activity and executing it asynchronously. The pipeline itself does not wait.

```json
"step3_log_event": {
    "description": "Log an event to the server. Does not block pipeline execution.",
    "type": "object",
    "properties": {
        "eventName": { "type": "string", "const": "UserAction" },
        "details": { "$ref": "#/properties/_reasoning_for_action" }
    }
}
```

#### Server-Context Steps (Blocking)

A step is explicitly defined as a **blocking server call** if its schema contains a top-level `output` property. This property signals to the workflow engine that it must pause execution, run a server-side activity, and wait for a result that conforms to the `output` schema. The other properties of the step implicitly define its input.

To make the schema consistent, the `output` property is made `nullable`. When an LLM generates a pipeline that includes a blocking step, its job is to set the `output` field to `

```question
When should a Stateful Process be used instead of a Direct Server Call?
* [x] For any task that is long-running or needs to survive server restarts.
* [x] When a server-side action is blocking and its output is required by a subsequent step in the pipeline.
* [x] For any workflow that requires durable state management for intermediate results.
* [ ] For any simple, non-blocking side effect like logging or sending a notification.
* [ ] When the entire workflow can be completed within a single LLM call.
```
