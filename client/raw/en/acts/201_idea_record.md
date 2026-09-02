# 201: Idea/Record

> [!DEFINITION] [Record Idea](./000_glossary.md)
> An Idea whose schema defines structured data. Its `solution` is the structured content itself.

> [!WARNING]
> This RFC is currently a placeholder and will be expanded in the future.

> Sidenote:
>
> - Requires:
>   - [101: Concept/Idea](./101_concept_idea.md)

A **Record Idea** has a `schema` that specifies both the structure of its content `solution` (often a JSON Schema) and potentially embedded "data tools" for working with that `solution`. The `solution` of a Record Idea _is_ the structured content itself, conforming to this `schema`, given a specific `context`.
