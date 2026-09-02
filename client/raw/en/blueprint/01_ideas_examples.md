### Examples of Vibe Structures

This document provides concrete examples to illustrate the fundamental structure of the different Vibe types discussed in "Chapter 1: Vibes & Their Manifestations." Each example focuses on showcasing a Vibe as a static record, comprising its `input`, its `schema` field (which contains the Vibe's own structural blueprint or operational definition), and its `solution` (the content, outcome, or configuration that conforms to that `schema` definition).

How these Vibes are created, evolve, or interact dynamically (e.g., through the `refine` primitive) is detailed in later chapters. The purpose here is to clarify the distinct nature and typical content of each Vibe type's core components.

### Understanding Schema References (`$ref`)

Vibe schemas often reference other schemas using the `$ref` keyword. These references follow a URI-like format, typically starting with `aug:`.

- **Absolute vs. Local References:**
  - An **absolute reference** to a globally shared schema starts with a slash directly after the scheme, e.g., `aug:/schemas/SharedSchemaName`. These are for common, system-wide definitions.
  - A **local reference** (without a leading slash after the scheme, e.g., `aug:schemas/UserSpecificSchema`) is considered context-dependent (e.g., user-specific, or relative to the current Vibe's namespace). For the examples in this document, most shared base schemas will use absolute paths.
- **Evergreen vs. Versioned References:**
  - To refer to the latest version of a schema (an **evergreen reference**), omit the version number at the end, e.g., `aug:/schemas/BaseVibe`. This is typical for extending base templates, ensuring they always use the most up-to-date definition of the base.
  - To lock into a specific version of a schema, include the version number, e.g., `aug:/schemas/SpecificDefinition:2`. This is useful for referencing specific tools or instruction sets where a particular version is critical.

Conceptually similar Vibes (e.g., all "UserProfile" Record Vibes, or all Vibes defining a "StandardProcess") would typically share identical or very similar `schema` definitions within their respective `schema` fields, ensuring structural consistency for Vibes of the same kind.

---

### 1. Record Vibe Example: User Profile

**Scenario:** A Vibe representing a user's profile information. Record Vibes are primarily used for structured data.

**Illustrative `UserProfileRecordVibe` JSON Structure (e.g., `aug:users/user-789?1`):**

```json
{
  "input": [
    {
      "type": "UserProfileContext",
      "$id": "aug:user-789",
      "query": "Initialize or update the user profile. If preferences are not fully known or provided, attempt to infer sensible defaults or suggest options.",
      "state": {
        "displayName": "Jane Doe",
        "email": "jane.doe@example.com"
        // 'preferences' field is deliberately omitted from state to allow LLM to infer or suggest.
      }
    }
  ],
  "schema": {
    "title": "User Profile Record",
    "description": "Defines the complete structure for a user's profile information, including identity, metadata, and user-specific details. Composed from base Vibe and RecordVibe schemas using evergreen references.",
    "allOf": [
      {
        "$ref": "aug:/schemas/Vibe",
        "$comment": "Base Vibe structure, referenced without a version to always use the latest. Provides properties like $id, $type, and $revision."
      },
      {
        "$ref": "aug:/schemas/RecordVibe",
        "$comment": "Base RecordVibe structure, referenced without a version. Provides common record properties like id, createdAt, lastModified."
      },
      {
        "type": "object",
        "title": "Vessel Details",
        "description": "Specific information and preferences associated with the user.",
        "properties": {
          "displayName": {
            "type": "string",
            "title": "Display Name",
            "description": "The name publicly shown for the user."
          },
          "email": {
            "type": "string",
            "format": "email",
            "title": "Email Address",
            "description": "The user's primary email address for communication and login."
          },
          "preferences": {
            "type": "object",
            "title": "User Preferences",
            "description": "Settings and choices made by the user to customize their experience.",
            "properties": {
              "newsletter": {
                "type": "boolean",
                "default": false,
                "title": "Newsletter Subscription",
                "description": "Indicates if the user wishes to receive newsletter updates."
              },
              "theme": {
                "type": "string",
                "enum": ["light", "dark"],
                "default": "light",
                "title": "Interface Theme",
                "description": "The selected visual theme for the user interface (e.g., light or dark)."
              }
            }
          }
          // id, createdAt, lastModified are inherited from RecordVibe layer
          // $id, $revision, $type are inherited from Vibe layer
        },
        "required": ["displayName", "email"]
      }
    ]
    // The effective 'required' list would be the union of 'required' from all layers:
    // e.g., ["$type", "$id", "$revision", "createdAt", "lastModified", "designation", "primaryCapability"]
  },
  "solution": {
    "$type": "UserProfileRecordVibe", // from Vibe layer
    "$id": "aug:vibes/user-profile-789", // from Vibe layer, this is the vibe's unique ID
    "$revision": 3, // from Vibe layer
    "id": "user-789", // from RecordVibe layer, this is the actual user record ID
    "displayName": "Jane Doe", // from User Details
    "email": "jane.doe@example.com", // from User Details
    "preferences": {
      // from User Details, potentially inferred/completed by LLM
      "newsletter": false,
      "theme": "light"
    },
    "createdAt": "2025-01-15T10:00:00Z", // from RecordVibe layer
    "lastModified": "2025-03-10T12:05:00Z" // from RecordVibe layer
  }
}
```

**Explanation:**

- The `input` is now an array of objects. The first object provides context (`$id`), a `query` for the LLM (replacing `directive`), and initial `state` (replacing `initialData`). The `query` suggests inferring missing `operationalParameters`.
- The `schema` field now has a top-level `allOf` array:
  - It `$ref`erences `aug:/schemas/Vibe` (providing $type).
  - It `$ref`erences `aug:/schemas/RecordVibe` (providing $id, $revision, `createdAt`, `lastModified`).
  - It includes an inline object defining vessel-specific details (`designation`, `primaryCapability`, `operationalParameters`, `capabilities`) with UI-friendly `title` and `description` fields.
- The overall `title` and `description` for the composed `VesselProfileRecord` schema are at the top level of the `schema` object.
- Comments indicate which properties are expected from the referenced layers.
- The `solution` includes $type (from `Vibe`) and $id (from `RecordVibe`), and the `operationalParameters` in the solution might be the result of LLM inference if they were not in the input state.

This structure makes the inheritance chain explicit and uses evergreen references. The `title` and `description` fields are aimed at being clear for both UIs and LLMs.

The referenced base schemas (`aug:/schemas/Vibe` and `aug:/schemas/RecordVibe`) would be defined elsewhere. For example:

**Hypothetical definition for `aug:/schemas/Vibe` (e.g., if its latest version is 1, it might be stored as `aug:/schemas/Vibe?1` but referenced as `aug:/schemas/Vibe` for evergreen use):**

---

### 2. Role Vibe Example: "Investment Advisor" Vessel

**Scenario:** This Vibe illustrates an Investment Advisor vessel - a specific instance that processes queries. The `input` contains the query and any initial state (like specific configurations for this vessel, such as `specialization` or `clientContext`). The `schema` defines the structure of this vessel by extending RoleVibe and adding specific capabilities and outputs. The `solution` showcases the results produced by this vessel: based on the input query, some capabilities produce data, while others are `null` when not relevant to the specific request.

**Illustrative `InvestmentAdvisorVesselVibe` JSON Structure (e.g., `aug:vessels/advisor-vessel-acme-001`):**

```json
{
  "input": [
    {
      "type": "text",
      "text": "You are financial assistant",
      "role": "system"
    },
    {
      "type": "query",
      "query": "Provide a quick market analysis of ACME Corp's quantum computing breakthrough impact."
    },
    {
      "type": "state",
      "state": {
        "specialization": "Technology Sector Analysis",
        "analysisDepth": "quick"
      }
    }
  ],
  "schema": {
    "title": "Investment Advisor Vessel",
    "description": "Represents a vessel instance of the Investment Advisor Role. It inherits common properties from base schemas and defines the specific outputs pertinent to this vessel.",
    "allOf": [
      {
        "$ref": "aug:/schemas/Vibe",
        "$comment": "Base Vibe structure. Provides the fundamental $id, $type, and $revision properties for this vessel Vibe."
      },
      {
        "$ref": "aug:/schemas/RoleVibe",
        "$comment": "Base RoleVibe structure. Provides common role properties like displayName, description, createdAt, lastModified. By extending this, we're defining this as a role-based Vibe."
      },
      {
        "type": "object",
        "title": "Vessel Output",
        "description": "Defines the specific outputs resulting from this Investment Advisor vessel's processing.",
        "properties": {
          "specializationUsed": {
            "type": "string",
            "title": "Active Specialization",
            "description": "The area of focus that was active for this specific vessel."
          },
          "analysisDepthUsed": {
            "type": "string",
            "enum": ["quick", "standard", "comprehensive"],
            "title": "Analysis Depth",
            "description": "The level of detail used in the analysis."
          },
          "_IntroductionNarrative": {
            "type": "string",
            "format": "markdown",
            "title": "Report Introduction",
            "description": "Opening narrative that introduces the analysis context and sets the stage."
          },
          "MarketAnalysis": {
            "type": ["object", "null"],
            "title": "Market Analysis",
            "description": "The market analysis findings. Null if not performed.",
            "properties": {
              "summary": {
                "type": "string",
                "title": "Analysis Summary",
                "description": "Main findings from the market analysis."
              },
              "sentiment": {
                "type": "string",
                "enum": ["very-negative", "negative", "neutral", "positive", "very-positive"],
                "title": "Market Sentiment",
                "description": "Overall market sentiment assessment."
              },
              "impactScore": {
                "type": "number",
                "minimum": 0,
                "maximum": 10,
                "title": "Impact Score",
                "description": "Numerical score representing the potential market impact."
              }
            },
            "required": ["summary", "sentiment", "impactScore"]
          },
          "_RecommendationTransition": {
            "type": "string",
            "format": "markdown",
            "title": "Recommendation Context",
            "description": "Narrative that bridges from analysis to recommendations.",
            "const": "Based on the market analysis above, the following investment recommendation takes into account both the immediate opportunities and the inherent risks in emerging technology investments."
          },
          "InvestmentRecommendation": {
            "type": ["object", "null"],
            "title": "Investment Recommendation",
            "description": "The final investment recommendation. Null if not requested.",
            "properties": {
              "recommendation": {
                "type": "string",
                "enum": ["strong-buy", "buy", "hold", "sell", "strong-sell"],
                "title": "Recommendation",
                "description": "The investment action recommendation."
              },
              "rationale": {
                "type": "string",
                "title": "Rationale",
                "description": "Explanation for the recommendation."
              },
              "confidenceLevel": {
                "type": "number",
                "minimum": 0,
                "maximum": 1,
                "title": "Confidence Level",
                "description": "Confidence in the recommendation (0-1)."
              },
              "timeHorizon": {
                "type": "string",
                "title": "Time Horizon",
                "description": "Suggested investment time horizon."
              }
            },
            "required": ["recommendation", "rationale", "confidenceLevel", "timeHorizon"]
          },
          "RiskAssessment": {
            "type": ["object", "null"],
            "title": "Detailed Risk Assessment",
            "description": "Comprehensive risk analysis. Null if not requested or if analysis depth is 'quick'.",
            "properties": {
              "overallRiskLevel": {
                "type": "string",
                "enum": ["very-low", "low", "moderate", "high", "very-high"],
                "title": "Overall Risk Level"
              },
              "primaryRiskFactors": {
                "type": "string",
                "title": "Primary Risk Factors",
                "description": "Key risks identified in the investment."
              },
              "mitigationStrategies": {
                "type": "string",
                "title": "Risk Mitigation Strategies",
                "description": "Recommended approaches to manage identified risks."
              }
            },
            "required": ["overallRiskLevel", "primaryRiskFactors", "mitigationStrategies"]
          },
          "_ConclusionNarrative": {
            "type": "string",
            "format": "markdown",
            "title": "Report Conclusion",
            "description": "Closing narrative that summarizes key takeaways."
          }
        },
        "required": [
          "specializationUsed",
          "analysisDepthUsed",
          "_IntroductionNarrative",
          "MarketAnalysis",
          "_RecommendationTransition",
          "InvestmentRecommendation",
          "RiskAssessment",
          "_ConclusionNarrative"
        ]
      }
    ]
  },
  "solution": {
    "$type": "InvestmentAdvisorVesselVibe",
    "$id": "aug:vessels/advisor-vessel-acme-001",
    "$revision": 1,
    "createdAt": "2025-03-18T10:00:00Z",
    "lastModified": "2025-03-18T10:05:00Z",
    "displayName": "ACME Corp Quantum Impact Analysis",
    "description": "Quick market analysis of ACME Corp's quantum computing breakthrough.",

    "specializationUsed": "Technology Sector Analysis",
    "analysisDepthUsed": "quick",

    "_IntroductionNarrative": "## Investment Analysis Report\n\n*Generated Analysis*\n\nThis report provides market analysis and investment recommendations based on the requested evaluation. The analysis depth and focus areas are tailored to the specific query parameters provided.",

    "MarketAnalysis": {
      "summary": "ACME Corp's quantum computing breakthrough represents a significant technological leap with potential to disrupt multiple sectors including AI, pharmaceuticals, and cryptography. The 'QuantumLeap' chip shows 1000x performance improvement over current solutions, positioning ACME as a potential leader in the $65 billion quantum computing market projected by 2030.",
      "sentiment": "very-positive",
      "impactScore": 8.5
    },

    "_RecommendationTransition": "Based on the market analysis above, the following investment recommendation takes into account both the immediate opportunities and the inherent risks in emerging technology investments.",

    "InvestmentRecommendation": {
      "recommendation": "buy",
      "rationale": "ACME's breakthrough positions them as a leader in the emerging quantum computing market. First-mover advantage and strong IP portfolio support positive outlook despite typical early-stage technology risks.",
      "confidenceLevel": 0.75,
      "timeHorizon": "12-18 months"
    },

    "RiskAssessment": null,

    "_ConclusionNarrative": "## Key Takeaways\n\nThis analysis has evaluated the investment opportunity based on the available information and specified analysis parameters. The recommendations provided should be considered within the context of your overall investment strategy and risk tolerance.\n\n*Note: This analysis was conducted at the requested depth level. Additional capabilities may be available for more comprehensive evaluation.*"
  }
}
```

**Explanation:**
This enhanced Vibe represents an **Investment Advisor Vessel** that produces article-like output:

- The `input` array contains:
  - System context
  - User query
  - State configuration with `specialization` and `analysisDepth` set to "quick"

- The `schema` uses `allOf` composition with:
  - Base schemas for fundamental properties
  - **Narrative blocks** that structure the output:
    - `_IntroductionNarrative`: Generic opening that provides consistent report structure
    - `_RecommendationTransition`: Fixed transition text (using `const`)
    - `_ConclusionNarrative`: Generic closing that provides consistent report structure
  - **Three capabilities**:
    - `MarketAnalysis`: Core analysis capability
    - `InvestmentRecommendation`: Recommendation based on analysis
    - `RiskAssessment`: **Not activated** in this example (null) because the query requested "quick" analysis

- The `solution` demonstrates:
  - **Article-like flow**: Reading from top to bottom, the narrative blocks and data sections create a cohesive report
  - **Selective capability usage**: `RiskAssessment` is `null` because it's only activated for "standard" or "comprehensive" analysis depths
  - **Markdown formatting** in narrative blocks for proper document structure
  - **Separation of structure and content**: Narrative blocks provide consistent vessel structure, while only the actual analysis capabilities (MarketAnalysis, InvestmentRecommendation) contain query-specific content (e.g., ACME Corp details)

This pattern shows how vessels can produce well-structured, readable outputs while intelligently activating only the capabilities relevant to the specific request.

**Selective Capability Usage Pattern:**
The vessel intelligently responds to the input query by only using relevant capabilities. This allows the same Role Blueprint to handle various types of requests efficiently:

- Full analysis requests would use all capabilities
- Quick analysis requests (like this example) use only essential capabilities
- Specific tool requests could use just those specific capabilities
  This pattern ensures efficient processing and clear, focused outputs aligned with user intent.

**Narrative Blocks Design Pattern:**

- **All narrative blocks** (properties starting with underscore): Provide consistent structural narrative for the vessel, independent of the specific query. They define the vessel's report framework.
  - `_IntroductionNarrative`: Generic introduction for any investment analysis report
  - `_RecommendationTransition`: Fixed transition text (using `const` in schema)
  - `_ConclusionNarrative`: Generic conclusion for any investment analysis report
- **Query-specific content** appears only in the capability outputs (MarketAnalysis, InvestmentRecommendation, etc.), not in narrative blocks
- This separation ensures the vessel maintains a consistent structural identity while adapting its analysis content to different queries

---

### 3. Capability Vibe Example: Resource Allocation and Task Delegation

**Scenario:** Capability Vibes represent delegated authority with integrated resource allocation. They serve as both the authorization to perform work and the specification of what work can be done, with resources allocated directly within the capability rather than through separate Resource Vibes.

#### 3.1 Budget Allocation Capability: Marketing Campaign

This capability represents authority to spend marketing budget with specific constraints and allocations.

**Illustrative `MarketingCampaignBudgetCapability` JSON Structure (e.g., `aug:capabilities/marketing-campaign-q1?1`):**

```json
{
  "input": [
    {
      "type": "query",
      "query": "Create a capability that delegates marketing campaign authority with $50K budget allocation"
    }
  ],
  "schema": {
    "title": "Marketing Campaign Budget Capability",
    "description": "A capability that authorizes marketing campaign activities with allocated budget and resource constraints.",
    "allOf": [
      {
        "$ref": "aug:/schemas/Vibe",
        "$comment": "Base Vibe structure"
      },
      {
        "$ref": "aug:/schemas/CapabilityVibe",
        "$comment": "Base CapabilityVibe structure"
      },
      {
        "type": "object",
        "title": "Budget Allocation Structure",
        "description": "Defines the budget allocation parameters and constraints",
        "properties": {
          "authority": {
            "type": "object",
            "title": "Delegation Authority",
            "description": "The source and scope of authority for this capability",
            "properties": {
              "grantedBy": {
                "type": "string",
                "title": "Granting Authority",
                "description": "The vibe ID of the authority granting this capability"
              },
              "scope": {
                "type": "string",
                "enum": ["organization", "department", "project", "personal"],
                "title": "Authority Scope"
              },
              "expiresAt": {
                "type": "string",
                "format": "date-time",
                "title": "Authority Expiration"
              }
            },
            "required": ["grantedBy", "scope"]
          },
          "budget": {
            "type": "object",
            "title": "Allocated Budget",
            "description": "Direct resource allocation within this capability",
            "properties": {
              "totalUsd": {
                "type": "number",
                "minimum": 0,
                "maximum": 50000,
                "title": "Total USD Budget"
              },
              "usedUsd": {
                "type": "number",
                "minimum": 0,
                "default": 0,
                "title": "USD Already Used"
              },
              "remainingUsd": {
                "type": "number",
                "minimum": 0,
                "title": "USD Remaining"
              },
              "allocations": {
                "type": "object",
                "title": "Budget Breakdown",
                "properties": {
                  "contentCreation": {
                    "type": "number",
                    "minimum": 0,
                    "maximum": 20000,
                    "title": "Content Creation Budget"
                  },
                  "advertising": {
                    "type": "number",
                    "minimum": 0,
                    "maximum": 25000,
                    "title": "Advertising Budget"
                  },
                  "analytics": {
                    "type": "number",
                    "minimum": 0,
                    "maximum": 5000,
                    "title": "Analytics Budget"
                  }
                },
                "required": ["contentCreation", "advertising", "analytics"]
              }
            },
            "required": ["totalUsd", "usedUsd", "remainingUsd", "allocations"]
          },
          "constraints": {
            "type": "object",
            "title": "Spending Constraints",
            "properties": {
              "maxSingleTransaction": {
                "type": "number",
                "maximum": 10000,
                "title": "Maximum Single Transaction"
              },
              "approvalRequired": {
                "type": "boolean",
                "title": "Requires Approval for Large Spending"
              },
              "approvalThreshold": {
                "type": "number",
                "title": "Threshold for Approval Requirement"
              }
            }
          }
        },
        "required": ["authority", "budget", "constraints"]
      }
    ]
  },
  "solution": {
    "$type": "CapabilityVibe",
    "$id": "aug:capabilities/marketing-campaign-q1-2025",
    "$revision": 1,
    "createdAt": "2025-01-10T09:00:00Z",
    "lastModified": "2025-01-10T09:00:00Z",
    "displayName": "Marketing Campaign Q1 Budget Authority",
    "description": "Authority to execute marketing campaigns with $50K allocated budget",

    "authority": {
      "grantedBy": "aug:roles/marketing-director-sarah",
      "scope": "department",
      "expiresAt": "2025-04-01T00:00:00Z"
    },

    "budget": {
      "totalUsd": 50000,
      "usedUsd": 0,
      "remainingUsd": 50000,
      "allocations": {
        "contentCreation": 15000,
        "advertising": 25000,
        "analytics": 10000
      }
    },

    "constraints": {
      "maxSingleTransaction": 10000,
      "approvalRequired": true,
      "approvalThreshold": 5000
    }
  }
}
```
