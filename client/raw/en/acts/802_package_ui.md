# 802: Package/UI

> [!DEFINITION] [Tree Style Architecture](./000_glossary.md)
> A schema-driven engine for generating dynamic, adaptable user interfaces directly from data structures. It treats the **Schema** as the backbone of design, allowing UIs to be stylized and reconfigured via declarative stylesheets and rendered through swappable adapters.

> Sidenote:
>
> - Requires:
>   - :term[800: Package/Schemistry]{href="./800_package_schemistry.md"}

The **UI Package** implements a rendering engine where **Structure is Design**. Instead of manually crafting templates for every data type, it generates the interface automatically from the underlying JSON Schema. This ensures that the UI is always in sync with the data model—change the schema, and the interface updates instantly.

## Core Philosophy: Structure as the Source of Truth

In traditional development, the data model and the UI often drift apart. This package solves that by making the **Schema** the single source of truth for the interface.

- **Dynamic Generation**: The UI is not hard-coded; it is derived. A change in the schema structure (e.g., adding a field, changing a type) is immediately reflected in the rendered output.
- **Always Up-to-Date**: Because the UI is a direct projection of the schema, it eliminates the class of bugs where the interface lags behind the data model.

## Declarative Styling & Reconfiguration

While the Schema dictates _what_ is rendered, the **Stylesheet** dictates _how_ it looks. This separation allows for radical redesigns without touching the underlying structure.

- **Declarative Layer**: Stylesheets act as a configuration layer that maps schema nodes to visual properties.
- **Reconfigurable**: You can completely change the layout, spacing, and visual hierarchy by swapping stylesheets, effectively "skinning" the raw data structure.

## Adapters & Interoperability

The engine is agnostic to the final rendering target. It uses an **Adapter Pattern** to translate the abstract component tree into concrete UI elements.

- **Design System Support**: Adapters can target specific design systems (e.g., Material UI, Ant Design), mapping abstract schema types to rich, pre-built components.
- **HTML Fallback**: A standard HTML adapter ensures that any schema can be rendered as semantic, accessible web content out of the box.

## Multi-Modal Views

The same schema and data can be projected into different contexts using **View Modes**.

- **Edit Mode**: Generates fully interactive forms for data entry and validation.
- **Show Mode**: Renders read-only, optimized presentations for data consumption.

This allows a single definition to serve multiple purposes across an application, reducing code duplication and ensuring consistency between creating and viewing data.

## Core Concepts

### Dynamic Property System with Derivations

The system is built around a self-extending controller where modular properties register themselves and their inter-dependencies, creating a powerful and extensible derivation graph.

- **Properties self-register**: Each property module (e.g., for `data`, `schema`, `vars`) automatically registers itself when imported.
- **Dependencies Declaration**: Properties can declare dependencies on other properties (e.g., `styles` depends on `vars` and `settings`).
- **Chained Derivations**: When a base property changes, the controller automatically re-derives any dependent properties in the correct order, ensuring the UI state is always consistent.
- **Type safety**: TypeScript infers the controller's and fields' types from the combined registered properties.
- **Modular architecture**: New properties, along with their derivation logic, can be added without modifying any existing code.

### User-Defined Properties

The dynamic property system is the key to this extensibility. You can create and register your own custom properties to add new features and control any aspect of field behavior. This allows you to build powerful, domain-specific functionality directly into the rendering engine. For example, you could implement:

- A `slots` property that depends on `styles` to determine which UI components to render.
- An `errors` property that depends on `data` and `schema` to perform validation.
- Custom styling properties that react to specific data conditions.

### Themable Component System

The second key to extensibility is the `Theme` system. It allows the rendering engine to be completely decoupled from any specific UI framework. This system orchestrates a clear flow of information from abstract data to a concrete UI:

1.  **Schema Provides Structure**: The `schema` defines the shape of the data and the overall hierarchy of the UI tree.
2.  **Controller Creates State**: The controller processes the schema and data, creating a specific `state` for every field in the tree.
3.  **Variables (`vars`) Declare Atoms**: CSS variables are used to declaratively assign `Atom` components to named `slots` (e.g., `--slot-title: 'TitleAtom'`).
4.  **Atoms are the Building Blocks**: `Atoms` are the leaf-level components (e.g., `<Input />`, `<Button />`) that bind to the field's `state` to display data.
5.  **Fields Orchestrate Rendering**: The `Field` component acts as an orchestrator, checking which `Atoms` are assigned to its named slots via variables.
6.  **Rendering is Data-Driven**: A `Field` only renders an `Atom` if the corresponding data for that slot exists in its `state`, ensuring a minimal UI.
7.  **Fields as Leaves**: Represents a single data point (e.g. a string), composing multiple `Atoms` into a complete input (label, widget, description).
8.  **Fields as Branches (Fieldsets)**: A `Field` can also represent a "branch" (object/array), acting as a "fieldset" that provides layout for its child `Fields`.

This clear separation of concerns allows for deep customization at every level, from the data processing logic to the final rendered pixel.

### State Management & Structural Sharing

The controller centrally manages the tree's state, distinguishing between raw props, processed state, and the last rendered state to enable efficient change detection.

- **Raw Props (`controller.props`, `controller.data`, etc.)**: The original props passed to the root component. They serve as the source of truth and are never mutated by the processing pipeline. This enables support for both controlled and uncontrolled modes.
  - **Controlled Mode**: When `data` or `vars` props are provided, the system uses these external values. Updates trigger `onChange`/`onVarsChange` callbacks.
  - **Uncontrolled Mode**: When only `initialData` or `initialVars` are provided, the controller manages state internally.
- **Current State (`controller.current`)**: The processed state. After the `store` phase, properties are processed (e.g., schema is collapsed, data is validated) and the result is stored in `controller.current`. This is the state that is distributed to fields.
- **Last State (`controller.last`)**: A shallow copy of the `current` state from the previous render cycle. It's used to compare against the new `current` state to detect exactly which properties and paths have changed.

**Structural Sharing:**

The architecture uses a single, shared state tree (`controller.current`) to minimize memory usage and ensure state consistency.

- **Zero Duplication**: Fields do not get their own copies of data. Instead, they hold references to slices of the `controller.current` state tree.
- **State Coherency**: Since fields directly reference slices of `controller.current`, the tree's state data is always coherent. UI updates are then batched and rendered on the next tick for performance.
- **Descendant Visibility**: Parent fields (e.g., for an object) have access to their entire sub-tree, including all nested data and schema.

### The Update & Derivation Pipeline

The controller uses a single, unified pipeline for both initial renders and all subsequent updates. This ensures a consistent and predictable state flow. Here is a step-by-step breakdown of the process:

- **Trigger**: An external event occurs (e.g., user input, API call). The corresponding property's `update` method is called, which checks for meaningful changes. If there are none, the process stops.

- **Root Re-render**: If a change is detected, a re-render of the root `<Form>` component is triggered. This initiates the controller's core processing cycle.

- **Store Raw Props**: During the render, the controller first stores the raw props from the `<Form>` component.

- **Process Properties**: The raw props are then processed into a consistent internal state (`controller.current`). For example, the `schema` is collapsed.

- **Build Field Tree**: The controller discovers all field paths from the processed state and ensures a `field` object exists for each one.

- **Distribute Changes**: The controller compares the new `controller.current` state with the previous state (`controller.last`) to identify the exact fields and properties that have changed.

- **Invalidate & Derive**: For each change detected, `controller.invalidate()` is called. This is the entry point for the reactive derivation system and triggers:
  - `controller.rederive()`: Computes new values for all dependent properties on the field (e.g., `settings`, `styles`) in the correct topological order.
  - `controller.cascade()`: Intelligently propagates the changes to descendant fields, triggering their own re-derive cycles.
  - Any field whose state is altered during this process is queued for a re-render.

- **Batched DOM Updates**: After the React render cycle is complete, a `useLayoutEffect` hook flushes the render queue. All fields that were queued during the derivation step are updated in the DOM in a single, efficient batch.

## Performance & Efficiency

### Efficient Rendering

The architecture is designed for high performance by minimizing React reconciliation and re-rendering overhead.

- **Precise Change Detection**: By diffing `controller.current` and `controller.last` for raw props, and using deep equality checks within the derivation cycle, the system knows exactly which fields and properties have changed, avoiding unnecessary updates.
- **Selective Invalidation & Derivation**: Only fields affected by a change are invalidated. The derivation chain ensures that only dependent properties are re-computed.
- **Deferred & Batched Rendering**: Field render requests are queued during the processing cycle. The controller then flushes this queue in a single batch within `useLayoutEffect`, minimizing render calls.
- **Render Deduplication**: If multiple changes affect the same field within one cycle, it still only renders once.

### Smart Cascading & Derivation

The system efficiently propagates variable changes down the field tree, similar to CSS variable inheritance, while minimizing re-computation.

- **Cached Dependency Graph**: The dependency relationship between all properties is calculated once and cached. The `rederive` process uses this cache to run derivations in the correct order without re-calculating it on every change.
- **Lazy Inheritance**: Variables are inherited up the tree on-demand when a field computes its styles, using `controller.inherit()`.
- **Selective Cascading**: When a CSS variable (`var`) changes on a field, the `cascade` method propagates the change down to its descendants. The cascade stops at any descendant that defines its own local override for that specific variable.
- **Differential Updates**: To avoid unnecessary re-renders, the `rederive` logic performs a deep-equality check on the result of each `derive` function. A field is only flagged for re-rendering if a cascaded `var` change actually resulted in a different final state (e.g., a different `style` object), preventing wasteful renders.
- **Consequence-Based Rendering**: This means that an external change (e.g., updating a `var`) will only trigger a re-render if it actually causes a meaningful change in a derived property that affects the UI. If a `var` change is overridden by a more specific rule and results in the same final `style` output, no wasteful render will occur.

## Example Update Flow

The unified pipeline handles all updates. The derivation cycle is an integral part of the "Distribute" phase.

```typescript
// User updates a CSS variable on the 'user.name' field
await controller.update('user.name', 'vars', { '--field-color': 'red' });
```

What happens internally:

- `controller.update()` calls the `update` method on the `VarsProperty`.
- The `update` method detects a change and calls `controller.render()`, triggering a re-render of the root component.
- The controller runs its processing cycle: it **stores** raw props, **processes** them into the new `controller.current` state, and **builds** the field tree.
- During the **distribute** step, the controller finds that `vars` on `user.name` has changed and calls `controller.invalidate('user.name', 'vars')`.
- `controller.invalidate()` is the entry point for the derivation logic:
  - It updates `field.vars` on the 'user.name' field.
  - It calls `controller.rederive(field, ['vars'])`.
  - It calls `controller.cascade(field, ['vars'])`.
- **Derivation & Cascade**: `rederive()` runs the dependency chain for the 'user.name' field, updating its derived `styles` and queueing it for a render. `cascade()` recursively propagates the change to children, triggering their `rederive` process.
- **`useLayoutEffect`** run, flushing the render queue and updating the DOM.

## Architecture Diagram: Update Lifecycle

```mermaid
graph TD
    subgraph "Trigger & Process"
        A["User Action / API Call"] --> B["<b>Property.update()</b>"];
        B -- "No change" --> Stop([Stop]);
        B -- "Change detected" --> C["<b>controller.render()</b><br/>Triggers root re-render"];
        C --> D["Store Raw Props"];
        D --> E["Process Properties<br/>(e.g., collapse schema)"];
        E --> F["Build Field Tree"];
    end

    subgraph "Distribute & Derive"
        F --> G["Distribute Changes<br/>Diff current vs. last state"];
        G --> H["<b>controller.invalidate(field)</b>"];
        H --> I["<b>controller.rederive(field)</b><br/>Computes new derived state"];
        I --> J{"Changed state?"};
        J -- Yes --> K["Queue field for re-render"];
        J -- No --> L;
        K --> L["<b>controller.cascade(field)</b>"];
        L --> M["...propagate to children..."];
    end

    subgraph "Render"
        K --> N["<b>useLayoutEffect</b><br/>(after React render)"];
        N --> O["<b>controller.flushRenderQueue()</b>"];
        O --> P["Affected Fields<br/>Re-render in DOM"];
    end
```

## API Reference

### Controller Methods

```typescript
// Update a field property
controller.update(path: string, property: string, value: any): Promise<boolean>

// Merge with existing property value
controller.merge(path: string, property: string, value: object): Promise<boolean>

// Get property value
controller.get(property: string, path?: string): any

// Inherit property value up the tree
controller.inherit(property: string, path: string, key?: string): any

// Register field subscriber
controller.register(path: string, forceRender: () => void): () => void
```

### Property Registration Example

Properties are self-contained objects defining lifecycle methods to manage a specific aspect of the tree's state.

```typescript
const StylesProperty = {
  priority: 50,
  fieldDefaults: { styles: {} },

  // Declare that this property depends on 'vars' and 'settings'
  dependencies: ['vars', 'settings'],

  // --- Lifecycle Methods ---

  // Computes the 'styles' object based on the field's current state.
  // Runs automatically when 'vars' or 'settings' change.
  derive: field => {
    const newStyles = getComputedFieldStyles(
      field.mode,
      varName => field.controller.inherit('vars', field.path, varName),
      field.type
    );
    return { styles: newStyles };
  },

  // Handles updates from a field, like controller.update('path', 'styles', ...)
  // This is less common for a purely derived property.
  update: (field, controller, value) => {
    return false; // Typically, derived properties are not manually updatable.
  },

  // Called by controller.update() to kick off the derivation process.
  invalidate: (field, controller, newValue, oldValue) => {
    // Invalidate is simpler for derived properties. The main logic is in `derive`.
    // The controller's rederive logic will handle re-computation.
    // For a base property like 'vars', it would trigger the chain.
    controller.rederive(field, ['styles']);
    controller.cascade(field, ['styles']);
  },
};

Property.register('styles', StylesProperty);
```

This architecture provides a robust foundation for complex UI rendering while maintaining excellent performance and developer experience.
