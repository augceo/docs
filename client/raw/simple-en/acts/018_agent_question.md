# 018: Agent/Question

> [!DEFINITION] [Question](./000_glossary.md)
> A way to ask someone else for input using a :term[Tool Call]{canonical="Tool Call"}. It provides a list of possible answers, plus a space to write something new if the options do not quite fit. Answers pile up in a single history folder, so people can change their minds later while keeping a record of why.

> Sidenote:
> - Requires:
>   - :term[002: Agent/Tool]{href="./002_agent_tool.md"}
>   - :term[004: Agent/Call]{href="./004_agent_call.md"}
>   - :term[008: Agent/Output]{href="./008_agent_output.md"}
> - Enhances:
>   - :term[010: Agent/Loop]{href="./010_agent_loop.md"}
>   - :term[014: Agent/Delegate]{href="./014_agent_delegate.md"}

If an agent cannot ask a question, it is forced to guess. The **Question Protocol** gives it a way to stop guessing and actually ask a person or another agent for help, leaving the conversation open until a decision is made.

It does not invent any new magic to pull this off. Asking a question is just using a :term[Tool Call]{canonical="Tool Call"}. Answering is also a :term[Tool Call]{canonical="Tool Call"}. The memory of that conversation is entirely handled by saving it to a standard :term[Variable]{canonical="Variable"}. Everything below is just those three fundamental things working together.

## Asking Is an Act, So Asking Is a Call

When you ask a question, you are doing something out in the world. You send a message out and expect someone to get back to you. This is exactly what a :term[Tool Call]{canonical="Tool Call"} does: it sends an action outward and waits for the result to come back to a specific folder.

Sometimes AI models are given an empty text box to just think out loud. That is completely different. Thinking out loud is just talking to yourself. Asking a question is handing a direct task directly to someone else.

Because of that, asking a question looks exactly like this in code:

```json
{
  "_why": "the vendor sets the terms, and guessing them would set them wrong",
  "_tool": "ask",
  "_activity": "ask",
  "title": "settlementTerms",
  "description": "On what terms is the balance settled?",
  "answers": [
    { "title": "net30", "description": "Balance due thirty days after acceptance." },
    { "title": "net60", "description": "Balance due sixty days after acceptance." }
  ],
  "_outputPath": "†question.settlementTerms",
  "_outputMethod": "push"
}
```

The `ask` :term[Activity]{canonical="Activity"} is a lot like a mail delivery service. The system might show this as a pop-up on your screen or send it as a chat message to a teammate. We only instruct the system to write the letter (the tool) and do not worry about how it physically gets delivered. Whatever the reply ends up being—like `{ "title": "net60" }`, maybe with a little note attached—the :term[Execution Loop]{canonical="Execution Loop"} safely files it away at `†question.settlementTerms`. It files it there because that is the exact address the tool set up beforehand.

Nothing about this is new. The tool is an ordinary tool, the activity is a normal activity, and the save paths are just the basic paths from acts 008 and 007.

## The Closed Menu

Imagine a strict multiple-choice test. Whoever writes the test gets to decide all the possible answers. If your true feeling is not on the list, you are forced to pick the closest wrong answer. Because you are boxed in, a wrong answer ends up looking like total agreement.

The second problem is timing. If you are forced to answer a question immediately before doing any research, you are making a blind guess.

Providing better multiple-choice options does not fix this. A good question needs to have space for answers the creator never imagined, and it needs to patiently wait for you to be ready to answer.

## The Comment

This is why we provide a `comment` box. It belongs to the person answering. The answering party does not just pick an option; they can pick an option **and** explain themselves. A completely unexpected thought has a safe place to go instead of being deleted.

It is also a perfect place for the "why." The chosen answer acts as the hard decision, while the comment acts as the reason behind it. Not every answer requires a grand explanation—which is exactly why the comment is strictly optional.

## Being Asked: the Matter Becomes a Tool

What happens when you are the one receiving the question? It shows up as a brand-new tool placed into your workspace. This is called a **Question Message**:

```ts
{
  type: 'question',
  question: {
    title: 'settlementTerms',
    description: 'On what terms is the balance settled?',
    answers: [
      { title: 'net30', description: 'Balance due thirty days after acceptance.' },
      { title: 'net60', description: 'Balance due sixty days after acceptance.' },
    ],
  },
}
```

This message politely places exactly one temporary tool on your desk. The tool is named after the topic, the description is the question being asked, and the available choices are built into it. To submit your answer, you just use that new tool:

```json
{
  "_why": "sixty days is affordable and the discount is worth more than the float",
  "_tool": "answerSettlementTerms",
  "_outputPath": "†question.settlementTerms",
  "_outputMethod": "push",
  "_output": {
    "title": "net60",
    "comment": "Thirty days is affordable, but only if the discount goes with it."
  }
}
```

Things like where the file gets saved (`_outputPath`) are fixed automatically. You cannot accidentally file your answer in the wrong spot or quietly erase a previous answer.

Each choice in the list carries its own title and description. You get to read exactly what each option guarantees before you click it.

## Answering Is Latent, Asking Is Explicit

There is only one real difference between asking and answering.

When a tool physically sends a request out into the world (like our mail carrier), it is **explicit**. When a tool simply writes down a decision you are making in your own head right now, it is **latent**.

Asking hands a job to someone else—so it is explicit. Answering is stating your own personal opinion, and only you can do that—so it is latent. Providing your answer does not require a complex delivery route; you simply log what you have decided right there. Again, the system relies on the tools we already built.

> Sidenote:
> - :term[002: Agent/Tool]{href="./002_agent_tool.md"} — the difference between handling tasks mentally (latent) and sending them out (explicit).
> - :term[104: Concept/Latent]{href="./104_concept_latent.md"}

## Identity

Every question requires a `title` (a name). Every answer neatly attaches to that exact name.

Without a name, a second answer would look like a completely unrelated question. Your system history would become a messy pile of random opinions instead of the clear history of one decision. By keeping the name, an updated answer just looks like someone changing their mind. The original thought stays perfectly intact. This also allows an agent to ask a question, go do some other chores, and finally read the answer when it arrives much later.

Naming the tool after the topic buys one more benefit. If someone asks the exact same question twice, the system realizes they are the same and just updates the original entry. It does not spawn two confusing clones fighting for the same piece of information.

## History Accrues

A question is safely stored as a :term[Variable]{canonical="Variable"}, and every new reply just gets pinned to the bottom of the board.

Because a new reply tacks itself onto the end instead of hitting delete, your previous choices are preserved perfectly. The system always reads the oldest entries first, which means the very last entry is your current final decision, showing the whole journey of how you got there.

Anyone reading this board later sees exactly what was decided, who decided it, and the comments explaining the pivot. If a later :term[Tool Call]{canonical="Tool Call"} needs to read `†question.settlementTerms`, it downloads the entire conversation history instead of just the final choice:

```json
{
  "_tool": "writeMemo",
  "_activity": "writeMemo",
  "exchange": "†question.settlementTerms",
  "_outputPath": "†state.memo"
}
```

## Either Party

A human does not always have to be involved. An agent can ask a subordinate "helper" agent (a :term[Delegate]{canonical="Delegate"}). That helper can reply back with a comment warning about an unlisted roadblock—raising a concern before starting the work instead of failing silently after.

A helper agent can also refuse to answer and bounce the whole question completely back up to the boss. It simply passes a Question Message back up the chain verbatim. From there, the boss can either answer it personally or leave it alone.

Helpers always keep their own separate local files, so their messy scratchpads never accidentally overwrite their boss's pristine workspace.

> Sidenote:
> - :term[014: Agent/Delegate]{href="./014_agent_delegate.md"}
> - :term[015: Agent/Scopes]{href="./015_agent_scopes.md"} — making a localized safe zone called `question` packages up the whole matter and its history without making a mess.

## Boundaries

Every question we create shares this exact same basic shape. If you need a fully custom questionnaire with unique rules, this system is not built for that.

Asking a question also does not completely freeze the system. A question sits open, and the agent can easily continue doing other tasks that do not depend on the answer. If an agent absolutely must have an answer to proceed, they clearly state that dependency. A :term[Tool Call]{canonical="Tool Call"} looking for a reply will patiently block that specific task until someone finally writes an answer.

We made it a hard tool requirement instead of a soft suggestion. If we just asked an AI to hopefully fill out a text box if it felt like it, some engines would just ignore it entirely. By making the question a solid tool, we stripped that lazy option away.

## Synergies

- **:term[Tool]{canonical="Tool"}:** A question is literally just a standard tool. Asking sends it outward, answering processes it internally.

  > Sidenote:
  > - :term[002: Agent/Tool]{href="./002_agent_tool.md"}

- **:term[Output Method]{canonical="Output Method"}:** Pinning new replies to the bottom of the list lets you legally change your mind without destroying the history tape.

  > Sidenote:
  > - :term[008: Agent/Output]{href="./008_agent_output.md"}

- **:term[Plan]{canonical="Plan"}:** A submitted answer instantly becomes part of the next action plan. You can visually confirm an answer was given as an actual step.

  > Sidenote:
  > - :term[012: Agent/Plan]{href="./012_agent_plan.md"}

- **:term[Delegate]{canonical="Delegate"}:** Questions give smaller helper agents a strict way to communicate back with their boss instead of just performing chores silently.

  > Sidenote:
  > - :term[014: Agent/Delegate]{href="./014_agent_delegate.md"}

- **:term[Advisor]{canonical="Advisor"}:** An advisor acts as a private reasoning engine producing private thoughts. A question is for asking *someone else*. One is how the "brain" works privately; the other is how you communicate publicly.

  > Sidenote:
  > - :term[017: Agent/Advisor]{href="./017_agent_advisor.md"}
