# Chapter 2.1: Instruction Examples

## Example 1: Process Scheduling

This example demonstrates how to structure a complex, multi-step process that involves different execution contexts (LLM, server, user). The process is designed to schedule a meeting, requiring participant identification, availability checks, time slot selection, and invitation management. Each context switch is explicitly modeled, and data dependencies between steps are clearly defined through references. The process shows how to handle both immediate LLM operations and long-running server tasks within a single, coherent workflow.

**Nested Expression:**

```ts
// Send the approved invitation
SendInvitation_Activity(
  // Get user approval for the drafted invitation
  ConfirmInvitation_User(
    // Create invitation with participants and selected time slot
    DraftInvitation(
      // Extract participant info from user's request
      participants: IdentifyParticipants(prompt),
      // Find a time that works for both participants
      slot: FindCommonSlot(
        // Get calendar data for identified participants
        FetchAvailability_Activity(IdentifyParticipants(prompt)))
      )
    )
  )
)
```

Or alternatively:

```typescript
// Extract participant info from user's request
const participants = IdentifyParticipants(prompt);

// Get calendar data for identified participants
const availabilityParams = FetchAvailability(participants);
const availabilityData = FetchAvailability_Activity(availabilityParams);

// Find a time that works for both participants
const commonSlot = FindCommonSlot(availabilityData);

// Create invitation with participants and selected time slot
const invitation = DraftInvitation(participants, commonSlot);

// Get user approval for the drafted invitation
const approval = ConfirmInvitation_User(invitation);

// Send the approved invitation
const result = SendInvitation_Activity(approval);
```

Flattened Expression in pseudo code:

```ts
Process(
  // LLM determines participants and prepares calendar query parameters
  LLM_Context({
    participants: IdentifyParticipants(prompt),
    availabilityParams: FetchAvailability({ participants }),
  }),

  // Server fetches actual calendar data
  Server_Context({
    availabilityData: FetchAvailability_Activity({ availabilityParams }),
  }),

  // LLM analyzes availability and creates invitation
  LLM_Context({
    commonSlot: FindCommonSlot({ availabilityData }),
    invitation: DraftInvitation({ participants, commonSlot }),
  }),

  // User reviews and approves the invitation
  User_Context({
    approval: ConfirmInvitation_User({ invitation }),
  }),

  // Server sends out the approved invitation
  Server_Context({
    result: SendInvitation_Activity({ invitation, approval }),
  })
);
```

This example demonstrates how a complex, multi-step process is initiated through a simple, focused instruction. The process has multiple steps (identifying participants, fetching availability, finding slots, drafting & confirming invitations), but the initial instruction focuses only on what's needed to start.

It's important to note that only the first LLM context (`LLM_Context` with `IdentifyParticipants` and `FetchAvailability`) serves as the entry point to the entire process - this initial LLM interaction becomes the inciting logic that starts the workflow, after which it becomes part of a process vibe that carries the state through its lifecycle, maintaining context and data dependencies between steps while ensuring that each subsequent action (whether performed by the server, LLM, or user) has access to the information it needs from previous steps. The vessel merely facilitates this interaction but does not itself become the process.

### The Process Schema

This schema is part of a vibe `aug:processes/schedule-meeting:1` that exists as a prototype of the complete meeting scheduling workflow with all its steps and data dependencies. Later in this explanation, we will show how a vessel can instantiate this process by completing its first step in the process of working with LLM context.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Process: Schedule Meeting",
  "description": "A durable process to schedule a meeting, involving both LLM and server steps.",
  "type": "object",
  "properties": {
    "llmContext1": {
      "title": "Initial Setup and Parameter Preparation",
      "description": "LLM identifies participants and prepares parameters for server operations",
      "type": "object",
      "properties": {
        "identifyParticipants1": {
          "title": "Identify Meeting Participants",
          "type": "object",
          "properties": {
            "organizer": {
              "type": "string",
              "description": "The person initiating the meeting."
            },
            "attendee": {
              "type": "string",
              "description": "The person being invited."
            }
          },
          "required": ["organizer", "attendee"]
        },
        "fetchAvailability": {
          "title": "Prepare Availability Fetch Parameters",
          "references": ["identifyParticipants"],
          "type": "object",
          "properties": {
            "organizerId": {
              "type": "string",
              "description": "The system identifier for the organizer, derived from identifyParticipants.organizer."
            },
            "attendeeId": {
              "type": "string",
              "description": "The system identifier for the attendee, derived from identifyParticipants.attendee."
            },
            "timeRange": {
              "type": "object",
              "properties": {
                "start": {
                  "type": "string",
                  "format": "date-time",
                  "description": "The start of the time range to check for availability."
                },
                "end": {
                  "type": "string",
                  "format": "date-time",
                  "description": "The end of the time range to check for availability."
                }
              },
              "required": ["start", "end"]
            }
          },
          "required": ["organizerId", "attendeeId", "timeRange"]
        }
      },
      "required": ["identifyParticipants", "fetchAvailability"]
    },
    "serverContext1": {
      "title": "Data Retrieval and Processing",
      "description": "Server fetches and processes calendar availability data",
      "type": "object",
      "properties": {
        "FetchAvailability_Activity": {
          "title": "Calendar Availability Results",
          "references": ["llmContext1.fetchAvailability"],
          "type": "object",
          "properties": {
            "organizerSlots": {
              "type": "array",
              "items": { "type": "string", "format": "date-time" },
              "description": "Available time slots for the organizer."
            },
            "attendeeSlots": {
              "type": "array",
              "items": { "type": "string", "format": "date-time" },
              "description": "Available time slots for the attendee."
            }
          },
          "required": ["organizerSlots", "attendeeSlots"]
        }
      },
      "required": ["FetchAvailability_Activity"]
    },
    "llmContext2": {
      "title": "Meeting Time Selection and Communication Draft",
      "description": "LLM analyzes availability and prepares meeting invitation",
      "type": "object",
      "properties": {
        "findCommonSlot": {
          "title": "Find Common Available Slot",
          "references": [
            "llmContext1.identifyParticipants",
            "serverContext1.FetchAvailability_Activity"
          ],
          "type": "object",
          "properties": {
            "selectedSlot": {
              "type": "string",
              "format": "date-time",
              "description": "The chosen meeting time that works for both participants."
            },
            "reasoning": {
              "type": "string",
              "description": "Explanation for why this slot was chosen."
            }
          },
          "required": ["selectedSlot", "reasoning"]
        },
        "draftInvitation": {
          "title": "Draft Invitation",
          "references": ["llmContext1.identifyParticipants", "findCommonSlot"],
          "type": "object",
          "properties": {
            "subject": { "type": "string" },
            "body": {
              "type": "string",
              "description": "The full text of the invitation email."
            }
          },
          "required": ["subject", "body"]
        }
      },
      "required": ["findCommonSlot", "draftInvitation"]
    },
    "userContext": {
      "title": "User Decision Point",
      "description": "Awaits user approval of the proposed meeting details",
      "type": "object",
      "properties": {
        "confirmInvitation": {
          "title": "Get User Confirmation",
          "references": ["llmContext2.draftInvitation"],
          "type": "object",
          "properties": {
            "decision": {
              "type": "string",
              "enum": ["Approve", "Reject"],
              "description": "The user's decision on the proposed meeting."
            }
          },
          "required": ["decision"]
        }
      },
      "required": ["confirmInvitation"]
    },
    "serverContext2": {
      "title": "Final Communication Dispatch",
      "description": "Server sends out the approved meeting invitation",
      "type": "object",
      "properties": {
        "sendInvitation": {
          "title": "Send Invitation",
          "references": [
            "llmContext1.identifyParticipants",
            "llmContext2.draftInvitation",
            "userContext.confirmInvitation"
          ],
          "type": "object",
          "properties": {
            "messageId": { "type": "string" },
            "status": { "type": "string" }
          },
          "required": ["messageId", "status"]
        }
      },
      "required": ["sendInvitation"]
    }
  },
  "required": ["llmContext1", "serverContext1", "llmContext2", "userContext", "serverContext2"]
}
```

### Focused Data Flow with `references`

The `references` meta-property seen in the schema above is the key to managing data flow and context size. The workflow engine uses this property to inject only the necessary data from previous steps into the LLM prompt for the current step.

Let's look at a focused example. Imagine a step `step5_draft_email` that needs the user's name from a previous step, `step4_find_user`. The schema would look like this:

```json
"step5_draft_email": {
    "description": "Draft a personalized email to the user found in the previous step.",
    "references": [ "step4_find_user.output" ],
    "type": "object",
    "properties": {
        "recipientName": {
            "description": "Use the 'userName' field from the 'step4_find_user.output' object provided in the prompt context.",
            "type": "string"
        },
        "emailBody": { "type": "string" }
    }
}
```

The workflow engine processes this:

1.  It sees `references: [ "step4_find_user.output" ]`.
2.  It retrieves the entire output object `{ "userId": "...", "userName": "..." }` from its state.
3.  It constructs a minimal context and injects it into the prompt for the LLM.

<pre>
--- Context from previous steps ---
{
    "step4_find_user": {
        "output": {
            "userId": "u-12345",
            "userName": "Jane Doe"
        }
    }
}
--- End Context ---

Please generate the JSON for the `step5_draft_email` step using the information provided above.
</pre>

4.  It sends the schema for `step5_draft_email` to the LLM, but **without the `references` property**. The LLM then uses the provided context to follow the instructions in the `description` fields. This provides a clean and powerful mechanism for piping data from server-side actions back into the LLM's reasoning process.

This ensures the LLM receives only the information it needs, preventing context overload and keeping the interaction efficient. The large `Process: Schedule Meeting` schema uses this pattern extensively to pass information between LLM, server, and user contexts.

## The Modular Schema

Instead of exposing the entire complex process schema to the LLM, we create a tool specifically for _initiating_ the process. This tool is derived by taking the first LLM context from Example 1 and enhancing it with additional cognitive steps through Instruction composition.

Let's trace how this composition works, starting from the original process structure.

### Using Instructions for Cognitive Enhancement

Instructions can compose with each other because their schemas merge like JavaScript objects, and JSON Schema preserves property order. This enables macro-like Instructions that wrap other Instructions while injecting additional steps:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Introspection_Mixin",
  "description": "An instruction that enhances any schema with pre-action analysis and post-action reflection through natural composition",
  "type": "object",
  "properties": {
    "_considerations": {
      "type": "string",
      "description": "Before performing the action, analyze the request and outline key considerations."
    },
    "default": {
      "type": "object",
      "description": "This is a placeholder that will be replaced by the target's schema at runtime."
    },
    "_feedback": {
      "type": "string",
      "description": "After completing the action, provide feedback about the decisions made."
    }
  },
  "required": ["_considerations", "default", "_feedback"]
}
```

When this Instruction composes with another, the `default` property acts as a slot that gets replaced by the other schema's properties. Because property order is preserved, we get `_considerations` first, then all the properties from the composed schema, then `_feedback` - creating a macro-like wrapper that injects thinking steps around the main task.

### Composing the Tool

Here's how we transform the first LLM context from Example 1 into our tool:

Original Process first LLM context chunk looks like this:

```ts
const llmContext1 = FetchAvailability(IdentifyParticipants(prompt));
```

Vessel wraps it with process initiation and introspection:

```ts
StartProcess({
  name: 'ScheduleMeeting',
  llmContext1: Introspect_Mixin(FetchAvailability(IdentifyParticipants(prompt))),
});
```

Introspect_Mixin is a mixin instruction that accepts named properties:

```ts
StartProcess({
  name: 'ScheduleMeeting',
  llmContext1: Introspect_Mixin({
    default: {
      identifyParticipants: IdentifyParticipants(prompt),
      fetchAvailability: FetchAvailability({ participants }),
    },
  }),
});
```

This yields jsonschema composition where the instructions are composed together. Note how we use the `description` field to instruct how arguments should be passed between steps - for example, the `fetchAvailability` description indicates that it should reuse participant emails from the identification step. This approach allows for clear data flow between instruction components without requiring explicit parameter declarations.

```ts
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Start Scheduling Process",
  "description": "Initiates the meeting scheduling workflow with participant identification and availability checking",
  "type": "object",
  "allOf": [
    { "$ref": "aug:/vibes/macros/StartProcess:1" },
    {
      "properties": {
        "name": { "const": "ScheduleMeeting" },
        "llmContext1": {
          "type": "object",
          "allOf": [
            { "$ref": "aug:/vibes/macros/Introspect_Mixin" },
            { "$ref": "aug:/vibes/macros/Loop_Mixin", description: "Loop at most 3 trimes" },
            {
              "properties": {
                "default": {
                  "type": "object",
                  "properties": {
                    "identifyParticipants": {
                      "allOf": [
                        { "$ref": "aug:instructions/identifyParticipants" },
                        { "description": "Extracts and identifies participant information from the user's prompt, determining who should be included in the meeting" }
                      ]
                    },
                    "fetchAvailability": {
                      "allOf": [
                        { "$ref": "aug:processes/schedule-meeting:1#fetchAvailability" },
                        { "description": "Retrieves calendar availability data for identified participants, reusing participant emails from the identification step to check their schedules: llm_context_1.identifyParticipant" }
                      ]
                    }
                  }
                }
              }
            }
          ]
        }
      }
    }
  ]
}
```

This is what it looks like when mixin is expanded and llm does its job

> Prompt: Assistant, schedule a meeting between alice and bob!

```ts
StartProcess({
  name: 'ScheduleMeeting',
  llmContext1: {
    _considerations: "Need to check both participants' availability within business hours",
    identifyParticipants: {
      organizer: 'alice@example.com',
      attendee: 'bob@example.com',
    },
    fetchAvailability: {
      organizerEmail: 'alice@example.com', // Reused from identifyParticipants
      attendeeEmail: 'bob@example.com', // Reused from identifyParticipants
      timeRange: {
        start: '2024-03-20T09:00:00Z',
        end: '2024-03-20T17:00:00Z',
      },
    },
    _feedback:
      'Prepared to check availability for Alice and Bob during their workday. Once complete, the process will continue with finding common slots.',
  },
});
```

This shows how the schema's structure guides the LLM to:

1. Think through considerations before acting (`_considerations`)
2. Provide complete, well-structured data for the initial operations
3. Reuse participant emails directly from the identification step
4. Reflect on both the immediate step and the process continuation (`_feedback`)
5. Start the `ScheduleMeeting` process explicitly on the server and continue there

### The Composed Tool Schema

This is the final schema presented to the LLM, with all references resolved. It merges vessel's own adjustments with tool definitions (e.g. instructions on how to pass output from one step to another)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Vessel Toolkit: Meeting Scheduler",
  "description": "A set of tools available to the Vessel for scheduling meetings.",
  "type": "object",
  "properties": {
    "name": {
      "type": "string",
      "const": "ScheduleMeeting",
      "description": "Identifies the process being started"
    },
    "llmContext1": {
      "type": "object",
      "properties": {
        "_considerations": {
          "type": "string",
          "description": "Before performing the action, analyze the request and outline key considerations."
        },
        "identifyParticipants": {
          "title": "Identify Meeting Participants",
          "type": "object",
          "description": "Identifies meeting participants from user input\nExtracts and identifies participant information from the user's prompt, determining who should be included in the meeting",
          "properties": {
            "organizer": {
              "type": "string",
              "format": "email",
              "description": "Email of the person initiating the meeting."
            },
            "attendee": {
              "type": "string",
              "format": "email",
              "description": "Email of the person being invited."
            }
          },
          "required": ["organizer", "attendee"]
        },
        "fetchAvailability": {
          "title": "Fetch Participant Availability",
          "type": "object",
          "description": "Fetches calendar availability for specified participants\nRetrieves calendar availability data for identified participants, reusing participant emails from the identification step to check their schedules",
          "properties": {
            "organizerEmail": {
              "type": "string",
              "format": "email",
              "description": "Email of organizer"
            },
            "attendeeEmail": {
              "type": "string",
              "format": "email",
              "description": "Email of attendee."
            },
            "timeRange": {
              "type": "object",
              "properties": {
                "start": {
                  "type": "string",
                  "format": "date-time",
                  "description": "The start of the time range to check for availability."
                },
                "end": {
                  "type": "string",
                  "format": "date-time",
                  "description": "The end of the time range to check for availability."
                }
              },
              "required": ["start", "end"]
            }
          },
          "required": ["organizerEmail", "attendeeEmail", "timeRange"]
        },
        "_feedback": {
          "type": "string",
          "description": "After completing the action, provide feedback about the decisions made."
        }
      },
      "required": ["_considerations", "identifyParticipants", "fetchAvailability", "_feedback"]
    }
  },
  "required": ["name", "llmContext1"]
}
```
