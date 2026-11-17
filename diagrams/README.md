# UML Use-Case Diagram Documentation
## Customizable Subscription Box Scheduler (Problem Statement #35)

This folder contains the visual and textual specifications for the UML Use-Case Model modeled for the Customizable Subscription Box Scheduler system.

---

## 1. Diagram Assets

- **High-Resolution PNG (300 DPI)**: [`use_case_diagram.png`](file:///Users/bharath/.gemini/antigravity/scratch/se-lab-1/diagrams/use_case_diagram.png)
- **Scalable Vector Graphic**: [`use_case_diagram.svg`](file:///Users/bharath/.gemini/antigravity/scratch/se-lab-1/diagrams/use_case_diagram.svg)
- **PlantUML Source Specification**: [`use_case_diagram.puml`](file:///Users/bharath/.gemini/antigravity/scratch/se-lab-1/diagrams/use_case_diagram.puml)

---

## 2. Model Structure

### Target Stakeholders & Actors
1. **Subscriber (`Sub`)** (*Primary Human Actor*):
   - The end customer subscribing to recurring product deliveries.
   - Interacts with the portal to manage preferences, customize items, review box summaries, and pause/skip cycles.
2. **Fulfillment Lead (`FL`)** (*Primary Human Actor - Operations*):
   - The warehouse operations manager responsible for dispatch logistics.
   - Aggregates finalized box choices, generates fulfillment manifests, and exports batch shipping labels.
3. **Payment Gateway & Billing Service (`PG`)** (*Secondary External System Actor*):
   - Automated financial infrastructure handling card tokenization, recurring subscription debits, and cycle shifts upon subscriber pauses.

---

### UML Stereotypes & Relationships

#### `<<include>>` Relationships (Mandatory Inclusions)
- **`UC-02 (Customize Monthly Box)` `<<include>>` `UC-05 (Validate Cutoff Window <48h)`**:
  - *Semantics*: Every time a subscriber attempts to swap an item or modify box contents, the system unconditionally executes the 48-hour cutoff validation check. Customization is strictly disallowed if the current time is within 48 hours of billing renewal.
- **`UC-02 (Customize Monthly Box)` `<<include>>` `UC-06 (Verify Real-Time Inventory Stock)`**:
  - *Semantics*: Swapping an item requires querying the active warehouse catalog in real time to ensure candidate replacement items have sufficient allocated physical stock.
- **`UC-03 (Pause or Skip Billing Cycle)` `<<include>>` `UC-05 (Validate Cutoff Window <48h)`**:
  - *Semantics*: Pausing or skipping must also be validated against the 48-hour cutoff deadline to prevent packaging conflicts in already staged batches.
- **`UC-10 (Generate Monthly Fulfillment Manifest)` `<<include>>` `UC-12 (Lock Customization Window)`**:
  - *Semantics*: Generating the active warehouse manifest triggers an irreversible lockout of subscriber modifications for that billing period.
- **`UC-10 (Generate Monthly Fulfillment Manifest)` `<<include>>` `UC-11 (Batch Export Courier Shipping Labels)`**:
  - *Semantics*: Finalizing the monthly fulfillment manifest prepares and formats the bulk data stream necessary to print compliant carrier shipping labels (PDF/ZPL).

#### `<<extend>>` Relationships (Optional Conditional Extensions)
- **`UC-07 (Select Premium Add-on / Gift Item)` `<<extend>>` `UC-02 (Customize Monthly Box)`**:
  - *Extension Point*: `[Add-on Item Requested]`.
  - *Semantics*: If the subscriber chooses to add extra items beyond their standard tier allowance, the base customization workflow is extended to handle additional item selection and incremental micro-billing.
- **`UC-08 (Apply Promotional Discount / Coupon)` `<<extend>>` `UC-02 (Customize Monthly Box)`**:
  - *Extension Point*: `[Promo Code Applied]`.
  - *Semantics*: If the subscriber applies a promotional voucher or referral reward during box customization, the price/reward adjustment flow conditionally executes.
- **`UC-03 (Pause or Skip Billing Cycle)` `<<extend>>` `UC-09 (Process Recurring Renewal Billing)`**:
  - *Extension Point*: `[Skip / Pause Requested]`.
  - *Semantics*: Modifies standard scheduled billing execution by advancing the next payment charge date and preventing renewal charges for that cycle.
