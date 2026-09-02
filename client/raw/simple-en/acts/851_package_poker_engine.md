# 851: Package/Poker Engine

> [!DEFINITION] [Poker Engine](./000_glossary.md)
> Think of the Poker Engine as the super-smart "brain" for a poker game. It's a solid set of rules for games like Texas Hold'em, but it's built so flexibly that it could run almost any game. It keeps everything organized by using a special language for the game's `State` (the memory), the `Game` (the live action), and the `Commands` (the player moves).

> Sidenote:
> - Brought to life by:
>   - :term[850: Package/Game Service]{href="./850_package_game_service.md"}
> - Shown on screen by:
>   - :term[852: Package/Poker UI]{href="./852_package_poker_ui.md"}

The **Poker Engine** (`@idealic/poker-engine`) is a toolbox for building and managing poker games. It provides all the specialized tools needed to start a game, play through it, and finish it. While it's focused on poker right now, the way it's designed could be used to create any game where players take turns.

## Reliability

The engine is built like a high-precision watch. Every single rule was written down and defined before any code was created. It's checked by about **3,000 automatic tests** that cover every situation you can imagine—from normal betting to really weird and complicated scenarios—to make sure it's always fair and never crashes.

Also:

- **Chaos Testing:** We constantly test the engine by throwing random, messy situations at it, like a real-world game where anything can happen, to make sure it stays stable.
- **Battle-Tested:** The engine's core logic comes from a system that's already been proven to work with the same format used by **PokerStars**. It has been tested against a library of over **10 million real poker hands** from the past.

## Design Principles

The engine is built on six main rules that it never breaks:

1.  **Never Erase History:** Every time something changes, the engine saves a new copy instead of changing the old one. This gives you a perfect, preserved history of the entire game.
2.  **Perfectly Replayable:** If you have the starting 'seed' (a secret number), you can replay any game exactly as it happened, card for card. It's like having a perfect video recording.
3.  **Check First, Act Second:** The engine always checks if a move is legal *before* it lets it happen. This prevents anyone from accidentally or intentionally breaking the rules and corrupting the game.
4.  **Everyone Has One Job:** Different parts of the engine do different things. The `State` just holds data, the `Game` runs the live action, and the `Processors` think about the rules. This keeps everything clean and organized.
5.  **A Story of Actions:** Everything that happens in the game is just a simple, clear command. This creates a step-by-step history of the game that's easy to follow.
6.  **A Common Language:** All parts of the engine use the same set of commands and messages to talk to each other, so there's no confusion.

## The Four Departments Pattern

Think of the engine as a TV studio control room with four main departments that work together.

### 1. The State Department (The Official Script)

**`Poker.State`** is the official record of the game.

- **A Universal Story:** It's written in a way that could describe any turn-based game, like Chess or Checkers, not just Poker.
- **The Single Source of Truth:** This is the one and only true history of everything that's happened in the game.
- **A Read-Only Record:** It's like a history book. You can't change what's already written; you can only add a new page.
- **Different Viewpoints:** It can show different versions of the script to different players. It will show you your secret cards, but keep them hidden from everyone else.

### 2. The Game Department (The Live Studio)

**`Poker.Game`** is the director running the live show.

- **The Live Game:** It takes the official script (`State`) and turns it into an active, live game that you can interact with.
- **The Referee:** It enforces all the rules, checks if your moves are legal, and manages the flow of the game.
- **A Multi-Tool:** It's like a Swiss-army knife for the game, keeping track of time, who the players are, and what's happening right now.
- **Making it Happen:** The `applyAction(game, action)` command is the main 'GO' button. When you make a move, this department updates the live game for everyone.

### 3. The Command Department (The Player's Remote)

**`Poker.Command`** is how you tell the game what to do.

- **The Translator:** When you click a button like 'Bet' or 'Fold', this department translates your click into a simple command the engine understands.
- **Smart Suggestions:** It's smart. It looks at the live game to figure out your options. If you click 'All-In', it knows exactly how much money that is for you.
- **Keeps Things Separate:** This keeps the buttons you see on the screen separate from the game's brain, which makes everything easier to manage.

### 4. The Format Department (The Packer & Shipper)

**`Poker.Format`** is in charge of packaging and sending data.

- **Packing & Unpacking:** It's an expert at shrinking game data into a tiny package to send over the internet and then unpacking it perfectly on the other side.
- **Reading History:** It can also read the text files from old poker games (like the ones from PokerStars) and turn them back into game data the engine can use.

## Statistics & Analytics

The engine has a dedicated **`Poker.Stats`** department for tracking everything.

- **The Big Idea:** It separates **Stats** (simple counts, like 'how many times you raised') from **Metrics** (smarter calculations, like 'the *percentage* of times you raised').
- **Location, Location, Location:** It knows *where* you were sitting when you made a move. Raising when you're first to act is different than when you're last, and it tracks that difference.
- **Creating Reports:** It can gather all this data to create reports on a single player, a specific part of the game, or the entire table.

## System Architecture

### Where the Engine Lives

The Poker Engine runs in two places at once: your device and the main server.

- **On Your Device:** A copy runs on your computer or phone to make the game feel instant. When you make a move, you see it happen right away.
- **On the Server:** The main engine lives on the server. The server is the final judge that checks every move, makes sure the game is fair, and controls the randomness of the cards.

### How It Connects

- **Backend:** The **[@idealic/game-service](../game-service/README.md)** handles saving games, connecting players, and managing the online parts.
- **Frontend:** The **[@idealic/poker-ui](../poker-ui/README.md)** is what draws the game on your screen.

## How a Game Plays Out

The Poker Engine works like a step-by-step assembly line.

1.  **Getting Started:** Players ask to join. The system finds a table for them. The game waits until at least two people are ready to play.
2.  **The Deal:** The server starts a new game with a secure, random shuffle. It deals the cards, saves the game's state, and sends it to all the players.
3.  **Taking Turns:** A player sends a move. The engine checks it, updates the game for everyone, and saves the new state. This loop continues until the hand is over.
4.  **The Finish Line:** The hand ends. The engine figures out the winner, moves the dealer button, and gets the next hand ready.
5.  **Coming and Going:** If a player wants to leave or take a break, the engine waits until the current hand is over before processing their request.
6.  **Waiting Around:** If there are fewer than two players at the table, the game pauses.
7.  **Time's Up:** A timer makes sure the game keeps moving. If someone takes too long to act, the engine will make a default move for them (like folding).

## Core Abilities

- **Game Tracking:** It perfectly keeps track of the cards, all the bets, and even really tricky situations like side pots.
- **Rule Following:** It strictly enforces all the rules for betting, moving to the next round, and figuring out the winner at the end.
- **Player Tracking:** It knows where everyone is sitting (like who is the Dealer or Big Blind), how much money they have, and everything they've done.
- **Game Flow:**
  - **The Coordinator:** A part of the engine called the 'Croupier' coordinates the game's flow.
  - **The Auto-Dealer:** Another part acts as an automatic 'Dealer' that handles dealing cards and moving from one round to the next.
  - **The Showdown:** It uses a super-fast lookup system to instantly compare everyone's cards and know who won.

## The Action Language

The engine uses a simple, text-based language for every event:

- **Player Moves:** `'p1 f'` (Player 1 folds), `'p2 cc 20'` (Player 2 calls 20), `'p3 cbr 100'` (Player 3 raises to 100).
- **Dealer Actions:** `'d dh p1 AsKs'` (Dealer deals hole cards Ace of Spades, King of Spades to Player 1), `'d db AhKhQh'` (Dealer deals the board: Ace, King, Queen of Hearts).
- **Chat:** `'p4 m Hello!'` (Player 4 sends a message).

This simple format is easy for people to read (which helps for checking game history) and is the exact same language computers use to send commands to the engine.
