# Chapter 2.2: Instruction Guidance

This document explains the key concepts and mechanisms behind the Instruction system demonstrated in the examples. It covers how context switches are handled, how Instructions can be composed like macros, and how data flows between steps.

## Context Switching with Execution Environments

### Understanding Context Suffixes

The Instruction system uses naming conventions to signal which execution environment should handle a particular step:

- **Base name (e.g., `IdentifyParticipants`)**: Executed in the LLM context by default
- **`_Activity` suffix (e.g., `FetchAvailability_Activity`)**: Executed on the server
- **`_User` suffix (e.g., `ConfirmInvitation_User`)**: Requires user interaction

These suffixes provide clear visual signals in the code about where execution will happen and help the system route operations to the appropriate environment.

```ts
// LLM operation (default)
const participants = IdentifyParticipants(prompt);

// LLM operation to prepare parameters
const availabilityParams = FetchAvailability(participants);

// Server operation
const availabilityData = FetchAvailability_Activity(availabilityParams);

// User interaction
const approval = ConfirmInvitation_User(invitation);
```

### Context Containers

In the flattened representation, we explicitly group operations by their execution context:

```ts
Process(
  // LLM operations
  LLM_Context({
    participants: IdentifyParticipants(prompt),
    availabilityParams: FetchAvailability({ participants }),
  }),

  // Server operations
  Server_Context({
    availabilityData: FetchAvailability_Activity({ availabilityParams }),
  })

  // More contexts...
);
```

This structure creates a clear separation of concerns, ensuring that operations run in their appropriate environments while maintaining data flow between them.

## Instruction Composition and Macros

Instructions can be composed like mixins, with one Instruction wrapping or enhancing another. This is accomplished through JSON Schema composition.

### Schema Merging

When Instructions are composed, their schemas are merged following JSON Schema's `allOf` semantics. This preserves the structure and requirements of both schemas while combining them into a single, coherent interface.

The key insight is that JSON Schema preserves property order, which enables mixin-like behavior:

```json
{
  "properties": {
    "_considerations": { "type": "string" },
    "default": true, // This is a placeholder that gets replaced
    "_feedback": { "type": "string" }
  }
}
```

When this schema composes with another, the `default` property acts as a slot that gets replaced by the other schema's properties. This creates a wrapper pattern where the mixin can inject properties before and after the wrapped schema's properties.

### Creating Mixin Instructions

The `Introspect_Mixin` demonstrates this pattern:

```json
{
  "title": "Introspection_Mixin",
  "properties": {
    "_considerations": {
      "type": "string",
      "description": "Before performing the action, analyze the request and outline key considerations."
    },
    "default": true, // Placeholder for the wrapped schema
    "_feedback": {
      "type": "string",
      "description": "After completing the action, provide feedback about the decisions made."
    }
  }
}
```

When applied to another Instruction, it injects cognitive steps before and after the main task:

```ts
Introspect_Mixin({
  default: {
    identifyParticipants: IdentifyParticipants(prompt),
    fetchAvailability: FetchAvailability({ participants }),
  },
});
```

This composition creates a new schema that guides the LLM to:

1. Think through considerations before acting
2. Perform the main operations
3. Reflect on the results afterward

## Data Flow Between Instructions

### Implicit Parameter Passing

Instructions can implicitly pass data between steps through their descriptions and the LLM's understanding of context. This is shown in the examples where participant information flows from `identifyParticipants` to `fetchAvailability`.

```ts
fetchAvailability: {
  organizerEmail: 'alice@example.com', // Reused from identifyParticipants
  attendeeEmail: 'bob@example.com',    // Reused from identifyParticipants
  timeRange: { /* ... */ }
}
```

The description in the schema explicitly instructs this data reuse. The actual schema structure uses `allOf` to combine the base schema reference with additional instructions:

```json
"fetchAvailability": {
  "allOf": [
    { "$ref": "aug:processes/schedule-meeting:1#fetchAvailability" },
    { "description": "Retrieves calendar availability data for identified participants, reusing participant emails from the identification step to check their schedules" }
  ]
}
```

This pattern allows us to:

1. Reference the base functionality and structure from the process definition
2. Extend it with contextual instructions specific to this usage
3. Guide the LLM on how to reuse data between steps without requiring explicit parameter declarations

### Explicit References

For more complex dependencies, Instructions can use explicit references to declare which previous steps they depend on:

```json
"findCommonSlot": {
  "references": [
    "LLM_1.identifyParticipants",
    "SERVER_1.fetchAvailabilityOutput"
  ]
}
```

These references serve multiple purposes:

1. They document data dependencies clearly
2. They guide the LLM in understanding which data to use
3. They help the system ensure that referenced data is available when needed

## Process Instantiation and Execution

### Process Prototypes

Processes are defined as "vibes" - reusable prototypes that can be instantiated:

```
aug:processes/schedule-meeting:1
```

These prototypes define the complete structure of a process, including all steps, contexts, and data dependencies.

### Process Initiation

A process is instantiated when a vessel completes its first step:

```ts
StartProcess({
  name: 'ScheduleMeeting',
  LLM_1: {
    // First step data...
  },
});
```

This creates a new instance of the process that will carry state through its lifecycle.

### Process Continuation

After initiation, the process continues automatically through its defined steps, with each step having access to the data it needs from previous steps. The vessel carries the process state, ensuring continuity across context switches.

## Best Practices for Instruction Design

1. **Use clear naming conventions** for context switching (base, \_Activity, \_User)
2. **Design modular Instructions** that can be composed and reused
3. **Use descriptions to guide data flow** between steps
4. **Explicitly document references** for complex dependencies
5. **Create macro Instructions** for common patterns like introspection, validation, or error handling
6. **Start with the minimal entry point** needed to initiate a process

By following these practices, you can create Instructions that are both powerful and flexible, enabling complex workflows that span multiple execution environments while maintaining clear data flow and dependencies.
