# 205: Idea/Budget

> **Budget Idea:** An Idea that combines **authority** (permission to act, defined in its `schema`) and **resources** (spendable assets, tracked in its `solution`).
>
> — [Glossary](./000_glossary.md)

A **Budget Idea** transforms static financial planning into a living economic engine. It's an Idea that combines **authority** (permission to act, defined in its `schema`) and **resources** (spendable assets, tracked in its `solution`). It governs all significant changes in the system through the `refine` primitive.

> Sidenote:
>
> - Requires:
>   - [Concept: Idea](./101_concept_idea.md)

This document outlines the core concepts of Budget Ideas, which introduce a way to think about budgets and economic systems. Traditional budgets are static numbers; Budget Ideas are living economic engines with their own rules, goals, and even custom currencies.

## The Living Budget

In this system, a **`Budget` is not merely a number; it is a living economic engine.** The `Budget` Vibe can be seen as the **strategic nervous system** of an entire enterprise—a living, executable model of the business itself. It is a portable, clonable, and auditable business model.

Using the `refine` primitive, we can design, fund, and deploy `Budget` Vibes that act as miniature economies, complete with their own rules, goals, and even custom currencies. This transforms budgeting from a static, top-down process into a dynamic, decentralized, and auditable system for orchestrating value flow.

```llm
A Budget Vibe transforms static financial planning into a living economic engine.
It separates planning (schema defining rules), funding (explicit transactions),
and authority (delegated permissions). This enables modeling anything from simple
escrows to complex treasuries with automatic reinvestment logic, creating dynamic,
decentralized systems for managing any quantifiable resource.
```

### The Constitution (`schema`): Strategic Planning

The **`schema`** is the **Constitution**. It is a public declaration of intent that creates total organizational alignment. It defines the rules, relationships, and priorities, answering: What are our goals? How do we handle crisis? How do we reward success? But it goes deeper than simple targets. It provides the **strategic context** for every metric, turning the `Budget` into an active decision-making tool.

The `schema` clarifies the _nature_ of each goal. Some metrics are meant for continuous, unbounded growth. Others are meant to be minimized. Crucially, the `schema` can also define "good enough" thresholds.

Beyond the numbers, the `schema` can also contain the qualitative interpretation of each metric. The `description` for a metric is the official definition of what it means to the business. Evolving this description is a strategic act, allowing the organization to refine its values and goals over time without breaking historical data.

By encoding these nuanced guardrails, the `schema` prevents the organization from blindly over-optimizing one metric at the expense of others. It provides the wisdom to know not just what to do, but when a goal is achieved and it's time to pivot. Changing this constitution is a strategic, high-rank `refine` operation because it means fundamentally altering the business model.

```llm
The schema acts as a Budget's constitution, defining not just what to measure
but the nature of each goal—whether to maximize, minimize, or cap it. This
strategic context prevents blind optimization of single metrics and provides
wisdom about when goals are achieved and resources should pivot to other
priorities. It transforms budgets from constraints into active decision tools.
```

### The Snapshot (`solution`): Real-Time Reality

The **`solution`** is the **Real-Time Snapshot**. It is the concrete, calculated result of applying a "solver" (which could be a linear program, an LLM, a simple script, or manual input) to the `schema`'s constitution using the latest available data. The result is simple, legible, and public. Recalculating the snapshot is a cheap, safe, low-rank operation. It is the tactical heartbeat of the system, not a strategic shift.

### Budget Solvability

Not every `Budget` can produce a valid `solution`. The schema may define **hard constraints**—minimum resource requirements that must be met before the solver can generate a meaningful allocation. When these constraints aren't satisfied, the `Budget` is **unsolvable**.

This isn't a failure; it's a feature. Unsolvability protects the system from making impossible promises. An unsolvable `Budget` still exists as a **plan**—a clear statement of intent and requirements. It shows stakeholders exactly what resources are needed to move from aspiration to execution. This transparency transforms fundraising from vague appeals into precise, justified requests.

```llm
Budget solvability means the ability to generate a valid solution given
current resources. Hard constraints in the schema define minimum thresholds
below which no meaningful allocation is possible. Unsolvable budgets remain
as plans, making resource gaps explicit and actionable. This protects against
impossible commitments while providing clear funding targets.
```

## The Budget as a Collective Goal

A `Budget` is fundamentally a **plan**—a statement of intent for a future period. This plan can, and often does, exist before it is fully funded.

**The Three Stages of a Budget's Life:**

1.  **Planning Stage**: The `Budget` exists as a pure vision—a `schema` without funding. Teams can collaborate on the plan, simulate scenarios, and build consensus.
2.  **Partial Funding Stage**: Resources begin flowing in. The `Budget` can operate in limited capacity if it has met certain minimum thresholds.
3.  **Full Activation**: All hard constraints are met. The `Budget` operates at full capacity.

The true goal of the enterprise becomes to collectively **fund the `Budget` on all fronts**. It is a target to be achieved, not just a wallet to be spent. Funding arrives asynchronously and in many different "currencies" from various stakeholders:

- **Financial Capital:** Investors or revenue contribute money.
- **Time & Attention:** A `Budget` can define a pool of labor and a calendar deadline.
- **Reputational Capital:** A marketing `Budget` might include a goal for positive media mentions.
- **Community Engagement:** A developer relations team's `Budget` could be funded by merged pull requests.
- **Computational Resources:** A research project can be budgeted in `GPU-Hours` or `LLM-Tokens`.

Each of these is a measurable resource that fills a specific line item in the `Budget`'s grand plan.

```llm
Budgets evolve through three stages: planning (unfunded vision), partial
funding (limited operation above minimum thresholds), and full activation
(all constraints met). They aggregate diverse currencies asynchronously,
with the schema defining both aspirational goals and hard constraints.
This staged approach transforms fundraising into a transparent, measurable process.
```

## Strategic Capabilities

The `Budget` is where human ingenuity and machine intelligence collaborate to navigate the complexities of business.

### Dynamic Planning and Simulation

Because the `schema` is a self-contained economic model, it is a powerful tool for simulation. Stakeholders can test different economic assumptions or forecast the impact of strategic changes before committing real resources. It can gracefully handle resource fluctuations:

- **In a deficit**, the logic finds the optimal compromise, allocating scarce resources to the highest priorities.
- **In abundance**, a surplus is allocated intelligently to growth, savings, or bonuses.

This allows teams to experiment with different economic models and strategies, fostering innovation and data-driven decision-making.

### Fractal Planning: Scaling Across Time

Combined with a time-series stats engine, a `Budget` becomes temporally fluid, enabling **fractal planning**.

1.  **Decomposition (Zooming In):** A high-level, long-term `Budget` can be automatically decomposed into actionable, short-term chunks. A yearly budget can be viewed quarterly or monthly.
2.  **Composition (Zooming Out):** Daily tactical actions are continuously aggregated upwards, providing a real-time, strategic view of progress against goals.
3.  **Extrapolation and "What-If" Scenarios:** The temporal hierarchy enables simulation, allowing extrapolation of current trends to forecast future impacts.

```llm
Fractal planning enables Budgets to fluidly scale across time horizons.
Yearly strategic plans decompose into daily tactical targets, while daily
actions aggregate up to show strategic progress. Combined with the stats
engine's hierarchical aggregations, this creates seamless alignment between
long-term vision and immediate execution, with powerful simulation capabilities.
```

### Evolution Through Refinement

A `Budget`'s `schema` can evolve through the `refine` primitive. Amending a `schema` is a significant, strategic operation, equivalent to amending its constitution. Changes are expected to be backward-compatible extensions—a `narrowing` of the schema—ensuring that processes designed to interact with the older `Budget` do not break.

The primary driver for this evolution is the need for greater granularity, such as splitting a single top-level `Budget` into dedicated sub-allocations for different departments. This is the foundational mechanism for creating a **hierarchy of delegation**.

### Hierarchical Demands: Budgets as Engines of Incentive

When a parent `Budget` delegates to a child, it issues a **charter for a micro-economy**. This charter is the child `Budget`'s `schema`, defining both the **provisions** (e.g., `USD`) and the **work order** (e.g., a target for `articles_written`).

By defining target metrics alongside the resources to achieve them, we establish powerful **incentives and dynamics**. We tell autonomous agents what to care about and give them the means to act. This is how the system orchestrates complex work, creating a market for specific outcomes by creating demand.

A `schema` could also include metrics for `positive_feedback_tokens` to unlock bonuses, or `complaint_tokens` to trigger reviews, creating a powerful incentive system.

#### Flexible Budgets and Backpressure

A parent `Budget` can grant a child `Budget` a flexible spending range (e.g., `base_budget` and `overdraft_allowance`). The child's `schema` can contain logic to tap into the overdraft allowance if certain performance metrics are met. This creates a "backpressure" mechanism, where demonstrated success can unlock further resources.

#### Bottom-Up Metrics and Dialogue

A `Budget`, being an active economic engine, also generates its own metrics (e.g., `average_cost_per_article`). If these metrics indicate the initial budget is insufficient, this information flows back up, triggering a dialogue for intelligent, data-driven adjustments to strategic plans.

### Automated Growth and Custom Currencies

In this model, any metric can be a currency. The `schema` contains the formulas for what to do when these currencies are recorded. For example, a `Budget` could invent a `ProcessImprovementToken`, earned by delivering high-quality features, which can then be "spent" to authorize time for refactoring.

The `Budget` can be set to periodically ingest updated stats and recalculate allocations. The `schema` then automatically allocates outcomes according to its rules, creating self-reinforcing growth engines.

```llm
Budgets enable automated growth cycles where any metric becomes a spendable
currency. Success in one area (e.g., customer satisfaction) automatically
unlocks resources in another (e.g., quality initiatives). The schema defines
these relationships, creating transparent formulas where achievement directly
fuels future capacity, turning budgets into self-reinforcing growth engines.
```

### Decentralized and Nested Economies

The `Budget` model enables decentralization, allowing for isolated financial ecosystems that can operate privately while interacting with a larger network. An enterprise can fund a master `Budget` on the main system and then run its own internal, private ledger to manage that `Budget`. All internal transactions are invisible to the main network, which only sees the high-level funding and settlement transactions.

## Funding and Authority: The Implementation Mechanics

The economic system is built on two simple actions orchestrated by `refine`: funding (a transaction) and delegation (no transaction).

### 1. Funding a Budget (A Transaction)

A "budget" is a distinct `Budget` Vibe, explicitly funded from a source account.

- A user `refine`s a `Budget` template, specifying an amount and source account.
- The router validates authority over the source account.
- A transaction is written to the ledger that debits the source account and credits the new `Budget` Vibe. The new `Budget` now holds those funds.

### 2. Delegating Authority (No Transaction)

Once a `Budget` is funded, the holder can delegate authority over it without creating further transactions.

- Authority is granted with specific permissions (e.g., max spend, allowed resource types).
- **No transaction is generated.** The funds are not moved. The new authorization is a "virtual card" permitted to draw from the parent `Budget`.

This two-step process—fund, then delegate—provides maximum flexibility and control.

```llm
The economic system operates through two foundational actions:
1. Funding creates a Budget Vibe via explicit transaction, moving value from
   source accounts into dedicated pools
2. Delegation grants spending authority without transactions, like issuing
   virtual cards that draw from the funded pool
This separation enables flexible, auditable value management where funding
and authority are independently controlled.
```

## Spending: A Two-Part Transaction

The definitive movement of value out of a `Budget` occurs when a task is posted and completed. This process is handled by two simple, separate transactions that ensure funds are committed and paid out correctly.

### 1. Committing Funds to a Task

When a manager uses their delegated authority to post a task against a `Budget`, the `refine` router validates the request and creates a single transaction: a **debit** from the `Budget`. The value is now considered "in-flight," committed to this task.

The `refine` router performs **three critical validations**:

**a. Rule Validation:** Does the presented authorization permit this action? The router validates the spend against the rules of the specific permission being used.
**b. Balance Validation:** Does the `Budget` have sufficient funds? The router performs a live check on the `Budget`'s current balance.
**c. Solvability Validation:** After this commitment, will the `Budget` still meet its hard constraints?

If all three checks pass, the router writes a single debit transaction to the ledger. If any check fails, the market **rejects the job posting entirely**. This protects workers from accepting jobs that lack proper funding and maintains the integrity of the economic system.

```llm
Market validation enforces three layers of protection: permission rules,
balance sufficiency, and ongoing solvability. A job posting is rejected if
committing its funds would violate any constraint or drop the Budget below
minimum thresholds. This creates a trustless marketplace where every accepted
job has guaranteed, verified funding that won't compromise the Budget's viability.
```

### 2. Settling the Payment

Upon successful completion of the task, a final `refine` call is made. The router creates the second, balancing transaction: a **credit** to the worker's account.

The lifecycle is complete. Two simple transactions—a debit from the payer's `Budget` and a credit to the payee's account—are all that's needed. If the task were cancelled, the credit would be issued back to the original `Budget`.

```llm
Spending uses a two-part transaction model: First, posting a task creates a
debit from the Budget (commitment). Second, task completion creates a credit
to the worker (settlement). This simple model ensures funds are properly
committed and books always balance, without complex escrow mechanisms.
Cancelled tasks simply credit back to the original Budget.
```

## Universal Applications

The `Budget` Vibe is a flexible primitive that can model a wide array of scenarios.

### Financial and Business Use Cases

1.  **Standard Project Funding:** A software project receives a `Budget` with its `schema` defining allocations like `"development": "60%", "design": "20%", "qa": "20%"`.
2.  **Per-Task Escrow:** For a simple freelance job, a `Budget` is created and funded with the exact task amount.
3.  **Subscription Management:** A user's monthly subscription fee funds a personal `Budget`. The `schema` allocates these funds to various services based on usage.
4.  **Team Operational Expenses:** A marketing team gets a quarterly `Budget`. The team lead can delegate spending authority for each category.
5.  **Sales Commission and Profit Sharing:** A company `Budget` tracks `GrossProfit`. Its `schema` dictates how profit is split between costs, reinvestment, and bonuses.

### Advanced and Non-Monetary Applications

6.  **Algorithmic Trading Bot:** A trading bot's `Budget` holds assets. The `schema` _is_ the trading strategy, with rules for execution.
7.  **Grant and Research Funding:** A university research grant is managed as a `Budget`. The `schema` enforces the strict spending rules mandated by the grant provider.
8.  **Usage-Based Resource Allocation:** A data science team might get a monthly `Budget` of `1000 GPU-Hours` and `50M LLM-Tokens`.
9.  **DAO Treasury Management:** A DAO manages its treasury as a master `Budget`. The `schema` reflects community-approved spending proposals.
10. **Game Economy:** In a video game, a player's inventory is a `Budget` holding resources like `Gold`, `Wood`, and `Stone`.
11. **Personal Development Engine:** An individual's `Budget` tracks efforts and skills. It can be funded with time and money, with a schema defining goals like increasing a `Python-Proficiency` metric. The `Budget` transforms effort and investment into tangible, measurable skills.

### Universal Resource Management

A `Budget` does not need to be funded with liquid assets. It can be funded with **unrealized potential**, like physical assets or abstract ones like intellectual property. The `Budget` then becomes the vehicle for a planned conversion, with a `schema` containing rules for how to convert "Expertise" into `ConsultingHours` (a currency), which can then be "sold" to generate `USD`.

The transaction ledger is resource-agnostic and can manage any quantifiable resource (`LLM-Tokens`, `GPU-Hours`, `Storage-GB`, `API-Credits`, `DeveloperReputation`, etc.).

```llm
The transaction ledger is resource-agnostic, managing any quantifiable value.
Budgets can be funded with unrealized potential (physical assets, expertise)
and contain rules for conversion. Any resource can be tracked: money, compute,
storage, reputation. The refine primitive enables cross-resource transactions,
creating a unified system for all forms of value exchange and transformation.
```

## The Technical Backbone

This section describes the key architectural components that make the dynamic `Budget` system possible.

### Decoupling for Speed: The Role of the Aggregation Layer

This model relies on a **decoupled, real-time aggregation layer** (e.g., TimescaleDB) for a real-time view of `Budget` balances.

- **The Ledger is the Source of Truth:** The `refine` router writes every transaction to the immutable ledger.
- **Events Feed the Aggregator:** The ledger emits an event for each new transaction.
- **The Aggregator Provides Live Balances:** The aggregation layer consumes these events to maintain a continuous, low-latency view of all `Budgets`.
- **Validation Happens Against the Fast Layer:** The router queries this fast aggregation layer, not the slow main ledger.

This architecture provides the integrity of an event-sourced ledger with the high performance of a real-time balance-checking system.

### Auditing the Flow: Transaction Lineage

A primary benefit of the immutable transaction ledger is perfect auditability. Because every transaction is linked to the `Budget` that funded it and the permission that authorized it, we can trace the complete lineage of any unit of value.

For example, a query on a payment can reveal its entire history:

- `CREDIT Balance:Designer-Jane +$1000` `(Settlement for AdCampaign-123)`
  - `└─ DEBIT Budget:Marketing-2025 -$1000` `(Commitment for AdCampaign-123)`
    - `   └─ CREDIT Budget:Marketing-2025 +$200k` `(Funded from Treasury)`
      - `      └─ DEBIT Balance:Company-Treasury -$200k` `(To create Marketing budget)`
        - `         └─ CREDIT Balance:Company-Treasury +$1M` `(Initial funding event)`

This clear, unbroken chain provides complete transparency into how funds are allocated and spent.

```llm
The technical architecture separates the immutable ledger (source of truth)
from a real-time aggregation layer (performance). Events flow from ledger
to aggregator, maintaining live balances for fast validation. Every
transaction preserves complete lineage, enabling perfect auditability from
initial funding through final settlement. This provides both integrity and speed.
```
