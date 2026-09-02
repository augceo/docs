# 21: Company/Proposal

> [!DEFINITION] Proposal Document
> A plan written down before starting a big project. It explains an idea for a change or a new feature before anyone starts building it.
>
> Sidenote:
> - Part of: :term[02: Company/Process]{href="./02_process.md"}
> - Followed by: :term[22: Company/Alignment]{href="./22_document_alignment.md"}
> - Formerly known as: Intent Document, Evolution Document

## 1. The Core Idea

A Proposal is like a **"Request for Ideas" (or RFC)**. It's written **one time** for each new project to announce that you want to make a change. We need proposals because big projects are too complicated to figure out all at once; this helps break the larger problem into smaller steps. Each proposal is like one big task, or **one 'Act'** of a project.

This document is **saved forever**. It becomes the official story of *why* we decided to make a change, serving as the **main source for understanding our original goal**.

## 2. Why We Need It

Why write a plan instead of just starting to code?

- **To Beat the 'Blank Page' Feeling:** People have ideas, but you can't just tell a computer to "build something cool." A proposal forces us to pull those thoughts out of our heads and put them on paper. **Talking out the idea** is the best way to get started when you feel stuck.
- **To Create a To-Do List:** This document is the starting point for all the small tasks (or "tickets") that engineers will work on. It makes sure every little job is part of the bigger plan.
- **To Separate 'What' from 'How':** It keeps the discussion about an idea ("What we want") separate from the technical details ("How we'll build it"). This lets the team agree on the goal first, without getting bogged down in how it will work.
- **To Safely Explore Big Ideas:** It lets you suggest big, ambitious changes without breaking anything. If the team decides against the idea, you just get rid of the document. No harm done.
- **To Give Clear Instructions:** AI helpers need very specific instructions to be useful. This document acts as the perfect "prompt" or command that tells the AI (and the human team) exactly what the goal is.

## 3. The Structure (A Template)

A Proposal follows a clear, story-like structure to make sure the idea is logical and safe. The sections below are the standard, but you can change them if it helps explain your idea better.

### 3.1. The Story So Far (The Evolution)

_The storytelling pattern:_ "We used to think X, but then we learned Y, so now we need to do Z."

1.  **What We Used to Believe:** What was our old way of thinking?
2.  **The Turning Point:** What new thing did we discover? (A bug, a new request, a new insight).
3.  **The New Plan:** Why is it important to make this change now?

### 3.2. The Problem (What Started This)

1.  **The Problem:** A simple, clear statement of what is broken or missing.
2.  **Why It's Urgent:** What might go wrong if we don't fix this now?

### 3.3. The Boundaries (The Agreement)

1.  **Rules We Can't Break:** What things must stay the same? (For example, speed, or working with older versions).
2.  **What We're NOT Doing:** A clear list of things this project will *not* do.
3.  **Before vs. After:** A simple chart showing the difference between the old way and the new way.

### 3.4. The Steps (The Small Changes)

**The most important part.** This is where thebig solution is broken down into small, separate steps.

_How to write each step:_

- **Title:** A short summary.
- **Type:** {Fixing a Bug / New Feature / Tidying Up / Reorganizing}
- **Where:** {Which part of the system / Which file}
- **Reason:** Why this specific step is needed.
- **The Plan:** A simple sketch of the change (like a flowchart or an outline). **Do not write actual code here.**

### 3.5. A Picture (Optional)

_Good to have:_ A **diagram** (made with a tool called Mermaid) that shows how things will work, like a flowchart or a map. Use this if words aren't enough to explain a complex idea.

### 3.6. Examples of Use (Optional)

_Good to have:_ Real-life examples of how people will use the new system. This helps prove that the new design is easy to use and makes sense.

### 3.7. Safety Check (Optional)

1.  **Risks:** What could possibly go wrong?
2.  **How We'll Prevent Problems:** Our plan to stop those bad things from happening.
3.  **Guarantees:** What must stay true (for example, "The app must not get any slower").

### 3.8. When Are We 'Done'?

A checklist to prove the work is truly finished (for example, all tests pass, the user guide is updated, etc.).

### 3.9. Other Ideas We Considered

A list of different solutions we thought about and why we decided not to use them.

---

> [!NOTE] It's Okay to Start Small
> If you're just writing down an idea from a conversation, it's fine to leave some sections blank or mark them as `(To be decided)`. The structure is just a guide to help you think everything through.

## 4. The Life of a Proposal

1.  **Draft:** Someone writes the first version, maybe by talking it out or using an AI helper.
2.  **Review:** It's shared with the team so everyone can see it.
3.  **Discussion:** The team talks about the plan and gives feedback.
4.  **Agreement:** Everyone agrees on the final plan in a separate document.
5.  **Final Polish:** The Proposal is updated with any changes from the discussion.
6.  **Freeze:** The Proposalis marked as "Accepted" and can't be changed anymore.
7.  **Action:** The plan is now used to update the main project blueprints.
8.  **Archive:** The Proposal is saved as part of the project's history.

## 5. Quick Facts

- **Time to Create:** About a day.
- **How Often It's Updated:** It's written once to capture the main idea.
- **Purpose:** Saved as a historical record.
- **It Defines:** The *reason* why we did something.
- **Think of it as:** A request for ideas, or one 'Act' of a bigger story.