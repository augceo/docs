# 51: Prompt/Evolution Draft

> [!DEFINITION] Evolution Draft
> Think of this as a quick, temporary sketch of a plan. It’s used to make sure everyone agrees on changes and understands why they’re happening before the official plan is updated. It catches new ideas right after they're discussed.
>
> Sidenote:
>
> - See the :term[02: Company/Process]{href="./02_process.md"} document to understand the full process.

## Purpose

The Evolution Draft is the bridge between a conversation about changes (like comments on a project) and the final, official rulebook. It's designed to capture the new understanding or plan that comes out of a discussion.

## Instructions for the Agent

When a user asks you to create an Evolution Draft, you **MUST** get a link to the project's discussion (the Pull Request) and a timeframe to look at.

### 1. Check for What You Need

1.  **Project Link:** If the user doesn't give you a link to the GitHub Pull Request, you have to **STOP** and ask:

    > "Please give me the link to the GitHub Pull Request you want me to look at."

2.  **Start Date:** If the user doesn't tell you how far back to look for comments, you have to **STOP** and ask:
    > "Please tell me a 'since' date (like YYYY-MM-DD) so I know which comments to check. If you want me to look at all of them, just say 'All'."

### 2. Getting the Information

**RULE:** You are only allowed to get information from these three places:

1.  **HTTP GET** to `https://idealic.academy/en/company/02_process.md/`
2.  **HTTP GET** to `https://idealic.academy/en/company/50_prompt_truth.md/`
3.  **A GitHub API call** to get comments (using the command below).

You are **NOT** allowed to look at any other files, commit histories, or code changes.

**Step 1: Get the Rulebooks (You Must Do This)**
You **MUST** use a tool to download the content from these exact web addresses. **Do NOT search the web for them.**

- [02: Company/Process](https://idealic.academy/en/company/02_process.md/)
- [50: Prompt/Truth](https://idealic.academy/en/company/50_prompt_truth.md/)

**Step 2: Get the Comments (Using a Single Command)**
Run this exact command. You'll need to replace `{PR_NUMBER}` with the project number (like 123) and `{SINCE_DATE}` with the date (like 2025-01-01). This command is specially designed to grab up to 200 comments at once and group them into conversation threads. It’s important not to change it.

**VERY IMPORTANT**: This command is like a magic spell. It organizes all the comments into neat conversation threads, includes the relevant piece of code for the first comment in each thread, and gets everything in one go. If you change it, you'll get the wrong results.

```bash
gh api "repos/{OWNER}/{REPO}/pulls/{PR_NUMBER}/comments?since={SINCE_DATE}&per_page=200" --paginate --jq 'map({id, body, user: .user.login, created_at, html_url, diff_hunk, in_reply_to_id}) | group_by(.in_reply_to_id // .id) | map(sort_by(.created_at) | .[0] as $root | [$root] + (.[1:] | map(del(.diff_hunk))))' | jq '.' > "comments_{SINCE_DATE}.json"
```

### 3. Understanding and Summarizing (Language: Russian)

**First:** You need to read and understand the rules in these two documents:

- :term[02: Company/Process]{href="https://idealic.academy/en/company/02_process.md/"} (This explains the purpose of an Evolution Draft.)
- :term[50: Prompt/Truth]{href="https://idealic.academy/en/company/50_prompt_truth.md/"} (This explains how to write clearly and directly.)

**Rule:** The final report you create must be in **Russian**. (You can keep common English words like "yeah" or "ok," or technical terms.)

**How to do it: Smart Summarizing**
This isn't just about copying and pasting. You need to act like a smart reporter who understands the conversation and writes a short, truthful summary.

- **Rephrase and Shorten:** Turn long comments into short, simple sentences that keep the original meaning.
- **Ignore the Noise:** Leave out comments that don't add anything important, like "Thanks!" or "Looks good."
- **The Goal:** Create a report that is short, clear, and accurate.

Read through the file of comments you downloaded. Your goal is to cover **everything**. Every single conversation thread needs to be included.

**For each conversation, figure out:**

1.  **Intent:** What does the person really want?
2.  **Explanation:** _Why_ do they want this change? What's the bigger reason? Use the whole discussion to figure this out and give context.
3.  **Action:** What needs to be done now?
4.  **Vision Impact:** Did this change the project's main goal or long-term plan?
5.  **Agreement Status:** Did people agree? Or was there a misunderstanding?
    - If the original author **never replied** to a comment, mark it as: "Needs acknowledgement."
6.  **Context:** Show your proof. Include short quotes from the conversation and the piece of code they were talking about.

### 4. Double-Checking Your Work

**CRITICAL STEP:** Before you finish, you must check your own work to make sure it's perfect.

1.  Read the summary you just wrote.
2.  Compare it against the original `comments_{DATE}.json` file.
3.  **Check Every Comment:** Make sure every conversation thread from the original file is mentioned in your summary.
4.  **Check Your References:** Make sure every point in your summary (like "Intent #3") actually exists. If you mention it in your final checklist, the section has to be in the report.
    - **Fix it:** If you find a reference to a point that you forgot to write about, you **MUST** go back and add the missing section. Don't just delete the reference from the checklist; write the summary for it.
5.  **Ask yourself:** "Did I miss _any_ conversation thread?"
6.  If the answer is yes, add it right away.

### 5. Creating the Document

Create a new file, for example: `evolution_{DATE}.md`.

**It must have this structure:**

````markdown
# Evolution Draft: {DATE}

> Status: Draft
> Source: {PR_LINK}

## Overview

{Write a short summary of the whole discussion in Russian. Answer these questions:}

1.  **What the reviewer wanted:** {What were the main suggestions or criticisms?}
2.  **What the author agreed with:** {What important changes did the author agree to make?}
3.  **The big picture:** {Were any misunderstandings cleared up? Did the main goal of the project change?}
4.  **Misunderstandings (if any):** {List things people were confused about.}
5.  **Open questions (if any):** {List things that were discussed but not decided.}
6.  **New discoveries:** {What new ideas or insights came from the discussion?}
7.  **Related documents:** {Add links to any other documents that were mentioned.}

## List of Intents

<!-- Do this for EVERY conversation thread you found -->

### {N}. {A Short Title in Russian}

- **Intent:** {What do we want to do?}
- **Reasoning:** {Explain _why_ this change is needed, based on the conversation.}
- **Action:** {What are the exact steps to take?}
- **Status:** {Agreed / Misunderstanding / Needs More Info / Vision Change / Needs Acknowledgement}
- **Old Understanding (if it changed):** {Briefly describe the old plan if it's different now.}
- **Result:** {In short: Was the big plan changed? Was an agreement reached? Was a misunderstanding cleared up?}
- **Context:**

  > [{Reviewer's Name}]({Link}): "{Short, rephrased version of their point}"
  >
  > [{Author's Name}]({Link}): "{Short, rephrased version of their answer}" (or "No reply")

  ```{lang}
  {Show 1-3 lines of the code they discussed}
  {Use ... for very long lines}
  {Keep this code block very small}
  ```

---

## Validation Report

| Comment ID    | Title (Summary)    | Intent # | Status          |
| ------------- | ------------------ | -------- | --------------- |
| [{ID}]({URL}) | {3-6 word summary} | {N}      | Included        |
| [{ID}]({URL}) | {3-6 word summary} | -        | Skipped (Noise) |
````
