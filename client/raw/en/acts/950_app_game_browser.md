# 950: App/Game Browser

> [!DEFINITION] [Game Browser](./000_glossary.md)
> A versatile, real-time operations dashboard designed to monitor game ecosystems, visualize table activity, and manage AI agents. It is a universal viewer that connects to any data source compliant with the game service standard.

> Sidenote:
>
> - Powered by:
>   - :term[850: Package/Game Service]{href="./850_package_game_service.md"}
> - Uses:
>   - :term[852: Package/Poker UI]{href="./852_package_poker_ui.md"}

The **Game Browser** is a client application that serves two distinct but complementary roles: **Operational Overwatch** for managing live venues and bots, and **Historical Forensics** for analyzing past sessions and importing external game data.

Unlike traditional game clients coupled to a specific server, the Game Browser is designed as a universal viewer. It can connect to any system that outputs data in the standard `Engine.State` format, whether that is a live WebSocket feed, a REST API, or a static log file.

## Core Philosophy: Games > Tables

A fundamental architectural decision in this application is the abstraction of the "Table."

- **No Table Synchronization**: We do not treat "Tables" as primary entities that need to be synchronized across systems. This eliminates complex state synchronization issues often found in lobby management.
- **Tables as Game Streams**: Instead, a "Table" is dynamically reconstructed as a series of sequential **Game States**. If we have the game records, we have the table.
- **Derived State**: By syncing only the _Games_ (the atomic units of gameplay), the system can deterministically reconstruct the entire state of a Venue, including active tables, seated players, and chip counts, without needing a separate "Lobby Service."

## Capabilities

### 1. Real-Time Venue Browsing

The application provides a "God Mode" view into any connected venue:

- **Live Feed**: Watch games unfold in real-time with zero latency (depending on data source).
- **Multi-Table View**: Monitor multiple tables simultaneously to spot trends, issues, or high-stakes action.
- **Session Reconstruction**: Seamlessly view a table's history. Because tables are streams of games, scrolling back in time is as natural as scrolling through a chat log.

### 2. Universal Data Ingestion

The Game Browser is agnostic to where the data comes from. It acts as a visualization layer for the standard `Engine.State` format.

- **Third-Party Integration**: Can ingest game streams from external casino systems, unifying disparate platforms into a single dashboard.
- **Log Restoration**: Capable of parsing and "replaying" flat logs (e.g., PokerStars hand histories, server dumps). This effectively turns static logs into an interactive, replayable session.
- **Hybrid Deployment**: Can simultaneously display live games from our internal engine and historical games from an imported dataset.

### 3. Aggregated Analytics

Because the system perceives the world as a collection of Games, statistics are naturally derived by aggregating these atomic units.

- **Player-Centric Stats**: As players participate in games across different tables and venues, the system aggregates their performance (VPIP, PFR, Win Rate) into a unified profile.
- **Venue Health**: Analyze table occupancy, hands per hour, and economy flow by aggregating game metadata.
- **On-Demand Calculation**: Stats can be computed retroactively on imported data, allowing for deep analysis of historical performance.

### 4. AI Control Plane

Beyond passive viewing, the Game Browser is the primary interface for the AI Runtime.

- **Bot Management**: Monitor the health, bankroll, and status of AI agents.
- **Decision Inspection**: (When permitted) View the internal reasoning or "thought process" of the AI during a hand.
- **Intervention**: Controls to pause, stop, or reconfigure bot deployments in real-time.

## Architecture & Extensibility

The Game Browser is built to be **Game Agnostic**.

- **Pluggable Renderers**: While currently optimized for Poker (Hold'em/Short Deck) using the :term[Poker UI]{href="./852_package_poker_ui.md"}, the application can load different renderers based on the `game.type` field. A single dashboard can monitor Poker, Blackjack, and Roulette tables side-by-side.
- **Flexible Contexts**: The app can be deployed in various modes:
  - **Standalone App**: A desktop-like experience for venue managers.
  - **Embedded Component**: A widget within a larger casino back-office.
  - **Public Observer**: A restricted view for public spectating (e.g., tournament rails).
