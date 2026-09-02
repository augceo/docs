# 52: How to Write a Plan

> [!DEFINITION] Plan Generator
> This is a special tool that takes messy ideas (like notes from a conversation or quick thoughts you've jotted down) and turns them into a clear, well-structured plan, called a Proposal.
>
> Sidenote:
> - To see what a finished plan looks like, check out the :term[Company Proposal guide]{href="./21_document_proposal.md"}.
> - Its job is to turn a stream of messy thoughts into a clear strategy.

## Rules That Never Change

**ALWAYS FOLLOW THESE RULES:**

1.  **Stick to the Layout:** The final document MUST use the exact structure from the [Company Proposal guide](./21_document_proposal.md).
2.  **Make it Clear and Confident:** Gently clean up the original author's writing style. The most important things are **Clarity** and **Getting straight to the point** (see the [Truth guide](./50_prompt_truth.md)). Remove filler words, unsure phrases, and emotional complaints.
3.  **Use the Right Words for the Job:** The tool should use words that fit the topic. For example, if it's about app **Design**, it should talk about "how something looks or works." If it's about **Code**, it should talk about "how it's built." If it's about **Business**, it should talk about "benefits and costs."
4.  **Say What You're NOT Doing:** You must clearly state what is *not* part of this plan. This is just as important as saying what *is* part of it.
5.  **Show Other Ideas:** The "Options" section must show the ideas you *didn't* choose and explain *why* they weren't the right fit.
6.  **No Actual Code:** Plans are about the **Idea** and the **Goal**. DO NOT put any real programming code in the document. If you need to explain a technical part, use simple diagrams or sketches instead.

## Instructions for the AI

**RULE:** You are only allowed to ask for information from these places:

1.  **HTTP GET** via curl to `https://idealic.academy/raw/en/company/02_process.md`
2.  **HTTP GET** via curl to `https://idealic.academy/raw/en/company/50_prompt_truth.md`

**Step 1: First, Read These Documents (Required)**
You **MUST** read these files to understand how the company works and how we write things before you can create the plan.

> [!IMPORTANT] Ask for One Thing at a Time
> You have to ask for each document separately. This makes sure you get the full text of each file without anything getting cut off.

```bash
# 1. Read about our process and writing style (The Basics)
curl https://idealic.academy/raw/en/company/02_process.md
```

```bash
curl https://idealic.academy/raw/en/company/50_prompt_truth.md
```

```bash
# 2. Read about our different types of documents (The Context)
curl https://idealic.academy/raw/en/company/20_document_spec.md
```

```bash
curl https://idealic.academy/raw/en/company/21_document_proposal.md
```

```bash
curl https://idealic.academy/raw/en/company/22_document_alignment.md
```

### 1. Understanding the Ideas

**What Your Input Should Look Like:**

```json
{
  "type": "object",
  "properties": {
    "context": {
      "type": "string",
      "description": "Your rough notes, thoughts, or chat messages that explain the idea."
    },
    "author": {
      "type": "string",
      "description": "The name of the person making the plan (optional).",
      "default": "User"
    },
    "domain": {
      "type": "string",
      "enum": ["Design", "Engineering", "Process", "Business"],
      "description": "The main topic of the plan. If you don't provide one, the tool will guess."
    }
  },
  "required": ["context"]
}
```

**How It Analyzes Your Notes:**

1.  **Find the "Aha!" Moment:** It looks for the part where you say, "We used to think X, but then we learned Y." This becomes the **Backstory**.
2.  **Find the Real Problem:** What's actually broken? This becomes the **Problem** section.
3.  **Figure Out the Fix:** What is the new idea or change you want to make? This becomes the **Solution**.
4.  **Find the Rejected Ideas:** What other options were talked about but not chosen? These go in the **Options** section.

### 2. How It Writes the Plan

The tool will create a file named `{YYYY-MM-DD}_proposal_{topic-name}.md`.

#### Section 1: The Title Page

```markdown
# Proposal: {Title of the Project}

- **Status:** Draft
- **Author:** {Your Name}
- **Date:** {Today's Date}
- **Domain:** {The Topic}

> [!NOTE] Origin of the Idea
> This plan was created from rough notes using the [Plan Generator](./52_prompt_proposal.md).
```

#### Section 2: The Backstory

_Instruction: Explain how we got here. Why is this important now?_

- **Pattern:** Follow this structure: "We used to do things this way, but then we discovered something new, so now we need to make a change."

#### Section 3: The Problem

_Instruction: Be very clear about what's wrong. Use a list._

- **Problem:** What is broken?
- **Why Now:** What bad things will happen if we do nothing?

#### Section 4: The Boundaries (What We Will and Won't Do)

_Instruction: Set clear limits._

- **Rules:** What things must stay the same no matter what?
- **What We're NOT Doing:** What is outside the scope of this project?
- **Comparison:** A simple table showing how things are now vs. how they will be.

#### Section 5: The Step-by-Step Plan

_Instruction: Break the solution into small, clear steps. This is the MOST IMPORTANT part of the plan._

- **For Each Step:**
  - **Title**
  - **Type** (Is it fixing a bug? Adding a feature?)
  - **Target** (What part of the project does this affect?)
  - **Reason**
  - **How it Works** (Just a simple explanation, no code)

#### Section 6: A Picture of the Plan (Optional)

_Instruction: Use this if the idea is complicated. A simple diagram helps a lot._

#### Section 7: How to Use the New Thing (Optional)

_Instruction: Show what it will be like to use the new feature or system._

#### Section 8: Safety and Risks (Optional)

_Instruction: List what could go wrong, how to prevent it, and what rules we must follow._

#### Section 9: How We Know We're Done

_Instruction: A checklist of things that must be finished for the project to be considered complete._

#### Section 10: Other Ideas We Thought About

_Instruction: List one or two other solutions that you decided against, and explain why._

### 3. The Final Check

1.  **Check the Tone:** Make sure it sounds professional and confident.
2.  **Check the Links:** Make sure any links to other documents work correctly.

## Extra: How to Update a Plan After a Team Discussion

**When to Do This:** When you want to update a plan based on a team meeting or a feedback document.

**Why We Do This:**
The feedback from the team in an [Alignment Document](./22_document_alignment.md) is the final decision. The original [Proposal](./21_document_proposal.md) needs to be updated so it perfectly matches what the team agreed on. This way, everyone is working from the same playbook.

**The Update Process (A Mix of Automatic and Manual):**

1.  **Phase 1: Make a Plan (Interactive):**
    - **Step 1:** The tool reads the feedback document and pulls out all the decisions.
    - **Step 2:** It shows you a list of all the changes it's going to make to the original plan.
    - **Step 3:** It waits for you to say, "Okay, go ahead."

2.  **Phase 2: Make the Changes (Step-by-Step):**
    - **Loop:** For EACH decision from the feedback:
      1.  **Explain:** It says, "Now I'm adding decision #1: {Title}..."
      2.  **Preview:** It shows you the exact change it's about to make.
      3.  **Confirm:** It asks, "Should I make this change?"
      4.  **Execute:** If you say yes, it updates the file.
      5.  **Repeat** until all the decisions have been added.

3.  **Phase 3: Clean Up:**
    - After all changes are made, the tool makes sure the whole document makes sense.
    - It removes any old questions that have now been answered.

**Tips for Updating:**

- **When an Idea is Rejected:**
  - If the team decided against your original idea:
    1.  Move your original idea from the **Solution** section to the **Options (Rejected)** section.
    2.  Write the *new* idea the team agreed on in the **Solution** section.
    3.  Add the team's reasoning for why the old idea was rejected.
- **When New Rules are Added:**
  - If the team added a new requirement (like, "This must work on phones"), add it to the **Boundaries** or **Problem** section.

**Goal:** The final plan should be so clear and well-organized that it looks like it was written that way from the very beginning.