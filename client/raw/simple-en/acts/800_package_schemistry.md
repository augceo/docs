# 800: Package/Schemistry

> [!DEFINITION] [Schemistry](./000_glossary.md)
> A tool that uses one master blueprint, called a **Schema**, to define how data should look and act. This single blueprint is used to automatically check data when the app is running, help programmers write code without mistakes, and give AIs clear instructions. It ensures that the way data is described during development and how it actually works in the live application are always perfectly in sync.

> Sidenote:
> - This makes the following possible:
>   - :term[001: Agent/Request]{href="./001_agent_request.md"}
>   - :term[101: Concept/Idea]{href="./101_concept_idea.md"}

**Schemistry** is the foundational toolkit that organizes all the data in our system. Its main job is to make the **Schema** the ultimate authority—the single set of rules that governs what data looks like and how it behaves for everyone, including developers and AIs.

## Philosophy & Core Concepts

### The Single Source of Truth

In many software projects, the way a programmer *thinks* data is structured (in the code) can be different from the rules the live app uses to *check* that data. This mismatch leads to confusion and bugs.

Schemistry fixes this by making the **Schema** the one and only source of truth. By defining data just once in the schema, we get everything we need automatically:

- **Runtime Validation:** Making sure the data isn't broken or incorrect when the app is actually running.
- **TypeScript Types:** Giving programmers a real-time guide in their code editor that helps them avoid mistakes.
- **LLM Integration:** Handing AIs a perfect instruction manual for the data they need to work with.
- **Application Logic:** Allowing the app to make smart decisions based on the data's design.

This approach is like having a single master recipe. Instead of the chef and the baker working from different notes, they both use the exact same recipe card. If the recipe changes, they both know instantly, so no mistakes are made.

### The "From Schema" Paradigm

A key feature of the library is a tool that acts like an automatic translator. It reads the data blueprint (the schema) and creates a matching guide for the programmer to use in their code.

- **Dynamic Type Inference:** When the blueprint changes, the code guide updates itself automatically.
- **Type Safety:** If a programmer tries to use data in a way that breaks the blueprint's rules, the system will show an error immediately, before the program is even run.

### The Schema Registry

To keep a big system from getting messy, Schemistry includes a registry, which is like a catalog for all your data blueprints.

- **Flat Structure:** Instead of creating one giant, complicated blueprint, you create small, reusable parts. The registry keeps track of all these parts.
- **Automatic Resolution:** When you need a specific part, you just ask for it by its ID, and the system finds and connects it for you. This is like building with a set of well-organized, labeled parts instead of a giant, tangled pile.

### Canonical Schema & Semantic Divergence

Sometimes, the blueprint needs to be described differently for the computer versus the human programmer, even though it means the same thing.

Imagine instructions for building a chair. For the robot in the factory (the computer), the instructions are very technical: "Combine Part A and Part B." For the designer (the programmer), the instructions might just be a picture of the finished chair leg. The technical details are different, but the *meaning*—a chair leg—is the same.

Schemistry allows for this difference because it makes life easier for the programmer while still being extremely strict with the computer at runtime.

### Performance & Caching

Translating the blueprint into a code guide takes computer brainpower. To avoid doing this work over and over, Schemistry is smart about saving its work.

- **Caching:** Once the system translates a blueprint, it saves the result.
- **Reusability:** The next time it needs that same translation, it just uses the saved copy instead of doing all the work again. This makes everything much faster.

## Evolution: From Composite to Strict

At first, the library treated all data blueprints as if they were in one big, generic box. This caused problems because the system couldn't tell the fine details apart—it was like trying to find a specific fish in a box labeled "Animals."

- **The "Composite" Trap:** This generic approach made it impossible for the code to understand specific rules, like "a fish has fins, but a lion has fur."
- **The Shift to Strictness:** We changed the system to use separate, clearly labeled boxes: "Objects," "Arrays," "Strings," etc. Now, when the code looks inside the "Object" box, it knows exactly what kind of rules to expect.
- **The Resolution:** We are getting rid of the big generic box completely. Every blueprint must now be specific and clear.

## Roadmap: Type-Safe Architecture

### Schema Manipulation

The library gives you tools to edit and combine your data blueprints, like digital scissors and glue. You can merge two blueprints, cut out parts you don't need, or pick only the parts you want.

Our goal is to make these tools perfectly safe, so that any change you make to a blueprint is instantly and correctly reflected in the programmer's code guide, with no chance of error.

### The `ToSchema` Strategy

To make this work, Schemistry uses a clever process that's like translating a document.

1.  **Extract (`FromSchema`):** First, we translate the technical blueprint from computer-language into a simple language that programmers understand (TypeScript types).
2.  **Operate:** Next, we make all the edits—combining, cutting, pasting—in this simpler language.
3.  **Reconstruct (`ToSchema`):** Finally, we translate the edited version back into the technical, computer-language blueprint.

```typescript
Schema.Intersection<A, B>(a: A, b: B): ToSchema<FromSchema<A> & FromSchema<B>>
```

**Why This Works:**
Because the translation is perfect in both directions, we can be confident that the final blueprint exactly matches the changes we made. This allows us to treat blueprints as simple building blocks that can be changed and combined with mathematical certainty.

### Goals: New Helpers & Extensibility

This translation strategy makes it much easier to build powerful new tools for working with data blueprints.

- **Intersection & Union:** Combining blueprints without losing any detail.
- **Pick & Omit:** Changing object blueprints while keeping the code guide perfectly accurate.

**Maintainability:**
By working in the simpler language, we can build better tools much faster and with fewer mistakes, because we don't have to worry about all the complex details of the computer-language version.

### Partial Schema Support

The official standard for data blueprints (JSON Schema) is flexible. For example, if you list a bunch of properties, it assumes you're describing an object. Our system, however, currently requires you to be very explicit and state `type: 'object'`.

- **Current Limitation:** Our translator can't yet understand these simpler, "partial" blueprints.
- **The Workaround:** For now, we require every blueprint to be fully and explicitly defined.
- **Internal State:** We've marked the places in our code where this feature could be added later.
- **The Blocker:** We need to make sure that teaching our translator to understand partial blueprints won't make it too slow or complicated.
- **Future Goal:** We plan to add support for these simpler blueprints if we can do it without hurting performance.

## Outro

By creating a single, shared language that both people (developers) and AIs can understand, Schemistry builds the strong foundation our system needs.

Now that we know how our data is organized, we can explore how it's used to communicate with AIs. :term[001: Agent/Request]{href="./001_agent_request.md"} will show how these blueprints are used to define our conversations with them.
