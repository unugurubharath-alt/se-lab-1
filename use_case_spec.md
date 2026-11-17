# Use-Case Flow Specification
## Core Use Case: UC-02 — Customize Monthly Box Items (Swap Products)

**System**: Customizable Subscription Box Scheduler Platform  
**Course**: PES University - Dept. of CSE | Lab 1: Requirements Engineering & UML Use-Case Modelling  
**Problem Statement #35**: Retail, E-Commerce & Finance  

---

### 1. Specification Overview & Metadata

| Attribute | Details |
| :--- | :--- |
| **Use Case ID** | **UC-02** |
| **Use Case Name** | **Customize Monthly Box Items (Swap Products)** |
| **Primary Actor** | **Subscriber** (Authenticated Customer) |
| **Secondary Actors** | **Warehouse Inventory Subsystem**, **Customer Notification Service** |
| **Associated FRs** | `FR-001` (Customization/Swap), `FR-002` (Preference Tagging), `FR-004` (Notifications) |
| **Trigger** | Subscriber selects "Customize Next Box" on the portal dashboard or clicks a deep link from an automated renewal alert. |
| **Brief Description** | Enables an active subscriber to inspect pre-curated monthly items (books, coffee, snacks), swap default selections for alternative catalog products matching preference tags, and confirm their monthly box before the 48-hour cutoff. |

---

### 2. Preconditions & Postconditions

#### Preconditions
1. The subscriber is authenticated with an active, non-paused subscription account.
2. The current timestamp is at least **48 hours prior** to the scheduled monthly billing renewal date ($T_{\text{renewal}} - T_{\text{current}} \ge 48\text{ hours}$).
3. The catalog for the upcoming monthly delivery cycle is published and active.

#### Postconditions
- **Success Guarantees (Main Postcondition)**:
  1. Selected replacement SKUs overwrite default items in the subscriber's monthly box manifest.
  2. Temporary inventory reservation locks are placed on chosen items in the warehouse inventory system.
  3. The provisional fulfillment manifest updates dynamically.
  4. An email and in-app confirmation receipt summarizing finalized box contents is transmitted to the subscriber.
- **Minimal / Failure Guarantees**:
  1. If the swap transaction aborts or fails, the subscriber's prior box configuration remains intact.
  2. No phantom inventory holds are retained.
  3. A clear, actionable diagnostic message is presented to the subscriber.

---

### 3. Main Success Scenario (Happy Path)

| Step | Actor Action | System Response |
| :---: | :--- | :--- |
| **1** | Subscriber accesses the "My Subscription" dashboard and clicks **"Customize Upcoming Box"**. | System retrieves subscriber profile and billing cycle metadata. |
| **2** | — | System invokes **UC-05 (Validate Cutoff Window)**; verifies $T_{\text{renewal}} - T_{\text{current}} \ge 48\text{ hours}$. |
| **3** | — | System queries catalog and displays current default curated items along with preference match badges (e.g., "Medium Roast", "Sci-Fi", "Gluten-Free") and countdown timer to cutoff. |
| **4** | Subscriber clicks **"Swap Item"** on a default product (e.g., swapping Dark Roast Coffee with Ethiopian Single-Origin). | System invokes **UC-06 (Verify Real-Time Inventory Stock)** for eligible replacement SKUs matching active tags. |
| **5** | — | System renders interactive swap modal displaying candidate products, flavor/genre notes, customer ratings, and real-time availability badges. |
| **6** | Subscriber selects the desired replacement product and clicks **"Confirm Swap"**. | System validates stock reservation and updates provisional box configuration in memory. |
| **7** | Subscriber optionally repeats steps 4–6 for remaining eligible box items (up to box slot limits). | System recalculates box summary, tag compatibility score, and confirms total item balance. |
| **8** | Subscriber reviews the finalized selection and clicks **"Save & Confirm Monthly Box"**. | System initiates an atomic persistence transaction: writes new SKU IDs, sets status to `CUSTOMIZATION_CONFIRMED`, and updates warehouse dispatch queue. |
| **9** | — | System sends an instant confirmation email and displays confirmation screen: *"Your custom box for [Month] is locked in!"* |

---

### 4. Alternate Flows

#### Alternate Flow A: Customization Attempted Within 48-Hour Lockout Window
- **At Step 2**: System evaluates cutoff condition and detects $T_{\text{renewal}} - T_{\text{current}} < 48\text{ hours}$.
- **A.1**: System disables all "Swap", "Add-on", and "Remove" interface elements.
- **A.2**: System displays persistent lockout notification:  
  > *"Customizations for your upcoming delivery closed 48 hours before billing renewal. Your box is currently staged for warehouse packing. You may adjust preference tags or schedule a pause for the subsequent month's cycle."*
- **A.3**: System redirects user to view-only box manifest with courier tracking placeholder. Use case terminates.

#### Alternate Flow B: Selected Swap Product Out of Stock (Concurrency Collision)
- **At Step 6**: Between modal display and subscriber confirmation, the selected replacement SKU stock drops to 0 due to concurrent subscriber checkouts.
- **B.1**: System traps inventory allocation error from warehouse service.
- **B.2**: System flags selected item with a red warning badge: *"Just ran out! Please select another option."*
- **B.3**: System automatically refreshes swap modal with the next best matching in-stock items based on subscriber preference tags.
- **B.4**: Subscriber selects another available SKU (resumes at Step 6) or cancels modal (reverts to Step 3).

#### Alternate Flow C: Subscriber Diverts to Skip or Pause Billing Cycle
- **At Step 4**: Instead of swapping items, subscriber clicks **"Skip This Month"** or **"Pause Deliveries"**.
- **C.1**: System invokes **UC-03 (Pause or Skip Billing Cycle)**.
- **C.2**: System presents pause duration picker (1, 2, or 3 months) or 1-month skip option.
- **C.3**: Subscriber confirms pause; system shifts renewal date forward, releases all item reservations for the cycle, updates status to `PAUSED`, and sends confirmation. Use case terminates.

---

### 5. Extension Points & Special Requirements

- **Extension Point 1 — Premium Add-on Selected (`UC-07`)**:  
  At Step 7, subscriber may click *"Add Extra Item to Box"*. System opens premium add-on catalog, adds item to box, and recalculates micro-billing increment.
- **Extension Point 2 — Promotional Voucher Applied (`UC-08`)**:  
  At Step 8, subscriber enters a promo coupon. System validates code and applies discount before final confirmation.
- **Performance Requirement**: Step 5 (fetching swap catalog and inventory availability) must render in **< 800 ms** under 95th percentile load.
- **Data Integrity Requirement**: Box item swaps must be wrapped in ACID transactions with optimistic locking to prevent double-allocation of limited-run items.
