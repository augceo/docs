# 52: Prompt/Proposal

> [!DEFINITION] Proposal Generator
> A structural engine that synthesizes raw context (dictation, notes, chats) into a rigorous Proposal Document (RFC).
>
> Sidenote:
>
> - See: :term[21: Company/Proposal]{href="./21_document_proposal.md"} for the definition.
> - Purpose: Turns "stream of consciousness" into "strategy."

## Invariance Rules

**STRICTLY MAINTAIN THESE RULES:**

1.  **Format Adherence:** Output MUST strictly follow the structure defined in [21: Company/Proposal](./21_document_proposal.md).
2.  **Voice Normalization:** Mildly preserve the author's voice, but prioritize **Clarity** and **Direct Assertion** (see [50: Prompt/Truth](./50_prompt_truth.md)). Remove fluff, hesitation, and emotional venting.
3.  **Domain Agnosticism:** Adapt terminology to the detected domain (e.g., Design = "Affordance/User Flow", Code = "Architecture/Contract", Business = "Leverage/Cost").
4.  **Negative Space:** You MUST infer or explicitly state what is **NOT** being done (The "Anti-Scope").
5.  **Alternatives:** You MUST structure the "Options" section to show not just the chosen path, but the rejected paths and _why_ they were rejected.
6.  **No Implementation Code:** Proposals are for **Intent** and **Vision**. Do NOT include implementation code, snippets, or diffs. Use high-level schemas, abstract interfaces, or diagrams to convey technical structure if necessary.

## Protocol for Agent

**RESTRICTION:** You are permitted to make ONLY the following external requests:

1.  **HTTP GET** via curl to `https://idealic.academy/raw/en/company/02_process.md`
2.  **HTTP GET** via curl to `https://idealic.academy/raw/en/company/50_prompt_truth.md`

**Step 1: Fetch Prerequisite Docs (Mandatory)**
You **MUST** fetch these documents to understand the Company Process and Writing Standards before generating the Proposal.

> [!IMPORTANT] Separate Requests
> You must fetch each document in a **separate** bash block/tool call. This prevents the output from being truncated, ensuring you receive the full context of every file.

```bash
# 1. Process & Truth (The Foundation)
curl https://idealic.academy/raw/en/company/02_process.md
```

```bash
curl https://idealic.academy/raw/en/company/50_prompt_truth.md
```

```bash
# 2. Artifact Definitions (The Context)
curl https://idealic.academy/raw/en/company/20_document_spec.md
```

```bash
curl https://idealic.academy/raw/en/company/21_document_proposal.md
```

```bash
curl https://idealic.academy/raw/en/company/22_document_alignment.md
```

### 1. Input Analysis

**Input Schema:**

```json
{
  "type": "object",
  "properties": {
    "context": {
      "type": "string",
      "description": "Raw text, dictation, or pasted chat logs containing the user's thoughts."
    },
    "author": {
      "type": "string",
      "description": "Name of the proposer (optional).",
      "default": "User"
    },
    "domain": {
      "type": "string",
      "enum": ["Design", "Engineering", "Process", "Business"],
      "description": "The primary domain of the proposal. If not provided, infer from context."
    }
  },
  "required": ["context"]
}
```

**Analysis Steps:**

1.  **Identify the Pivot:** Find the "We thought X, but learned Y" moment. This is the **Context**.
2.  **Isolate the Pain:** What is explicitly broken? This is the **Problem**.
3.  **Define the Change:** What is the specific intervention? This is the **Solution**.
4.  **Extract the Conflicts:** What other ideas were mentioned and discarded? These are the **Options**.

### 2. Document Generation Workflow

Generate a markdown file named `{YYYY-MM-DD}_proposal_{topic_slug}.md`.

#### Section 1: Frontmatter & Header

```markdown
# Proposal: {Title of Initiative}

- **Status:** Draft
- **Author:** {Author}
- **Date:** {Today}
- **Domain:** {Detected Domain}

> [!NOTE] Intent Source
> Derived from raw context via [52: Prompt/Proposal](./52_prompt_proposal.md).
```

#### Section 2: Context (The Evolution)

_Instruction: Synthesize the background. Why are we talking about this now?_

- **Pattern:** "We thought X, but we learned Y, so we must do Z."

#### Section 3: The Problem (The Trigger)

_Instruction: Be specific. Use bullet points._

- **Problem:** What is broken?
- **Why Now:** The cost of doing nothing.

#### Section 4: The Scope (The Contract)

_Instruction: Define boundaries._

- **Invariants:** What must remain true?
- **Negative Space:** What are we **NOT** doing?
- **Comparison:** Current vs. Proposed table.

#### Section 5: Intents (Atomic Changes)

_Instruction: Breakdown the solution into granular units. This is the CORE of the proposal._

- **Structure per Intent:**
  - **Title**
  - **Type** (Bug Fix/Feature/etc)
  - **Target** (File/Module)
  - **Rationale**
  - **Diff/Logic** (Abstract only)

#### Section 6: Visual Model (Optional)

_Instruction: Recommended if the flow is complex. Use Mermaid._

#### Section 7: Usage Patterns (Optional)

_Instruction: Show how the new system is consumed (UX/DX)._

#### Section 8: Safety & Risks (Optional)

_Instruction: Risks, Mitigations, and Invariants._

#### Section 9: Completion Criteria

_Instruction: Checklist for "Definition of Done"._

#### Section 10: Options Considered

_Instruction: List 1-2 alternatives that were rejected and why._

### 3. Final Polish

1.  **Check Tone:** Ensure it sounds professional but not robotic. Use active voice.
2.  **Verify Links:** Ensure internal links to other company docs use the correct relative paths if referenced.

## Appendix: Workflow - Applying Alignment

**Trigger:** User requests "Apply Alignment to Proposal" or provides an Alignment Document to update a Proposal.

**Context:**
The [Alignment Document](./22_document_alignment.md) represents the _Consensus_ achieved during review. It is transient. The [Proposal](./21_document_proposal.md) must be updated to reflect this consensus so it remains the "Source of Truth for Intent."

**Instructions (Semi-Auto Mode):**

1.  **Phase 1: Planning (Interactive):**
    - **Step 1:** Read the Alignment Document and extract all **Intents**.
    - **Step 2:** Present a **Plan of Action** to the user.
      - List every Intent to be applied.
      - Briefly explain _where_ in the Proposal it will be integrated.
    - **Step 3:** Pause and ask for user confirmation to proceed.

2.  **Phase 2: Execution (Iterative):**
    - **Loop:** For EACH Intent in the Plan:
      1.  **Explain:** "Applying Intent #{N}: {Title}..."
      2.  **Preview:** Show the specific diff or text change you are about to make.
      3.  **Confirm:** Ask the user "Apply this change?"
      4.  **Execute:** If confirmed, edit the file.
      5.  **Repeat** until all intents are applied.

3.  **Phase 3: Cleanup:**
    - Once all intents are applied, verify the document flows logically.
    - Remove any "Open Questions" that were resolved by the intents.

**Strategies for Integration:**

- **Handling Rejections:**
  - If the Consensus (Alignment) rejected the Author's original idea:
    1.  Move the original text from **Proposed Solution** to **Options (Rejected)**.
    2.  Write the _new_ agreed approach in **Proposed Solution**.
    3.  Add the reasoning from the Alignment to the "Why Rejected" field.
- **Handling Constraints:**
  - If the Alignment identified new constraints (e.g., "Must support mobile"), add them to **Scope** or **Problem**.

**Goal:** The resulting Proposal should read as if it were written this way from the start—confident, clear, and aligned.
