# Chapter 2: Parallel Realities - Versioning & Branching

## New Ideas in This Chapter

- **Hierarchical Versions**: Instead of simple version numbers, versions are hierarchical identifiers (e.g., `1.2.feature-x.3`). This unifies the concept of a version with the concept of a branch or draft into a single, powerful structure.
- **A Duality of Branches**: Two types of branches work together: **Branches** (like `main` or `staging`) define _where_ a Vibe is visible, while **Branched Revisions** (like `.feature-x.`) within versions create alternative lineages that can be used for development, configuration overlays, or system extensions.
- **The Search Path as an Overlay Engine**: Resolution is governed by a prioritized **search path** (e.g., `['feature-x', 'main']`). This list acts as an overlay mechanism, enabling developers to see their branched work seamlessly layered on top of the stable system.
- **Branch-Aware URIs**: The `aug:` reference scheme navigates this multi-branch reality, allowing for requests that are either implicit (respecting the search path) or explicit (targeting a specific branch).
- **The Controlled Ripple Effect**: Promoting a branched version with breaking changes can safely and automatically update dependencies for consumers who have opted into that compatibility band, making evolution predictable.

---

## The Challenge: Managing Change and Parallelism

Any evolving system faces two fundamental challenges: managing change (versioning) and enabling parallel work (branching). The system addresses both through a unified model where these two concepts are deeply intertwined. This chapter explains the mechanics of these "parallel realities," showing how versions, branches, and a prioritized resolver work together to create a safe, flexible, and powerful environment for development and experimentation.

> **Alice:** "So we're tackling two big problems at once here - how things change over time and how different people can work on things at the same time?"
> **Bob:** "Exactly. And instead of treating them as separate issues, we've created a unified approach where versioning and branching work together seamlessly."
> **Alice:** "That sounds powerful. How does it actually work?"
> **Bob:** "Let's break it down piece by piece, starting with how we identify Vibes in this system."

## Part 1: The Anatomy of Identity - Versions

A Vibe's identity consists of two components: its **`path`** (a textual identifier, like `articles/common/button`) and its **`version`** (a hierarchical identifier). A version isn't just a simple number - it's a rich, dot-separated hierarchy that tells the story of a Vibe's evolution. This elegant structure seamlessly combines traditional versioning with branching and drafting capabilities.

In its simplest form, a version might just be a single number like `1` or `2`. But the real power comes from its hierarchical nature. For example, a version like `1.2.feature-x.3` tells us this is the third revision of a feature branch that was created from version 1.2.

```llm
A Vibe's identity is its `path` and its `version`. The version is a hierarchical
identifier like `1.2.feature-x.3` that captures the Vibe's lineage and state.
```

> **Alice:** "So instead of just 'version 5', a Vibe could be 'version 1.4.my-hotfix.2'?"
> **Bob:** "Exactly. And just by looking at that version, you know it's the second revision of the 'my-hotfix' branch, which was forked from version 1.4 of the public Vibe."

## Part 2: The Anatomy of Visibility - Branches & Search Paths

While the Version represents the state and history of a Vibe, the branching model determines where and how it can be seen. This operates through two complementary concepts: Branches and the Search Path.

### Branches: The "Where"

Every Vibe version in the database is associated with one or more **Branches** (e.g., `["main", "feature/new-billing"]`). These branches act as channels that control visibility. A Vibe is only visible to a user or process if it is published to at least one of the branches included in their request's search path.

### The Search Path: The "How"

The system's real power comes from the **Search Path**. This is an ordered list of branch names that tells the resolver where to look for a Vibe, and in what order of priority. A typical search path during development might be:

`['feature/my-new-idea', 'staging', 'main']`

This configuration creates a cascading overlay system:

1.  First, look for a Vibe published to the `feature/my-new-idea` branch.
2.  If not found, look for one published to the `staging` branch.
3.  Finally, fall back to a Vibe published to the `main` branch.

This allows a developer to see a complete, functional environment composed of their specific changes layered on top of the shared, stable system, without needing to duplicate every Vibe.

```llm
Visibility is controlled by branches, which define where a Vibe is visible.
The Search Path (`['feature-x', 'main']`) is a prioritized list that tells
the resolver where to look and in what order. This creates a seamless overlay,
showing branched work on top of the stable system.
```

> **Alice:** "So if my search path is `['my-feature', 'main']`, and I ask for `common/button`, it will first look for a version of `common/button` published to my feature branch?"
> **Bob:** "Exactly. And if you haven't touched it and no version is published there, it will fall back to find the standard one published to `main`."
> **Alice:** "So I get my local changes, plus the stable base from `main`, all seamlessly."
> **Bob:** "You got it. That's the overlay effect in action."

## Part 3: The Advanced Version Structure

Now that we understand the basics of versions and branches, let's explore the version structure in more detail.

### Version Components

A version consists of revisions, which can be:

- **Integer revisions**: Used for sequential, public versions (like `1` or `2.3`)
- **Branched revisions**: Named segments used for branches and drafts (like `feature-x` in `1.2.feature-x.3`)

> **Alice:** "So we have regular numbered revisions for the main line, and these named branched revisions for feature work?"
> **Bob:** "Right. The integer revisions give us a clean, sequential numbering for public versions, while the named segments let us create branches for specific features or experiments."
> **Alice:** "And they work together in the same version structure? That's elegant."
> **Bob:** "It is! It gives us a unified way to express both linear progress and parallel development."

### Evolution Patterns

Versions evolve in several ways:

- **Compatible Changes**: A non-breaking edit typically creates a new minor revision.

  - Editing version `1.2` might create `1.2.1`, `1.2.2`, etc.

- **Incompatible Changes**: Breaking changes affect compatibility at different levels of the version hierarchy.

  - Within a single prefix (e.g., `1.2.x`), different revisions may be incompatible with each other, but they should remain compatible with their parent (`1.2`).
  - If a change breaks compatibility with the parent prefix, it must "bubble up" to a higher level. For example, a change that breaks compatibility with `1.2` would create a new version `1.3`.
  - In cases of significant structural changes, a version might break compatibility with multiple levels, potentially bubbling all the way up to a major version change (e.g., `1.x` to `2.0`).
  - The system determines how far up the change needs to bubble based on which prefix levels are affected by the incompatibility.
  - **Automatic Detection**: The system automatically detects incompatibilities by analyzing schema changes between versions of the vibe. When an incompatibility is found, the system determines the appropriate version level to increment, ensuring that semantic versioning principles are maintained without requiring manual decision-making.

  > **Alice:** "So I don't need to figure out whether my change breaks compatibility with version 1.2 or all of version 1? The system does that for me?"
  > **Bob:** "Exactly. The system analyzes your changes and automatically determines whether you need a minor bump like 1.2.3 to 1.2.4, a more significant bump like 1.2 to 1.3, or even a major version change from 1.x to 2.0."
  > **Alice:** "That sounds much safer than having developers guess about compatibility impacts."
  > **Bob:** "It is. It prevents both accidental breaking changes and unnecessary version inflation. The system ensures that version numbers accurately reflect the true compatibility boundaries."

- **Creating Branched Revisions**: When creating a branch in the version structure, two revision segments are added at once: a named branch segment followed by a numeric segment.
  - When a developer forks version `1.2` to work on a feature, it creates a version like `1.2.feature-x.0`.
  - The branch segment (`feature-x`) identifies the development lineage, while the numeric segment (`0`) allows for sequential versioning within that branch.

> **Alice:** "So when I branch, I'm not just adding a name but also starting a new numbering sequence within that branch?"
> **Bob:** "Exactly. The `1.2.feature-x.0` shows you're branching from version 1.2, and the .0 gives you a starting point for your own sequence of revisions in that branch."
> **Alice:** "And as I make changes in my branch, I'd get versions like `1.2.feature-x.1`, `1.2.feature-x.2`, and so on?"
> **Bob:** "Precisely. Your work evolves with its own numbering, while still clearly showing its relationship to the main version it branched from."

It's important to note that branches (like `main` or `feature/new-login`) are separate from branched revisions in versions (like `feature-x` in `1.2.feature-x.0`). Branches act as publishing channels that control where a Vibe is visible, while branched revisions within versions define lineage and development history. The search path coordinates both of these concepts - it determines which branches to look in and also which branched revisions to prefer when multiple versions are available.

### Branch Visibility and Version Selection

The search path creates a sophisticated resolution system that considers both branches and branched revisions while respecting version requirements:

1. **Branch Visibility**: A Vibe must be published to at least one branch in the search path to be visible at all.

2. **Version Requirements**: Any version requirements in the reference (e.g., `?1.2`) must always be satisfied, regardless of branch priority.

3. **Selection Criteria**: Within the set of Vibes that satisfy the version requirements, the system prioritizes based on branch position in the search path:
   - **Branch Priority**: Vibes published to branches earlier in the search path are strongly prioritized - to the point that the system will select a matching Vibe from a higher-priority branch even if more recent versions exist on lower-priority branches
   - **Branched Revision Visibility**: Branched revisions in versions (like `feature-x` in `1.2.feature-x.3`) create alternative lineages that are visible when their names match entries in the search path, but they still must fall within the version prefix specified in the reference

> **Alice:** "This branch priority sounds really important. So if I have `['feature-x', 'main']` as my search path, it will always prefer something from my feature branch even if there's a newer version in main?"
> **Bob:** "Yes, as long as it meets any version requirements you've specified. That's what makes the overlay system work - your changes take precedence in your view."
> **Alice:** "And the branched revision part just makes things visible, but doesn't override the priority?"
> **Bob:** "Exactly. Having a matching name in the version might make something visible, but it's the branch it's published to that determines priority in the search path. The branched revision creates an alternative lineage that can be used for all sorts of purposes."
> **Alice:** "So these alternative lineages aren't just for development work?"
> **Bob:** "Not at all. They could be for overlaying different configurations, extending the system in various ways, or any situation where you need parallel versions of the same content."

Let's explore some examples with a search path of `['feature-x', 'main']` when looking for `common/button`:

- **Scenario 1**: If `common/button` at version `1.3` is published to the `feature-x` branch, it will be selected over any version on the `main` branch (as long as it's within the version prefix specified in the reference).

- **Scenario 2**: If `common/button` is not published to the `feature-x` branch at all, the system will select a compatible version from the `main` branch.

- **Scenario 3**: If `common/button` at version `1.2.feature-x.3` is published to the `main` branch (note the branched revision matches a name in the search path), this version is visible because the branched revision creates an alternative lineage that matches an entry in the search path. However, it will only be selected if it falls within the version prefix specified in the reference.

This approach enables creating alternative lineages for various purposes. It can be used for development work, creating safe spaces for editing and drafting, overlaying different configurations, or extending the system in parallel ways. The search path mechanism ensures users see their preferred lineages seamlessly integrated with the rest of the system.

```question
How are breaking changes represented in a Version?
* [x] By incrementing a major revision number (e.g., a breaking change to `1.2.5` results in `1.3`).
* [ ] By adding a `!` symbol to the end of the version.
* [ ] By creating a completely new root version (e.g., `1.2.5` becomes `V2/1.0`).
* [ ] By converting integer revisions to text revisions.
```

## Part 4: Tying It All Together - The `aug:` Reference

The `aug:` URI scheme serves as the language for navigating this versioned, branched reality. It provides syntax for both simple requests and fully resolved, locked references that ensure perfect reproducibility.

### Components of an `aug:` Reference

An `aug:` reference consists of several key components:

- **Scheme**: The `aug:` prefix that identifies this as an Augmented reference
- **Branch**: An optional branch specification (e.g., `~main/`) that explicitly targets a branch
- **Resolved Branch**: In fully resolved references, the branch that was used for resolution (e.g., `~:staging`)
- **Path**: The textual identifier of the Vibe (e.g., `articles/common/button`)
- **Version Prefix**: An optional version constraint (e.g., `?1.2`) that specifies compatibility requirements
- **Resolved Version**: In fully resolved references, the exact version that was found (e.g., `:1.2.3`)
- **Error Indicator**: If resolution failed, an error code (e.g., `!404`)
- **Fragment**: An optional pointer to a specific part within the Vibe (e.g., `#schema/properties/name`)
- **Text Fragment**: An optional text fragment identifier (e.g., `:~:text=exact%20text`) that points to specific text content within the Vibe, following the [Text Fragments](https://wicg.github.io/scroll-to-text-fragment/) standard

Examples:

- `aug:schemas/UserProfile` - Simple request for the latest version
- `aug:~experiments/schemas/UserProfile` - Request specifically from the "experiments" branch
- `aug:/schemas/UserProfile` - Request specifically from the "main" branch (shorthand for `aug:~main/schemas/UserProfile`)
- `aug:schemas/UserProfile?1.2` - Request compatible with version 1.2
- `aug:~:staging/schemas/UserProfile?1.2:1.2.staging.4` - A fully resolved reference

### The Dual Role of the Search Path

When resolving a reference, the search path is used to match against **both** the Branch and any branched revisions in the Version. This is the magic that makes personal drafts and feature branches work.

Consider a search path of `['yarik', 'casino', 'core']`. When resolving `aug:articles/cows`, the system:

1.  Finds all `articles/cows` Vibes published to the `casino` branch.
2.  From that set, it sees a version `1.yarik.1`. Because `yarik` is in the search path with the highest priority, this version is selected.
3.  If no `yarik` version existed, it would find the latest version without a special branched revision, like `1.2`.

> **Alice:** "So the search path doesn't just look at branches, but also at the branched revision names within versions?"
> **Bob:** "Exactly. That's what makes it so powerful. It can find your work whether it's published to your personal branch or just has your name in its version."
> **Alice:** "That's really flexible. I could have my personal changes visible just to me, while still working within the main system."
> **Bob:** "That's the idea. It creates these parallel realities where everyone sees their own work integrated seamlessly with the shared foundation."

### Reference States

- **Implicit Request (The "Ask")**: `aug:schemas/UserProfile?1.2`

  - This asks for the latest version of `UserProfile` that is compatible with the `1.2` prefix, resolved against the current search path.

- **Explicit Branch Request**: `aug:~main/schemas/UserProfile`

  - The `~` prefix indicates an explicit **Branch** request. This asks for a Vibe published to the `main` branch, ignoring the rest of the search path for this lookup.
  - **`main` branch shorthand**: An explicit request for the `main` branch can be shortened. For example, `aug:~main/schemas/UserProfile` can be written as `aug:/schemas/UserProfile`. The leading slash acts as a shorthand for `~main/`.

- **Resolved Reference (The "Answer")**: `aug:~:staging/schemas/UserProfile?1.2:1.2.staging.4`
  - This is the permanent, unambiguous record of a resolution.
  - `~:` indicates the original request was implicit.
  - `staging` is the branch from the search path that successfully resolved to a Vibe.
  - `?1.2` was the requested version prefix.
  - `:1.2.staging.4` is the exact, full Version that was resolved.
  - **The `main` branch shorthand**: For clarity, the `main` branch is treated as the default and is not explicitly shown in a resolved reference. For example, if the above reference had resolved on the `main` branch, it would be written as `aug:schemas/UserProfile?1.2:1.2.main.4`, omitting the `~:main/` specifier.

It's important to note that branches (like `main` or `feature/new-login`) are separate from branched revisions in versions (like `feature-x` in `1.2.feature-x.0`). Branches act as publishing channels that control where a Vibe is visible, while branched revisions within versions define lineage and development history. The search path coordinates both of these concepts - it determines which branches to look in and also which branched revisions to prefer when multiple versions are available.

## Part 5: A Complete Development Workflow

These concepts enable a safe and efficient workflow:

1.  **Branching**: A developer starts work on a Vibe with path `processes/onboarding`, which is at version `1.3` and published to the `main` branch. They "branch" it for their feature, `new-login`.

    - The system creates a new Vibe version.
    - **New Version**: `1.3.new-login.0`
    - **Published to Branches**: `["feature/new-login"]`
    - The developer's search path is set to `['feature/new-login', 'main']`.

2.  **Developing**: The developer works on their Vibe, creating versions `1.3.new-login.1`, `1.3.new-login.2`, etc. They need a standard "Send Email" Instruction. They reference it as `aug:instructions/send-email`. The resolver first checks for a version published to the `feature/new-login` branch, finds nothing, and falls back to find the stable version published to `main`.

3.  **Overriding**: The developer needs a custom email template. They fork `records/email-templates/welcome` and save a new version. This new version is created and published to the `feature/new-login` branch with a version like `2.1.new-login.0`. Now, their process automatically picks up their custom template because their feature branch has top priority in their search path.

4.  **Promoting**: Once complete, the developer "promotes" their work. The Vibe at version `1.3.new-login.2` is copied, its Version is cleaned to `1.4`, and it is now published to the `main` branch (e.g. its `branches` array becomes `["main", "feature/new-login"]`). It is now the new, stable version.

5.  **The Ripple Effect**: Another `Process` was referencing `aug:processes/onboarding?1`. Because the new version `1.4` is compatible with the `?1` prefix, the next time that `Process` is resolved, it will automatically and safely pick up the new `1.4` version. This controlled update is a core feature of the system's architecture.

> **Alice:** "This workflow is really comprehensive. So I can branch, develop in my own space, and then promote my changes when they're ready?"
> **Bob:** "Yes, and the system handles all the complexity of making sure your changes are visible to you during development, and then properly integrated when promoted."
> **Alice:** "And this ripple effect - that means other parts of the system automatically get my updates if they're compatible?"
> **Bob:** "Exactly. If they asked for version `?1`, they'll get your new `1.4` automatically. But if they wanted something more specific like `?1.3` exactly, they wouldn't get the update unless they changed their reference."

This workflow, powered by the unified branching and versioning system, allows for parallel development to happen safely and efficiently. It is the core mechanism that allows the Vibe ecosystem to evolve in a controlled, predictable, and scalable way.

---

## Advanced Concepts & Implementation Details

### Mental Checkpoints

- **Versions are Lineage**: `1.2.feature.3` tells a story of forking from `1.2`.
- **Dual Visibility Control**: A vibe can be _published to_ the `main` branch but have a _version_ of `1.2.yarik.1` where `yarik` is a branched revision that makes it preferentially visible to users with `yarik` in their search path.
- **Search Path is an Overlay**: `['yarik', 'main']` means "show me `yarik`'s work layered on top of `main`."
- **Promotion is Shortening**: A draft like `1.2.yarik.5` gets promoted by being copied to a new, shorter version like `1.3`.
- **The Ripple is a Feature**: Promoting `1.3` will be picked up by anything asking for `?1`, which is the intended upgrade path.
- **Pinning for Stability**: To avoid the ripple, ask for a more specific version prefix, like `?1.2`.

> **Alice:** "These mental checkpoints are really helpful. So the version structure itself tells a story about how the Vibe evolved?"
> **Bob:** "Exactly. Just by looking at `1.2.feature.3`, you know it's the third revision of a feature branch that started from version 1.2."
> **Alice:** "And the search path is like laying my work on top of the main system, like transparent overlays."
> **Bob:** "That's a perfect analogy! Your changes sit on top, and anywhere you haven't changed something, you see through to the main version underneath."
