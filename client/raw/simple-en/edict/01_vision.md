# Part I: The Vision

---

_The big idea here is a major rethink of how the internet works. Right now, a few big companies control everything. This vision is about building a new digital world where power is handed back to individuals and communities, letting everyone be in charge of their own digital lives._

---

### (Chapter 1) The Core Idea & Principles

The whole system is built on a few simple but strong rules. These rules make sure everything works together smoothly.

- **Everything is an Idea.** At the very center of this system, everything you create or use—an app, a tool, a document, a plan—is called an “Idea.” This keeps things simple and organized, because there’s only one type of building block for everything.

  > Sidenote:
  > [RFC 101: Concept/Idea](../acts/101_concept_idea.md)
  >

- **Self-Contained and Self-Describing.** Every Idea comes with its own instruction manual built right in. This means any person, program, or AI can look at an Idea and immediately understand what it is and how to use it, without needing any prior information.

  > Sidenote:
  > [RFC 001: Agent/Request](../acts/001_agent_request.md)
  >

- **Any Idea Can Have a Home.** Any Idea can be given its own website address (a domain). This gives it a permanent place on the internet and a basic webpage to interact with. If it's a common type of Idea (like an article), it gets a nice, custom-fit page. If it’s something more complex (an Ideator), it gets a flexible, do-anything kind of page.

  > Sidenote:
  > [RFC 102: Concept/Sovereignty](../acts/102_concept_sovereignty.md)
  >

- **Ideators are Ideas with Input.** An “Ideator” is just an Idea that’s designed to take instructions from you. Think of it like a simple machine or a tool. While a basic Idea might just be information (like a recipe), an Ideator is a tool that uses that information to *do* something (like a mixer that takes flour and eggs and turns it into batter).

  > Sidenote:
  > - [RFC 103: Concept/Ideator](../acts/103_concept_ideator.md)
  > - [RFC 005: Agent/Input](../acts/005_agent_input.md)

- **Composable by Design.** This system is made for connecting things together. Some Ideators can take one Idea and change it into another. This means you can line them up in a chain to create powerful automated jobs, where the output of one tool becomes the input for the next.

- **The AI is the Universal Interpreter.** We use Artificial Intelligence (specifically, a large language model or LLM) as a smart helper. Because every Idea explains itself, the AI can read its “instruction manual” and figure out how to run it. This makes the system incredibly flexible and able to handle new kinds of Ideas without needing to be reprogrammed.

  > Sidenote:
  > [RFC 104: Concept/Latent](../acts/104_concept_latent_.md)
  >

- **A Protocol of Living Ideas.** We're not just creating a way to share files that sit there and do nothing. We're building a system for sharing living, functional things that can grow, change, and connect with each other over time.

---

### (Chapter 2) The Diagnosis — The Gilded Cage of Big Tech

The main problem with today's internet is that big tech companies have offered us a deal that seems great at first but is actually a trap. At first, they gave us social networks that promised we could reach millions of people. This pulled us into their private playgrounds, where they collected our data and limited our choices. Now, they're offering us powerful AI tools that are too good to pass up.

But this convenience has a hidden price. The more we rely on their platforms, the more we depend on them. We end up trapped in a beautiful cage where we don't own our own digital lives anymore; the platform does.

> Sidenote:
> This is a typical centralized network. The Platform in the middle owns all the data and controls everyone connected to it.
>
> ```mermaid
> graph LR
>     P((Platform))
>     A((" ")); B((" ")); C((" "));
>     E((" ")); F((" ")); G((" ")); H((" ")); I((" "));
>
>     A --> P; B --> P; C --> P;
>     P --> E; P --> F; P  --> G; P --> H; P --> I
> ```

---

### (Chapter 3) The Solution — A Decentralized Renaissance

The answer isn't to build a slightly better cage. It's to leave the cages behind and create a new, open world. We're going back to the original spirit of the internet, where everything was open and decentralized, but we're updating it for the age of AI.

This new world is built on two big pillars:

- **A Vision of Autonomy:** We want to move from platforms to a protocol. A protocol is like a shared language that everyone agrees to use, instead of a private playground owned by one company. In this system, you use **AI models that run on your own computer** and connect directly with others through **peer-to-peer networks**. This means your data and digital tools stay completely in your control. The goal is to create an operating system for ideas, where you can share interactive, intelligent tools—like a budget that manages itself, or a project plan that automatically updates—as easily as you share a text document today.

> Sidenote:
> In a decentralized network, everyone (peers and services) is equal and connects directly to each other without needing a middleman.
>
> ```mermaid
> graph TD
>         A((" ")) <--> B((" "))
>         A <--> C((" "))
>         A <--> D{{"Service"}}
>         B <--> C
>         B <--> E((" "))
>         C <---> F{{"Service"}}
>         D <--> E
>         D <--> F
>         E <--> F
>         F <--> G((" "))
> ```

- **A Philosophy of Connection:** This new way of building things also changes how we connect with each other. Today, “sharing” online often feels like shouting into a crowd for attention. We want to bring sharing back to what it should be: a **thoughtful, deliberate act of giving** something to people you trust. It's about moving away from a flood of global content and toward meaningful conversations in small, trusted groups. This lets people create their own “hidden worlds” online, where they have total control over who sees their ideas and how they are used.
