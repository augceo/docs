# 20: Company/Specification

> [!DEFINITION] Specification
> The permanent, evolving source of truth that defines the product vision, architecture, and implementation details.
>
> Sidenote:

> - Part of: :term[02: Company/Process]{href="./02_process.md"}
> - Preceded by: :term[22: Company/Alignment]{href="./22_document_alignment.md"}

## 1. The Core Philosophy

The Specification is the **destination**. It is a major labor of love—a document that the author builds, refines, and updates continuously. It is not written in a day; it is the result of weeks of deep thought and multiple rounds of iteration.

It is **Evergreen**. It is the absolute **Source of Truth for the System** (Code, Presentation, Architecture). In a system where we build complex, autonomous agents, we cannot rely on tribal knowledge or transient chats.

**The Vision:** We believe that with a correct and complete Specification, the rest of the work (code generation, presentation design) can be automated. We write this document to eventually feed it into the machine.

## 2. The Need

Why do we maintain this document?

- **Adherence to Truth:** It strictly follows the standards defined in :term[00: Company/Truth]{href="./50_prompt_truth.md"}. It must be self-contained and free from ambiguity.
- **Correctness over Speed:** Unlike transient documents (Proposal/Alignment), the Specification must be rigorously correct. It is better to leave a section undefined than to define it falsely.
- **The Map:** It describes _where we are going_. It allows every team member to understand the broader picture without needing to hold the entire plan in their head.
- **Handling Complexity:** For large systems with a "large surface area," it is impossible to hold the entire context in memory. The Specification allows the system to evolve while maintaining a coherent source of truth.

## 3. The Rules of Maintenance

1.  **Never Stagnant:** If the code changes, the document changes. Ideally, the document changes _before_ the code.
2.  **Expansion through Review:** The specification is never "finished," only "current." As we review it, we identify blind spots. Addressing these expands the document's scope to achieve the clarity required for execution.
3.  **Source of Truth:** If there is a conflict between the code and the Specification, the Document is correct, and the code is a bug (or the Document needs an update via the Proposal process).

## 4. Domain Mapping

The "System" is not always code. The Specification adapts to the domain:

| Domain           | Specification (The Truth)   | Artifact (The Output)       |
| :--------------- | :-------------------------- | :-------------------------- |
| **Engineering**  | Architecture Doc / API Spec | Typescript Code / DB Schema |
| **Presentation** | Narrative Outline / Script  | Slides / Figma              |
| **Process**      | Policy Document             | The Team's Actions          |
| **Design**       | User Flow / Wireframe       | UI Components               |

## 5. Characteristics

- **Timeframe:** Weeks to create, continuous updates.
- **Rounds:** Multiple iterations and refinements.
- **Role:** The "Big Doc" that anchors the project.
- **Nature:** Evergreen.
- **Source of Truth:** **For the System**.
