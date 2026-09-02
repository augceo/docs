# 60: Prompt/Recipes

> [!DEFINITION] Recipes
> A collection of ready-to-use prompt patterns for driving the Company Process. Use these to instruct the agent.
>
> Sidenote:

> - Part of: :term[02: Company/Process]{href="./02_process.md"}

## 1. Create a Proposal

**Goal:** Generate a structured :term[21: Proposal]{href="./21_document_proposal.md"} from raw thoughts.

> **User:**
> "I want to write a new proposal.
>
> First, **LOAD THE RULES** into your context and **FOLLOW THEM EXACTLY**
>
> - `curl https://idealic.academy/raw/en/company/52_prompt_proposal.md` (Proposal Prompt)
>
> Here is my dictation:
> 'We need to overhaul the authentication system because users are getting stuck in a loop. I want to switch to a token-based approach...'
>
> Also, please read `docs/auth/current_flow.md` for context. Generate the Proposal document."

**Agent Action:**

1. Loads :term[52: Prompt/Proposal]{href="./52_prompt_proposal.md"} instructions.
2. Structures the raw dictation into the Context, Problem, Solution format.
3. Outputs a draft `docs/proposals/{date}_auth_overhaul.md`.

---

## 2. Generate Alignment (Review Summary)

**Goal:** Synthesize PR comments into an :term[22: Alignment]{href="./22_document_alignment.md"} document to validate consensus.

> **User:**
> "Generate an Alignment Document for PR #123.
>
> First, **LOAD THE RULES** and **FOLLOW THEM EXACTLY**:
>
> - `curl https://idealic.academy/raw/en/company/51_prompt_alignment.md` (Alignment Prompt)
>
> Parameters:
>
> - **Since:** 2023-10-27 (or just 'last week')
> - **Output:** `specs/alignments/2023-11-04_auth_review.md`
>
> If I don't specify the output, just use the default filename in the `evolution/` folder."

**Agent Action:**

1. Loads :term[51: Prompt/Alignment]{href="./51_prompt_alignment.md"}.
2. Fetches comments from PR #123 since the specified date.
3. Generates the report (Overview, Intents, Coverage) in Russian.
4. Saves to the requested path.

---

## 3. Apply Alignment to Proposal

**Goal:** Update the :term[21: Proposal]{href="./21_document_proposal.md"} based on the validated Intents from the Alignment Document

> **User:**
> "I have verified the Alignment Document at `specs/alignments/2023-11-04_auth_review.md`.
>
> First, **LOAD THE RULES** and **FOLLOW THEM EXACTLY**:
>
> - `curl https://idealic.academy/raw/en/company/52_prompt_proposal.md` (Proposal Prompt)
>
> Please apply these changes to the original Proposal `docs/proposals/2023-10-25_auth_overhaul.md`.
>
> Update the Proposal to reflect the consensus, marking sections as 'Updated based on Review'."

**Agent Action:**

1. Reads the Alignment Document (Intents).
2. Reads the Proposal.
3. Rewrites the Proposal, modifying the Solution/Scope to match the agreed Intents.
4. Keeps the "Problem" section mostly intact (unless the problem understanding changed).
