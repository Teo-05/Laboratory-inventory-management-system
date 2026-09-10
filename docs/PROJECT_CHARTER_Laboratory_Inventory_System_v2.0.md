# Laboratory Inventory Management System
## Project Charter / Core Vision
**Version:** 2.0  
**Status:** Stable core vision; initial MVP scope aligned with specification v2.0  
**Purpose:** Preserve the stable project vision and distinguish the initial MVP from later delivery phases.

**Document alignment:** This is document revision 2.0, not a declaration of implemented software. `MVP_Laboratory_Inventory_System_v2.0.md` defines the initial release rules and `USE_CASES_MVP_v2.0.md` defines its eight operational flows. Earlier v1.0 scope statements are superseded by this revision.

---

# 1. Project Idea

The **Laboratory Inventory Management System** is an internal inventory system for laboratories or clinical environments focused on:

- lot-level inventory control,
- expiration management,
- inventory movement tracking,
- and traceability from reception to consumption.

The system is not intended to be a generic store inventory application.

Its main purpose is to allow laboratory staff to know, at any moment:

1. What products are available?
2. How much inventory is available?
3. Which lot does the inventory belong to?
4. When does each lot expire?
5. What condition is the inventory in?
6. When and how did the inventory enter the system?
7. How and to whom did the inventory leave the system?
8. What happened to inventory that is missing, damaged, expired, or adjusted?

---

# 2. Main Problem

A simple inventory system may only store:

```text
Product: Nitrile Gloves
Quantity: 800
```

For a laboratory, this is not enough.

The same product may exist in several different lots:

```text
Nitrile Gloves

Lot A234
- 300 units
- Expires: October 2026

Lot B719
- 500 units
- Expires: February 2027
```

Each lot may have:

- a different expiration date,
- a different arrival date,
- a different available quantity,
- a different condition,
- and a different movement history.

The system therefore must not only answer **how much inventory exists**, but also **where that quantity came from and what happened to it**.

---

# 3. Primary Objective

> Build a laboratory inventory management system that controls supplies at lot level and maintains traceability of quantities, expiration dates, condition, and inventory movements from reception to consumption.

The system should help prevent:

- negative inventory,
- accidental use of expired inventory,
- loss of movement history,
- unexplained inventory differences,
- and unnecessary expiration of older lots.

---

# 4. Core Principle

**Every change in a lot's recorded balance must remain traceable through a movement.**

| Example movement | Balance effect |
| --- | --- |
| RECEIVED | +100 |
| ISSUED | -20 |
| DAMAGED | -5 |
| Resulting recorded balance | 75 |

The recorded balance is the net effect of confirmed movements. Issue availability equals that balance for a valid lot and zero for an expired lot. Expiration changes calculated availability without inventing a disposal or changing the balance. Actual disposal requires a documented movement.

Positive movement magnitude and movement effect are separate concepts: RECEIVED increases the balance; ISSUED and DAMAGED decrease it; ADJUSTED explicitly chooses increase or decrease.

---

# 5. Main Users

## 5.1 Inventory Staff

Registers products and lots, receives inventory, records damage and adjustments, adds notes or reasons, searches inventory and inspects movement history.

## 5.2 Inventory Consumer

Searches inventory, selects a lot and quantity, reviews and confirms an actual issue for use.

These are conceptual patterns of interaction, not enforced permission groups in the initial MVP. Every movement records the declared name of the person entering it. Receptions also identify the physical receiver; issues identify the person receiving material for use. These people may coincide. Names are required, but identity is not authenticated.

Preparing an issue creates neither a reservation nor an approval request. Confirming it represents an actual issue. Authentication, authorization and employee administration remain later work.

---

# 6. Main Domain Structure

| Relationship | Cardinality |
| --- | --- |
| Product to lots | One product has zero or many lots; each lot belongs to exactly one product. |
| Lot to movements | One lot has zero or many movements; each movement affects exactly one lot. |
| Confirmed inventory operation to movement in the initial MVP | Exactly one lot and one movement per operation. |

Movement types are RECEIVED, ISSUED, DAMAGED and ADJUSTED. Product and lot registration are independent catalog operations and create no stock. The actual database representation is defined during the ERD and implementation design, subject to the documented integrity rules.

---

# 7. Product Concept

A product describes a specific material and variant. Required information is a system-generated identifier, a unique internal product code, name including relevant variant, category, manufacturer and one base counting unit.

Code comparison trims surrounding whitespace and uses uppercase while preserving leading zeros, punctuation and internal spaces. The code is text, not a number. A unique code prevents that code from being duplicated; it cannot automatically recognize equivalent materials entered under different codes. Staff must inspect existing products before registering another.

The initial MVP counts indivisible units such as individual gloves, pairs, bottles, sealed boxes or packages. All movements of a product use the same unit. Fractional quantities and conversions between boxes and pieces are outside this release.

A new shipment reuses the product. It also reuses its lot when the product and lot number already match; a new shipment does not necessarily mean a new lot. Minimum desired stock and low-stock alerts belong to a later phase.

---

# 8. Lot Concept

A lot has a system-generated identifier, product reference, lot number and complete expiration date. Product plus normalized lot number is unique; the same number may be used by different products. Lot number normalization follows the product-code convention.

Creating a lot leaves its balance at zero and generates no movement. First recorded reception date and quantity come from the first confirmed RECEIVED movement; before it, there is no reception date and the initial received quantity is zero. Total recorded received quantity sums all RECEIVED movements.

Later receptions reuse the lot and add movements. The expiration date is not overwritten. A disagreement between the physical information and the recorded expiration stops the operation for identification/data review. Confirmed catalog metadata has no edit or delete workflow in this MVP.

Recorded balance is derived logically from movement effects. Whether it is physically calculated on read or stored and maintained is an implementation choice that must preserve the same invariant.

---

# 9. Inventory Reception

The initial MVP confirms one product/lot and one RECEIVED movement per operation. A physical delivery containing several lots is entered as independent operations, without a collective all-or-nothing promise.

Staff selects or registers a product and lot, checks the identity and expiration against the material, enters a positive integer quantity in the product's base unit, identifies the person recording and the physical receiver, reviews and confirms.

An expired lot can be recorded when the material exists physically, with a warning, explicit acknowledgment and a required note. The operation increases recorded balance while issue availability stays zero. Material rejected for damage before entry is excluded from the quantity incorporated; its rejection can be described in a note, without a second damage deduction.

A reception and its balance effect are indivisible and recoverable after a repeated confirmation. Catalog records already confirmed remain if the subsequent reception is canceled.

**Later phase:** a reception header grouping several products/lots may be added to reflect a complete delivery. This is not an initial MVP requirement.

---

# 10. Inventory Issue

The initial MVP confirms one issue from one selected lot. The user searches, selects a valid lot, enters a positive integer quantity, identifies the person recording and the person receiving for use, reviews and confirms.

Preparing the form does not reserve inventory. Final validation uses the current lot, business date and issue availability under protection against concurrent operations. An expired lot or insufficient availability causes rejection. There is no partial issue, automatic lot replacement or automatic distribution between lots.

The ISSUED movement and balance reduction are indivisible. Repeating the same confirmation recovers the original outcome rather than executing it again.

**Later phase:** a multi-product or multi-lot issue may be introduced with separately defined grouping and confirmation rules.

---

# 11. Inventory Movements

Supported types are RECEIVED, ISSUED, DAMAGED and ADJUSTED. Each confirmed movement contains the affected lot, type, positive integer magnitude, direction for ADJUSTED, system identifier and registration timestamp, the person recording, the reception/issue participant where relevant, note or required reason, and a correction reference when applicable.

Confirmed movement fields are immutable. Quantity errors are explained through an additional ADJUSTED movement with a reason and, for a known source movement, a valid reference belonging to the same lot. References do not automatically reverse a movement or move it to another lot.

The system preserves a stable confirmation order per lot and a recoverable outcome for each confirmation identifier. A known rejection has no inventory effect; a lost response is an unknown outcome until the original attempt is recovered. Registration time is assigned by the system; separate backdated physical event times are outside the initial MVP.

---

# 12. Expiration Management

Every lot in the initial MVP requires an actual, complete calendar expiration date. Incomplete dates and products without expiration are outside this release; the system must not invent a date.

A lot is expired when its expiration date is earlier than the current calendar day in the configured laboratory time zone. The expiration date itself is included as the last valid day. Final validation uses the system clock, not the browser's chosen date or time zone.

Validity is calculated, not maintained as a conflicting editable status. Expiration does not alter recorded balance. It makes issue availability zero and prevents every ISSUED operation. Receiving expired inventory requires explicit acknowledgment and a note. Damage and adjustment remain possible against recorded balance, without making the expired material available for use.

An actual disposal for expiration uses a decreasing ADJUSTED movement with the reason. Expiring-soon indicators and alerts belong to a later phase.

---

# 13. FEFO Principle — Later Phase

FEFO prioritizes material that expires first. For example, when two valid lots have different expiration dates, a later feature can recommend the earlier-expiring lot.

The initial MVP records and displays expiration and prevents issues from expired lots, but does not recommend or automatically select lots by FEFO. FEFO remains part of the planned evolution and the long-term goal of reducing expiration waste. It is not required to declare the initial MVP complete.

---

# 14. Damaged or Unavailable Inventory

Part of a lot can be damaged while the rest remains usable. DAMAGED removes the identified affected quantity from recorded balance and requires a reason. It cannot exceed recorded balance, including for an expired lot. Already written-off units must not be deducted or counted again merely because they remain physically in a discard area.

ADJUSTED handles justified count differences, recorded quantity corrections or actual expiration disposal. It requires an explicit increase/decrease direction, a positive magnitude, reason and a prior RECEIVED for that lot. It does not replace a new delivery or the first reception.

Before an adjustment confirms, the system detects any movement since the reviewed lot state, even if the balance returned to the same number. It then requires a fresh count review and confirmation. Adjustments cannot produce negative balance or change expiration metadata.

Quarantine, recall, detailed rejected-delivery handling and collective corrections across lots are later workflows.

---

# 15. Search and Filtering

The initial MVP searches by a partial product-name match or an exact normalized lot-number match. A blank query lists products. Results identify product code, name/variant, manufacturer, unit, lot, expiration, recorded balance and issue availability.

The system distinguishes no matching product, product without lots, lot without reception, exhausted lot, expired lot and failed query. Exhausted and expired lots remain searchable with their history. A failed query must not be reported as zero inventory.

Product availability, when displayed, is the sum of its lots' issue availability. The initial MVP does not reserve quantities through search. Advanced filters by category, manufacturer, date windows and condition, as well as low-stock and expiring-soon views, belong to a later phase.

---

# 16. Alerts Inside the System — Later Phase

Low-stock alerts and expiring-soon alerts are not required for the initial MVP. Showing a lot's current validity, warning when recording expired inventory and rejecting an expired issue are included operational behaviors.

Later versions may add internal alerts for low stock and approaching expiration. Email, SMS, push and other external notifications remain outside the core initial scope.

---

# 17. Traceability Questions

The initial MVP must answer which products and lots exist, what their recorded balances and issue availability are, when each lot expires, what its first recorded reception was, how much was recorded as received in total, and why its balance changed.

The history must show each movement's effect, confirmation order, registration time, person recording, reception/issue participant and required reason or correction reference. Declared names provide basic operational attribution, not authenticated proof of identity.

Users can inspect dates to compare lots. An automatic FEFO recommendation is a later feature. Movement totals and current balance must come from a coherent view, while availability additionally depends on the business date.

---

# 18. Core Business Rules

1. Every lot belongs to one existing product; product plus normalized lot number is unique.
2. Catalog creation produces no inventory movement or units.
3. Quantities are positive integers in the product's fixed base unit; movement type and adjustment direction determine the effect.
4. Recorded balance equals the net confirmed movement effects and never becomes negative.
5. Issue availability is balance for valid lots and zero for expired lots; final issue validation cannot exceed it.
6. Every balance change and its movement are recorded indivisibly. Time-based expiration changes availability without fabricating a stock movement.
7. Damage and adjustments require reasons, and movement history is immutable.
8. Every movement records who entered it; reception/issue also identifies the participant, without claiming authenticated identity.
9. Each initial MVP confirmation affects one lot and produces one movement.
10. Concurrent operations preserve current balance and all confirmed effects; stale adjustments require review again.
11. A repeated confirmation recovers its original outcome and never creates a second movement. Lost responses are resolved using the same attempt.
12. Multiple receptions can affect the same lot without duplicating the lot or overwriting its expiration.

The authoritative detailed definitions BR-01 through BR-24 appear identically in the MVP and use-case v2.0 specifications. FEFO and multi-item grouping are delivery-phase goals, not requirements of this initial release.

---

# 19. Initial MVP Scope

The initial MVP includes product and lot registration, a single-lot reception, a single-lot issue, damage and bidirectional adjustments, basic search, expiration validation, immutable movement history, traceable declared participants, and safe handling of concurrency and repeated confirmations.

It uses whole counting units and complete expiration dates. It does not include multi-item confirmation, FEFO recommendation or automation, low-stock/expiring-soon alerts, dashboards, advanced filters, fractional units, conversions, products without expiration, catalog editing/deletion, employee administration, reservations, approvals or historical backdated events.

Completion requires the acceptance scenarios in MVP specification v2.0, not merely producing this documentation. This limited first delivery demonstrates the core concept before adding later phases.

---

# 20. Possible Future Improvements

Later phases may add multi-item reception/issue grouping, FEFO recommendations and automation, low-stock and expiring-soon alerts, advanced filters, fractional units and conversions, products without expiration, controlled catalog corrections, multi-lot corrections, employee records, authenticated access, physical event dates distinct from registration time, and richer reports.

Other possible extensions include locations, quarantine and recall, barcode/QR support, suppliers and detailed delivery rejections. Each extension needs its own rules and must preserve movement traceability and quantity integrity.

---

# 21. Explicitly Out of Scope

The following features are not part of the core project:

- billing,
- sales,
- payments,
- complete purchasing system,
- complete supplier management,
- product images,
- complex authentication,
- complex permissions,
- mobile application,
- multiple laboratories,
- email/SMS notifications,
- chat,
- unrelated scheduling systems.

These features should not be added simply to make the project appear larger.

---

# 22. Technology Direction

The intended technology stack is:

## Version Control
- Git
- GitHub

## Database
- PostgreSQL

## Backend
- Python
- FastAPI

## Frontend
- HTML
- CSS
- JavaScript

The frontend will remain intentionally minimal.

The main technical emphasis is:

- relational database design,
- backend architecture,
- business rules,
- API design,
- data integrity,
- testing,
- and professional development practices.

---

# 23. Project Identity

The initial release should be described as:

> A laboratory inventory system focused on lot-level traceability, expiration control and complete tracking of confirmed movements from reception to issue and disposal.

FEFO-based prioritization is part of the intended evolution. It should only be presented as a delivered capability once implemented and verified.

---

# 24. Non-Negotiable Core

The stable core is laboratory materials organized as products and lots, with recorded movements explaining each balance change, expiration determining issue eligibility, and preserved history identifying participants and reasons.

An improvement should strengthen traceability, stock integrity or expiration control. FEFO supports that direction as a later capability without being mandatory for the initial MVP. The project must not expand into unrelated sales, billing or scheduling merely to appear more complex.

---

# 25. Rule for Changing the Project

The project may evolve, but the core concept should only change if a new requirement:

1. solves a real problem in the laboratory inventory domain,
2. strengthens traceability, stock control, or expiration management,
3. does not unnecessarily expand the project into another type of system,
4. and can be justified technically.

New features should not be added only to make the project look more complex.

---

# 26. One-Sentence Definition

> **Laboratory Inventory Management System: an internal system for controlling laboratory supplies at lot level, maintaining traceability of inventory movements, quantities, condition, and expiration from reception to consumption.**
