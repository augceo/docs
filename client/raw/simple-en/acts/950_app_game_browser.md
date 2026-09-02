# 950: App/Game Browser

> [!DEFINITION] [Game Browser](./000_glossary.md)
> Think of this as a control room for online games. It’s a special dashboard where you can watch games live, see what’s happening at every table, and manage the computer-controlled players (the AIs). The best part is that it’s a universal remote—it can connect to any game system, as long as that system speaks a language it understands.

> Sidenote:
> - Powered by:
>   - :term[850: Package/Game Service]{href="./850_package_game_service.md"}
> - Uses:
>   - :term[852: Package/Poker UI]{href="./852_package_poker_ui.md"}

The **Game Browser** is an app that does two big things. First, it’s like a **live security camera**, letting you watch over games as they happen. Second, it’s like a **detective's tool**, letting you look back at old games to see exactly what went down.

Most game apps are built to work with only one specific game server. This Game Browser is different; it's a universal viewer. It can connect to anything that gives it game data in the right format—whether that's a live video feed, a website, or just a simple text file of old games.

## Core Idea: Games Are More Important Than Tables

A key decision we made was how to think about a game "Table."

- **No Keeping Tables in Sync**: We don't treat the "Table" as the most important thing. Trying to make sure every computer agrees on what a table looks like is very complicated.
- **Tables are Streams of Games**: Instead, we think of a Table as just a story told one sentence at a time. Each "sentence" is one game or one hand of poker. If you have all the sentences in the right order, you have the whole story of the table.
- **Figuring Things Out on the Fly**: By only tracking the individual *Games*, our system can perfectly rebuild the entire casino scene—who is at which table, how many chips they have—without needing a separate program to manage the lobby.

## What It Can Do

### 1. Watch Live Games

The app gives you a “god mode” view of any casino it's connected to:

- **Live Feed**: Watch games play out in real-time, just as they happen.
- **Multi-Table View**: Keep an eye on many tables at once to spot important moments or problems.
- **Rewind History**: You can easily see what happened earlier at a table. Since a table is just a stream of games, scrolling back in time is as easy as scrolling up in a chat message.

### 2. Connect to Anything

The Game Browser doesn't care where the game information comes from. It just needs the data to be in the standard `Engine.State` format, and it will know how to show it.

- **Works with Other Systems**: It can connect to game systems from other companies, putting everything you need to see onto one screen.
- **Replay Old Games**: It can take a boring text file of an old game (like a PokerStars game history) and turn it into an interactive movie that you can watch and explore.
- **Mix and Match**: You can watch a live game from our system and a replay of an old game from another system, side-by-side on the same screen.

### 3. Smart Statistics

Since the system sees the world as a collection of tiny game pieces, it’s really good at adding them up to get interesting statistics.

- **Player Stats**: As players move between tables and casinos, the system tracks their performance and builds a single profile showing how they play.
- **Casino Health**: You can check the casino's pulse by looking at how busy the tables are, how many hands are played per hour, and how money is moving around.
- **Analyze the Past**: You can run an analysis on old game data to understand how players or the casino performed in the past.

### 4. AI Remote Control

Besides just watching, the Game Browser is the main remote control for all the AI players.

- **Manage Bots**: Check on the AI players to see if they are working correctly, how much money they have, and what they are doing.
- **See the AI's Thoughts**: If you have permission, you can look inside the AI's “brain” to see its reasoning for making a certain move.
- **Step In**: You have controls to pause, stop, or change the instructions for the AI players at any time.

## Design & Flexibility

The Game Browser is built to work with **any kind of game**.

- **Pluggable Visuals**: Think of the app like a game console. You can pop in a “Poker cartridge” to see poker games, or a “Blackjack cartridge” to see blackjack games. The main system stays the same; you just swap out the part that knows how to draw the specific game. With this, you can watch Poker, Blackjack, and Roulette all on the same screen.
- **Different Modes**: The app can be used in a few different ways:
  - **Full App**: A program for casino managers to use on their computers.
  - **Small Widget**: A small window that can be placed inside a bigger management program.
  - **Spectator Mode**: A view-only version for the public to watch games, like for a big tournament.