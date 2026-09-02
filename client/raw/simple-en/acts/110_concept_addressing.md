# 110: How We Name and Find Ideas

> [!DEFINITION] [idea:](./000_glossary.md)
> Think of `idea:` as a special kind of web address. It's a system for finding an :term[Idea]{canonical="Idea"} in a world where there are many different versions and drafts. You can use it to make a simple request, like "get me the latest version," or to create a permanent link that always points to one exact version.

> Sidenote:
> - You'll want to understand these first:
>   - :term[Visibility]{href="./108_concept_visibility.md"} (How things are found and seen)
>   - :term[Meta]{href="./016_agent_meta.md"} (Data about data)

The `idea:` address system is the language we use to get around in the world of branching, versioned :term[Ideas]{canonical="Idea" href="./101_concept_idea.md"}. It lets you ask for :term[Ideas]{canonical="Idea" href="./101_concept_idea.md"} in a flexible way and also create permanent links that will always work, so you can always find the exact same thing again.

> Sidenote:
> - This address system works by navigating through :term[Branches]{canonical="Branch" href="./108_concept_visibility.md#branches-publication-and-partitioning"} (drafts) and the :term[Search Path]{canonical="Search Path" href="./108_concept_visibility.md#the-search-path-prioritized-retrieval"} (the order you look through drafts). You can read more about these in the chapter on :term[Visibility]{href="./108_concept_visibility.md"}.

## The Parts of an `idea:` Address

An `idea:` address is like a detailed mailing address, but for digital concepts. Here are the parts it can have:

- **Scheme**: Always starts with `idea:` to show what kind of address it is.
- **Domain**: This is like the city and state, such as `//my-project.com`. It tells you which big collection of ideas to look in. If you leave it out, it assumes you're looking within your current collection.
- **Branch Specifier**: This is like asking to look in a specific department, like `~main/`. The `main` branch is usually the official, finished version. A `/` at the beginning is a shortcut for `~main/`.
- **Path**: The name of the :term[Idea]{canonical="Idea" href="./101_concept_idea.md"}, like the street address (e.g., `schemas/UserProfile`).
- **Version Prefix**: A way to ask for a version without knowing the exact number, like saying "find me something that starts with version 1.2" (e.g., `?1.2`).
- **Resolved Version**: This is the exact, final version number that was found, like the apartment number (e.g., `:1.2.3`). It makes the link permanent.
- **Fragment**: Points to a specific part inside the :term[Idea]{canonical="Idea" href="./101_concept_idea.md"}, like pointing to a specific chapter in a book (e.g., `#schema`).

## Different Kinds of Addresses

An `idea:` address can be an "ask" or an "answer."

1.  **A Simple Ask (Relative Request):** `idea:schemas/UserProfile?1.2`
    - This means: "In my current project, find the latest version of `UserProfile` that matches version `1.2`."

2.  **A Specific Ask (Absolute Request):** `idea://my-project.com/schemas/UserProfile?1.2`
    - This is more specific: "Go to the `my-project.com` collection and find the version of `UserProfile` that matches `1.2`. Ignore my current project."

3.  **An Ask for a Specific Draft (Explicit Branch Request):** `idea:~main/schemas/UserProfile` (or `idea:/schemas/UserProfile`)
    - This says: "Look *only* in the `main` draft (or branch) for `schemas/UserProfile`. Don't check any other drafts first."

4.  **The Final Answer (Resolved Reference):** `idea://my-project.com/:staging/schemas/UserProfile?1.2:1.2.staging.4`
    - This is a permanent record. It says: "I looked in the `my-project.com` collection, specifically in the `staging` draft, and I found the exact version `1.2.staging.4` which matched the request for `?1.2`."

## How It Works in a Real Project

This system makes it easy for many people to work on the same project without getting in each other's way.

1.  **Make a New Draft**: A programmer starts a new task by creating their own draft of the project, called a :term[branch]{canonical="Branch"}, named `feature/user-onboarding-v2`. When they look for things, the system will check their draft first, and then the official `main` version.

2.  **Build Something New**: They create a new :term[Idea]{canonical="Idea" href="./101_concept_idea.md"} for a new user sign-up process: `idea:processes/onboarding`. Because they are working in their `feature/user-onboarding-v2` draft, this new :term[Idea]{canonical="Idea" href="./101_concept_idea.md"} is saved there automatically. They can change it as much as they want without affecting the main project.

3.  **Use an Existing Part**: Their new process needs to send an email. They add a reference to a pre-existing part: `idea:activities/send-email`. The system first looks in their personal draft. It's not there, so it looks in the `main` draft and finds the official, stable version to use.

4.  **Make a Custom Version of a Part**: The programmer needs a special "Welcome Email" template. They make a new version of `idea:records/email-templates/welcome`. This new version is saved only in their `feature/user-onboarding-v2` draft. Now, their new sign-up process will automatically use their custom email template, while the rest of the project continues to use the old one from the `main` draft.

5.  **Merging the Changes**: When they're finished, the programmer merges their work. Their new version of the sign-up process is copied from their draft to the `main` :term[branch]{canonical="Branch"}. It gets a new official version number (like `1.4`) and becomes the new standard for everyone.

6.  **Controlled Updates**: Imagine another process was using the old sign-up process by asking for `idea:processes/onboarding?1`. Since the new version `1.4` still matches this request, that process will automatically start using the new, updated version. If a developer wants to prevent this and stick with an older version for stability, they can "pin" it by asking for a more specific version, like `?1.3`.