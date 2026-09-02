### Detailed Example: Evolving Record Vibe Schemas

The "schema refinement principle" discussed during bootstrapping—making schemas more specific—isn\'t limited to the initial setup of the system. It\'s a fundamental aspect of how the system adapts and evolves. Just as Capabilities can authorize changes _to the data_ within a Record Vibe\'s `solution`, they can also authorize changes _to the very structure_ defined in a Record Vibe\'s own `schema` field. This is achieved by `Refine`ing a Record Vibe with an `instruction` Vibe, whose `solution` defines the evolution of the target Vibe. This process results in a new Record Vibe with an evolved `schema` field and a `solution` conforming to this new structure. This allows for controlled evolution of data structures as business needs change, aligning with the content-first philosophy where interactions with content drive its transformation.

Consider the scenario of evolving a simple promotional website into a full-fledged e-commerce platform.

**Initial State:**
The company initially has Record Vibes for its products, such as `aug:products/superwidget?1` (where `?1` means revision). These Vibes conform to the structure defined in an exemplar Vibe, say `aug:products/example?1` (representing a basic product structure).

- The `schema` field of `aug:products/example?1` (and thus of `aug:products/superwidget?1`) holds the JSON Schema definition for this initial basic product structure.
- The `solution` field of `aug:products/example?1` contains placeholder/example data for a basic product.

This initial structure (`aug:products/example?1#schema`) might look like this:

```json
// Content of the `schema` field for `aug:products/example?1` and its derivatives
{
  "type": "object",
  "properties": {
    "productName": { "type": "string" },
    "description": { "type": "string" },
    "promoImage": { "type": "string", "format": "uri" }
  },
  "required": ["productName", "description"]
}
```

**The Business Goal & Required Schema Change:**
As the business decides to launch a full online shop, a critical new requirement emerges: tracking inventory and price. This necessitates evolving product Record Vibes to include `stockLevel` and `price` fields, meaning the JSON Schema definition in their `schema` field must change.

**The `Refine` Operation: Evolving the Vibe\'s Schema and Solution**
To implement this, a `Refine` operation targets the existing `aug:products/superwidget?1` (let\'s consider its initial state to be `aug:products/superwidget?1`):

- **`target`**: `aug:products/superwidget?1`.
- **`instruction`**: An `aug:migrations/evolve-to-shop-item?1` Vibe. The `solution` of this instruction Vibe is crucial; it specifies how to modify the JSON Schema definition found in the target Vibe\'s `schema` field, and how the target\'s `solution` (data) should adapt:
  ```json
  // Example: Content of aug:migrations/evolve-to-shop-item?1\'s solution
  // The format does not matter
  {
    "schemaModifications": {
      "addProperties": {
        "stockLevel": { "type": "integer", "minimum": 0, "description": "Current available stock" },
        "price": { "type": "number", "format": "currency", "description": "Retail price" }
      },
      "addToRequired": ["stockLevel", "price"]
    },
    "solutionUpdate": {
      "stockLevel": 0, // Initialize new stockLevel field in the solution
      "price": "0.00" // Initialize new price field in the solution
    }
  }
  ```
- **`capability`**: A Capability held by a role like "System Architect," authorizing the refinement of basic products to shop-ready products.

The `Refine` call would effectively be:

`aug:products/superwidget?2 = Refine(aug:products/superwidget?1, aug:migrations/evolve-to-shop-item?1, aug:capability-architect-master?1)`

This operation results in a new revision of the same conceptual Vibe (`aug:products/superwidget?1`) because the `migrations/evolve-to-shop-item?1` leads to a schema that is a compatible refinement (more specific version) of the original. This means that any system or process that was designed to understand the basic product structure can still understand this new, evolved structure; it would simply ignore the new `stockLevel` and `price` fields if it wasn\'t programmed to use them. The system\'s ledger records this as the next revision for `aug:products/superwidget?1`. Had the schema change been fundamentally incompatible, it would typically result in a Vibe with a new distinct identity.

The result is a **new revision** of the Vibe, now identifiable as `aug:products/superwidget?2` (the system manages actual versioning internally, this notation is for example clarity). This new revision _is_ the shop-ready version:

- Its `schema` field now contains the new, evolved JSON Schema definition (the "Shop Product Structure"):
  ```json
  // Content of the `schema` field for aug:products/superwidget?2
  {
    "type": "object",
    "properties": {
      "productName": { "type": "string" },
      "description": { "type": "string" },
      "promoImage": { "type": "string", "format": "uri" },
      "stockLevel": { "type": "integer", "minimum": 0, "description": "Current available stock" },
      "price": { "type": "number", "format": "currency", "description": "Retail price" }
    },
    "required": ["productName", "description", "stockLevel", "price"]
  }
  ```
- Its `solution` field now contains the original data plus the newly initialized `stockLevel` and `price` fields, conforming to this new "Shop Product Structure":
  ```json
  // aug:products/superwidget?2#solution
  {
    "productName": "Super Widget",
    "description": "The best widget for your needs.",
    "promoImage": "http://example.com/superwidget.png",
    "stockLevel": 0,
    "price": "0.00"
  }
  ```
  The original `aug:products/superwidget?1` remains untouched in the ledger. Other existing basic product Vibes (like `aug:products/anotheritem?1`) would undergo similar `Refine` operations to evolve them into their respective shop-ready iterations (e.g., `aug:products/anotheritem?2`).

**Subsequent Data Updates under the Evolved Schema:**
With `aug:products/superwidget?2` now having the "Shop Product Structure" in its `schema` field, an "Inventory Manager" can be granted capabilities to update its `solution`.

- **`target`**: `aug:products/superwidget?2`.
- **`instruction`**: An `aug:updates/update-stock?1` Vibe, whose `solution` specifies `{ "stockLevel": 150 }`.
- **`capability`**: A Capability held by the "Inventory Manager."

The `Refine` call:
`aug:products/superwidget?3 = Refine(aug:products/superwidget?2, aug:updates/update-stock?1, aug:capability-inventory-mgr?1)`

This produces `aug:products/superwidget?3`, a new revision of the product Vibe with the updated stock count in its `solution`. Its `schema` field remains the "Shop Product Structure" (as defined in `aug:products/superwidget?2`).

### Advanced Schema Evolution: Migrating to a New Schema Version

The previous example showed additive schema refinement. However, when a more fundamental "breaking" change is needed (e.g., renaming a field, significantly altering its type, or removing it entirely), a robust strategy is to migrate product instances to conform to a new schema version. This aligns with the content-first principle: the new schema isn\'t an independent, abstract Vibe but is defined directly within the `schema` field of the new product Vibe instances.

The core idea is to use an old product Vibe\'s data (as an `instruction`) to populate and finalize a new product Vibe (`target`) that is already structured according to the new schema version and represents the specific entity being migrated.

**Scenario: Introducing `priceInCents` and removing `price`**

Let\'s say our existing V1 product Vibes (e.g., `aug:products/superwidget?3`) were created based on an exemplar Vibe, named `aug:products/example?1`.

- The `schema` field of `aug:products/example?1` (and thus of `aug:products/superwidget?3`) defines a V1 product structure that includes a `price` string field.
- The `solution` field of `aug:products/example?1` would contain example placeholder values for a V1 product.

We now want to migrate these V1 products to a new V2 structure, represented by a new exemplar Vibe, `aug:products/example-v2?1`. This new V2 structure will use an integer `priceInCents` field for better precision and will omit the old `price` field.

1.  **Conceptualize the `aug:products/example-v2?1` Exemplar Vibe**:
    This Vibe establishes the pattern for all V2 products. It would encapsulate:

        - **`input`**: Example or typical input fields relevant for `aug:products/example-v2?1` (e.g., `{"id": "generic-products/example-v2-guid", "productId": "PROD-V2-EXEMPLAR"}`).
        - **`schema`**: This field contains the new "ProductSchemaV2" JSON definition:
          ```json
          // "ProductSchemaV2" - to be embedded in the .schema field of `aug:products/example-v2?1` and its derivatives
          {
            "type": "object",
            "properties": {
              "productId": {
                "type": "string",
                "description": "Unique business identifier for the product"
              },
              "productName": { "type": "string" },
              "description": { "type": "string" },
              "promoImage": { "type": "string", "format": "uri" },
              "stockLevel": { "type": "integer", "minimum": 0 },
              "priceInCents": {
                "type": "integer",
                "minimum": 0,
                "description": "Retail price in cents"
              }
            },
            "required": ["productId", "productName", "description", "stockLevel", "priceInCents"]
          }
          ```
        - **`solution`**: An example `solution` conforming to "ProductSchemaV2", showing default or typical V2 data for `aug:products/example-v2?1`:
          ```json
          {
            "productId": "PROD-V2-EXEMPLAR",
            "productName": "Generic Product V2",
            "description": "This is a standard Version 2 product.",
            "promoImage": "http://example.com/generic-v2.png",
            "stockLevel": 0,
            "priceInCents": 0
          }
          ```
          When migrating a specific product like SuperWidget, the `target` Vibe (e.g., `aug:products/superwidget-v2-base?1`) will be an instance or derivative that adopts this V2 structure (i.e., its `schema` field will be identical to `aug:products/example-v2?1#schema`, and its `solution` will have V2-specific fields based on `aug:products/example-v2?1#solution` and SuperWidget specifics).

2.  **The `Refine` Operation for Migration (Example: SuperWidget)**:
    To create a V2 Vibe for SuperWidget, conforming to the structure defined in `aug:products/example-v2?1#schema` and populated with data from `aug:products/superwidget?1`:

        - **`target`**: The `aug:products/example-v2?1` exemplar Vibe itself. This Vibe (defined in Step 1) provides the structural blueprint (its `schema` field) and default V2 values (its `solution` field) for the new Vibe to be created.

        - **`instruction`**:
          Generally, the `instruction` Vibe\'s `solution` provides the data or directives for the `Refine` operation. Its format is highly flexible and is not prescribed by the system; it could be natural language, executable code, structured data (like JSON or XML), or even before-and-after examples. The process authorized by the `capability` is responsible for interpreting this `solution`, potentially using tools like LLMs or specific parsers and logic.

          In this specific migration scenario, we illustrate a straightforward approach where the existing V1 product Vibe itself serves as the `instruction`. For migrating SuperWidget:
            - The `instruction` is `aug:products/superwidget?3` (the V1 SuperWidget Vibe, using the revision number from your prior edits).
            - Its `solution` (containing all V1 product data like original `productName`, `description`, `price`, `stockLevel`, etc.) directly provides the source material for the migration.

          The process authorized by the `capability` (see below) is then responsible for interpreting this V1 data. It will typically:
            - Use the `aug:products/example-v2?1#schema` (from the `target` Vibe) as the structural template for the new V2 product.
            - Extract relevant data from the `instruction` Vibe\'s `solution` (i.e., `aug:products/superwidget?3#solution`).
            - Perform necessary transformations (e.g., converting a `price` string to an integer `priceInCents`).
            - Populate the new V2 structure. Any V2-specific fields not directly derivable from the V1 data (e.g., a new `productId` for the V2 version if desired, or default values for entirely new V2 fields) would either be generated by this migration process logic or taken from the default `solution` of the `target` exemplar (`aug:products/example-v2?1#solution`).

        - **`capability`**: A Capability, say `aug:migrations/product#v1-to-v2`. This permit authorizes `Refine`ing the `aug:products/example-v2?1` exemplar using a V1 product Vibe (like `aug:products/superwidget?3`) as the instruction to produce a new V2 product Vibe. The logic for data transformation and V2 field population is encapsulated by the process acting under this capability.

        The `Refine` call would then result in a new Vibe with a distinct identity:
        `aug:products/superwidget-v2?1 = Refine(target: aug:products/example-v2?1, instruction: aug:products/superwidget?3, capability: aug:migrations/product#v1-to-v2)`

3.  **Result (SuperWidget)**:
    A **new** Record Vibe, `aug:products/superwidget-v2?1`, is created (note the new identity):

    - Its `schema` field is identical to `aug:products/example-v2?1#schema` (inherited from the `target` exemplar).
    - Its `solution` field is now populated according to `aug:products/example-v2?1#schema`. The data is derived by the migration process (authorized by the `capability`) using the V1 data from `aug:products/superwidget?3#solution` as the primary input:
      ```json
      // aug:products/superwidget-v2?1#solution
      {
        "productId": "SW-001", // Carried over from V1 to maintain product identity
        "productName": "SuperWidget Pro", // Potentially updated by the migration process
        "description": "The next generation SuperWidget.", // Potentially updated by the migration process
        "promoImage": "http://example.com/superwidget-v2.png", // Potentially updated by the migration process
        "stockLevel": 150, // Carried over from V1 data (aug:products/superwidget?3#solution)
        "priceInCents": 1999 // Transformed from V1 price (aug:products/superwidget?3#solution)
      }
      ```
    - The original Vibes (`aug:products/superwidget?3` and `aug:products/example-v2?1`) remain untouched.

    _(Note: The `productId` shown here is the business identifier for the product, defined by its schema and part of its `solution`. This is distinct from the Vibe\'s unique system/ledger UUID, which is metadata external to the `solution`.)_

4.  **Migrating Another Product (e.g., SuperGizmo)**:
    To migrate another V1 product, say `aug:products/supergizmo?1` (assuming it\'s a V1 product), to its V2 version:
    - The `target` remains the `aug:products/example-v2?1` exemplar Vibe.
    - The `instruction` would be `aug:products/supergizmo?1` itself.
    - The same `capability` (`aug:migrations/product#v1-to-v2`) can be used.
    - The `Refine` call:
      `aug:products/supergizmo-v2?1 = Refine(target: aug:products/example-v2?1, instruction: aug:products/supergizmo?1, capability: aug:migrations/product#v1-to-v2)`
      This creates `aug:products/supergizmo-v2?1` conforming to the schema defined in `aug:products/example-v2?1#schema`.

**Benefits of this Migration Approach:**

- **Clean Schema Versions**: New Vibes (`aug:products/superwidget?1`) are clean instances of the new schema version without carrying legacy fields directly in their structure.
- **Explicit Transformation Logic**: The `instruction` Vibe (or the process interpreting it, as authorized by a `capability`) explicitly defines the transformation or migration logic from V1 to V2. This logic is itself auditable and can be versioned.
- **Full Immutability and Lineage**: The historical Vibe (`aug:products/superwidget?1`) remains, providing a complete audit trail. The lineage of `aug:products/superwidget?1` back to its V1 origins is clear.
- **Phased Rollout Possible**: Systems consuming product data can be updated over time to understand "ProductSchemaV2". During a transition, some systems might read V1 Vibes (if they still exist and are relevant) and others V2 Vibes.

This strategy allows for robust and clean evolution of data structures, fully embracing the system\'s immutability and explicit transformation principles. It provides a structured way to handle significant schema changes beyond simple additive refinement.

**Chain of Authorization and Evolution:**
This example illustrates a chain:

1. A high-level business need (launching a shop) triggers...
2. Authorized personnel (System Architect) to perform schema evolution by `Refine`ing existing Record Vibes with specific `instruction` Vibes. The `solution` of these `instruction` Vibes dictates changes to both the `schema` field content (the structure) and the `solution` field content (the data) of the target Record Vibe, under an authorizing Capability.
3. This results in new Record Vibes, each containing the evolved structural definition in its `schema` field and corresponding data in its `solution` field.
4. This evolved structure then enables other roles (Inventory Manager) to be granted Capabilities to perform new types of data operations on the `solutions` of these evolved Record Vibes.

This capability-driven approach treats schema evolution as an integral part of data evolution. Changes to a Vibe\'s structure (its `schema` field) are managed through the same `Refine` primitive and permit-based authorization that governs all other transformations, ensuring that data structures can evolve in a controlled, auditable, and secure manner directly reflecting changes in business requirements.

---

## Practical Permission Management for `Refine` Operations

Permissions are granular and task-oriented, granted via specific permits. These permits can authorize actions on specific Vibe instances or, more powerfully, on any Vibe that conforms to a specified schema (as defined in an exemplar Vibe). Here are e-commerce examples illustrating this, with each scenario broken down into steps, using `aug:/...` for global/absolute paths and `aug:...` for local paths (implicitly within the current company context):

- **Product Managers & New Product Launch:**
  A Product Manager is launching various new electronic gadgets.

  - **Permit Issued:** "Permit to Create New Product Listings from Approved Product Exemplars using Standard Launch Instructions," an instance of `aug:/permits/spawn-policy?1`.
  - **Action:** The Product Manager is authorized to _refine_ any target exemplar Vibe whose `schema` field defines an electronics product structure (e.g., `aug:products/electronics-product?1`, which serves as the base Vibe for electronic products) using any `instruction` that conforms to schema `aug:schema-instruction-launch?1`.
  - **Outcome:** This allows creation of various new product Vibes. For instance, to create a specific smartwatch, the `target` might be `aug:products/smartwatch?1` (an exemplar Vibe for smartwatches, itself potentially derived from `aug:products/electronics-product?1`), and the `instruction` would be `aug:announcements/smartwatch-gen5-details?1`. The result would be `aug:products/smartwatch-gen5?1`. Similarly for `aug:products/earbuds-pro?1` from an `aug:products/earbuds?1` exemplar.

- **Inventory Managers & Stock Level Adjustments:**
  An Inventory Manager needs to update stock levels for any product based on incoming feeds.

  - **Permit Issued:** "Permit to Update Stock Counts for Any Inventory-Tracked Product via System Feeds," an instance of `aug:/permits/input-change?1`.
  - **Action:** The Inventory Manager is authorized to _refine_ any target product Vibe whose `schema` field conforms to the structure defined in an `aug:products/inventory-tracked-product?1` exemplar, using a specific instruction like `aug:feeds/stock-feed?1`.
  - **Outcome:** The `stockLevel` field within any valid product Vibe (e.g., `aug:products/smartwatch-gen5?1`, `aug:products/charger-c30?1`) can be updated.

- **Marketing Specialists & Promotional Campaign Setup:**
  A Marketing Specialist needs to define various types of discount rules for sales events.

  - **Permit Issued:** "Permit to Define New Discount Rules in Promotion Engine via Approved Rule Structures," an instance of `aug:/permits/schema-govern?1`.
  - **Action:** The Marketing Specialist is authorized to _refine_ the target `aug:schema-promo-rules?1` using any instruction Vibe that conforms to `aug:schema-instruction-discount?1` (e.g., instructions for percentage-off, BOGO, or tiered discounts).
  - **Outcome:** Various new discount rule structures (e.g., `percentage-off-category-x`, `bogo-on-accessories`) can be added to the global promotion rules schema.

- **Content Writers & Product Description Updates:**
  A Content Writer needs to update marketing copy for any product page.

  - **Permit Issued:** "Permit to Update Product Page Content for Any Product via SEO Guidelines," an instance of `aug:/permits/editorial?1`.
  - **Action:** The Content Writer is authorized to _refine_ any target product page Vibe conforming to schema `aug:schema-prod-page?1` using the specific instruction `aug:guide-seo-desc?1`.
  - **Outcome:** The `description` and `marketingCopy` fields in the solution of any valid product page Vibe (e.g., `aug:products/smartwatch-gen5?1`) can be updated.

- **Customer Support Leads & Order Adjustments:**
  A Customer Support Lead needs to process various types of standard order adjustments.
  - **Permit Issued:** "Permit to Adjust Customer Orders via Standard Service Protocols," an instance of `aug:/permits/merge-policy?1` (or a more specific adjustment permit).
  - **Action:** The Support Lead is authorized to _refine_ any target customer order Vibe conforming to schema `aug:schema-order?1` using any instruction that conforms to an approved `aug:schema-instruction-order-adj?1` (e.g., for partial refunds, shipping changes, item swaps defined by specific protocols).
  - **Outcome:** New versions of various customer order Vibes (e.g., `aug:order-12345?1`, `aug:order-67890?1`) can be created, reflecting the authorized adjustments.

When any Vibe attempts to _refine_ another, the system verifies it holds a Capability containing a valid permit. This permit must explicitly authorize the `target` (either a specific Vibe or one matching a schema) to be transformed by the specific `instruction` (either a specific Vibe or one matching a schema). The permit itself, through its definition and associated schemas for the target and instruction using these ID conventions (global `aug:/` vs. local `aug:`), clearly defines the scope and constraints of the authorized action.
