# Chapter 8: Branches - Managing Parallel Realities

## New Ideas in This Chapter

- **Branches as Isolated Workspaces**: We introduce branches as parallel, isolated environments within the same database, allowing teams to experiment, develop, and test new features without disrupting the stability of the main (`main`) version.
- **Content-Addressable, Branch-Aware References**: The `aug:` URI scheme is extended to be fully branch-aware. It allows for both explicitly requesting a Vibe from a specific branch and implicitly resolving a Vibe against a prioritized list of branches.
- **Hierarchical Resolution via Search Paths**: Vibe resolution is not a simple lookup. It follows a configurable, ordered **search path** (e.g., `['feature-x', 'staging', 'main']`), creating a cascading system of fallbacks and overrides that is essential for development workflows.
- **Time-Traveling Queries**: Branch-based resolution is also time-sensitive, allowing the system to resolve dependencies as they existed at any point in the past, ensuring perfect reproducibility for any Vibe.
- **Branches as the Foundation for Safe Evolution**: We show how branches are the mechanism that enables the safe `refinement` and evolution of all Vibe types—from `Records` and `Instructions` to `Processes` and `Budgets`—in a controlled, auditable manner.

---

## The Power of Parallelism

In any evolving system, managing change is a critical challenge. How do you introduce new features, fix bugs, or experiment with new ideas without destabilizing the core, production-ready version of your system? The answer is **branches**.

A branch is a complete, parallel version of the Vibe ecosystem. It allows developers and creators to work in an isolated environment, making changes and creating new Vibe revisions that are invisible to other branches until they are explicitly merged. This provides two fundamental benefits:

1.  **Isolation**: Work on a new feature (e.g., on a `feature/new-billing` branch) doesn't interfere with the stable `main` branch. This prevents half-finished or buggy code from affecting production systems.
2.  **Experimentation**: Branches are cheap and easy to create. This encourages experimentation. You can spin up a branch to test a new `Instruction`, try out a different `Budget` model, or develop a new `Process`, and simply discard the branch if the experiment doesn't work out.

At a technical level, this is enabled by the `vibes` database table, which includes a `branches` array column. A single Vibe `id` can have multiple revisions, each associated with one or more branches. This allows a Vibe like `aug:ui/theme` to have a stable `revision 2` on the `main` branch while `revision 5` is being developed on the `feature/redesign` branch.

## Referencing and Resolving Across Branches

The system's real power comes from its ability to intelligently resolve `aug:` references in a branch-aware context. This is governed by the **search path**, an ordered list of branch names that tells the system where to look for a Vibe, and in what order of priority.

### The Search Path

When you request a Vibe without specifying a branch, the system resolves it against your current search path. A typical search path during development might be:

`['feature/my-new-idea', 'staging', 'main']`

This configuration means:

1.  First, look for the Vibe on the `feature/my-new-idea` branch.
2.  If it's not found there, look on the `staging` branch.
3.  If it's still not found, fall back to the `main` branch.

This cascading lookup is powerful. It allows a developer working on the `feature/my-new-idea` branch to have a workspace that contains:

- The specific Vibes they have created or modified on their feature branch.
- All the stable Vibes from `staging` and `main` that they haven't touched.

This creates a complete, functional environment without needing to duplicate every Vibe.

### `aug:` Reference Syntax for Branches

The `aug:` URI scheme has specific syntax for handling branches:

- **Implicit Request**: `aug:schemas/UserProfile`

  - This asks for the latest revision of the `UserProfile` schema, resolved against the current search path.

- **Explicit Request**: `aug:~main/schemas/UserProfile`

  - The `~` prefix indicates an explicit branch request. This asks for the latest revision of the `UserProfile` schema **specifically from the `main` branch**, ignoring the rest of the search path for this Vibe.

- **Resolved Reference**: `aug:~:staging/schemas/UserProfile?:2`
  - This is the format of a reference after it has been successfully resolved. It provides a permanent, unambiguous record of the resolution:
    - `~:` indicates the original request was implicit (no `~` prefix).
    - `staging` is the branch where the Vibe was found.
    - `schemas/UserProfile` is the path.
    - `?:2` indicates the latest version was requested (`?`) and resolved to revision `2` (`:2`).

The SQL test suite (`03_test_find_vibes_for_resolution.sql`) provides numerous concrete examples of these resolution rules in action, from simple priority checks to complex fallbacks with minimum version requirements.

```llm
Branch resolution is governed by an ordered search path (e.g., `['feature', 'main']`).
An implicit reference (`aug:my-vibe`) is resolved by checking each branch in
order. An explicit reference (`aug:~main/my-vibe`) targets a specific branch,
overriding the search path. Resolved references record the outcome, creating
a permanent, auditable link (e.g., `aug:~:main/my-vibe?:5`).
```

> **Alice:** "So if my search path is `['my-feature', 'main']` and I ask for `aug:common/button`, it will first look for a version of `common/button` on my feature branch. If I haven't touched it, it will find the standard one on `main`?"
> **Bob:** "Exactly. You get your local changes, plus the stable base from `main`, all seamlessly. But if you needed to test against the `main` version of the button specifically, you could ask for `aug:~main/common/button` to bypass your local version."

```question
How does the search path influence Vibe resolution?
* [x] It defines an ordered priority list of branches to search.
* [x] It allows developers to override stable Vibes with their own versions.
* [x] It enables a cascading fallback mechanism from feature branches to main.
* [ ] It requires all Vibes to exist on all branches.
* [ ] The order of branches in the search path does not matter.
```

## A Typical Development Workflow

Let's see how branches enable a safe and efficient development workflow for creating a new `Process`.

1.  **Create a Branch**: A developer starts by creating a new branch, `feature/user-onboarding-v2`. Their search path is set to `['feature/user-onboarding-v2', 'main']`.

2.  **Develop the Process**: The developer creates a new `Process` Vibe, `aug:processes/onboarding`. Because they are on the `feature/user-onboarding-v2` branch, this new Vibe and all its revisions are associated with that branch. They can freely `refine` and test the process, creating `Instruction` Vibes and supporting `Record` schemas as needed. All these new Vibes live on the feature branch.

3.  **Use Existing Components**: The new `onboarding` process needs a standard "Send Email" `Instruction`. The developer references it as `aug:activities/send-email`. The resolution system first checks the `feature/user-onboarding-v2` branch. Since it doesn't exist there, it falls back and finds the stable version on `main`.

4.  **Override a Component**: The developer realizes they need a custom "Welcome Email" template. They create a new revision of `aug:records/email-templates/welcome` and save it. This new revision is saved to the `feature/user-onboarding-v2` branch. Now, when their process references this Vibe, the resolution system finds their new version first and uses it, leaving the original on `main` untouched.

5.  **Testing**: The entire `Process` can be tested in isolation on the feature branch. It uses a combination of the new components from the feature branch and the stable components from `main`.

6.  **Merging**: Once development and testing are complete, the changes from `feature/user-onboarding-v2` can be merged into `main`. This is typically a controlled process where the new revisions of the Vibes created on the feature branch are now also associated with the `main` branch, making them the new "production" standard.

This workflow, powered by the branching and resolution system, allows for parallel development to happen safely and efficiently. It is the core mechanism that allows the Vibe ecosystem to evolve in a controlled, predictable, and scalable way.
