# 301: Storing an Idea

The **Storage System** is like the brain's long-term memory for our world of Ideas. Its main job is to save every `Idea` in a safe place so it can be remembered and used later. Think of it as a special machine that takes an `Idea`, gives it a permanent home, and keeps a record so other parts of the system, like the [Resolver](./302_ideator_resolver.md) (the part that finds Ideas), can look it up.

## 2. Main Idea: Storage as a Transformation Machine

The Storage system follows a simple pattern: it takes an `Idea` in and gives an `Idea` back out.

- **What it takes in (Input)**: A complete `Idea`, which has three parts: its blueprint (`schema`), its answer (`solution`), and its background story (`context`).
- **What it does (Process)**: When the system gets an `Idea`, it gives it a unique ID number (like a library book's code) and saves it permanently in a safe, digital bookshelf (like a database).
- **What it gives back (Output)**: It hands back the same `Idea` you gave it, but now with a little note attached—the new ID and version number. This is your receipt, proving the `Idea` is safely stored.

## 3. Storage as a Hand-off Point

A very important job of the Storage system is to be a clear **hand-off point**, like a relay racer passing the baton. When someone (a person or another computer program) gives an `Idea` to Storage and gets the “saved” confirmation back, that first step is officially done.

This makes the whole system strong and able to grow. It’s like an assembly line. Once an `Idea` is saved, other helpers (called `Watchers`) can see that a new `Idea` has arrived and start their own separate jobs with it. This prevents one single task from getting too long and complicated. It creates a clean, traceable chain of events where one step finishes cleanly before the next one begins.

## 4. Key Features

### 4.1. A “Black Box” for Saving

The Storage system is like a magic box. You put your `Idea` in, and you don’t need to know *how* it stores it inside—whether it uses magic crystals or tiny robots. All you need to know is that once you give it an `Idea`, it promises to keep it safe and findable. The details of how it works are managed for you.

### 4.2. Versions and Never Changing the Past

The system believes that the past should never be changed. When you want to “update” an `Idea`, the system doesn't erase the old one. Instead, it saves a brand-new version and links it to the old one, creating a complete history. This means you can always go back and see exactly what an `Idea` looked like at any point in time. It's like saving a new draft of a drawing instead of erasing your old one.

### 4.3. Every Idea Has a Unique Address

Every `Idea` that is saved gets its own unique address, made up of its name and version number. This is like every house on a street having its own mailing address. It means any program can ask for a specific `Idea` with total confidence it will get the right one.

## 5. How You Talk to It

The Storage system is designed to be very simple to talk to. Its main job is to listen and save things.

- **Main Job**: The only thing you can do is give it an `Idea` to save.

- **It Doesn't Fetch Things**: To keep things simple and organized, the Storage system doesn't handle requests to *find* or *get* Ideas. That's a different job for a different system, called the **Resolver**. This way, each part of the system has one clear job: Storage saves things, and the Resolver finds things.