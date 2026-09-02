# 51: Instructions for Summarizing Discussions

> [!DEFINITION] Alignment Document
> This is a summary that the computer creates automatically after people have reviewed a project. It brings together everyone's ideas, shows which problems were solved, and gives clear instructions on what to do next.

> Sidenote:
> - For more details, see: :term[22: Company/Alignment]{href="./22_document_alignment.md"}.

## The Golden Rules

**YOU MUST ALWAYS FOLLOW THESE RULES:**

1.  **One Notebook Only:** You are allowed to create one special file (`{OUTPUT_DIR}/{FILENAME}.ndjson`) to jot down notes about the comments. Don't create any other temporary files.
2.  **Ask Once:** Get all the information you need (like all the comments) in one single request. Don't go back and ask for more little by little.
3.  **Use Your Notes:** When you need to check something, look at the information you already have. Don't go back to the original source to double-check.
4.  **Speak the Right Language:** The final summary must be written in the language the user asked for. Technical terms and direct quotes from people can stay in their original language.
5.  **No Comment Left Behind:** Every single comment must be mentioned in the final checklist, called the Coverage Report.
6.  **Fix, Don't Restart:** If you notice a problem (like a missing comment), just fix that one thing. Don't throw everything away and start all over again.
7.  **Think For Yourself:** Do all the work of analyzing, counting, and summarizing in your own digital brain. Don't use other programs or scripts to help you.

## Why We Do This

The Alignment Document acts as a bridge between talking about ideas (in a Pull Request) and actually doing the work. It captures the main points and agreements that come out of a team's review.

## The Recipe for the AI Agent

When a user asks for an Alignment Document, you **MUST** first figure out the settings they want.

### 1. Figure Out the Settings

**What the user can specify:**

```json
{
  "type": "object",
  "required": ["since_date", "repo", "pr_number", "output_dir", "filename", "language"],
  "properties": {
    "since_date": {
      "type": "string",
      "description": "A date to start looking from (YYYY-MM-DD). If you don't provide one, it will look at the last week.",
      "default": "{ONE_WEEK_AGO}"
    },
    "repo": {
      "type": "string",
      "description": "The name of the project. If you don't provide one, it will use the current project.",
      "default": "{CURRENT_REPO}",
      "example": "idealic-ai/platform"
    },
    "pr_number": {
      "type": "integer",
      "description": "The number of the Pull Request. This is REQUIRED.",
      "required": true
    },
    "output_dir": {
      "type": "string",
      "description": "The folder where the summary will be saved. The default is 'alignments'.",
      "default": "alignments"
    },
    "filename": {
      "type": "string",
      "description": "The name for the summary file. By default, it uses the PR number and the date. Don't add an extension like .md.",
      "default": "pr{PR_NUMBER}_{SINCE_DATE}"
    },
    "language": {
      "type": "string",
      "description": "The language for the summary (like Russian or English). The default is Russian.",
      "default": "Russian"
    },
    "auto_post": {
      "type": "boolean",
      "description": "If this is true, the summary will be automatically posted on GitHub.",
      "default": false
    },
    "merge_alignments": {
      "type": "boolean",
      "description": "If true, this will combine the new summary with any previous one. If false, it will start fresh.",
      "default": true
    },
    "include_instructions": {
      "type": "boolean",
      "description": "Tells the system whether to include extra instructions and warnings in the final document.",
      "default": null
    },
    "post_summary_viewer": {
      "type": "string",
      "description": "If you put a username here, the system will create a special summary just for that person.",
      "default": null
    }
  }
}
```

**Step 0: Confirm the Settings**

1.  Read what the user wrote and pull out these settings.
2.  **Check for Required Info:** If the `pr_number` is missing, **STOP** and ask the user for it.
3.  **Use Defaults:** For any setting the user didn't mention, use the default value.
    - If `include_instructions` wasn't set: If `auto_post` is true, set it to `false`. Otherwise, set it to `true`.

**Step 1: Show Your Work (First Reply)**
As your very first reply, you **MUST** show the user the settings you've decided on. Use the default value if none was provided.

```text
> Alignment Config:
--------------------------------
- Project:      {repo}
- PR Number:    {pr_number}
- Start Date:   {since_date}
- Output Folder:{output_dir}
- Filename:     {filename}
- Language:     {language}
- Auto Post:    {auto_post}
- Merge Summaries: {merge_alignments}
- Show Instructions: {include_instructions}
- Personal Summary For: {post_summary_viewer}
--------------------------------
Starting analysis...
```

### 2. Gather the Information

> [!WARNING] NO EXTRA FILES
> **DO NOT** save the information you download into extra temporary files (like `process.md` or `pr.json`).
> **Why?** It makes a mess and breaks Golden Rule #1.
> **The Only Exception:** You are allowed to save the raw comments into the special `{OUTPUT_DIR}/{FILENAME}.ndjson` file.
>
> Everything else you get should just be displayed directly.

**RESTRICTION:** You are only allowed to get information from these four places:

1.  **HTTP GET** using curl to `https://idealic.academy/raw/en/company/02_process.md`
2.  **HTTP GET** using curl to `https://idealic.academy/raw/en/company/50_prompt_truth.md`
3.  A **GitHub API** call to find out who created the Pull Request.
4.  A **GitHub Comments API** call (using the specific command below) to get all the comments.

Do not try to get other information, like the code changes or the list of commits. That's not part of your job.

**Step 1: Get the Rulebooks (Required)**
Get these documents. **Handle each one separately.**
**Rule:** You must use a **separate command** for each document. DON'T chain them together with `&&` or `;`.

1.  **The Process Document:**

    ```bash
    curl -s https://idealic.academy/raw/en/company/02_process.md
    ```

2.  **The Truth Document:**

    ```bash
    curl -s https://idealic.academy/raw/en/company/50_prompt_truth.md
    ```

3.  **The Alignment Definition:**
    ```bash
    curl -s https://idealic.academy/raw/en/company/22_document_alignment.md
    ```

**Step 2: Get PR Details (Find Out Who's Who)**
Get the details of the Pull Request to find out who the Author is.

```bash
gh api "repos/{OWNER}/{REPO}/pulls/{PR_NUMBER}" --jq '{author: .user.login, title: .title}'
```

**Step 3: Get the Comments**

1.  **Save Comments to Your Notebook:** Run this exact command to save all the comments into your one allowed temporary file: `{OUTPUT_DIR}/{FILENAME}.ndjson`.

    ```bash
    gh api graphql -F owner='{OWNER}' -F repo='{REPO}' -F pr={PR_NUMBER} -f query='
      query($owner:String!, $repo:String!, $pr:Int!) {
        repository(owner:$owner, name:$repo) {
          pullRequest(number:$pr) {
            reviewThreads(first: 100) {
              nodes {
                isResolved
                comments(first: 50) {
                  nodes {
                    databaseId
                    body
                    author { login }
                    createdAt
                    path
                    diffHunk
                    replyTo { databaseId }
                    url
                    reactions(first: 10) { nodes { content } }
                  }
                }
              }
            }
          }
        }
      }
    ' --jq '
      .data.repository.pullRequest.reviewThreads.nodes[]
      | .isResolved as $resolved
      | .comments.nodes[]
      | {
          id: .databaseId,
          body,
          user: (.author.login // "ghost"),
          created_at: .createdAt,
          path,
          diff_hunk: .diffHunk,
          in_reply_to_id: .replyTo.databaseId,
          html_url: .url,
          reactions: ((.reactions.nodes | map({(.content): 1}) | add) // {}),
          is_resolved: $resolved
        }
    ' | jq -s '
        group_by(.in_reply_to_id // .id)
        | map(sort_by(.created_at))
        | sort_by(.[0].created_at)
        | flatten
        | to_entries
        | map(.value + {index: ("§" + ((.key + 1) | tostring)), anchor: (.value.html_url | split("#") | last)} | del(.html_url))
        | group_by(.in_reply_to_id // .id)
        | sort_by(.[0].index | ltrimstr("§") | tonumber)
        | map(map({
        index,
        id,
        anchor: .anchor,
        path,
        body,
        user,
        created_at,
        diff_hunk: (.diff_hunk | if length > 350 then .[:350] + "..." else . end),
        in_reply_to_id,
        reactions: (.reactions | with_entries(select(.value > 0))),
        is_resolved
        }))
        | map(.[0] as $root | [$root] + (.[1:] | map(del(.diff_hunk, .is_resolved, .path))))
        | map(map(del(.in_reply_to_id)))
        | .[]
    ' -c > {OUTPUT_DIR}/{FILENAME}.ndjson
    ```

### 3. Manage the Work

**Step 0: Create a To-Do List**

You **MUST** create a to-do list for yourself using the `todo_write` tool.

**Your plan must have these steps:**

1.  **Phase 1: Setup & Get Data** (to-do)
2.  **Phase 2: Read All the Notes** (to-do)
3.  **Phase 3: Write the Overview** (to-do)
4.  **Phase 4: Create Action Items** (to-do)
5.  **Phase 5: Combine with Old Summary** (to-do)
6.  **Phase 6: Make the Checklist** (to-do)
7.  **Phase 7: Give Your Opinion** (to-do)
8.  **Phase 8: Final Check & Tidy Up** (to-do)
9.  **Phase 9: Post the Summary** (to-do)
10. **Phase 10: Write Personal Summary** (to-do)

**IMPORTANT: How to Report Your Progress:**
After you finish **EACH** phase, you **MUST** do these two things before starting the next one:

1.  **Tell the User (THIS IS REQUIRED):**
    You must post a little summary in the chat so the user knows what you're doing. Use this exact format:

    ```text
      **Phase {N} Complete**
      Summary: {A short sentence or two on what you just did.}
      Details: {For example, "Found 5 conversations", "Created 3 action items", etc.}
    ```

2.  **Update Your To-Do List:**
    Use the `todo_write` tool to mark the phase you just finished as `completed` and the next one as `in_progress`.

**Rule:** You are not allowed to skip telling the user what you did. It's important for them to see your progress.
**Rule:** **One Step at a Time.** Don't start the next phase until you've finished and reported on the current one. Don't try to get information for future steps ahead of time.
**Rule:** **Wait for the Green Light.** You must wait to see a success message from your tools before you start thinking about the next phase.

### 4. How to Think: Smart Summarizing

**Your Method: Smart Summarizing**

- :term[02: Company/Process]{href="https://idealic.academy/en/company/02_process.md/"}
- :term[50: Prompt/Truth]{href="https://idealic.academy/en/company/50_prompt_truth.md/"}

**Rule:** The final document must be in the **Target Language** ({language}).

- **Translate These:** All titles, descriptions, your analysis, and action item summaries.
- **Keep These the Same:** Technical terms, file paths, pieces of code, direct quotes, and the word **"Alignment"** (in titles).
- **Word Guide (if the language is Russian):**
  - Proposal: Предложение
  - Intent: Намерение
  - Agreed: Согласовано
  - Done: Готово
  - Rejected: Отклонено
  - Discussion: Обсуждение
  - Clarification: Уточнение
  - Deferred: Отложено
  - Outdated: Устарело

**Getting it right is the most important thing:**

- **Capture the Details:** Don't lose any important technical instructions.
- **One Idea at a Time:** Each "Action Item" should only be about one single, specific change.
- **Cover Everything:** Make sure every comment is connected to an action item or is otherwise accounted for.

### 5. The Step-by-Step Guide to Writing the Document

#### 5.1. Phase 1: Setup & Get Data

1.  **Get the Info:** Download the documents, PR details, and comments.
2.  **Create the File:**
    - Make a new file called `{OUTPUT_DIR}/{FILENAME}.md` using this **EXACT** template (and translate all the text like 'Status' and 'Author' to the **{language}**):

    ```markdown
    # Alignment: {DATE}

    - Status: Draft of Agreed Ideas
    - Author: {PR_AUTHOR}
    - Source: {PR_LINK}
    - Range: {SINCE_DATE} - {NOW}

    {{WARNING_PLACEHOLDER}}

    ## Overview

    {{OVERVIEW_PLACEHOLDER}}

    ## List of Action Items (The Plan)

    {{INTENTS_PLACEHOLDER}}

    {{QUESTIONS_PLACEHOLDER}}

    {{COVERAGE_PLACEHOLDER}}

    {{OPINION_PLACEHOLDER}}

    {{INSTRUCTIONS_PLACEHOLDER}}
    ```

#### 5.2. Phase 2: Read Your Notes

1.  **Get a Sense of the Data:**
    - **Count Conversations:** Run `wc -l {OUTPUT_DIR}/{FILENAME}.ndjson`. Each line represents one conversation thread.
    - **Report Back:** Tell the user how many conversation threads you found.

2.  **Load the Notes:**
    - Read the comments from your `{OUTPUT_DIR}/{FILENAME}.ndjson` file into your memory, about 25 lines at a time.

#### 5.3. Phase 3: Write the Overview

**Thinking About It:**
**IMPORTANT**: Answer these 8 questions very clearly in a bulleted list. This list will become the OVERVIEW.

1.  **What was the reviewer trying to achieve?**
2.  **What did the author agree with?**
3.  **What did the author disagree with?**
4.  **What's the general topic of this discussion?**
5.  **Were there any misunderstandings?**
6.  **Are there any questions that still need answers?**
7.  **Did anyone discover something new or unexpected?**
8.  **Are there other documents related to this?**

**Writing It:**

- Replace the `{{OVERVIEW_PLACEHOLDER}}` with your answers.

#### 5.4. Phase 4: Create Action Items

**Thinking About It:**
Goal: **Break everything down into tiny, specific steps.**

- **The 'AND' Rule:** If a suggestion has 'and' in it, it should probably be two separate action items. For example, "change the color AND move the button" are two different tasks.
- **One at a Time:** One Action Item = One single technical change.
- **Details Matter:** **DO NOT SUMMARIZE TOO MUCH.** Explain the full idea, including the "What" and the "Why." It's better to be too detailed than not detailed enough.
- **Show the Code:** **ALWAYS** include the relevant piece of code (`diff_hunk`) that the comment is about. If there are several:
  - **List them all** if they are short and important.
  - **Shorten them** smartly if they are too long (like `... code ...`), but make sure the important changes are still visible.
- **Figure Out the Category:** Decide what area each action item belongs to:
  - **Specification:** A change to the plan or rules (usually in markdown files).
  - **Code:** A change to how the program works (usually in ts/tsx files).
  - **Tests:** A change to how the code is tested.
  - **Process:** A change to how the team works or to documentation.
- **How to Know the Author's Decision:**
  - **They Said So:** If the author wrote a reply that clearly says yes or no, use that.
  - **They Used an Emoji:** If there's no written reply, look for an emoji reaction on the _reviewer's_ comment (like 👍, 🚀, or 👀). You can assume the author put it there to show they agree or saw it.
  - **Don't Make Things Up:** If there's no reply and no emoji, just mark the status as "Pending/No Response". DO NOT guess what the author was thinking.
- **Status Meanings:**
  - **Done:** The author said they finished it, and nobody has disagreed.
  - **Agreed:** Everyone agrees it's a good idea, but the work hasn't been done yet.
  - **Rejected:** The author explained why they won't do it, and the discussion ended.
  - **Discussion:** People are still actively talking about it; no decision has been made.
  - **Clarification:** Someone needs more information before a decision can be made.
  - **Deferred:** It's a good idea, but it will be done later, not right now.
  - **Outdated:** This was an action item from an old summary that is no longer relevant.
- **Understanding the Context:**
  - **Markdown Files (`*.md`):** Comments on these files are usually about the **Plan** or **Specification**. (For example, "Remove this feature" means "Update the plan to say we're removing the feature," not necessarily "delete the code right this second.")
  - **Code Files:** Comments on these are about the actual code.
  - **Be Clear:** If it's not obvious whether it's about the plan or the code, say so in the `Reasoning` or `Intent` section.

**Making a Draft (Required):**

1.  **Draft in the Chat:** Based on your analysis, write a full list of new "Fresh Action Items" directly into the chat.
2.  **Use the Template:** Make sure each one follows the standard format for an action item.
3.  **Give it a Title:** Start the list with the title `### Fresh Draft Intents`.
4.  **Wait!** Don't write this to the main summary file yet.

#### 5.5. Phase 5: Combine with the Old Summary

**Goal:** Blend the "Fresh Action Items" you just created with any "Previous Summary" that might already exist.

1.  **Check the Settings:**
    - If `merge_alignments` is **false**: Skip the next two steps. Just use your "Fresh Draft Action Items" and move on.

2.  **Get the Previous Summary:**
    - Run this command to find the **ID** and the **Text** of the last summary comment (and show it):
      ```bash
      gh api "repos/{OWNER}/{REPO}/issues/{PR_NUMBER}/comments" \
      --jq 'map(select(.body | contains("# Alignment"))) | sort_by(.created_at) | last | {id: .id, body: .body}'
      ```
    - **Next:** Read what the command printed out.
    - **Save It:** Keep the text (`.body`) as your "Baseline" to merge with.
    - **REMEMBER THIS:** In your Phase 5 progress report, you must mention the `id` you found (e.g., "Found existing Alignment Comment ID: 12345"). You will need this number later in Phase 9.

3.  **How to Merge:**
    - **Your Ingredients:** Use your "Fresh Draft Action Items" (from Phase 4) and the "Baseline" summary (from Step 2).
    - **Find Matches:** Compare each fresh action item to the ones in the baseline summary.
    - **If it's a Match:** Keep the **original number** (`#{N}`). **UPDATE** everything else (Title, Details, Status, code sample) with the newest information. Don't keep old text if the new information is better.
    - **If it's New:** If a fresh action item is completely new, give it a **new number** (one higher than the highest number from the baseline).
    - **If it's Old:** If an action item from the baseline summary isn't in your fresh list (meaning no one talked about it this time), **keep it exactly as it was**. Don't change its status or title.

4.  **Write the Final List:**
    - **The Logic:**
      - **If a previous summary exists AND you're supposed to merge:** Combine the "Fresh Draft Action Items" with the "Baseline."
      - **Otherwise:** Just use the "Fresh Draft Action Items" by themselves.
    - **Action:** Replace `{{INTENTS_PLACEHOLDER}}` with the **Final List of Action Items**.
    - **Action:** Replace `{{QUESTIONS_PLACEHOLDER}}` with any open questions.
    - **The Template for Each Item:**

````markdown
### {N}. {Short Title}

- **Category:** {Logic / Design / ...}
- **Context:** {Plan / Code / Tests / ...}
- **Action Item:** {A detailed sentence about what needs to be done.}
- **Author's Original Idea:** {How it was first built.}
- **Reviewer's Suggestion:** {What the reviewer recommended.}
- **Author's Decision:** {What the author said, or what emoji they used. If nothing: "Pending"}
- **Reviewer's Reaction:** {Any follow-up comments.}
- **Reasoning:** {Why this change is being made.}
- **Status:** {Agreed / Done / Rejected / Discussion / Clarification / Deferred / Outdated} {Add ✅ if the conversation is resolved/closed}
- **Result:** {A brief note on what happened. Did the plan change? Did everyone agree?}

> [{Reviewer Name}]({anchor}): "{A short summary of their point}"
>
> [{Author Name}]({anchor}): "{A short summary of their response}"

_(If there's a code sample, put it here. If NOT, leave this whole code block out.)_

```{lang}
{The relevant piece of code. Include all the important parts.}
```
````

    - **Separation:** If you added any brand new action items, put a line `---` before them to show they are new.

    **Progress Report Rule:**
    In your progress report for this phase, you **MUST** tell the user:
    - How many **New** action items you created.
    - How many **Existing** action items you updated.
    - How many **Old** action items you kept without changing.

#### 5.6. Phase 6: Make the Checklist (Coverage Report)

**Thinking About It:**

- **Check Your Work:** Read through your **Final List** of action items. Compare it against the original comments to make sure you didn't miss anything.
- **Map Every Comment:** Make sure every single comment index (like `§1`) is accounted for.

**Writing It:**

- Replace `{{COVERAGE_PLACEHOLDER}}`.
- **IMPORTANT:** Don't create this table piece by piece. You must generate the **entire** table all at once after you have everything you need.
- **The Template:**

  Create one **SINGLE** table. Group the comments by the day they were posted. Put a special "Header Row" for each new date. Start with this title:

  ```markdown
  ---

  ## Coverage Report

  | {Index}           | {Date/Time}    | {User} | {Title (Summary)} | {Action Item}  | {Reaction} |
  | ----------------- | -------------- | ------ | ----------------- | --------- | ---------- |
  |                   | **{Date}**     |        |                   |           |            |
  | [{Idx}]({anchor}) | {dd.MM HH:mm}  | {User} | {A 4-6 word summary} | #{N}      | {Emojis}   |
  | [{Idx}]({anchor}) | {dd.MM HH:mm}  | {User} | └ {A 4-6 word summary} | #{M},#{N} | {Emojis}   |
  |                   | **{NextDate}** |        |                   |           |            |
  | [{Idx}]({anchor}) | {dd.MM HH:mm}  | {User} | {A 4-6 word summary} | -         | {Emojis}   |
  | [{Idx}]({anchor}) | {dd.MM HH:mm}  | {User} | ├ {A 4-6 word summary} | #{N}      | {Emojis}   |
  | [{Idx}]({anchor}) | {dd.MM HH:mm}  | {User} | └ {A 4-6 word summary} | #{N}      | {Emojis}   |
  | [{Idx}]({anchor}) | {dd.MM HH:mm}  | {User} | {A 4-6 word summary} | -         | {Emojis}   |
  ```

  **Rules for the Table:**
  1.  Use the `index` (like `§1`) from your notes file.
  2.  **One Row Per Comment:** Every single comment gets its own row. Don't group them like `§1-§5`.
  3.  **Show Replies:** Use the `├` and `└` symbols to show which comments are replies to others.
  4.  **Show What's Resolved:** If a conversation thread is marked as resolved, you **MUST** put a `✅ ` at the beginning of the **Title (Summary)**. (e.g., `✅ Fix typo...`).
  5.  **How to Choose a Reaction Emoji:**
      - **Level 1 (Real Emojis):** If people reacted with emojis on GitHub (like `+1`, `heart`, `rocket`), use those.
      - **Level 2 (Guess from Text):** If there are no real emojis, figure out the feeling of the comment and use one of these:
        - `❓` (It's a question)
        - `⚠️` (It's a warning)
        - `🚧` (It's about work in progress)
        - `🗑️` (Something was rejected)
        - `💡` (It's a new idea)
        - `🤝` (People are agreeing)
        - `🗣️` (It's a normal reply)
        - `✅` (Someone confirmed something is done)
        - `👍` (Someone is approving)
  6.  **Sorting:** You **MUST** keep the comments in order by their index number (§1, §2, etc.). Your notes file is already sorted this way, so don't change the order.
  7.  **Grouping by Date:** Put a Date Header `| | **{Date}** | | | | |` only when a **new conversation thread starts on a new day**.
      - **Rule:** **NEVER** put a date header in the middle of a conversation. Even if a reply is posted days later, it stays grouped with the original comment.
      - **How it looks:** `[Date Header] -> [First Comment of Thread 1] -> [Replies to Thread 1] -> [First Comment of Thread 2]...`
      - **Example:**
        ```
        | | **Dec 4th** | | ...
        | §1 | ... | Inviz | The first comment | ...
        | §2 | ... | User2 | └ A reply (from Dec 8th) | ... (NO NEW DATE HEADER HERE)
        | §3 | ... | Inviz | The start of a new thread | ...
        ```

#### 5.7. Phase 7: Give Your Opinion

**Writing it:**

- Replace `{{OPINION_PLACEHOLDER}}`.
- **What to Write About:**
  1.  **Teamwork:** How well did people work together? Was the feedback helpful?
  2.  **Review Quality:** Was the review easy to understand? Were ideas broken down into small pieces?
  3.  **Potential Problems:** Point out any conversations that seemed confusing or might cause problems later.
  4.  **Biggest Roadblocks:** If anything is stopping this from being finished, list the top 1 or 2 issues.
  5.  **What We Learned:** What's a good (or bad) pattern from this discussion that we can learn from?
  6.  **Extra Thoughts (Optional):** If you have any other really useful ideas, you can add up to 10 more points. If not, **SKIP** this part.

#### 5.8. Phase 8: Final Check & Tidy Up

1.  **Think It Over:** Read the whole document one last time to make sure it's complete and makes sense.
2.  **Fix the Links:**
    - **The Problem:** While you were writing, you used short IDs for links (like `[Index](12345678)`) to save space. Now you need to turn them into real, clickable web links.
    - **The Fix:** Change things like `(ID)` into `(https://github.com/...#issuecomment-ID)`.
    - **How to do it:** Run this command to fix all the links at once.
    - **Command:**
      ```bash
      sed -E 's~]\((discussion_r[0-9]+|issuecomment-[0-9]+|pullrequestreview-[0-9]+)\)~](https://github.com/{OWNER}/{REPO}/pull/{PR_NUMBER}#\1)~g' {OUTPUT_DIR}/{FILENAME}.md > {OUTPUT_DIR}/{FILENAME}.tmp && mv {OUTPUT_DIR}/{FILENAME}.tmp {OUTPUT_DIR}/{FILENAME}.md
      ```
3.  **Read the File:** Check to make sure all the `{{PLACEHOLDERS}}` are gone.
4.  **Clean Up:** Delete your temporary notes file: `{OUTPUT_DIR}/{FILENAME}.ndjson`.

#### 5.9. Phase 9: Post the Summary (Optional)

1.  **Check the Settings:**
    - If `auto_post` is **false**: Mark this phase as complete and you're **DONE**.

2.  **Get the Content:**
    - Read the final text from your `{OUTPUT_DIR}/{FILENAME}.md` file.

3.  **Post or Update:**
    - **Remember?** Look back at your Phase 5 progress report. Did you find an ID for an old summary comment?
    - **If YES, Update It:** If you found an ID, use this command to edit the existing comment (replace `{ID}` with the real number):
      ```bash
      jq -n --rawfile content {OUTPUT_DIR}/{FILENAME}.md '{body: $content}' | \
      gh api -X PATCH "repos/{OWNER}/{REPO}/issues/comments/{ID}" --input -
      ```
    - **If NO, Post a New One:** If you didn't find an ID, use this command to post a brand new comment:
      ```bash
      jq -n --rawfile content {OUTPUT_DIR}/{FILENAME}.md '{body: $content}' | \
      gh api -X POST "repos/{OWNER}/{REPO}/issues/{PR_NUMBER}/comments" --input -
      ```

4.  **Final Touches (Maybe):**
    - If `include_instructions` is **true**:
      - Replace `{{WARNING_PLACEHOLDER}}` with this message:
        ```markdown
        > [!WARNING] COMPUTER-GENERATED: DO NOT EDIT THIS MANUALLY, DO NOT SAVE TO GIT
        > This document is a tool to keep everyone on the same page. It does three things:
        >
        > 1. **It Checks:** It makes sure 100% of comments were heard.
        > 2. **It Summarizes:** It turns a long discussion into a clear plan.
        > 3. **It Instructs:** It tells the next person (or AI) what to do.
        >
        > If you think this summary is wrong, **do not edit it**. Instead, add another comment to the Pull Request and ask for a **new summary**.
        >
        > **DO NOT ADD THIS FILE TO THE PROJECT'S CODE.**
        >
        > - More info: [22: Alignment](https://idealic.academy/raw/en/company/22_document_alignment.md)
        > - Only these files should be saved in Git: [Proposal](https://idealic.academy/raw/en/company/21_document_proposal.md) and [Specification](https://idealic.academy/raw/en/company/20_document_spec.md).
        ```
      - Replace `{{INSTRUCTIONS_PLACEHOLDER}}` with this:

        ```markdown
        ## Instructions for the AI Agent (Next Step)

        > [!IMPORTANT] Read These First
        > Before you make any changes to the project's plan, you **MUST** read these documents to make sure you understand all the rules:
        >
        > 1. `curl https://idealic.academy/raw/en/company/02_process.md` (The Process)
        > 2. `curl https://idealic.academy/raw/en/company/50_prompt_truth.md` (The Truth)
        > 3. `curl https://idealic.academy/raw/en/company/20_document_spec.md` (The Specification)
        > 4. `curl https://idealic.academy/raw/en/company/21_document_proposal.md` (The Proposal)
        > 5. `curl https://idealic.academy/raw/en/company/22_document_alignment.md` (Alignment)
        ```

    - If `include_instructions` is **false**:
      - Just delete `{{WARNING_PLACEHOLDER}}` and `{{INSTRUCTIONS_PLACEHOLDER}}` completely.

5.  **Clean Up:** Nothing to clean up here.

#### 5.10. Phase 10: Write a Personal Summary (Optional)

**Goal:** Post a separate, friendly summary of new comments for one specific person.

> [!IMPORTANT] DON'T GET THE DATA AGAIN
> You **already have all the comments** saved in `{OUTPUT_DIR}/{FILENAME}.ndjson`. You MUST use that file.
> **DO NOT** go back to the internet to get the comments again.

1.  **Check the Settings:**
    - If `post_summary_viewer` is **empty**: Mark this phase as complete and you're **DONE**.

2.  **Find the Starting Point:**
    - **Search:** Look for the last comment that looks like this: `# Review round {N} by {person's_username}`.
      ```bash
      gh api "repos/{OWNER}/{REPO}/issues/{PR_NUMBER}/comments" | \
      jq --arg user "{post_summary_viewer}" 'map(select(.body | test("# Review round [0-9]+ by " + $user))) | sort_by(.created_at) | last | {created_at, body}'
      ```
    - **The Logic:**
      - If you **Found One**:
        - The `Watermark` (your starting point) is when that comment was created.
        - The `LastRound` is the number `{N}` you found.
        - The `NextRound` is `LastRound + 1`.
      - If you **Didn't Find One**:
        - The `Watermark` is the very beginning of today.
          - `date -u +%Y-%m-%dT00:00:00Z`
        - The `NextRound` is 1.

3.  **Filter for New Comments (In Your Head):**
    - **Source:** Use the comment data you **already have in your memory** from Phase 2.
    - **Rule:** **DO NOT** read the file again. Use what you already know.
    - **Logic:** Look through the comments you remember for any that are:
      - **From:** `{post_summary_viewer}`
      - **Posted:** After the `{WATERMARK}` time.
    - **Next:** These are the comments you will include in the summary.

4.  **Write the Summary:**
    - **Your Instructions:**
      "Analyze the new comments from @{post_summary_viewer}.
      1.  **Language (VERY IMPORTANT):** The summary MUST be in the **{language}** the user asked for. Translate everything.
      2.  **Tone:** Be friendly and encouraging, but also detailed.
      3.  **Group Them:** Organize the comments into groups like "Critical," "New ideas," or "Small Fixes."
      4.  **Explain:** For each point, explain the main _Idea_, how it should be _Implemented_, and _Why_ it's a good change.
      5.  **Add a Link Symbol:** Put the '§' symbol at the end of each title.
      6.  **Stats:** At the very beginning, mention: '{N} comments posted since {WATERMARK}'."
    - **The Template:**

      ```markdown
      # Review round {NextRound} by {post_summary_viewer}

      {A friendly opening - like "Great progress! I've posted {N} comments since {Date}. Here's a summary:"}

      ## {Group Title (e.g., Critical Fixes)}

      ### {Title} [§]({url})

      {Explain the main idea here.}

      {Include code or other details if you need to.}

      _{Reasoning}: {Explain why this is a good idea}_

      ...

      {A friendly closing - like asking for feedback or what to do next.}
      ```

5.  **Post the Comment:**
    - **Action:** Post this summary as a **NEW** comment on the Pull Request.
      ```bash
      jq -n --arg body "{GENERATED_BODY}" '{body: $body}' | \
      gh api -X POST "repos/{OWNER}/{REPO}/issues/{PR_NUMBER}/comments" --input -
      ```

### 6. Final Report

Give a final summary of what you did, the changes you made, and any thoughts you have.
