# 60: Recipes for Your AI Assistant

> [!DEFINITION] Recipes
> A collection of ready-to-use command patterns for guiding the company's work. Think of them like cooking recipes, but for telling an AI assistant how to complete specific tasks.


> - Part of: :term[02: Company/Process]{href="./02_process.md"}

## 1. Create a Plan (Proposal)

**Goal:** Turn a bunch of scattered thoughts into a clear, organized plan, called a :term[21: Proposal]{href="./21_document_proposal.md"}.

> **User:**
> Here's how you'd ask the AI assistant:
> 
> "I need to make a new proposal.
>
> First, **DOWNLOAD THE INSTRUCTION MANUAL** and **FOLLOW IT PERFECTLY**.
>
> - `curl https://idealic.academy/raw/en/company/52_prompt_proposal.md` (This link is the instruction manual for making a Proposal)
>
> Now, here are my thoughts:
> 'We need to fix the way people log in because they keep getting stuck. I think we should use a newer, token-based system...'
>
> Also, please read the file at `docs/auth/current_flow.md` to understand the background story. Then, create the Proposal document for me."

**What the Assistant Does:**

1. It downloads and reads the instructions from the :term[52: Prompt/Proposal]{href="./52_prompt_proposal.md"} link.
2. It takes your spoken ideas and organizes them into sections like: **Background**, **Problem**, and **Solution**.
3. It creates a new draft file and saves it, like this: `docs/proposals/{date}_auth_overhaul.md`.

---

## 2. Create a Team Agreement Summary (Alignment)

**Goal:** Read all the comments and feedback on a project and combine them into a single report, called an :term[22: Alignment]{href="./22_document_alignment.md"} document. This proves everyone is on the same page.

> **User:**
> "Create an Alignment Document for project #123.
>
> First, **DOWNLOAD THE INSTRUCTION MANUAL** and **FOLLOW IT PERFECTLY**:
>
> - `curl https://idealic.academy/raw/en/company/51_prompt_alignment.md` (This is the manual for making an Alignment report)
>
> Here are the details:
>
> - **Since:** 2023-10-27 (meaning, only look at comments from this date forward)
> - **Output:** `specs/alignments/2023-11-04_auth_review.md` (where to save the report)
>
> If I don't tell you where to save it, just use a default name and folder."

**What the Assistant Does:**

1. It reads the :term[51: Prompt/Alignment]{href="./51_prompt_alignment.md"} instruction manual.
2. It finds all the comments for project #123 that were posted after the date you gave.
3. It writes a summary report with sections like **Overview**, **Decisions Made**, and **Topics Covered**.
4. It saves the report to the file path you asked for.

---

## 3. Update a Plan with Team Feedback

**Goal:** Use the team's agreed-upon ideas from the Alignment document to update the original :term[21: Proposal]{href="./21_document_proposal.md"}.

> **User:**
> "I've checked the Alignment Document at `specs/alignments/2023-11-04_auth_review.md` and it's correct.
>
> First, **DOWNLOAD THE INSTRUCTION MANUAL** for proposals and **FOLLOW IT PERFECTLY**:
>
> - `curl https://idealic.academy/raw/en/company/52_prompt_proposal.md` (Proposal Prompt)
>
> Now, please apply the agreements from that report to the original Proposal, which is located at `docs/proposals/2023-10-25_auth_overhaul.md`.
>
> Make sure to update the Proposal and add a note to the changed parts that says 'Updated based on Review'."

**What the Assistant Does:**

1. It reads the Alignment Document to understand what the team agreed on.
2. It reads the original Proposal.
3. It rewrites parts of the Proposal—mainly the **Solution**—to match the team's decisions.
4. It usually doesn't change the **Problem** section, since the problem itself is likely the same.
