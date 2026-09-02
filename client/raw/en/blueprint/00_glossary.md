# Chapter 0: Glossary of Core Concepts

This glossary provides a central reference for the fundamental terms and concepts used throughout the documentation.

---

### B

- **[Budget](06.%20budgets.md)**: A living economic engine, not just a number. It's a Vibe that combines **authority** (permission to act, defined in its `schema`) and **resources** (spendable assets, tracked in its `solution`). It governs all significant changes in the system through the [`refine` primitive](04.%20refinement.md).

### C

- **Capability Vibe**: A Vibe that represents a unified grant of authority and resources. It functions as a permission slip, a wallet, and a budget all in one, controlling what actions are permitted and providing the means to execute them. (See [Vibes](01.%20vibes.md))

- **[Content Determinism](12.%20determinism.md)**: A spectrum of control relating to the predictability and consistency of generated output, particularly from LLMs. It ranges from highly varied and exploratory (low determinism) to entirely fixed and certain (high determinism).

### I

- **Input**: The contextual information and parameters (e.g., a user prompt, data) that define the specifics for a given interaction. It is the `input` part of the `{input, schema, solution}` triplet that forms a Vibe. (See [Vibes](01.%20vibes.md))

- **[Instruction](02.%20instructions.md)**: A structured, reusable "recipe" for action, defined as a JSON Schema. It provides a clear, actionable guide for a `refine` operation, detailing _how_ a change should be made. It is the core building block for both simple edits and complex, multi-step `Processes`.

### M

- **Meme**: A self-propagating unit of functionality, often referred to as a "tool." Each meme represents a discrete capability, from reasoning and analysis to communication and process management, that can be composed within Vibe schemas. (See [Vibes](01.%20vibes.md))

### P

- **[Process Vibe](08.%20processes.md)**: A Vibe whose `schema` specifies a sequential, deterministic workflow of steps. An instance of a Process Vibe is a **Workflow Run**. Processes are designed for stateful, long-running, or complex tasks that may involve server-side logic, and they provide durability and reliability.

### R

- **Record Vibe**: A Vibe whose `schema` defines structured data. Its `solution` _is_ the structured content itself (e.g., a specific invoice). These Vibes can be made interactive by embedding tools and trackers, turning passive data into active resources. (See [Vibes](01.%20vibes.md))

- **[Refine](04.%20refinement.md)**: The single, universal primitive for all forms of Vibe modification and generation. It is the mechanism for making changes, creating new Vibes from templates, and evolving existing Vibes in a controlled, auditable way. Its signature is `refine(targets, instructions, capabilities)`.

- **[Resource (Economic)](05.%20exchange.md)**: A broad category of assets, qualifications, or conditions required for or consumed by a `refine` operation. Resources can be consumable (e.g., money, tokens), qualifying (e.g., skills, reputation), or computational (e.g., GPU hours).

- **Role Vibe**: A Vibe whose `schema` orchestrates a collection of tools (memes) concurrently to produce `solutions` that exhibit emergent behavior. An instance of a Role Vibe is a **Vessel**. (See [Vibes](01.%20vibes.md))

### S

- **[Schema](01.%20vibes.md)**: The "blueprint" or "constitution" within a Vibe. It is a JSON Schema object that defines the rules, structure, constraints, and logic that the `solution` must conform to. In a `Budget`, the `schema` defines its rules of authority.

- **[Solution](01.%20vibes.md)**: The resulting content or outcome of an interaction, which conforms to the `schema` given a specific `input`. In a `Budget`, the `solution` is the real-time snapshot of its state and balances.

- **[Structural Determinism](12.%20determinism.md)**: A spectrum of control concerning the definition and constraints of a process or schema itself. It ranges from highly flexible and adaptive (low determinism) to entirely defined and unyielding (high determinism).

### T

- **Tracker**: A specialized tool associated with a Record Vibe's `schema` that activates automatically based on triggers (e.g., being viewed or used), enabling the `solution` to become an active, aware participant in workflows. (See [Vibes](01.%20vibes.md))

### V

- **Vessel**: An instance of a **Role Vibe**. It is an active agent in the system (e.g., a bot) that uses a collection of capabilities (tools, `Instructions`) to perform complex tasks and exhibit emergent, compositional behaviors. (See [Vibes](01.%20vibes.md))

- **[Vibe](01.%20vibes.md)**: The fundamental unit of interaction and knowledge in the system. It is an immutable, self-contained record of an interaction, encapsulating the `{input, schema, solution}` triplet.

### W

- **Workflow Run**: An instance of a **Process Vibe**. It represents a single, stateful execution of a defined process or workflow. (See [Processes](08.%20processes.md))
