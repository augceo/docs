# 018: Agent/Question

> [!DEFINITION] [Question](./000_glossary.md)
> An unsettled matter put to another party as a :term[Tool Call]{canonical="Tool Call"}, carrying the answers worth considering and a place to say something the answers do not cover. Answers accumulate at one path, so a position can change and the reasoning survives with it.

> Sidenote:
>
> - Requires:
>   - :term[002: Agent/Tool]{href="./002_agent_tool.md"}
>   - :term[004: Agent/Call]{href="./004_agent_call.md"}
>   - :term[008: Agent/Output]{href="./008_agent_output.md"}
> - Enhances:
>   - :term[010: Agent/Loop]{href="./010_agent_loop.md"}
>   - :term[014: Agent/Delegate]{href="./014_agent_delegate.md"}

An agent that cannot ask is an agent that must guess. The **Question Protocol** gives it a way to put a matter to someone else — a person, or another agent — without pretending the matter is closed.

It invents nothing to do it. Asking is a :term[Tool Call]{canonical="Tool Call"}, answering is a :term[Tool Call]{canonical="Tool Call"}, and the record is an accumulating write at a :term[Variable]{canonical="Variable"} path. Everything below is those three, arranged.

## Asking Is an Act, So Asking Is a Call

A question has an addressee and an effect in the world. Someone is shown something and is expected to come back. That is what a :term[Tool Call]{canonical="Tool Call"} is: an action dispatched outward whose result lands in the context at the path the call named.

A field on the response that the model may fill when it feels moved to describes something else entirely. A field is a place to record a thought. Nothing is dispatched, nobody is addressed, and whether the field is filled at all depends on a provider's willingness to leave a property optional. A matter put to a human being is not a thought the model had.

So a question is a call:

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

The `ask` :term[Activity]{canonical="Activity"} is the addressee. A host implements it as a prompt on a screen, a message to a person, or a hand-off to another agent; what ships is the tool and never the delivery, because who is being asked is not something a protocol can know. Whatever the activity returns — `{ "title": "net60" }`, with a `comment` if the answering party had something to add — the :term[Execution Loop]{canonical="Execution Loop"} files at `†question.settlementTerms`, because that is what a call's :term[Output Path]{canonical="Output Path"} already means.

Nothing in that call is new. The tool is an ordinary tool, the activity is an ordinary activity, `_outputPath` and `_outputMethod` are act 008's, and `†question.settlementTerms` is act 007's.

## The Closed Menu

A question with a fixed set of answers is composed by one side and closed by the other. Whoever writes it decides the whole space of admissible replies, so a party whose actual position was never listed must pick the nearest wrong one. A menu always returns something, and the wrong answer arrives looking like agreement.

The second cost is timing. A menu is answered once, at the moment least likely to be informed — a choice made before a :term[Tool]{canonical="Tool"} has run is made without its result.

Neither is fixed by writing better options. A question has to admit what its author did not anticipate, and it has to stay answerable after the answer.

## The Comment

`comment` is optional and belongs to the answer. It is what separates a question from a menu: the answering party may choose **and** speak, so a position the author failed to anticipate has somewhere to go instead of being rounded to the nearest listed answer.

It is also where a reason can live. An answer records what was decided; a comment records why, at the moment it was still known. Most answers will not need one — which is the point of leaving it out rather than demanding it.

## Being Asked: the Matter Becomes a Tool

The other side of the exchange is a matter that arrives from outside and stands in your context until you take a position on it. That is a **Question Message**:

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

A Question Message declares one tool into the room and does nothing else. The tool is named for the matter, its description is the matter, its answers are an `enum`, and its :term[Output Path]{canonical="Output Path"} is fixed to the matter's own path. Answering is calling it:

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

`_outputPath` and `_outputMethod` are constants on the schema rather than choices, so an answer cannot be misfiled and cannot silently overwrite the position it supersedes.

Each answer is an object carrying its own `title` and `description`, so what choosing it means travels beside it — an answer whose consequence is unstated can be picked but not weighed. The `description` reaches the answering party as the tool's own prose; the titles reach it as the set it may choose from.

## Answering Is Latent, Asking Is Explicit

The two directions differ in exactly one respect, and it is a distinction the tool protocol already draws.

A tool with an :term[Activity]{canonical="Activity"} is **explicit**: code computes the result. A tool without one is **latent**: the model computes the result with its own reasoning, in the same breath as the call, and writes it into `_output`.

Asking hands a matter to someone else, so someone else produces the value — explicit. Answering is stating your own position, and nobody but you can produce it — latent. There is no activity to dispatch, no round trip, and nothing to register. The direction of the question decides the execution mode, and neither direction needed a mechanism invented for it.

> Sidenote:
>
> - :term[002: Agent/Tool]{href="./002_agent_tool.md"} — latent and explicit execution.
> - :term[104: Concept/Latent]{href="./104_concept_latent.md"}

## Identity

The `title` names the matter, and every answer attaches to it — the same way an answer's own `title` names it. There is no separate identifier: a title is what a thing is called and what it is addressed by.

Without one, a second answer is a second matter, and the record becomes a heap of opinions rather than the history of one decision. With one, a change of mind is legible as a change of mind — same question, later answer, earlier position intact. It also lets a question outlive the turn that asked it: an agent may ask, carry on with work that does not depend on the answer, and be answered several turns later.

Naming the tool after the matter buys one more thing. A tool declared twice under one name is a tool declared once — the later declaration replaces the earlier. So a matter raised again under a title it has already used refreshes its own tool rather than standing up a second, rival copy of it. Two rival copies would each carry their own answer set while sharing one path, and an answer naming that title would satisfy whichever one was reached first.

## History Accrues

A question is a :term[Variable]{canonical="Variable"} at a path, and every answer is a write to it.

Because a write declares its :term[Output Method]{canonical="Output Method"}, an answer that supersedes an earlier one appends rather than overwrites. The resolver replays oldest-first, so the standing position is the latest entry and the route to it is the sequence. The history is a consequence of how writes already work — not a second structure kept beside them.

So a party reading a question already answered sees what was settled, by whom, and against what comment. A downstream :term[Tool Call]{canonical="Tool Call"} reading `†question.settlementTerms` receives the whole exchange rather than the last word:

```json
{
  "_tool": "writeMemo",
  "_activity": "writeMemo",
  "exchange": "†question.settlementTerms",
  "_outputPath": "†state.memo"
}
```

## Either Party

Nothing requires a human on the other side. A :term[Delegate]{canonical="Delegate"} may be asked, and may answer with a comment its parent had no way to anticipate — which is how a subordinate surfaces a constraint before the work rather than after it.

A delegate may also decline the matter it was given and put one back. An :term[Activity]{canonical="Activity"} whose output is already a Message enters its caller's context verbatim, so a delegate returning a Question Message is a subordinate raising a matter with the party that consulted it. From there it is an ordinary standing question: it declares its tool into the caller's room, and the caller answers it or does not.

A delegate's record stays its own. Nothing joins a sub-run's `†question` to its caller's, because a clean room keeps its own paths.

> Sidenote:
>
> - :term[014: Agent/Delegate]{href="./014_agent_delegate.md"}
> - :term[015: Agent/Scopes]{href="./015_agent_scopes.md"} — a scope naming `question` carries the matter and every position taken on it into the room together.

## Boundaries

Every question in one exchange shares a shape, so an answer looks the same across them even as the content varies. A question needing its own attribute set is outside what this offers.

Asking does not gate the :term[Loop]{canonical="Loop"}. A standing question is a tool that may be called, not a demand that must be met, so work that does not depend on an answer proceeds while the question stands open. A party that must have an answer first says so as a dependency: a :term[Tool Call]{canonical="Tool Call"} reading `†question.<title>` cannot run until something is written there.

That dependency is the only thing that should gate on an answer, and it is worth saying why the alternative was rejected. A response field the model may leave empty is optional or mandatory according to whether a provider tolerates optional properties — so the same protocol pressed for an answer on one provider and never mentioned it on another. Reaching for an answer through the call queue takes that choice away from the provider: a room with a tool in it has calls to make, on every route, and which calls to make is a question the act answers rather than a feature flag.

## Synergies

- **:term[Tool]{canonical="Tool"}:** A question is a tool and nothing besides. Asking is an explicit call, answering is a latent one, and both are dispatched, queued and recorded by machinery that predates this act.

  > Sidenote:
  >
  > - :term[002: Agent/Tool]{href="./002_agent_tool.md"}

- **:term[Output Method]{canonical="Output Method"}:** Accumulating writes are what let an answer change without erasing the one before it.

  > Sidenote:
  >
  > - :term[008: Agent/Output]{href="./008_agent_output.md"}

- **:term[Plan]{canonical="Plan"}:** A turn's calls become the next turn's plan, so an answer given is visible afterwards as an act taken and not only as a value at a path.

  > Sidenote:
  >
  > - :term[012: Agent/Plan]{href="./012_agent_plan.md"}

- **:term[Delegate]{canonical="Delegate"}:** A question gives delegation a return path — the subordinate answers rather than only reporting.

  > Sidenote:
  >
  > - :term[014: Agent/Delegate]{href="./014_agent_delegate.md"}

- **:term[Advisor]{canonical="Advisor"}:** An advisor is a reasoning surface: an opinion produced alongside the answer, addressed to nobody, which is why it stays a property of the response. A question is addressed, dispatched and answered, which is why it is a call. One is how an agent thinks; the other is something an agent does.

  > Sidenote:
  >
  > - :term[017: Agent/Advisor]{href="./017_agent_advisor.md"}
