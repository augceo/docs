## Conversation-to-Insight Log — Detailed Markdown Prompt

### 0 · Scope

Transform raw transcripts (chat, call, meeting) into a **single markdown file** that preserves ideas, explanations, reactions, promises, actionables, and open threads in a form that can be mined later. **All content must be translated to English**, including idioms and expressions from other languages.

### 1 · Explicit Goals

1. **Idea Lifecycle** – record when each idea first appears, how it evolves (including explicit changes), and when it resurfaces in later talks.
2. **Explanation Capture** – store metaphors, analogies, and framing stories used to clarify ideas.
3. **Reaction Logging** – keep who responded, how, and any direct quotes showing support, skepticism, or questions.
4. **Promise Ledger** – list every commitment, deadline, or "I'll send you X" statement.
5. **Actionables** – enumerate concrete next steps with owners, even when no deadline is given.
6. **Open Threads** – flag unresolved questions, hypotheses, or follow-ups.
7. **Mining-Friendly Structure** – use consistent handles, bullets, and nesting so later scripts can parse the file reliably.

### 2 · Formatting Rules

- Markdown only.
- **Translation required**: All content in English, including proper translation of idioms and colloquialisms.
- Bullet levels: `-` for level 1, two leading spaces for level 2, four for level 3.
- **Headline syntax**
  `- [Exploration] Headline sentence #handle – Speaker`
  Use **\[Exploration]** only when the transcript clearly shows novelty with phrases like "What if we…", "I'm just thinking…", etc., or when prior logs contain no mention of the concept.
- **Tags**: Use additional tags to mark content type:
  - `[Suggestion]` for proposed solutions or recommendations
  - `[Idea]` for conceptual thoughts or brainstorming
  - `[Question]` for unresolved queries
- Sub-bullets under each headline (omit if empty):

  1. **Evolution** — additions, clarifications, or **Changed →** notes.
  2. **Explanation** — metaphors, stories, analogies used to teach the idea.
  3. **Reactions** — key feedback (quote or paraphrase).
  4. **Data/Links** — inline URLs or hard facts.

- Maximum nesting depth = 3.
- After the Idea Log, add three sections with H2 headings: **Promises**, **Actionables**, **Open Threads**.
- Keep links inline and human-readable.
- Write plainly; avoid corporate fluff or moralising.

### 3 · Document Skeleton

```markdown
# <Title>

**Date:** <YYYY-MM-DD>
**Participants:** <names>
**Setting:** <friendly brainstorm / formal review / etc.>

## Idea Log

- [Exploration] [Idea] Headline sentence #handle – Speaker

  - Evolution — …
  - Explanation — …
  - Reactions — …
  - Data/Links — …

- [Suggestion] Another headline #different-handle – Speaker

  - Evolution — …
  - Explanation — …
  - Reactions — …
  - Data/Links — …

  … repeat for each idea …

## Promises

- …

## Actionables

- …

## Open Threads

- …
```
