# 850: Package/Game Service

> [!DEFINITION] [Game Service](./000_glossary.md)
> A special program that runs in the background to manage turn-based games, like chess or poker. Think of it as the game's official referee. It knows the true state of the game, handles every player's move, uses different rulebooks to advance the game, and saves everyone's progress.

> Sidenote:
> - Powered by:
>   - :term[851: Package/Poker Engine]{href="./851_package_poker_engine.md"}
> - Compatible with:
>   - :term[852: Package/Poker UI]{href="./852_package_poker_ui.md"}

The **Game Service** is a special backend program, like the engine of a car, that runs and manages turn-based games. It's built like a game console that can play different game cartridges. We're starting with poker, but it’s designed to handle any kind of turn-based game. It also acts as a safe playground for developers to create and test new game ideas without breaking anything.

## Core Architecture

The service works like a smart vending machine—it doesn't remember who you are from one turn to the next. Every time you make a move, your app sends the game's entire history to the service. The service checks this against its official record, approves the move, and then sends the newly updated history back to everyone playing.

### Pluggable Game Engines

The service itself doesn't know the rules to any specific game. It's just a host, like a TV that can play movies from a Blu-ray player or a streaming stick. The actual game rules are inside **Pluggable Engines**—which are like the game cartridges or the streaming apps.

- **Registry Pattern:** The service keeps a list of all the different game "cartridges" it knows how to load.
- **Consistent Interface:** Each game engine or "cartridge" must have the same basic buttons, like `Next Turn` or `Join Game`, so the service knows how to talk to it.
- **Separation of Concerns:**
  - **Game Engine (e.g., Poker):** This part handles the actual rules of the game—what moves are legal and what happens next.
  - **Game Service (This Program):** This part handles everything *around* the game, like connecting players over the internet, saving progress, managing turns, and enforcing time limits.

### The "State" as Source of Truth

The most important concept for the service is the **State Object**. Think of this not just as a photo of the game board right now, but as a complete written diary of every single move ever made, just like the notes a chess master takes during a match.

- **Stateless Communication:** Every time a player makes a move, their app sends this entire game diary to the server. The server adds the new move and sends the complete, updated diary back out to everyone.
- **Determinism:** The game diary even includes moments of "luck," like how the cards were shuffled. This means you can perfectly replay any game from the beginning, which is great for checking if everything was fair.
- **Player Perspective:** The server is smart. Before sending the game diary out, it creates a personal copy for each player, hiding the things they're not supposed to see (like another player's cards in poker).

## I/O Abstraction Layer

The service doesn't care *how* it communicates with the outside world—whether through a website, a phone app, or something else. All of its communication abilities are kept in a single toolbox. If you want to use a different communication method, you can just swap the tools in the toolbox without changing the service itself.

- **State Persistence:** `saveGame` and `loadGame` are the tools for saving and loading the official game diary.
- **Session Management:** `fetchPlayerStacks` and `savePlayerStacks` are tools that talk to other systems to manage a player's chips or money.
- **Real-time Communication:** `broadcastToPlayers` is a tool that acts like a loudspeaker, sending the latest game updates to all players at once.
- **Background Processes:** `fetchTimedOutGames` lets a timer check in with the service to see if any players have taken too long to make their move.

## Table Service Concept

This explains how game tables (like a poker table) are created. Instead of having a fixed number of tables waiting around, a "table" is simply any game that is currently being played.

- **Dynamic Allocation:** Tables are created only when someone needs one. If you want to play a game of Texas Hold'em and all the current tables are full, the system instantly creates a new one just for you.
- **Lifecycle:**
  1.  **Find:** You ask to play a specific game. The system finds you an open seat or makes a new table.
  2.  **Observe:** You can watch the game to see what's happening before you join.
  3.  **Join:** You decide to play and send a signal to take a seat.
  4.  **Play:** The game continues with you in it.
  5.  **Terminate:** Players are removed from the game if they run out of time or money.

## How It Works

Here is the step-by-step loop of how the service operates:

1.  **Trigger:** An event happens, usually a player making a move.
2.  **Load:** The service loads the official, saved history of the game.
3.  **Merge:** It adds the player's new move to the official history.
4.  **Advance:** The service uses the game's rulebook (the engine) to run the game forward until it needs another player to do something.
5.  **Complete:** If a round of the game ends, the service wraps it up, hands out any winnings, and sets up the next round.
6.  **Persist & Broadcast:** It saves the new, updated game history and sends it to all players so they can see what just happened.