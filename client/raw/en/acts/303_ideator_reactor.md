# 303: Ideator/Reactor

> **Reactor:** A specific `Idea Transformer` designed as a universal runtime for stateful, turn-based agent interactions. It takes a game/process state (`Idea`) and produces the next state (`Idea`).
>
> — [Glossary](./000_glossary.md)

This document outlines the vision for refactoring the existing poker runtime into **"The Reactor"**—a universal and extensible system for context management for real-time agents. This evolution transforms our powerful Temporal-based workflow system from a single-purpose application into a protocol for intelligent, autonomous agents to interact with any turn-based environment, be it a zero-sum game, a collaborative simulation, or a complex business process. The Reactor will be the first "reactor engine" for our broader idea network.

## From Poker-Specific to Protocol-Driven

Our current runtime is a success. It uses Temporal workflows to manage a complex microcosm of poker bots that play, compute, and react within a casino environment. We have also designed an API for an abstract game engine to separate game logic from the runtime. Our poker engine is the first implementation of this API, setting the stage for a future where various game engines can be plugged in and out seamlessly.

However, the inter-communication within our system is still hard-coded to a specific game notation (Poker Hand History Format). To truly unlock the potential of our system, we will move beyond this limitation.

The core of this refactoring is the adoption of the **"Idea Triplet"** as the fundamental unit of communication. Instead of sending a simple game state object, every message within the system will be a self-contained triplet:

1.  **Schema:** A JSON schema that provides a universal, semantic definition of the game state. It not only makes the data self-describing but also contains sufficient information to determine the next valid action or state transition.
2.  **Context:** Additional information, such as game rules, bot instructions, or historical data, that informs the interpretation of the solution.
3.  **Solution:** The actual game state, which conforms to the schema.

Together, the `Schema` and the `Solution` form a complete, self-contained snapshot of the game state. The `Solution` by itself is an ambiguous representation of the game; it is merely data. The `Schema` provides the critical context, defining the vocabulary, structure, and rules. It is sufficient to understand what can be added to the game notation and how another player can react, making it possible to determine the next valid move. This means any component can understand not just the current state, but its immediate possibilities, without prior knowledge of the game's implementation. A key technical decision supporting this is that the triplet will be the sole unit of data passed through the critical path of all workflows; we are moving away from passing partial states.

This makes the `Idea` a true computational primitive—a building block for creating complex, evolving systems. Unlike a simple, ephemeral request to a chatbot, an `Idea` is a self-contained, stateful artifact. It packages the `input`, the `output` (`solution`), the rules (`schema`), and the entire `context` of its creation into a single, portable unit. It's not just a question; it's the question, the answer, and the complete formula that connects them, enabling a persistent, composable system, not just a one-off transaction.

## From Latent to Explicit: A Crystallization Strategy

This architecture enables a powerful approach where gameplay logic begins with a flexible **Latent Ruleset** and can gradually crystallize into a deterministic **Explicit Ruleset**. The system is designed to operate with the Latent Ruleset by default, treating an Explicit Ruleset as a powerful, progressive enhancement rather than a prerequisite.

### 1. Latent Ruleset

Our primary design principle is to start with the "worst case" scenario: a game for which no explicit engine exists. The system is built to function seamlessly in this mode by default.

Here, the LLM acts as a **"Universal Interpreter,"** effectively becoming the game's dealer and referee. It uses the `Schema` and `Context` from the triplet to understand the rules, validate actions, and advance the game state intelligently. The ruleset is "latent"—contained within the vast knowledge of the LLM and the information provided in the triplet, rather than in explicit code. The baseline "universal engine" will be minimal, managing only the seating of participants and the basic turn-taking sequence. The LLM handles the rest, allowing for unprecedented adaptability.

### 2. Explicit Ruleset

For games where we have a dedicated engine (like our existing poker engine), the system operates in an accelerated, deterministic mode. The engine provides an Explicit Ruleset that does not replace the triplet's core function, but powerfully augments it. It serves two primary functions:

- **Ensuring Determinism and Predictability:** The engine validates all actions against the game's strict rules, confirming what a player can do at any given moment and verifying the actions chosen by the bots. This guarantees predictable and fair gameplay.

- **Context Engineering:** The engine acts as a sophisticated context provider. It enriches the `Context` part of the triplet with high-value, domain-specific information. This includes historical statistics on player behavior, gameplay patterns, and other relevant analytics. This "engineered context" allows our bots to make far more intelligent and strategic decisions.

In this mode, the game engine provides explicit, hard-coded logic that acts as a source of truth and enhances the decision-making quality of our agents, ensuring the system is both reliable and smart.

### 3. From Simulation to Specification: A New Development Paradigm

This strategy of crystallization from latent to explicit logic unlocks a revolutionary approach to development. Instead of writing a specification for a new game from scratch, we will first use "The Reactor" with its default Latent Ruleset to _play_ the game.

1.  **Simulate:** We provide the Reactor with a triplet describing a new game (e.g., Dominoes), even a completely novel one. The LLM, acting as the universal interpreter, will begin to simulate gameplay.
2.  **Generate:** As the LLM plays, it generates logs, reasoning, and a wide variety of gameplay scenarios.
3.  **Specify:** We then use this rich, simulated output as the raw material to write a robust, battle-tested specification and a comprehensive suite of tests.

The simulation precedes and informs the code. We can understand and explore a game's dynamics and edge cases before a single line of deterministic engine code is written. This dramatically accelerates development and leads to more robust, well-defined game engines.

## Stateful Agents and Long-Term Memory

A critical capability of The Reactor is that the agents operating within it are stateful. They are not simple bots executing one-off commands. Because the system is built on persistent workflows, each agent can maintain a long-term memory of its interactions.

An agent can remember what happened in previous rounds, recall strategies from entirely different games, and learn the tendencies of other players over time. This statefulness is a core differentiator, transforming agents from simple reactive components into entities that learn and adapt within their environment. This allows for far more sophisticated and strategic behavior than a stateless request-response model could ever achieve.

## Business Implications

This architectural shift has profound business implications:

- **Rapid Onboarding:** We can offer clients the ability to integrate new games and even complex business processes with minimal engineering effort. The universal path allows us to start running bots for a new game the moment we receive its schema, even before a dedicated engine is implemented.
- **A Clear Specification:** We can provide a clear specification format (the game state `Schema`) to clients or third-party developers, allowing them to build new game engines that plug directly into our runtime. Our simulation-first approach will help us and our partners generate these specifications faster and more accurately.
- **Specification as a Service:** For clients struggling to define complex systems (like a membership or rewards program), we can treat their process as a "game." Our Reactor can simulate the process using the Universal Path, helping them clarify the rules and automatically generating a robust specification and test suite. This solves a major business bottleneck and becomes a valuable service in itself.
- **Immediate Functionality:** Our system can start running bots for a new game the moment we receive its schema, even before a dedicated engine is implemented.

By refactoring our runtime to be protocol-driven, we are not just improving an existing product. We are building a future-proof foundation for an ecosystem of autonomous agents capable of mastering any game, creating limitless opportunities for expansion and innovation.

## Alignment with the Broader Vision

This strategic shift does more than just enhance our gaming product; it aligns our entire product suite under a single, unified vision. This work is an immediate and practical extension of our goal to build a decentralized, peer-to-peer network for ideas, as outlined in the _Edict of Autonomy_.

**The First Client and a Proof of Concept**
This refactored runtime will serve as the first custom client for our **"Idea Triplet"** protocol. It is a critical proof of concept, demonstrating in a real-world, commercial application how this abstract model can create tangible value, flexibility, and intelligence.

**Beyond Gaming: A Universal Interaction Model**
The architecture we are building is not confined to games. It provides a generic and powerful model for AI agents to react to discrete changes in any environment. If you squint, everything can be viewed as a turn-based game: a conversation, a financial transaction, a project management workflow.

The Reactor is the first of many such "Idea-Transformer" microservices. The same protocol will support a composable ecosystem of services, including:

- A **Resolver** service that enriches an `Idea` by fetching and embedding linked data from a database.
- A **Stats** service that analyzes an `Idea` and appends historical or statistical context.
- A **Validation** service that checks an `Idea` against a set of rules and provides feedback for retries.

By building a system that masters this interaction model, we are creating a foundational technology—a composable "Unix-like tool" in a larger ecosystem of services—that can be applied to a vast array of future products and services, making this project a cornerstone of our company's future.
