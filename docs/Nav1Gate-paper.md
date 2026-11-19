# Nav1Gate: A Desktop Customer-Routing and PC-Parts Inventory System

## Abstract

Nav1Gate is a Python desktop application for customer service routing and parts
inventory management in a computer-parts store. It organizes customer requests
in first-in, first-out (FIFO) waitlists for five service sectors, assigns the
next customer to available staff in the relevant sector, tracks service
progress, and stores completed-service history. Its inventory functions
support product creation, lookup, editing, deletion, search, sorting,
nearest-price lookup, and queued restocking. A Tkinter graphical interface
provides Dashboard, Waitlist, Staff, PC Parts, and Service History screens; a
command-line interface is also available. The project demonstrates common
data structures and algorithms in its implementation, including queues,
stacks, linked lists, a hash table, a binary search tree, linear search,
binary search, and merge sort. Inventory and helpdesk records are persisted
locally as JSON, with actions recorded in an audit log. Automated tests cover
the principal business rules, structures, algorithms, validation, and restart
persistence. Nav1Gate is a prototype and does not provide authentication,
access control, encryption, or multi-user synchronization.

**Keywords:** inventory management, customer service, queueing, data structures,
Tkinter, Python

## 1. Introduction

Computer-parts stores support customers with both product sales and technical
services. Staff may answer retail questions, assemble computers, or
troubleshoot hardware and software. A consistent process is needed to route
requests, track staff availability, preserve service records, and keep an
accurate parts catalog.

Nav1Gate combines these activities in a lightweight desktop application. It
organizes customer requests by service sector, assigns waiting customers to
available staff, records service status and history, and provides basic
inventory operations for PC parts. Its data structures and algorithms are
visible in the source code, making the project a practical demonstration of
how these concepts can support application workflows.

## 2. Problem Statement

A small computer-parts store needs a simple way to track customer requests and
staff workload while maintaining its parts inventory. The system should
preserve the order of waiting customers within each service sector, avoid
assigning unavailable or out-of-sector staff, support inventory lookup and
maintenance, and retain important records after the application closes.

The solution is scoped to a single desktop installation. It focuses on core
workflow and data-structure demonstrations rather than shared access across
devices, user accounts, or integration with suppliers and point-of-sale
systems.

## 3. Objectives

Nav1Gate aims to:

1. Register customers under Assembly, Retail, Hardware Repair, Software Repair,
   or General Service.
2. Maintain a separate FIFO waitlist for each sector.
3. Assign the earliest waiting ticket only when suitable sector staff are
   available, updating staff availability as service begins and ends.
4. Track active and completed service tickets, preserve service history, and
   restrict deletion to finished tickets.
5. Maintain staff and PC-parts records with validation and local persistence.
6. Provide inventory searches, sorted views, nearest-price lookup, and queued
   restock processing.
7. Demonstrate data structures and algorithms through working operations and
   automated tests.

## 4. Scope and Functional Requirements

### 4.1 Customer service workflow

Each ticket records a generated ID, customer name, sector, service
description, status, assigned staff ID, creation time, and (when completed)
completion time. A new customer enters a sector waitlist with status
**Waiting**. Assignment changes the ticket to **Active**, records the staff
member, and marks that member unavailable. Completing an active ticket changes
its status to **Finished**, records the completion time, releases the staff
member, and adds the ticket to service history.

Waiting and active tickets cannot be deleted from service history. Finished
tickets can be explicitly deleted, following confirmation in the GUI. Staff
can be added, edited, and removed; a busy staff member cannot be removed or
moved to another sector.

### 4.2 Inventory workflow

Products contain a SKU, name, category, quantity, price, and derived stock
status. A quantity of five or fewer is labeled **Restock needed**. The system
supports create, read, update, and delete (CRUD) operations, rejects duplicate
SKUs, negative quantities and prices, fractional quantities, and non-finite
prices, and writes inventory changes to JSON. An inventory action stack can
undo the most recent product create, update, or delete during the current
session.

Users can search by name or category, look up an exact SKU, sort products by
name or price, and find the product nearest a target price. Restock requests
are queued; processing one increases the product quantity by 10 units.

### 4.3 Interfaces and persistence

The default interface is a Tkinter desktop application with Dashboard,
Waitlist, Staff, PC Parts, and Service History tabs. A terminal interface is
available by running `main.py --cli`. The GUI validates required customer
fields and numeric prices, presents operation errors, and refreshes views
after successful operations.

The application stores inventory in `inventory.json`, service tickets and
staff in `services.json`, and an append-only activity record in `audit.log`.
Files are placed in an operating-system-specific per-user data directory. At
startup, indexes and in-memory queues and stacks are rebuilt from saved
product and service records.

## 5. Design and Implementation

The project is separated into the following areas:

- `main.py` provides input helpers and the command-line menu, and starts the
  GUI by default.
- `gui.py` constructs the Tkinter screens, collects user actions, displays
  records, and delegates business operations to the application layer.
- `app.py` defines `InventorySystem`, which owns application state and
  enforces inventory, staff, and ticket rules.
- `inventory.py` defines product, staff, and service-ticket records, JSON
  storage helpers, and the audit log.
- `data_structures.py` and `algorithms.py` provide the structures and
  algorithms used by application operations.

The application layer maintains collections and indexes in memory during a
run. On loading, product records are re-indexed by SKU and price. Saved
service records rebuild each sector's waiting queue, active-ticket
collection, finished-ticket history, and staff availability. The GUI reads
this state through `InventorySystem` and refreshes tables and dashboard
metrics after mutations.

## 6. Data Structures and Algorithms

| Structure or algorithm | Use in Nav1Gate |
|---|---|
| ArrayList | Wraps Python's list and holds products, staff, tickets, and active services. |
| Queue | Stores sector waitlists in FIFO order and pending restock requests. |
| Stack | Tracks inventory changes for undo and organizes active and finished service records. |
| Linked list | Stores completed service history and recently viewed products. |
| Hash table | Indexes products by SKU, staff by ID, and tickets by ID; collisions are handled with separate chaining. |
| Binary search tree | Indexes products by price as part of the project's data-structure implementation. |
| Set | Tracks unique inventory categories and supported service sectors. |
| Linear search | Finds an inventory product whose name or category matches a query. |
| Binary search | Looks up an SKU after products have been sorted by SKU. |
| Merge sort | Sorts product collections by name or price and prepares SKU data for binary search. |
| Recursive nearest-price search | Sorts by price, then recursively searches for the product closest to a target price. |

For `n` products, linear search takes O(n) time and merge sort takes
O(n log n) time. Binary search takes O(log n) after sorting; because SKU
lookup sorts the products for each call, the full operation is O(n log n).
Nearest-price lookup likewise sorts first and then searches recursively, so
the full operation is O(n log n), with O(log n) search depth after sorting.
Queue operations at the ends and stack push/pop use the underlying deque/list
operations. Linked-list append traverses the list and therefore takes O(n).

These complexity descriptions match the current implementation and its
preconditions. Binary search requires sorted input, and nearest-price lookup
sorts the product collection before searching. Although a price tree is
maintained as an index, the user-facing price sort and nearest-price operation
use merge-sort-based paths.

## 7. Testing and Evaluation

The project includes a Python `unittest` suite in `test_app.py`, run with:

```text
python -m unittest -v test_app.py
```

Persistence cases use temporary directories. The suite covers:

- Product CRUD, SKU lookup, stock status, invalid numeric values, and
  persistence across reloads.
- Linked-list, stack, queue, binary-search-tree, and hash-table operations.
- Linear search, binary search, merge sort, and nearest-price search.
- Inventory undo for create, update, and delete operations.
- Waitlist registration, sector assignment, service completion, and staff
  availability.
- Waiting, active, and finished ticket restoration after restart.
- Staff roster persistence and restrictions on removing busy staff.
- Finished-ticket deletion and protection of waiting or active tickets.
- Retry behavior for invalid optional numeric CLI input.

The project test report records these cases as passing. This coverage
demonstrates core rules and data operations; it does not establish
performance with large inventories, security, accessibility, or usability
with real store personnel. The GUI is a separate manual demonstration in the
project documentation.

## 8. Limitations and Future Work

Nav1Gate is a local prototype. The project documentation identifies the
absence of authentication, role permissions, encrypted records, and
multi-user synchronization. JSON files support the demonstration but do not
provide concurrent transaction handling or database-level recovery. The
inventory undo stack and restock queue are in-memory structures and are not
restored after restart; the persisted inventory and ticket records are the
durable state.

Possible future work includes a database-backed storage layer, controlled
multi-user access, role-based permissions, encrypted sensitive data, supplier
and purchase-order workflows, barcode scanning, configurable restock amounts,
and expanded GUI and integration testing. These are prospective enhancements,
not capabilities claimed for the current version.

## 9. Conclusion

Nav1Gate combines customer routing, staff availability, service history, and
PC-parts inventory in a small desktop application. It applies FIFO queues to
sector waitlists, tracks ticket state transitions, supports common inventory
operations, and demonstrates foundational data structures and algorithms in
context. Local JSON persistence enables the application to restore inventory,
staff, and service tickets after restart, while automated tests exercise
important business rules and algorithm behavior.

The project meets its prototype goals as an integrated educational system.
Its local-only architecture also defines a clear boundary: before use as
shared production software, it would need stronger persistence, user and
access controls, data protection, and operational testing.

## Project Materials

This paper is based on the Nav1Gate implementation and the accompanying
project documentation: [README](../README.md), [presentation
outline](presentation.md), [test report](test-report.md), [system
flowchart](flowchart.md), and [UML diagram](uml.md), together with `main.py`,
`gui.py`, `app.py`, `inventory.py`, `data_structures.py`, `algorithms.py`,
and `test_app.py`.