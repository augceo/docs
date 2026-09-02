# Chapter 4: Refinements

## New Ideas in This Chapter

- **`refine` as the Universal Verb for Change**: Instead of multiple commands like `create`, `edit`, or `spawn`, the system uses a single, powerful primitive, `refine`, for all significant, state-altering events.
- **The Trinity of Change: `targets`, `instructions`, `budgets`**: Every `refine` call is defined by what is being changed (`targets`), how it's being changed (`instructions`), and the authority/resources for that change (`budgets`).
- **Budgets as Active Authorizers**: Authorization isn't just a simple permission check. `Budgets` are living economic engines whose `schema` acts as a constitution, evaluating if a `refine` operation is permissible.
- **Delegation as Budget Refinement**: Authority isn't just passed down; it's delegated by `refining` a parent `Budget` into a new, more specific child `Budget` with a `narrowed` scope and allocated funds.
- **Schema Refinement as Universal Evolution**: The principle of making schemas more specific (`narrowing`) applies consistently across all Vibe types, from `Records` to `Roles` and `Budgets`, providing a unified model for evolution.

## The `Refine` Primitive: Making Changes

The `refine` primitive is how we change and create Vibes. It's the core mechanism that:

- Takes a general Vibe and makes it more specific
- Creates new Vibes from templates
- Modifies existing Vibes in controlled ways

Every significant change in the system happens when you `refine` something. This action:

- Create a clear record of how Vibes evolve
- Track the creation of new data and processes
- Manage how experiments and changes happen

## The `Refine` Primitive: Unified Change

At the heart of how Vibes evolve, become more specific, and ultimately manifest as concrete solutions lies a single, powerful primitive: **`refine`**. This is the universal mechanism for all forms of Vibe modification and generation, guiding a Vibe from a more general or abstract state towards a more defined and concrete instance. Think of it as a process of **refinement** and specification, effectively `narrowing` its scope, where each time you `refine` a Vibe, it brings it closer to a tangible outcome, a more focused purpose, or eventually, a collapsed solution. This journey of progressive **refinement**, driven by instructions, and guided by budgets, is fundamental to how the system works and makes decisions. Crucially, the act of `refining` represents the definitive, state-altering events within the system. They constitute the primary lineage—a 'source tree'—of significant changes, including the creation of new data items, the evolution of processes, and the instigation of experiments. This distinguishes them from other, more transient Vibe interactions or general messaging, which, while potentially part of the history, do not directly form this core evolutionary record.

> **Primitive call**
>
> `outputVibe(s) = refine(targets, instructions, budgets)`
>
> This single verb, `refine`, takes three primary arguments. Each argument can accept either a single Vibe or an array of Vibes:
>
> - **`targets`**: One or more Vibe records that will be refined or template Vibes from which new Vibes will be spawned. If an array of `targets` is provided, the `refine` primitive is applied to each target Vibe, effectively performing a batch action that may result in an array of output Vibes.
> - **`instructions`**: One or more Vibes (which could be data or process) that describe _how_ the `targets` are to be refined or how new Vibes are to be generated. If an array of `instructions` is provided, the authorizing Budget's `schema` must define how these multiple instructions are to be interpreted (e.g., as a sequence to be applied or as a set of inputs to be merged).
> - **`budgets`**: One or more `Budget` Vibes that provide the **authorization** and, if necessary, the **resources** for the operation. The `schema` of a `Budget` acts as its constitution, defining what actions it can approve and what resources it can spend. If a `refine` call requires funding, the provided `Budget`(s) must have a sufficient balance. If the call requires specific permission, the `schema` of the `Budget`(s) must authorize the transformation of the given `targets` by the given `instructions`. In some contexts, a default or ambient `Budget` may be used implicitly.
>
> The result when you `refine` is **always** a fresh, immutable Vibe or an array of fresh, immutable Vibes if multiple `targets` were specified; the original Vibes remain untouched.

The `refine` primitive is versatile, supporting various contexts. For instance, a Vibe can `refine` itself (self-**refinement**), one Vibe can influence another (cross-**refinement**), or divergent versions can be reconciled (merge-**refinement**). A particularly crucial application is generative-**refinement** (spawn), where a new Vibe is created from a template `target` based on the `instructions`. When `refine` is used generatively to spawn new entities, these acts are fundamental to the system's evolutionary record, contributing significantly to the primary lineage of change.

Another common application of `refine` is budget allocation and delegation. Rather than splitting abstract capabilities, the `Budget` Vibe itself—a living economic engine—is the subject of `refine`. A broad, high-level `Budget` can be `refined` to create and fund new, more specific "child" `Budget` Vibes with constrained allocations. This creates a hierarchy of budgets, allowing authority to be delegated to teams or projects in a clear, auditable manner.

For instance, to allocate a portion of a department's budget to a specific project:

Let's assume we have:

- `DepartmentBudget`: A `Budget` Vibe with a balance of $50,000 for general marketing.
- `DelegateBudgetInstructions`: An `instructions` Vibe like `{ "createSubBudget": { "name": "Project Alpha Marketing", "amount": 10000, "purpose": "..." } }`.
- A `Budget` held by a manager that authorizes them to refine the `DepartmentBudget` to create sub-budgets.

The `refine` call would be conceptualized as:
`NewBudget = refine(targets: DepartmentBudget, instructions: DelegateBudgetInstructions, budgets: ManagerBudget)`

This single operation performs two crucial functions:

1.  **Creates a New Budget Vibe**: A new, immutable `ProjectAlphaBudget` Vibe is created. Its own `schema` defines its specific purpose and spending rules, effectively forming a "charter for a micro-economy."

2.  **Funds the New Budget**: An atomic transaction is written to the ledger, debiting $10,000 from the `DepartmentBudget` and crediting $10,000 to the new `ProjectAlphaBudget`. The `solution` (live balance) of both budgets is updated to reflect this.

This approach, detailed further in the chapter on Budgets, replaces abstract capability-splitting with the direct manipulation of `Budget` Vibes, creating a clear, auditable trail of how funds are allocated and delegated.

```llm
The `refine` primitive is the cornerstone of change. Its signature is `refine(targets, instructions, budgets)`.
It takes three arguments, each can be a single Vibe or an array of Vibes:
`targets` (what to change/spawn from), `instructions` (how to change), and `budgets` (the authority and resources for the change).
It always produces new, immutable Vibe(s), leaving originals intact.
```

> **Alice:** "So, instead of a whole bunch of different commands like 'create', 'edit', 'spawn', or 'refine', we just have this one `refine` primitive for everything that makes a Vibe more specific or brings a new one into existence?"
> **Bob:** "Exactly! One core primitive to guide how Vibes evolve. You bring your `targets` – the Vibe or Vibes you want to use as a base. Then you present your `instructions`, detailing how the new Vibe should be shaped. Finally, you might provide one or more `budgets`. The `budget` is your authorization slip and your wallet, all in one. Its internal rules—its `schema`—determine if the operation is allowed. If the action costs something, the `budget` also provides the funds. If the `budget` approves, poof! A brand new, immutable Vibe (or set of Vibes) appears!"

```question
What are the three mandatory arguments for the `refine` primitive?
* [x] `targets`: The Vibes to be refined or the templates to spawn from.
* [x] `instructions`: The Vibes describing how the refinement should occur.
* [x] `budgets`: The Vibes providing authorization and resources for the refinement.
```

```question
What is a guaranteed characteristic of the `refine` primitive's output?
* [x] It always produces a new, immutable Vibe.
* [ ] It modifies the `targets` in-place.
* [ ] It can sometimes return multiple new Vibes.
* [ ] The `budgets` are consumed and invalidated after use.
* [ ] It always requires LLM intervention to complete.
```

## Budgets: Authorizing Change

Authorization for all `refine` operations is managed through `Budget` Vibes. A `Budget` is not just a pool of funds; it is a living economic engine whose `schema` acts as a constitution, defining the rules, goals, and limits of what can be done under its authority. It is the gatekeeper for all significant changes.

### Core Principles of Budget-Driven Authority

- **Unified Authority and Resources:** A `Budget` Vibe combines permission to act with the resources required for action. Its `schema` defines the rules (the authority), and its `solution` tracks the spendable assets (the resources). An action is authorized if it complies with the `schema`'s rules and if the `solution` contains sufficient funds.

- **The Constitution (`schema`):** A `Budget`'s authority is explicitly defined by its `schema`. This "constitution" specifies what kind of `targets` can be refined and by what `instructions`. For example, a `Marketing Budget`'s `schema` might only authorize `refine` calls that create advertising-related Vibes.

- **Delegation as Budget Refinement:** Authority is delegated by `refining` a parent `Budget` to create a new, more specific "child" `Budget`. The `refine` operation allocates a portion of the parent's funds to the child and, crucially, gives the child its own `schema` that `narrows` the scope of authority. The Marketing Director can `refine` the main `Marketing Budget` to create a `Social Media Budget` for their team lead, which has a smaller pool of funds and a `schema` that only permits spending on social media platforms.

- **Immutability & Traceability:** `Budget` Vibes are immutable. When a `Budget` is refined to delegate authority, a new `Budget` Vibe is created. This ensures a clear, auditable chain of delegation, tracing every sub-budget back to its parent. Revocation of a `Budget` is handled by creating a new Vibe that marks it as void, invalidating any child `Budgets` that draw from it.

### Bootstrapping the System: Initial `refine` Operations

The `refine(targets, instructions, budgets)` primitive requires existing Vibes to function. This naturally leads to a question: how are the very first Vibes in the system created? This "bootstrap" or "genesis" phase relies on foundational elements provided by the platform itself.

1.  **Initial `budgets` (The Platform's Authority):**
    The initial authorization is provided by a foundational "Platform Budget." This acts as a foundational license, enabling the first set of `refine` calls. Its `schema` is configured to permit the creation of the core, foundational Vibe schemas (for defining Vessels, Processes, etc.). This `Budget` provides the "permission slip" to start building, typically without containing any financial resources itself.

2.  **Initial `targets` (Base Templates):**
    The platform provides a set of minimal, "initial" template Vibes that serve as the initial `targets` (the first argument) for creating fundamental entities. These typically include:
    - `aug:/types/Vessel?1`: A very generic template for creating new Vessel types.
    - `aug:/types/Process?1`: A minimal template for defining new Process types.
    - `aug:/types/Data?1`: A basic template for new structured Record types (e.g., might only contain a root `id` field or basic metadata structure).

3.  **Initial `instructions` (User Intent):**
    The `instructions` (the second argument) for these first refinements are typically crafted by the user or a setup/onboarding process. It specifies _how_ the user wants to modify or specialize one of the initial templates. For example, instructions might detail how to _refine_ the `aug:/types/Data?1` to create a new `Invoice` schema Vibe.

**The Schema Refinement Principle in Bootstrapping:**

When you modify a base template's schema by using `refine`, you should make it more specific (`narrowing` its definition) - not completely different. Think of it like this:

- You can add new fields and details to the schema
- You should stay within the template's original purpose
- You can't change its fundamental nature

For example:

- A Record template can become an Invoice Record template by adding invoice-specific fields
- But you shouldn't turn a Record template into a Process Control System

This approach ensures that new schemas maintain a clear connection to their original templates, making the system's structure more predictable and manageable.

```llm
Bootstrapping the system with `refine(targets, instructions, budgets)`
relies on platform-provided elements:
1. Initial `targets`: Minimal base templates like
   `aug:/types/Vessel?1`, `aug:/types/Process?1`, and
   `aug:/types/Data?1`.
2. Initial `instructions`: User-defined instructions specifying how to
   customize these base templates.
3. Initial `budgets`: A foundational "Platform Budget" from the owners,
   enabling initial actions. Its `schema` permits these foundational
   creations, usually without requiring financial resources.
Schema **refinement** typically follows a principle of `narrowing`, specializing templates
within the constraints of the authorizing `Budget`'s schema.
```

> **Alice:** "Alright, `refine` needs `targets` and `instructions`. But for the _very_ first one, where does the authority come from? Does a stork bring it?"
> **Bob:** "Ha! Not quite a stork, but the platform plays delivery person. It provides the basic template `targets` (plain dough) and a foundational `Platform Budget` (your license to bake). You just provide the `instructions` (your first recipe)."

```question
How is the initial authority for the first refinements provided?
* [x] It is provided by a foundational "Platform Budget" from the platform administrators/owners.
* [ ] It is generated automatically from the `targets`.
* [ ] The user creates it using a special command.
* [ ] It is cloned from the `instructions`.
```

```question
What serves as the initial `targets` in the bootstrapping phase?
* [x] Minimal, initial template Vibes provided by the platform (e.g., for Vessels, Processes, Data).
* [ ] The user must create these from scratch using external tools.
* [ ] Any existing Vibe in a public repository can be used.
* [ ] The `budgets` themselves also act as the first `targets`.
```

```question
What is the "schema **refinement** principle" in the context of **refining** base templates?
* [x] Resulting schemas should generally be specializations (`narrowed` versions), adding detail within the template's conceptual boundaries.
* [ ] Schemas can be arbitrarily widened without restriction.
```

### How Budgets Authorize `refine` Operations

Here's how the system checks if a `refine` call is allowed when one or more `budgets` are provided:

1.  **Iterate through `budgets`**: The system will examine each provided `Budget` Vibe one by one.

2.  **For each `Budget`, perform checks**: For the `refine` call to be authorized by a specific `Budget`, all the following conditions must be met:
    - **Authority Check**: The `Budget`'s `schema` (its constitution) must permit the requested operation. This involves validating the provided `targets` and `instructions` against the rules defined in the `schema`.
    - **Balance Check**: If the operation requires spending resources (e.g., currency, tokens, credits), the `Budget`'s `solution` (its live balance) must contain sufficient funds.
    - **Solvability Check**: Committing the required funds must not leave the `Budget` in an "unsolvable" state, meaning it still must meet its own minimum financial or operational constraints defined in its `schema`.

3.  **Final Decision**:
    - If _any single provided `Budget`_ successfully validates all the above conditions, then `refine` is approved.
    - The `refine` call then proceeds. If the action was economic, a transaction is written to the ledger, debiting the authorizing `Budget`.
    - If no provided `Budget` can authorize the operation, the `refine` call is rejected.

This system keeps a clear audit trail. Every `refine` action is linked to the specific `Budget` that authorized and funded it, providing a complete and unbroken lineage for every change and every unit of value spent.

```llm
Budget authorization for `refine` involves a multi-step check for each `Budget` provided:
1.  **Authority Check**: Does the `Budget`'s `schema` permit this specific `refine` operation on the given `targets` with the given `instructions`?
2.  **Balance Check**: Does the `Budget` have enough funds in its `solution`?
3.  **Solvability Check**: Will spending the funds leave the `Budget` able to meet its own core requirements?

If any single provided `Budget` passes all three checks, the operation is approved. Otherwise, it's rejected. This ensures granular, auditable control over all Vibe transformations.
```

> **Alice:** "So it's like using a credit card. When I try to `refine` something, the system checks my `Budget` Vibe. It asks, 'Does the `Budget`'s rulebook (`schema`) allow this type of purchase (`targets` and `instructions`)? Is there enough money in the account (`solution`)? And will this purchase drain the account so much that I can't pay my rent (`solvability`)?' If it's all good, the transaction goes through."
> **Bob:** "Perfect analogy! And if you have multiple `budgets` in your wallet, it just needs to find one that says 'yes' to the whole operation."

```question
What is the final condition for `refine` to be approved by a `Budget` (or set of `Budgets`)?
* [x] At least one provided `Budget` must have a `schema` that authorizes the operation and sufficient resources to fund it without becoming unsolvable.
* [ ] All provided `Budgets` must unanimously approve the call to `refine`.
* [ ] The `targets` Vibes must contain a special marker indicating they allow refining by the presented `budgets`.
* [ ] The `instructions` Vibes must be digitally signed by the owner of the `budgets`.
```

```question
During the authorization process for a `refine` call, what aspects of a `Budget` are checked?
* [x] The `Budget`'s `schema`, to see if the rules authorize the specific `targets` and `instructions`.
* [x] The `Budget`'s `solution`, to ensure it has a sufficient balance of the required resources.
* [x] The `Budget`'s overall solvability, to ensure the action doesn't violate its core constraints.
* [ ] The historical number of times the `Budget` has been successfully used.
* [ ] The computational cost of the proposed `refine` operation.
```

---

## Evolving Vibe Structures: Schema Refinement

The `refine` primitive, once authorized by a `Budget`, can be used to evolve not just the `solution` (data content) of a Vibe, but also the very structural definition held within its `schema` field. This is a form of `narrowing` the schema. This applies to all Vibe types, allowing for a consistent mechanism of controlled evolution across the entire system.

### The Principle of Schema Refinement

Schema **refinement** is the process of making a Vibe's `schema` definition more specific. This often involves adding new fields, making existing fields mandatory, tightening constraints (e.g., `minLength` for a string, `maximum` for a number), or **refining** data types. The key idea is that the new schema is an extension or **refinement** of the old one, preserving its core conceptual nature while adding specificity. This ensures that Vibes evolve coherently. For instance, a generic "Product" Record Vibe might be refined to an "Electronic Product" Record Vibe by adding fields specific to electronics, but it wouldn't be transformed into something unrelated like a "User Profile."

Crucially, a Vibe with a **refined** schema remains compatible with the original, more general schema. Systems or processes designed to work with Vibes conforming to the original schema can still interact with the **refined** Vibe; they will simply be unaware of or ignore the additional, more specific fields or stricter constraints. For example, if a system expects a "Vehicle" Vibe with a `color` field, it can still process a "Car" Vibe (a **refined** "Vehicle") that has `color` and also an additional `numberOfDoors` field; the system will just use the `color` field as expected.

```llm
Schema **refinement** makes a Vibe's `schema` definition more specific, like adding
fields, tightening constraints, or **refining** types. It's an extension of the old
schema, preserving the Vibe's core concept while adding detail. A **refined** Vibe
remains compatible with systems expecting the original, general schema; those
systems will simply ignore the newer, specific parts. This ensures
Vibes evolve coherently (e.g., a generic "Product" can become a more specific
"Electronic Product" but not an unrelated "User Profile").
```

> **Alice:** "So, schema **refinement** is like taking a general idea, say 'a vehicle,' and getting more specific, like 'a two-door electric sports car with vegan leather seats'? We're not changing it into a 'bicycle' or a 'boat'?"
> **Bob:** "Exactly! You're adding details and constraints but keeping the fundamental nature. The goal is **refinement** and increased precision, not a total identity swap. The new, **refined** schema will still satisfy the requirements of the original, more general schema. If a system only knows how to deal with 'vehicles' and their basic properties, it can still handle your 'two-door electric sports car' by just looking at the parts it understands."

```question
What is the primary goal of "schema **refinement**"?
* [x] To make a Vibe's `schema` definition more specific while preserving its core conceptual nature and ensuring compatibility with systems expecting the original schema.
* [ ] To completely change a Vibe's `schema` definition into something unrelated.
* [ ] To reduce the number of fields in a Vibe's `schema` definition to make it simpler.
* [ ] To make a Vibe's `schema` definition more generic and less constrained.
* [ ] To convert a Record Vibe's schema into a Process Vibe's schema.
```

### Schema Refinement Across Vibe Types

The principle of "schema **refinement**"—making a Vibe's guiding `schema` definition more specific (i.e., `narrowing` it)—applies to all Vibe types, not just Record Vibes. Each time you `refine` modifies a Vibe's `schema` definition, it results in a new Vibe with this **refined** structure. The nature of this **refinement** varies depending on the Vibe type:

| Vibe Type            | Schema Changes               | Input Changes                  | Refinement Effect             | Result Created             |
| :------------------- | :--------------------------- | :----------------------------- | :---------------------------- | :------------------------- |
| **Record Vibe**      | Fields added, constraints    | Updated context, examples,     | More specific JSON Schema     | New Record Vibe with       |
|                      | tightened, types **refined** | validation rules               | definition                    | evolved `schema` and       |
|                      |                              |                                |                               | conforming `solution`      |
|                      |                              |                                |                               |                            |
| **Instruction Vibe** | Parameters added, options    | Enhanced context, more         | More specific transformation  | New Instruction Vibe with  |
|                      | expanded, validation rules   | detailed transformation rules, | logic, clearer mapping of     | specialized parameters and |
|                      | strengthened                 | additional examples            | inputs to outputs             | more precise execution     |
|                      |                              |                                |                               |                            |
| **Role Vibe**        | Tool/meme collection becomes | Enhanced prompts, additional   | Specialized expertise and     | New Role Vibe (Vessel)     |
|                      | more specialized, focused    | context, behavior guidelines   | focused behavior patterns     | with **refined** tools     |
|                      | configuration                |                                |                               | and operational parameters |
|                      |                              |                                |                               |                            |
| **Process Vibe**     | Sequential workflow DAG gets | Step instructions, error       | Increased predictability and  | New Process Vibe with      |
|                      | more detailed, constrained   | handling, validation criteria  | more specific execution steps | specialized pipeline and   |
|                      |                              |                                |                               | enhanced error handling    |
|                      |                              |                                |                               |                            |
| **Budget**           | `schema` rules become more   | Delegation rules, `narrowed`   | More specific spending        | New child `Budget` with    |
|                      | restrictive, constraints on  | spending categories, resource  | authority, delegation of      | a smaller, dedicated       |
|                      | spending are tightened       | requirements                   | limited permissions           | scope and funds            |

This consistent application of schema **refinement** via the `refine` primitive allows the entire system, across all Vibe types, to evolve in a controlled, auditable, and progressively more **refined** (and `narrowed`) manner.

### Instance Migration During Refinement

When a Vibe undergoes refinement, the user is given choices on how to propagate the refined changes to existing instances of that Vibe:

1. **User-Controlled Propagation**: The system presents options for how to apply the refinement changes to existing instances, allowing the user to choose the appropriate migration strategy.

2. **Refinement Migration**: Rather than regenerating instances from scratch, the system applies the specific refinement changes to each individual existing Vibe, preserving the original instance while incorporating the refined schema and input modifications.

3. **Targeted Application**: Only the specific changes introduced in the refinement (new fields, updated constraints, enhanced prompts, etc.) are applied to existing instances, maintaining their existing data and context where compatible.

4. **Migration Strategies**: Users can choose from different propagation approaches:
   - **Immediate migration**: Apply refinements to all instances at once
   - **Selective migration**: Choose specific instances to migrate
   - **Gradual rollout**: Migrate instances incrementally with validation
   - **Branch-based testing**: Test refinements on copies before applying to originals

This migration mechanism ensures that refinements can propagate throughout the system in a controlled manner, with user oversight determining how and when the evolution occurs across the Vibe architecture.

```llm
Schema **refinement** applies to all Vibe types, making their guiding `schema` definitions
more specific. For Record Vibes, the JSON Schema **refines**. For Instruction Vibes, parameters
and transformation rules become more detailed. For Budgets, rules
become more restrictive (delegation, `narrowing` authority). For Roles, tool configurations specialize.
For Processes, workflows get more detailed. Each time you `refine`, it yields a new
Vibe with the **refined** schema, ensuring controlled, auditable evolution system-wide.
Users control how refinement changes migrate to existing instances.
```

> **Alice:** "So this schema **refinement** isn't just for data records? If I have a `Budget` to rule the world, I can `refine` it to just rule, say, my local bakery, effectively `narrowing` its power? And that's also schema **refinement**?"
> **Bob:** "Precisely! The `Budget`'s `schema` of authority gets more specific—stricter rules, smaller allocations. Same for a Role Vibe; its schema of tool configurations can be **refined** to make it a specialist. Or a Process Vibe can have its workflow schema **refined** to handle a very specific sub-task with extra validation. It's a universal principle."

```question
How does schema **refinement** manifest in a `Budget` Vibe (i.e., how is its authority `narrowed`)?
* [x] Its `schema` becomes more restrictive, and its funds are typically partitioned, representing delegation of a smaller scope of authority.
* [ ] New, unrelated tools are added to its `solution`.
* [ ] The `target` schemas it can operate on become broader.
* [ ] Its `issuerRef` metadata is removed.
* [ ] It gains the ability to bypass `refine` operations.
```

```question
What is the outcome of schema **refinement** on a Role Vibe?
* [x] Its `schema` definition (tool/meme collection and orchestration) becomes more specialized.
* [ ] It transforms into a Process Vibe.
* [ ] It loses all its embedded tools.
* [ ] Its ability to activate tools concurrently is removed.
* [ ] It can only produce Record Vibe `solutions`.
```

### Evolving Record Vibe Schemas: Conceptual Overview

The "schema **refinement** principle"—making schemas more specific—is fundamental to system adaptation. `Budget` Vibes can authorize changes not only to a Record Vibe's `solution` (data) but also to the structural definition in its `schema` field. This is done by using `refine` on a Record Vibe with an `instruction` Vibe that defines the evolution.

Two main scenarios for schema evolution are:

1.  **Additive Refinement**: New fields are added, or constraints are tightened. The new schema is a direct extension of the old one. For example, adding `stockLevel` and `price` to a basic product schema when evolving a promotional listing into a full e-commerce product.
2.  **Major Version Migration**: For more fundamental ("breaking") changes (e.g., renaming or removing fields, significantly altering types), a new schema version is defined (e.g., "ProductSchemaV2" from "ProductSchemaV1"). Existing Vibes are migrated by using `refine` where the old Vibe's data (as part of the `instruction`) is used to populate a new Vibe instance conforming to the new schema version. This ensures clean separation and explicit transformation.

This budget-driven approach treats schema evolution as integral to data evolution, managed by the same `refine` primitive and budget-based authorization. This ensures data structures evolve controllably, auditably, and securely, reflecting business requirement changes.

```llm
Record Vibe schema evolution (i.e., `narrowing` of the schema) is managed by `refine` calls authorized by
`Budget` Vibes. It typically involves either additive **refinement** (new fields,
tighter constraints) or major version migration (defining a new schema version
and transforming old Vibe data to fit). This ensures controlled, auditable
evolution of data structures, treating schema changes as part of data evolution.
```

> **Alice:** "So if my product Vibe needs to add, say, a 'carbon_footprint' field, that's 'additive **refinement**'? And if I want to completely change how I store prices, from a string like '$19.99' to an integer for cents, that's more like a 'major version migration'?"
> **Bob:** "You nailed it. Additive is like adding a new room to your house. Migration is like moving to a new house designed with different plumbing – you need a process to pack up your stuff from the old house and arrange it properly in the new one. Both use `refine`, but the `instruction` Vibe looks different."

```question
What are the two main scenarios for evolving Record Vibe schemas as described?
* [x] Additive Refinement: Adding new fields or tightening constraints on the existing schema.
* [x] Major Version Migration: Defining a new schema version and transforming data from old Vibes to conform to it.
* [ ] Schema Deletion: Removing the schema field entirely from a Record Vibe.
* [ ] Automatic Schema Generalization: The system automatically makes schemas less specific over time.
```

_For detailed walkthroughs of these schema evolution scenarios, including `refine` call examples and JSON structures, please see the accompanying document: `04_refinements_examples.md`._

---

## Practical Permission Management for `refine` Operations: Conceptual Overview

Permissions for using `refine` are granular and task-oriented, granted via the `schema` of a `Budget` Vibe. These `schema` rules can authorize actions on specific Vibe instances or, more powerfully, on any Vibe(s) conforming to specified schemas for their `targets` and `instructions` arguments.

This ensures that all changes, from data updates to schema evolutions (schema `narrowing`), and including actions that consume or require specific resources (like budget tokens or metric achievements held in the `Budget`), are controlled and auditable. The system verifies that the presented `Budget` (or one of them if multiple are provided) contains rules in its `schema` that explicitly authorize the `targets` Vibe(s) to be transformed by the specific `instructions` Vibe(s), and has sufficient resources to do so.

```llm
Permissions for `refine` are managed through the `schema` of a `Budget`
Vibe. This `schema` defines granular, task-oriented authorizations,
allowing actions on Vibe instances or any Vibe(s) matching `target`
and `instruction` schemas, and providing the resources for the action.
This system ensures all changes are controlled, auditable, and verified
against explicit `Budget` definitions.
```

> **Alice:** "So, these `Budget` schemas are super specific? Like, one `Budget` might let me change the _price_ of a product, but another, separate `Budget` is needed to change its _description_? And that second `Budget` might not even have any money in it, just the authority?"
> **Bob:** "Exactly. Or a single `Budget`'s `schema` might allow a Product Manager to `refine` a 'Product Template Vibe' using an 'Approved Product Launch Instruction Vibe,' but only if that `Budget` also has a balance of at least 10,000 'Marketing Dollars'. Granular control across targets, instructions, and resources is key."

```question
How does the system ensure that when you `refine`, it is controlled and auditable?
* [x] By verifying that a presented `Budget` Vibe has a `schema` that explicitly authorizes the `targets` and `instructions`, and has sufficient `resources` in its `solution`.
* [ ] By requiring all `refine` calls to be approved by a human administrator in real-time.
* [ ] By encrypting all `instructions` Vibes so only authorized `targets` Vibes can decrypt them.
* [ ] By limiting `refine` to only affect the `solution` field, never the `schema` field.
* [ ] By allowing any Vibe to be a `budget` as long as its name starts with "budget-".
```

_For e-commerce examples illustrating practical permission management for various roles and tasks (e.g., Product Managers launching products, Inventory Managers adjusting stock), please see the accompanying document: `04_refinements_examples.md`._

This consistent way to `refine` schema allows the entire system, across all Vibe types, to evolve in a controlled, auditable, and progressively more **refined** (and `narrowed`) manner.
