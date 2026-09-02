# 001: Agent/Request

> [!DEFINITION] [Request](./000_glossary.md)
> Think of a :term[Request]{canonical="Request"} as a single, complete mission you give to an AI. You give it a packet of information (the :term[context]{canonical="context"}) and a fill-in-the-blanks answer sheet (the :term[schema]{canonical="schema"}), and it gives you back the filled-out sheet (the :term[solution]{canonical="Solution"}).

> Sidenote:
>
> - Used in:
>   - :term[101: Concept/Idea]{href="./101_concept_idea.md"}
> - Learn more:
>   - [NPM: @idealic/agent](https://www.npmjs.com/package/@idealic/agent)

The :term[Request]{canonical="Request"} is the most basic building block of this AI system. It’s a way to talk to an AI that is organized, repeatable, and can be built upon. Instead of just a simple prompt, a :term[Request]{canonical="Request"} is a complete package that has everything the AI needs to do one job from start to finish. It’s the engine that powers everything else the AI can do.

## Context: A Mission Briefing of Messages

> [!DEFINITION] Context
> A carefully selected list of messages that gives the AI a complete briefing—with all the instructions, data, and conversation history it needs to do one specific job.

> Sidenote:
> The AI will use the :term[context]{canonical="context"} (mission briefing) to create a :term[solution]{canonical="Solution"} (filled-out form) that matches the :term[schema]{canonical="schema"} (blueprint).
>
> ```mermaid
> graph TD
>     subgraph User Input
>         direction LR
>         Context[\"Context (Briefing)"\]
>         Schema[\"Schema (Blueprint)"\]
>     end
>
>     Process{{"Request"}}
>
>     subgraph LLM Output
>         direction LR
>         Solution[/"Solution (Answer)"/]
>     end
>
>     Context --> Process
>     Schema --> Process
>     Process --> Solution
>     Schema -.-> Solution
>
>     linkStyle 2 stroke-width:2px,fill:none,stroke:gray,stroke-dasharray: 5 5;
>     linkStyle 3 stroke-width:2px,fill:none,stroke:gray,stroke-dasharray: 5 5;
> ```

The foundation of any :term[Request]{canonical="Request"} is its :term[context]{canonical="context"}, which is just a list of `Message` objects. Each `Message` is very simple: it has a `role` (like who is talking: the `"system"`, the `"user"`, or the `"assistant"`) and `content` (what they said). This lets us show the AI a whole conversation as part of its mission briefing.

Unlike a normal chatbot that just keeps adding new messages to a long, messy conversation, the :term[context]{canonical="context"} for each :term[Request]{canonical="Request"} is a clean, sealed package. It's built specifically for one task and isn't cluttered with old, irrelevant chats. The AI's answers aren't automatically added back in; we build a fresh :term[context]{canonical="context"} for every new job. This makes sure every mission is repeatable and that the AI has exactly the right information, without forgetting important details or getting confused.

This carefully managed :term[context]{canonical="context"} is how we give the AI its instructions and data. Everything it needs to know, except for the format of the final answer, is packed into these messages.

A simple list of messages might look like this:

```json
[
  { "role": "system", "content": "You are a helpful assistant." },
  { "role": "user", "content": "What is the capital of France?" }
]
```

The system makes this even more powerful by allowing the `content` of a message to be more than just text. It can also hold special packages of information called **custom content types**. For example, a message's content could be a structured package like `{ "type": "input", "input": { ... } }`.

> Sidenote:
> Special message types are explained in other documents:
>
> - :term[005: Agent/Data]{href="./005_agent_data.md"} - Gives data and its meaning to the AI.
> - :term[006: Agent/Input]{href="./006_agent_input.md"} - A structured prompt for the AI to use.
> - :term[009: Agent/State]{href="./009_agent_state.md"} - A way to remember things between steps.
> - :term[012: Agent/Plan]{href="./012_agent_plan.md"} - A pre-made plan for a multi-step job.
> - :term[016: Agent/Meta]{href="./016_agent_meta.md"} - Tells the AI about the Idea's identity.

This ability to send special packages makes the :term[context]{canonical="context"} the main place where the system can be expanded and taught new tricks.

Each custom content type comes with its own mini-program, or handler, that knows what to do with it. These handlers work together like an assembly line. Before the :term[Request]{canonical="Request"} is sent to the AI, this assembly line processes every message in the :term[context]{canonical="context"}. A handler can change the mission parameters on the fly:

- **AI Settings**: It can change things like which AI model to use or how creative it should be.
- **:term[Schema]{canonical="schema"}**: It can change the final answer sheet the AI needs to fill out.
- **:term[Context]{canonical="context"}**: It can change the list of messages sent to the AI, for example, by turning a special package into plain text or adding new instructions.

This powerful assembly line allows the AI to work with complex ideas, automatically building the perfect instructions it needs for any task.

## Schema: The Blueprint for the Answer

> [!DEFINITION] Schema
> A blueprint (specifically, a JSON Schema) that tells the AI the exact structure of the answer it needs to give. It’s like a fill-in-the-blanks form that the AI must follow, making sure the final answer is always perfectly organized.

> Sidenote:
>
> - Learn more about the blueprint format at [json-schema.org](https://json-schema.org/)

The :term[schema]{canonical="schema"} is a blueprint that defines the exact structure of the perfect answer, or :term[solution]{canonical="Solution"}. This is a very powerful system that can describe any kind of data, from a single word to a really complicated structure with many nested parts. The AI is required to create a :term[solution]{canonical="Solution"} that perfectly matches this blueprint, which guarantees that the output is always neat, organized, and predictable.

As these blueprints get more detailed, they can guide not just the final answer, but also the AI's thinking process. For example, a :term[schema]{canonical="schema"} can have a space for the final answer, but also have another space that asks the AI to first explain its reasoning or how confident it is. This turns the blueprint into a tool for shaping how the AI thinks.

A key idea here is that we can build complex blueprints by snapping together simpler, reusable pieces. This lets us teach the AI new skills in a way that is organized and easy to expand.

## Getting the Solution

> [!DEFINITION] Solution
> The final, perfectly organized answer (in a format called JSON) returned by the AI. It always follows the blueprint provided by the :term[schema]{canonical="schema"} and is the end result of a :term[Request]{canonical="Request"}.

After the mission briefing (the :term[context]{canonical="context"}) is prepared, the final list of messages and the blueprint (the :term[schema]{canonical="schema"}) are sent to the AI in one go. The AI's response is the :term[solution]{canonical="Solution"}—a perfectly structured document that matches the blueprint exactly.

You can think of this process as the AI telling a short story. Since an AI works by predicting the very next word, it fills out the :term[solution]{canonical="Solution"} from top to bottom, following the structure of the :term[schema]{canonical="schema"}. The way we design and order the fields in the blueprint directly changes the story the AI tells.

For example, if a blueprint first asks for a field called `"my_thinking_process"` before asking for the final `"answer"`, the AI is forced to first write down its thoughts before giving the final answer. Its own written thoughts then become part of the story, helping it generate a better final answer. This is a powerful way to guide the AI's thinking and have more control over the result by shaping the path it takes to get there.

> [!HEADSUP] Heads up
> This whole package—the :term[context]{canonical="context"}, the :term[schema]{canonical="schema"}, and the final :term[solution]{canonical="Solution"}—forms a single, repeatable unit. When you save this unit, it becomes what the system calls an :term[101: Concept/Idea]{href="./101_concept_idea.md"}.

## From Answers to Actions

A :term[Request]{canonical="Request"} is a great way to get a single, well-structured answer from an AI. But to build truly smart agents, we need more than just answers. We need a way to give the AI a menu of different tools or actions it can choose from to solve a problem. This means we need a system for defining these actions so the AI can pick the right one for the job.

> Sidenote:
>
> - :term[002: Agent/Tool]{href="./002_agent_tool.md"}

The next document, :term[002: Agent/Tool]{href="./002_agent_tool.md"}, explains how we create this menu of capabilities.
