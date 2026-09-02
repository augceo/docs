# 090: Agent/Typing

> [!DEFINITION] [Typing](./000_glossary.md)
> The discipline by which a schema's inferred TypeScript type is computed once, carried on the schema itself, and made to survive the boundaries between modules — so that the shape of an agent's answer is known statically at the place the question is asked.

> Sidenote:
>
> - Requires:
>   - :term[001: Agent/Request]{href="./001_agent_request.md"}
>   - :term[800: Package/Schemistry]{href="./800_package_schemistry.md"}

Everything the preceding chapters describe — a :term[Tool Call]{href="./004_agent_call.md"} with typed arguments, a :term[State]{href="./009_agent_state.md"} object whose paths are known, a :term[Plan]{href="./012_agent_plan.md"} whose steps compose — rests on a single claim: that a JSON Schema can be lifted into the type system and stay there. That claim is affordable only because of a small number of deliberate decisions, none of which are obvious from reading the code, and one of which is four lines long.

What follows is the account of those decisions, why each is load-bearing, and where the limits actually sit.

## The Answer's Type Is a Function of the Question

The reason to care about any of this is visible in one signature.

```typescript
Response = Identity<{ output: FromSchema<S> } & Content.InferResponseIntersection<Messages>>
): Promise<readonly Response[]>
```

A :term[Request]{href="./001_agent_request.md"} returns a type computed from two sources: the schema the caller supplied, and the context messages the caller passed alongside it. The first term is the caller's own schema lifted into TypeScript. The second is whatever the content handlers contribute — because a handler may rewrite the request's response schema, and the type layer computes that same widening statically.

This is the point of the whole apparatus. Add a message to the context and the type of the answer changes at the call site, before anything runs. The runtime pipeline and the type-level pipeline are two implementations of one rule, kept in step by construction rather than by discipline.

It is also the reason inference cost matters at all. The expense does not scale with how large a schema is; it scales with how many places ask a question.

## The Value and Its Type Share a Slot

JSON Schema already has a keyword that means *this schema's value is exactly this*: `const`. The library takes that keyword at its word and uses the same slot at the type level to mean *this schema's value has exactly this type*.

The symmetry is exact, and it runs in both directions:

- `Schema.const(value)` takes a value and produces a schema carrying it — at runtime.
- `FromSchema<S>` takes a schema type and produces its value type.
- `ToSchema<T>` takes a value type and produces a schema type.
- `Cached<S>` parks `FromSchema<S>` in the schema type's `const`.

So `Computed<S>` is `Omit<S, 'const'> & Cached<S>` — replacing the value with the type of the value. Not a collision with the keyword, but the same idea one level up.

> Sidenote:
>
> The helper that installs it, `compute()`, is a runtime identity. It returns the schema it was given, unchanged. Everything it does happens in the type domain; the object that flows on is the same object.

## Why the Cache Is Not Merely Faster

A cache that only avoided repeated work would be an optimisation. This one changes what is possible.

`FromSchema` fails with `TS2589` — *type instantiation is excessively deep* — at **sixteen levels of nesting**. Fifteen infers correctly; sixteen does not compile. A single `compute()` boundary placed part-way down makes a **twenty-level** schema infer completely, at 27,132 instantiations. That is not slow becoming fast. That is impossible becoming possible.

The mechanism is four lines in the vendored parser. The `const` branch is hoisted above the object branch, and `ParseConst` returns the cached type **without descending into the rest of the schema**. A schema carrying a computed `const` therefore short-circuits on the first branch test. Without that hoist the cache is inert — the parser would walk past it and re-derive everything underneath.

The cost profile follows from the same fact. Measured against a base schema of 16 and 64 properties, with N types derived from it:

| | fixed | per derived use |
|---|---|---|
| direct, 16 properties | 3,527 | 3,093 |
| direct, 64 properties | 9,095 | 10,293 |
| cached, 16 properties | 9,155 | **416** |
| cached, 64 properties | 28,211 | **416** |

The per-use figure is identical at both sizes. What schema size changes is the one-time term. **The cache does not remove the work; it moves the size-dependent part out of the per-use term and into a term paid once.** Break-even arrives at one extra derived use.

## The Boundary That Matters Is Declaration Emit

Within a single compilation, TypeScript already memoises a type reference instantiated with the same arguments, so writing `FromSchema<typeof S>` twice in one file costs almost nothing the second time. This is why the cache can look redundant when measured in one file — and why measuring it in one file is misleading.

The boundary that matters is not between modules. It is between a source tree and its emitted declarations.

The declaration emitter records a **type alias** as the expression the author wrote, unevaluated:

```typescript
// alias.ts  →  alias.d.ts
export type T = FromSchema<typeof S>;
```

Every consuming file re-derives it, forever — 10,724 instantiations each.

But a **value's** type must be serialised structurally, because there is no expression to record:

```typescript
// value.ts  →  value.d.ts
export declare const C: {
    readonly type: "object";
    readonly properties: { readonly a: { readonly type: "string" } };
    readonly required: readonly ["a"];
    readonly const: {
        [x: string]: unknown;
        a: string;
    };
};
```

`FromSchema` appears nowhere. The inferred type is written out. Consumers pay **zero** instantiations — the same figure as a hand-written declaration file, which is the floor.

This is why the schema must be exported **through a value**. Every alternative that looks equivalent fails: `Identity<>`, a mapped-type expansion, and `interface extends` all record the reference rather than the result. Only passing the schema through `compute()` and exporting the constant materialises it.

The practical consequence is that `tsc --declaration` over a computed schema *is* the precompiler. There is nothing to build.

## Where the Limits Actually Sit

Ambition is cheap to write and expensive to check. The measured shape of the cost:

- **Flat properties are linear** — 266 instantiations per property, unchanged from 16 to 128.
- **Nesting depth is linear** — 1,252 per level, until it stops at the depth-16 cliff.
- **Enum members are nearly free** — 20 each. A ten-thousand-member enum is cheaper than a forty-property object.
- **`anyOf` is the one superlinear axis** — roughly quadratic in the number of arms; 512 arms costs 1.74M instantiations and still completes.
- **`$ref` is cheaper than inlining, not dearer** — 1,912 against 2,848 for the same shape written out. An unused entry in the references tuple costs nine.

Two conclusions follow. Ordinary schema size does not threaten anything. And the depth cliff is reachable by an ordinary business object, which makes the placement of a `compute()` boundary a design decision rather than a tuning knob.

> Sidenote:
>
> The caps are properties of the type checker, not of this library, and they do not move. The native TypeScript compiler carries every one of them over unchanged — identical error codes at identical thresholds, and instantiation counts within a fraction of a percent. It is three to twelve times faster in wall-clock, and the speedup shrinks as the work grows. A faster engine is not a cheaper one.

## What Survives a Transformation

A cached type is only useful while it remains true. The library's operations divide sharply:

- `minimal` and `maximal` **carry it correctly**, rewriting the `const` slot to match what they did.
- `Intersection` **drops it**; the result is reconstructed from scratch.
- `merge` **loses everything**, returning an untyped value.
- `map` **returns the base definition**, discarding the type.
- `pick` and `omit` **keep it, and it becomes false** — they return the input type unchanged while the runtime removes properties, so the cached type promises fields that no longer exist.

That last case deserves emphasis, because it is not a performance matter. A stale cache is a type that lies without erroring, and the caching makes a pre-existing inaccuracy in the schema type into an inaccuracy in the value type as well. Any operation that changes a schema's shape must rewrite the slot or clear it.

## Composition, and the Function That Was Not Closed

For a long time `Intersection` could not be nested. Two calls deep, with a single property in each operand, it cost **6,639,171** instantiations and produced no usable type at all — while one call cost 7,509 and schema size made no difference. Nesting alone was what failed.

The number was a disguise. Bind the inner call to a variable before passing it on, and the same computation costs 25,722 and reports the actual problem in one line:

```
Argument of type 'ToSchemaObject<…>' is not assignable to parameter of type 'Definition'
```

**The function was not closed over the type it consumed.** `Intersection` returned a `ToSchemaObject`, which is not a schema, so its own output could never be its own input. It was never composable at any price, and the millions of instantiations were a type error the checker was never allowed to finish stating — inline, it must test assignability against an unevaluated expression, and it expands the whole tree trying.

This is the rule the rest of the library should be held to: **an operation on a schema must return something that is still a schema.** The cheap way to test it is to bind a result to a variable and pass it back in.

The repair follows from that. Rather than unboxing both operands, intersecting, and rebuilding a schema around the result, the operation intersects the **already-computed `const` slots** and leaves the result schema-shaped. Nesting then closes, and the cost collapses from 6,639,171 to **1,037**. The runtime keeps its own merge — including, now, the `items` case it had always been missing, which had silently discarded one operand's element schema entirely.

> Sidenote:
>
> Two shapes that look like the same idea both fail. Wrapping the *return type* in `compute()` measures 11,636,861 — the worst variant tried. An earlier signature that cast the result to a cached intersection measured 16.6M. Both box an expression that has not been evaluated, which re-enters the derivation from the other side. Intersecting slots that are already computed never leaves the type domain; boxing an unevaluated expression re-enters it.

### The rule underneath

The reason the naive repair still cost 8,695,786 is worth stating on its own, because it generalises well past this one function.

**While resolving a still-generic conditional type, TypeScript instantiates both branches.** A branch headed by another conditional therefore forces the schema parser to run, even on the path that will be discarded. Every branch of a conditional in this layer must be headed by something inert — a mapped type, an interface, or `Array<…>` — never by another conditional.

The fix that closed the gap was one line: writing the merged element type with an indexed access instead of `infer`.

### What the closure costs

`properties` and `required` lose their literal detail, becoming `Record<string, Schema.Definition>` and `readonly string[]`, in exchange for an exact `const`. Since `$id`, `title` and `description` were already absent from the result, nothing that survived before is lost now. The trade is the whole bargain of this chapter restated: **the schema's own description grows vaguer so that the value's type can stay exact.**

## Outro

The typing layer is not a convenience placed over the schema library; it is what makes an agent's answer knowable before it is asked for. It costs one hoisted branch in a parser, one shared slot between a value and its type, and one rule about exporting through values rather than aliases. Those three decisions carry the whole of :term[001: Agent/Request]{href="./001_agent_request.md"}, and their limits are the limits of everything built above them.

With the shape of an answer established, :term[101: Concept/Idea]{href="./101_concept_idea.md"} turns from how a request is typed to what the system is ultimately composing.
