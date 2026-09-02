# 21: Company/Proposal

> [!DEFINITION] Proposal Document
> A transient proposal artifact used to outline a desire for change, improvement, or expansion before execution begins.
>
> Sidenote:
>
> - Part of: :term[02: Company/Process]{href="./02_process.md"}
> - Followed by: :term[22: Company/Alignment]{href="./22_document_alignment.md"}
> - Formerly known as: Intent Document, Evolution Document

## 1. The Core Philosophy

The Proposal Document is essentially an **RFC (Request for Comments)**. It is written **once** per initiative to signal a desire for change. It arises because we cannot write the Specification in one go; we must decompose the problem into steps. One Proposal roughly corresponds to **One Act** of work.

It is **Preserved**. It serves as the historical record of the vision change and becomes the **Source of Truth for Intent**.

## 2. The Need

Why do we write a Proposal Document instead of just coding?

- **Overcoming the "Blank Slate":** We cannot automate human intent. The Proposal forces us to extract thoughts from the subconscious. **Dictation** is the preferred method to break paralysis and generate the initial seed content.
- **Source of Tickets:** This document is the seed from which specific engineering tickets are derived. It ensures work is broken down into atomic, manageable chunks (e.g., 1-day tasks).
- **Separation of Concerns:** It separates "What we want" from "How we implement it." This allows us to debate the value of the idea without getting bogged down in syntax.
- **Safe Exploration:** It allows us to propose radical changes without breaking the source of truth. If a Proposal is rejected, nothing is lost but a text file.
- **Context for the Machine:** LLMs require precise context. This document provides the "instruction prompt" for the machine (and the team).

## 3. Structure (Template)

A Proposal Document follows a rigorous narrative structure designed to validate logic and safety. The following sections are the **Standard**, but can be adapted if a different structure better serves the truth.

### 3.1. Context (The Evolution)

_Narrative Pattern:_ "We thought X, but we learned Y, so we must do Z."

1.  **Original Assumption:** What did we believe before?
2.  **The Pivot (Discovery):** What changed? (Bug found, new requirement, insight).
3.  **The Conclusion:** Why is change necessary now?

### 3.2. The Problem (The Trigger)

1.  **The Problem:** Specific, falsifiable statement of what is broken or missing.
2.  **Why Now:** The cost of doing nothing.

### 3.3. The Scope (The Contract)

1.  **Invariants:** What must remain true? (Performance, Compatibility, etc.)
2.  **Negative Space:** Explicit list of what we are **NOT** doing.
3.  **Comparison:** A "Before vs. After" table.

### 3.4. Intents (Atomic Changes)

**The Core.** Decomposition of the solution into granular, independent units of change.

_Format per Intent:_

- **Title:** Concise summary.
- **Type:** {Bug Fix / Feature / Cleanup / Refactor}
- **Target:** {Module / File / Component}
- **Rationale:** Why this specific change is needed (historical context).
- **Diff/Logic:** Abstract representation of the change (Pseudo-code, Interface, or Logic description). **No implementation code.**

### 3.5. Visual Model (Optional)

_Recommended:_ A **Mermaid Diagram** representing the workflow, state machine, or interaction. Use if the text is insufficient to capture the complexity.

### 3.6. Usage Patterns (Optional)

_Recommended:_ Concrete examples of how the new system will be _consumed_. Prove that the design is ergonomic.

### 3.7. Safety & Risks (Optional)

1.  **Risks:** What could break?
2.  **Mitigations:** How do we prevent it?
3.  **Invariance:** What must remain true (e.g., "Performance must not drop below X").

### 3.8. Completion Criteria

A checklist of verification steps (Tests, Docs, Integration) defining "Done."

### 3.9. Options Considered

Alternatives that were rejected, and _why_.

---

> [!NOTE] Partial Completion
> If generating a proposal from dictation or limited context, it is acceptable to mark sections as `(TBD)` or `(Placeholder)`. The structure serves as a guide for what _should_ be known, even if it is not yet known.

## 4. Lifecycle

1.  **Draft:** Created by an author via Dictation or AI Prompt.
2.  **PR:** Submitted as a Pull Request.
3.  **Discussion:** Debated by the team.
4.  **Consensus:** :term[Alignment Document]{href="./22_document_alignment.md"} generated and verified.
5.  **Patching:** The Proposal is updated to match the Alignment (Consensus).
6.  **Freeze:** The Proposal is marked "Accepted" and becomes immutable.
7.  **Execution:** The Proposal drives updates to the Specification.
8.  **Archive:** Preserved as history.

## 5. Characteristics

- **Timeframe:** Created within ~1 day.
- **Updates:** Written once to capture the vision change.
- **Nature:** Preserved/Historical.
- **Source of Truth:** **For Intent**.
- **Analogy:** RFC / One Act.
