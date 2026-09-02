# Laboratory Inventory Management System
## Project Charter / Core Vision
**Version:** 1.0  
**Status:** Core concept frozen  
**Purpose:** Serve as the stable reference for the project so the main idea does not drift during development.

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

The central concept of the project is:

> **Inventory must be traceable.**

Inventory quantities should not simply change without explanation.

Example:

Incorrect model:

```text
Quantity: 100
Quantity: 70
```

There is no explanation for the missing 30 units.

Preferred model:

```text
+100 RECEIVED
 -20 ISSUED
  -5 ISSUED
  -5 DAMAGED
----------------
  70 AVAILABLE
```

Every meaningful inventory change should be represented by a recorded movement.

---

# 5. Main Users

The system conceptually has two types of interaction.

## 5.1 Inventory Staff

People responsible for receiving and organizing inventory.

Main actions:

- register new products,
- register new lots,
- receive inventory,
- record quantities,
- record expiration dates,
- record who received the material,
- add notes to a reception,
- record damaged or adjusted inventory,
- search inventory,
- and inspect inventory history.

## 5.2 Inventory Consumer

Doctors, nurses, technicians, receptionists, or other staff who use or request inventory.

Main actions:

- search for products,
- view availability,
- select products and quantities,
- prepare an inventory issue,
- and confirm the products being removed.

Authentication and complex permissions are not required for the first version.

---

# 6. Main Domain Structure

The conceptual structure is:

```text
INVENTORY
    |
    v
 PRODUCT
    |
    | 1:N
    v
   LOT
    |
    v
MOVEMENTS
    |
    +--> RECEIVED
    +--> ISSUED
    +--> ADJUSTED / DAMAGED
```

A **product** represents what the material is.

A **lot** represents a specific batch of that product.

A product may have many lots.

Each lot may have its own:

- lot number,
- expiration date,
- arrival date,
- available quantity,
- and inventory history.

---

# 7. Product Concept

A product describes the material itself.

Possible information:

- name,
- category/type,
- manufacturer,
- unit of measurement,
- minimum desired stock.

Example:

```text
Product: Nitrile Gloves
Category: Medical Supply
Manufacturer: Example Manufacturer
Unit: units
Minimum stock: 200
```

A product should not be duplicated every time a new shipment arrives.

If the product already exists, a new **lot** is registered instead.

---

# 8. Lot Concept

A lot belongs to one product.

Possible information:

- lot number,
- expiration date,
- date received,
- quantity received,
- available quantity,
- condition/status when relevant.

Example:

```text
Product: Nitrile Gloves
Lot: A234
Received: 2026-08-31
Expires: 2027-03-01
Initial quantity: 500
```

Different lots of the same product may have different expiration dates and quantities.

---

# 9. Inventory Reception

The system must support receiving one or multiple products/lots in a single reception.

Example:

```text
RECEPTION #0042

Received by: Maria
Date: 2026-08-31
Notes: Weekly shipment

Items:
- Nitrile Gloves / Lot A234 / +500
- Alcohol / Lot B910 / +20 bottles
- Masks / Lot C128 / +1000
```

This avoids forcing the user to create many unrelated reception records for one delivery.

---

# 10. Inventory Issue

A user should be able to prepare an inventory issue containing multiple products.

Conceptual flow:

```text
Search
  |
Select product
  |
Select quantity
  |
Add more products
  |
Review issue
  |
Confirm
```

Example:

```text
ISSUE #0084

Received/Requested by: Employee

Items:
- Nitrile Gloves x20
- Alcohol x1
- Masks x5
```

The system must reject an issue if the requested quantity exceeds the available stock.

---

# 11. Inventory Movements

The project should preserve a history of inventory changes.

Initial movement concepts:

- RECEIVED
- ISSUED
- ADJUSTED
- DAMAGED

Example:

```text
Lot A234

2026-08-01   +500   RECEIVED
2026-08-03    -20   ISSUED
2026-08-05     -5   DAMAGED
2026-08-08    -30   ISSUED
```

The movement history is one of the most important features of the system.

---

# 12. Expiration Management

The database should store the actual expiration date.

Example:

```text
expiration_date = 2027-03-01
```

Statuses such as:

- VALID
- EXPIRING SOON
- EXPIRED

should preferably be calculated from the expiration date rather than permanently stored as duplicated information.

The system should make it easy to identify:

- expired lots,
- lots approaching expiration,
- and valid lots.

---

# 13. FEFO Principle

When multiple lots of the same product exist, the system should help prioritize the lot that expires first.

**FEFO = First Expired, First Out**

Example:

```text
Product X

Lot A -> 10 units -> expires September
Lot B -> 30 units -> expires December
```

If 5 units are needed, the system should recommend Lot A first.

The objective is to reduce waste and unnecessary expiration.

For the first version, FEFO may be implemented as a recommendation rather than a fully automatic process.

---

# 14. Damaged or Unavailable Inventory

The project should avoid representing an entire lot only as:

```text
GOOD
BAD
```

because part of a lot may be damaged while the rest remains usable.

Example:

```text
+100 RECEIVED
 -10 DAMAGED
----------------
  90 AVAILABLE
```

This preserves the reason for the inventory reduction.

More advanced lot states such as:

- ACTIVE
- QUARANTINED
- EXPIRED
- RECALLED

may be considered later if the project requires them.

---

# 15. Search and Filtering

The system should allow inventory to be searched or filtered using relevant information such as:

- product name,
- category/type,
- manufacturer,
- lot number,
- expiration date,
- availability,
- and condition.

Important inventory views may include:

- Available inventory
- Low stock
- Expiring soon
- Expired
- Damaged/unavailable inventory

---

# 16. Alerts Inside the System

The system may display internal alerts or status indicators for:

- low stock,
- expired lots,
- lots approaching expiration.

The first version does not require email, SMS, push, or external notifications.

---

# 17. Traceability Questions

A successful implementation should be able to answer questions such as:

- What inventory is currently available?
- Which lots contain a specific product?
- Which lot expires first?
- How much inventory was originally received?
- How much inventory remains?
- Why did the quantity change?
- Who registered a reception?
- Who received or requested an inventory issue?
- When was a specific lot used?
- How much inventory was damaged or adjusted?
- What movements affected a specific lot?

If the system cannot answer these questions, the design should be reconsidered.

---

# 18. Core Business Rules

The first version should respect these principles:

1. A lot belongs to an existing product.
2. One product may have multiple lots.
3. Different lots may have different expiration dates and quantities.
4. Inventory must never become negative.
5. A user cannot issue more inventory than is available.
6. Damaged inventory is not considered available inventory.
7. Expired inventory should not normally be issued.
8. Important quantity changes must remain traceable through movements.
9. A single reception may contain multiple items.
10. A single issue may contain multiple items.
11. The system should prioritize or recommend inventory using FEFO when appropriate.

---

# 19. MVP Scope

The core version of the project will focus on:

- product management,
- lot management,
- inventory reception,
- inventory issue,
- inventory adjustments,
- lot-level quantities,
- expiration tracking,
- damaged inventory,
- low-stock detection,
- inventory movement history,
- traceability,
- search and filtering,
- FEFO recommendation,
- and multi-item reception/issue operations.

This is sufficient complexity for the main project.

---

# 20. Possible Future Improvements

These features may be added later only if they support the existing system:

- storage locations,
- richer product descriptions,
- employee records,
- reporting and analytics,
- quarantine workflows,
- recall workflows,
- barcode or QR support,
- supplier information,
- authentication and authorization.

They are not required to prove the main project concept.

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

This project should not be presented as:

> "A website to manage inventory."

It should be presented as:

> **A laboratory inventory management system focused on lot-level traceability, expiration control, FEFO-based inventory usage, and complete tracking of inventory movements from reception to consumption.**

---

# 24. Non-Negotiable Core

The following ideas define the project and should not change casually:

```text
LABORATORY INVENTORY
        |
        v
PRODUCTS
        |
        v
LOTS
        |
        v
MOVEMENTS
        |
        +--> RECEPTION
        +--> ISSUE
        +--> DAMAGE / ADJUSTMENT
        |
        v
TRACEABILITY
        |
        +--> STOCK CONTROL
        +--> EXPIRATION CONTROL
        +--> FEFO
```

If a proposed feature does not reinforce this structure, it is probably outside the project.

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
