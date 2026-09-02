# Laboratory Inventory Management System

A laboratory inventory management system focused on **lot-level traceability, expiration tracking, stock control, and inventory movement history**.

## Overview

Laboratory inventory requires more than tracking the total quantity of a product. The same product may exist in multiple lots, each with its own:

- expiration date,
- received quantity,
- available quantity,
- reception date,
- and movement history.

This project is designed to maintain traceability from **inventory reception to consumption**, ensuring that meaningful inventory changes can be explained through recorded movements.

## Core Concept

```text
PRODUCT
   ↓
LOT
   ↓
MOVEMENTS
   ↓
RECEPTION / ISSUE / DAMAGE / ADJUSTMENT
   ↓
TRACEABILITY
```

The central principle of the project is:

> Inventory is not simply a number. Every meaningful inventory change must be traceable.

## MVP Features

The first version of the system will support:

- Product management
- Lot management
- Inventory reception
- Inventory issue
- Inventory adjustments
- Damaged inventory tracking
- Lot-level available quantities
- Expiration tracking
- Inventory movement history
- Basic search by product name and lot number
- Prevention of negative inventory
- Prevention of normal issues from expired lots

## Main Business Rules

- A lot must belong to an existing product.
- A product may have multiple lots.
- Inventory cannot become negative.
- Users cannot issue more inventory than is available.
- Expired lots cannot be used for normal inventory issues.
- Damaged inventory is removed from available stock.
- Every meaningful quantity change must generate a movement record.
- Reception and issue operations must identify the person related to the operation.

## Example

```text
Product: Nitrile Gloves
Lot: A234

+100 RECEIVED
 -20 ISSUED
  -5 DAMAGED
----------------
  75 AVAILABLE
```

The current quantity can always be explained using the movement history.

## Technology Stack

### Version Control
- Git
- GitHub

### Database
- PostgreSQL

### Backend
- Python
- FastAPI

### Frontend
- HTML
- CSS
- JavaScript

The frontend will remain intentionally simple. The main technical focus is on:

- relational database design,
- backend architecture,
- business rules,
- API design,
- data integrity,
- testing,
- and professional development practices.

## Project Status

**Current stage:** Requirements and system design.

The project currently includes:

- Project Charter
- MVP definition

Planned next steps include:

1. Formal use cases
2. Domain model / ERD
3. Database design
4. Backend structure
5. API implementation
6. Minimal frontend
7. Testing and validation

## Scope

This project is intentionally focused on laboratory inventory management.

Features such as billing, payments, purchasing systems, complex authentication, mobile applications, multiple laboratories, and external notifications are outside the initial MVP.

## Future Improvements

Possible future additions include:

- FEFO recommendations
- Low-stock alerts
- Advanced search and filtering
- Barcode or QR support
- Storage locations
- Supplier information
- Reporting and analytics
- Authentication and authorization

## License

This project is currently intended as a software engineering portfolio project.
