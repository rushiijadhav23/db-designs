# Comic-Con Parking System - ER Diagram Documentation

## Business Context

Comic-Con India is a large multi-day convention venue hosting thousands of visitors. The venue has a structured parking facility divided into multiple zones and levels. Vehicles of different types - bikes, cars, SUVs, cabs and EVs - enter and exit the facility across multiple days.

Some parking areas are reserved for special categories like VIP guests, cosplayers with props, exhibitors, staff and EV charging vehicles.

The system tracks vehicle entry and exit, assigns parking spots, generates tickets, manages spot availability and handles payments.

---

## Core Flow

```
Vehicle Arrives
      ↓
System checks vehicle type → vehicle_categories
      ↓
System finds available spot → parking_zones → parking_levels → parking_spots
      ↓
Parking Session created → entry time recorded
      ↓
Parking Ticket generated → given to driver
      ↓
Vehicle Exits → exit time recorded → fee calculated
      ↓
Payment collected → session marked COMPLETED → spot marked AVAILABLE
```

---

## Core Design Decisions

### 1. Vehicle Categories - Separate Table, Not Enum

Instead of storing vehicle type as an enum inside the `vehicles` table, a separate `vehicle_categories` table is used. This stores the type name, hourly fee rate and description.

**Why not enum?**
Adding a new vehicle type with enum requires altering the database structure. With a separate table, adding a new category is just adding a new row - no structural changes needed.

Each vehicle has a `category_id fk` pointing to its category. One category can have many vehicles.

---

### 2. Parking Structure - Zone → Level → Spot Hierarchy

The parking facility is modeled as a three-level hierarchy:

```
parking_zones     → Zone A, Zone B, Zone C (physical areas of the venue)
      ↓
parking_levels    → Level 1, Level 2, Level 3 (floors within each zone)
      ↓
parking_spots     → Individual spots like A101, A102 (actual parking spaces)
```

Each level knows which zone it belongs to via `zone_id fk`. Each spot knows which level it belongs to via `level_id fk`. This makes it easy to navigate and query the entire structure.

---

### 3. Spot Categories - Reserved Parking

A separate `spots_categories` table stores the types of reserved parking:
- GENERAL
- VIP
- EV
- COSPLAYER
- EXHIBITOR
- STAFF

Each spot has a `spot_category_id fk` pointing to its category. This makes it easy to find available VIP spots, EV charging spots etc. without scanning the entire spots table.

---

### 4. Parking Sessions - The Core of the System

A parking session is created every time a vehicle enters the facility. It records:
- Which vehicle entered
- Which spot was assigned
- Entry timestamp
- Exit timestamp (filled when vehicle leaves)
- Current status (ACTIVE or COMPLETED)
- Total fee (calculated from entry/exit time and vehicle category rate)

**One vehicle can have many sessions** - the same car can park on Day 1 and Day 3 of the event. **One spot can serve many sessions over time** - just not simultaneously. This is a one-to-many relationship from both vehicles and spots to sessions.

**Spot availability** is tracked via the `status enum('AVAILABLE', 'OCCUPIED')` on each `parking_spot`. When a session starts the spot is marked OCCUPIED. When the session ends it is marked AVAILABLE again.

---

### 5. Parking Tickets - Customer Facing Document

A parking ticket is the **customer-facing document** generated at entry. It contains:
- A unique ticket number for reference
- A QR code for scanning at exit

The ticket is linked to a session via `parkingSession_id fk`. All other details (vehicle, spot, times) are reachable through the session - no need to duplicate them in the ticket. This follows the same normalization principle applied throughout the design.

---

### 6. Payments - Linked to Sessions

Payments are linked to parking sessions since the fee is calculated based on session duration. Multiple payment attempts are supported per session to handle failed UPI or card transactions.

Each payment records:
- Amount paid
- Payment method (CASH, CARD, UPI)
- Payment status (SUCCESS, PENDING, FAILED)
- Timestamp of payment

---

## Entity Summary

| Entity | Purpose |
|---|---|
| `vehicle_categories` | Master list of vehicle types with hourly fee rates |
| `vehicles` | Registered vehicles with owner info and category |
| `parking_zones` | Physical zones within the venue |
| `parking_levels` | Floors within each zone |
| `spots_categories` | Types of parking spots (VIP, EV, GENERAL etc.) |
| `parking_spots` | Individual parking spaces with availability status |
| `parking_sessions` | Entry/exit tracking per vehicle visit |
| `parking_tickets` | Customer-facing ticket issued at entry |
| `payments` | Payment records per parking session |

---

## Relationship Summary

| Relationship | Type | Reason |
|---|---|---|
| vehicle_categories → vehicles | One-to-Many | One category has many vehicles |
| parking_zones → parking_levels | One-to-Many | One zone has many levels |
| parking_levels → parking_spots | One-to-Many | One level has many spots |
| spots_categories → parking_spots | One-to-Many | One category has many spots |
| vehicles → parking_sessions | One-to-Many | One vehicle can park many times |
| parking_spots → parking_sessions | One-to-Many | One spot can serve many sessions over time |
| parking_sessions → parking_tickets | One-to-One | One session generates one ticket |
| parking_sessions → payments | One-to-Many | Multiple payment attempts per session |

---

## Files

- `code_erd.md` - Eraser.io ERD source code
- `CCPS_ERD.png` - Exported diagram image from eraser.io