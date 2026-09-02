# 302: Ideator/Resolver

**System: Resolver** is an `Idea Transformer` that enables the composition and dynamic linking of `Ideas`. As a critical companion to [System: Storage](./301_ideator_storage.md), the Resolver's purpose is to take a high-level `Idea` containing references to other `Ideas` and transform it into a complete, self-contained, and executable `Idea` with all references resolved.

This mechanism allows `Ideas` to become modular and reusable building blocks, preventing data duplication and enabling the creation of complex systems from simpler, versioned components.

## 2. Core Concept: Resolution as an Idea Transformer

The Resolver service functions as a stateless `Idea Transformer` that enriches an `Idea` by fetching and embedding its dependencies.

- **Input**: An `Idea` whose `schema` or `context` contains one or more references to other `Ideas`.
- **Process**: The Resolver parses the input `Idea` to identify these references. For each reference, it queries a configured set of `Storage` services to fetch the corresponding `Idea` content. It then intelligently injects the content of the fetched `Ideas` into the original one.
- **Output**: A new, "hydrated" `Idea` where all references have been replaced by their resolved content, making it a complete, self-contained unit ready for execution or further processing.

## 3. Key Characteristics

### 3.1. Reference Syntax

While the exact implementation can vary, references should be expressed in a clear, unambiguous URI-like format. This format must be capable of specifying the `Idea`'s unique name as well as an optional version or branch.

_Example: A schema composing multiple `Ideas` using `allOf`._

```json
{
  "type": "object",
  "allOf": [
    { "$ref": "idea://my-org/article-template?version=1.2.0" },
    { "$ref": "idea://my-org/system-prompts/chain-of-thought?branch=latest" }
  ]
}
```

### 3.2. Deep (Recursive) Resolution

The resolution process is recursive. If a resolved `Idea` itself contains further references, the Resolver will continue to fetch and embed them until the entire dependency graph is resolved, resulting in a final `Idea` that contains all necessary information.
