# 302: Ideator/Resolver

**System: Resolver** is a special tool we call an `Idea Transformer`. Its job is to let you build big `Ideas` by snapping smaller `Ideas` together, kind of like connecting different train cars to make one long train.

The Resolver works closely with the [System: Storage](./301_ideator_storage.md), where all the `Ideas` are kept. It takes a big-picture `Idea` that points to other `Ideas` and pulls them all together into one complete, ready-to-use package.

This is super useful because it lets you treat `Ideas` as reusable building blocks. You can build big, complex things from smaller, simpler parts without having to copy and paste the same information over and over.

## 2. Core Concept: How the Resolver Transforms Ideas

The Resolver works like a helpful assistant who doesn't need to remember anything about past requests. It simply takes an `Idea` and makes it whole by finding and adding all the other pieces it depends on.

- **What you give it**: An `Idea` that contains links or references to other `Ideas`.
- **What it does**: The Resolver reads your `Idea` and finds all the links. For each link, it asks the `Storage` system to fetch the `Idea` it points to. Then, it neatly inserts the content of those other `Ideas` right into the original one.
- **What you get back**: A new, “filled-in” `Idea` where all the links have been replaced by their actual content. It’s now a complete, self-contained unit that's ready to go.

## 3. Key Characteristics

### 3.1. How to Write Links

Links to other `Ideas` should be written in a clear, simple way, a bit like a web address (URI). This special address needs to include the `Idea`'s unique name and can also specify a particular version.

_Example: A design that combines two other `Ideas`._

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

The process of finding and adding `Ideas` can go many layers deep. If a linked `Idea` also contains its own links, the Resolver will follow them, too. It keeps doing this until every single piece has been found and slotted into place, creating one final, complete
