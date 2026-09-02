# 801: Package/Agent

> [!DEFINITION] [Agent](./000_glossary.md)
> A reference implementation of the **Acts of Emergence** protocols. It provides the runtime engine for executing schema-driven, AI-native workflows, from atomic Requests to complex, stateful Agents.

> Sidenote:
>
> - Implements:
>   - :term[001: Agent/Request]{href="./001_agent_request.md"}
>   - :term[010: Agent/Loop]{href="./010_agent_loop.md"}
> - Links:
>   - [NPM: @idealic/agent](https://www.npmjs.com/package/@idealic/agent)

The `@idealic/agent` library is the canonical implementation of the agentic architecture defined in the Acts. Unlike frameworks that prioritize prompt engineering, this library prioritizes **protocol compliance**. It implements a rigorous state machine where every interaction is strictly typed, schema-driven, and distinct.

Two consequences follow, and between them they are the shape of the package. Each Act is a discrete capability rather than a layer of one framework, so simple tool use costs nothing of claims, instancing or delegation. And every capability the runtime has was specified before it was written, which is why the register below is the package's table of contents rather than a summary of it.

The library's own README answers a different question — whether this does what a particular reader needs, what each capability costs them in caveats, and where the door is. The register says where a rule lives; the README says what the thing is like to use. A number belongs to neither: a measurement carries a date, a model and an endpoint, and lives in [measurements](../measurements/README.md).

## What the Package Admits

A capability enters the library when an Act specifies it, and in that order — the Act first, the runtime after. A behaviour with no Act behind it has nothing to be checked against, which is the sense in which the register is also the boundary: the library holds no memory store, no retriever and no hosted service, because none of those is a protocol the Acts define.

## The Register

### Core Primitives

- :term[001: Agent/Request]{href="./001_agent_request.md"} — the atomic unit of computation. Transforms a `Context` and a `Schema` into a structured `Solution`, supports multiplexing, and is the foundation every higher capability is built on.
- :term[002: Agent/Tool]{href="./002_agent_tool.md"} — a schema definition of a capability, defining the interface for an action separately from its implementation.
- :term[003: Agent/Activity]{href="./003_agent_activity.md"} — the deterministic code behind a Tool, connecting the abstract schema to real work. A Tool with no Activity is answered from the model's own reasoning instead.
- :term[004: Agent/Call]{href="./004_agent_call.md"} — a concrete, parameterized instance of a Tool use, and the standardized transport between the model's intent and the system's action.

### Data & State

- :term[005: Agent/Data]{href="./005_agent_data.md"} — the uniform envelope for structured information in the context, and the rules by which two envelopes merge.
- :term[006: Agent/Input]{href="./006_agent_input.md"} — strict input parameters that turn a generic Request into a reusable, type-safe function.
- :term[009: Agent/State]{href="./009_agent_state.md"} — the shared scratchpad that survives across steps, which is what makes a run resumable.
- :term[016: Agent/Meta]{href="./016_agent_meta.md"} — identity and lineage: versioning, branching and origin, so an agent can evolve and record that it did.

### Wiring & Flow

- :term[007: Agent/Variables]{href="./007_agent_variables.md"} — `†kind.path` references that let a Call read from the context by address instead of by copy.
- :term[008: Agent/Output]{href="./008_agent_output.md"} — `_outputPath`, which says where a result is written and therefore what becomes readable next.
- :term[011: Agent/Expressions]{href="./011_agent_expressions.md"} — a parameter that computes over references rather than looking one up. Three operators the host language cannot parse — `<|>` choose, `*>` gate, `|>` flow — so none of them can be quietly read as JavaScript's own.

### Orchestration

- :term[010: Agent/Loop]{href="./010_agent_loop.md"} — the execution engine: request, drain the Calls the context already permits, feed the results back, repeat until a solution is written.
- :term[012: Agent/Plan]{href="./012_agent_plan.md"} — the workflow as a data-flow graph of Calls, separating what is planned from what is executed.
- :term[013: Agent/Instancing]{href="./013_agent_instancing.md"} — `_instance` grouping, which aims one Plan at many subjects without copying it per subject.
- :term[014: Agent/Delegate]{href="./014_agent_delegate.md"} — sandboxed sub-requests, which is how an agent becomes a Tool of another agent.
- :term[015: Agent/Scopes]{href="./015_agent_scopes.md"} — `_scopes`, which strictly bound what a Delegate or Activity is allowed to see.
- :term[017: Agent/Advisor]{href="./017_agent_advisor.md"} — personas that reason before the primary agent acts, with weighted voting over the guidance they give.
- :term[018: Agent/Question]{href="./018_agent_question.md"} — putting an unsettled matter to another party as a Call, with answers accumulating at one path so a position can change and its reasoning survives.

### Cross-Cutting Disciplines

These apply at several seams rather than at a stage of the run.

- :term[090: Agent/Typing]{href="./090_agent_typing.md"} — computing a schema's inferred type once, carrying it on the schema, and making it survive module boundaries.
- :term[091: Agent/Caching]{href="./091_agent_caching.md"} — ordering a context so the unchanging part comes first, because a provider caches by prefix and a prefix ends at the first byte that moved.
- :term[092: Agent/Limits]{href="./092_agent_limits.md"} — what a run may spend in turns, wall clock, tokens, money and held cache: shown to the model as advice, enforced by the loop as law.
- :term[093: Agent/Compaction]{href="./093_agent_compaction.md"} — releasing the account a run gives of its work while the writes that account narrated still resolve.
- :term[094: Agent/Config]{href="./094_agent_config.md"} — model, provider, thinking budget and sampling, shown to the model as a message and changeable by it within a declared allowance.
- :term[095: Agent/Claims]{href="./095_agent_claims.md"} — a named expression kept for the length of the run, which a Call both reads and waits on. Two Calls reading claims that cannot both hold are two branches decided without spending a turn.
- :term[096: Agent/Hooks]{href="./096_agent_hooks.md"} — one registration primitive with two verbs: an interceptor sits in the path and may replace what flows through it; an observer sits beside it and may change nothing.

### The Concept Behind the Default

- :term[104: Concept/Latent]{href="./104_concept_latent.md"} — when a Tool has no registered Activity, the model owes the output itself. A workflow can therefore be prototyped entirely in schemas, with code added only where a real implementation is wanted, and the Tool's interface does not move when it arrives.

## Outro

The register is deliberately a list of doors rather than a description of what is behind them: each Act carries its own rule, and the library is answerable to that rule rather than to any summary of it.

The same relationship holds one level up, between a running agent and the surface a person meets it through. :term[802: Package/UI]{href="./802_package_ui.md"} takes the schemas these protocols pass around and turns them into the interface that renders and edits them.
