# 110: Concept/Addressing

> [!DEFINITION] [idea:](./000_glossary.md)
> A URI scheme for navigating the versioned, branched reality of the :term[Idea]{canonical="Idea"} ecosystem. It provides syntax for both simple, dynamic requests and fully resolved, permanent references.

> Sidenote:
>
> - Requires:
>   - :term[Visibility]{href="./108_concept_visibility.md"}
>   - :term[Meta]{href="./016_agent_meta.md"}

The :term[idea:]{canonical="idea:"} URI scheme provides the language for navigating the versioned, branched reality described in the previous document. It offers a syntax for requesting :term[Ideas]{canonical="Idea" href="./101_concept_idea.md"} and for creating permanent, reproducible links to specific, resolved versions.

> Sidenote:
>
> - The concepts of :term[Branches]{canonical="Branch" href="./108_concept_visibility.md#branches-publication-and-partitioning"} and the :term[Search Path]{canonical="Search Path" href="./108_concept_visibility.md#the-search-path-prioritized-retrieval"}, which this URI scheme navigates, are detailed in the :term[Visibility]{href="./108_concept_visibility.md"} chapter.

## Anatomy of an `idea:` Reference

An :term[idea:]{canonical="idea:"} reference can include several components:

- **Scheme**: :term[idea:]{canonical="idea:"}
- **Domain**: An optional sovereign namespace, prefixed with `//` (e.g., `//my-project.com`). If omitted, the path is considered relative to the current namespace.
- **Branch Specifier**: An optional, explicit :term[branch]{canonical="Branch"} target (e.g., `~main/`). A leading `/` is shorthand for `~main/`.
- **Path**: The textual identifier of the :term[Idea]{canonical="Idea" href="./101_concept_idea.md"} (e.g., `schemas/UserProfile`).
- **Version Prefix**: An optional version constraint (e.g., `?1.2`).
- **Resolved Version**: In a resolved reference, the exact version found (e.g., `:1.2.3`).
- **Fragment**: A pointer to a specific part of the :term[Idea]{canonical="Idea" href="./101_concept_idea.md"} (e.g., `#schema`).

## Reference States

An :term[idea:]{canonical="idea:"} reference can exist in several states:

1.  **Relative Request (The "Ask"):** `idea:schemas/UserProfile?1.2`
    - This asks for the latest version of `UserProfile` compatible with the `1.2` prefix, resolved against the :term[Search Path]{canonical="Search Path" href="./108_concept_visibility.md#the-search-path-prioritized-retrieval"} within the **current sovereign namespace**.

2.  **Absolute Request:** `idea://my-project.com/schemas/UserProfile?1.2`
    - This is a fully-qualified request that targets a specific domain, ignoring the current namespace.

3.  **Explicit Branch Request:** `idea:~main/schemas/UserProfile` (or `idea:/schemas/UserProfile`)
    - The `~` prefix indicates an explicit :term[Branch]{canonical="Branch"} request. This ignores the :term[Search Path]{canonical="Search Path" href="./108_concept_visibility.md#the-search-path-prioritized-retrieval"} and targets the `main` :term[branch]{canonical="Branch"} directly, within the current (or specified) namespace.

4.  **Resolved Reference (The "Answer"):** `idea://my-project.com/:staging/schemas/UserProfile?1.2:1.2.staging.4`
    - This is a permanent, unambiguous record of a resolution. It shows the request was resolved within the `my-project.com` domain, on the `staging` :term[branch]{canonical="Branch"}, and found the exact version `1.2.staging.4`.

## A Complete Development Workflow

These concepts enable a safe and efficient workflow for parallel development.

1.  **Create a Branch**: A developer starts by creating a new :term[branch]{canonical="Branch"}, `feature/user-onboarding-v2`. Their :term[Search Path]{canonical="Search Path" href="./108_concept_visibility.md#the-search-path-prioritized-retrieval"} is set to `['feature/user-onboarding-v2', 'main']`.

2.  **Develop a Process**: The developer creates a new `Process` :term[Idea]{canonical="Idea" href="./101_concept_idea.md"}, `idea:processes/onboarding`. Because they are on the `feature/user-onboarding-v2` :term[branch]{canonical="Branch"}, this new :term[Idea]{canonical="Idea" href="./101_concept_idea.md"} and all its subsequent versions are automatically associated with that :term[branch]{canonical="Branch"}. They can freely refine and test the process, creating `Instruction` and `Record` :term[Ideas]{canonical="Idea" href="./101_concept_idea.md"} as needed.

3.  **Use Existing Components**: The new `onboarding` process needs a standard "Send Email" `Instruction`. The developer references it as `idea:activities/send-email`. The resolution system first checks the `feature/user-onboarding-v2` :term[branch]{canonical="Branch"}. Since it doesn't exist there, it falls back and finds the stable version on `main`.

4.  **Override a Component**: The developer realizes they need a custom "Welcome Email" template. They create a new version of `idea:records/email-templates/welcome` and save it. This new version is saved to the `feature/user-onboarding-v2` :term[branch]{canonical="Branch"}. Now, when their process references this :term[Idea]{canonical="Idea" href="./101_concept_idea.md"}, the resolution system finds their new version first and uses it, leaving the original on `main` untouched.

5.  **Promoting**: Once complete, the developer "promotes" their work. The :term[Idea]{canonical="Idea" href="./101_concept_idea.md"} at version `1.3.new-login.2` is copied, its version is cleaned to `1.4`, and it is now also published to the `main` :term[branch]{canonical="Branch"}. It becomes the new, stable version for everyone.

6.  **The Controlled Ripple Effect**: Another process was referencing `idea:processes/onboarding?1`. Because the new version `1.4` is compatible with the `?1` prefix, the next time that process is resolved, it will automatically and safely pick up the new `1.4` version. To prevent this and ensure stability, a process can "pin" its dependency by requesting a more specific version, like `?1.3`.
