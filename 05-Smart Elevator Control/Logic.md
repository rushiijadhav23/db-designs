# Smart Elevator Control System - ER Diagram Documentation

## Business Context

LiftGrid Systems builds intelligent elevator control platforms for large commercial buildings across India. Their software is used in corporate towers, malls, airports, hospitals and high-rise residential complexes where dozens of elevators operate together across many floors.

The platform helps operations teams monitor performance, track ride history, manage maintenance and ensure elevators remain safe, efficient and available at all times.

---

## Core Flow

```
Buildings → Floors + Elevators
                ↓
     elevator_floors (junction)
                ↓
     Floor Request Generated
                ↓
     Elevator Assigned to Request
                ↓
     Ride Created (passenger travels)
                ↓
     Ride Logged for Analytics
                ↓
     Maintenance Tracked Separately
```

---

## Core Design Decisions

### 1. Buildings, Floors and Elevators - Separate Entities

Everything in this system belongs to a building. Floors and elevators are both separate entities that link back to `buildings` via FK.

```
buildings  → common info (name, location)
floors     → each floor of a building (building_id fk)
elevators  → each elevator of a building (building_id fk)
```

This separation means you can independently query all floors of a building or all elevators of a building without joining unnecessary data.

---

### 2. Elevators and Floors - Many-to-Many via Junction Table

A single elevator can serve multiple floors. A single floor can be served by multiple elevators. This is a many-to-many relationship that requires a junction table.

```
elevator_floors {
  elevator_id fk
  floor_id fk
  is_active boolean  ← does this elevator currently stop at this floor?
}
```

Without this junction table you could only store one floor per elevator - making it impossible to model real-world elevator systems where elevators serve ranges of floors.

---

### 3. Static vs Dynamic Elevator Data

A common mistake is storing everything about an elevator in one table. This system separates:

- **`elevators`** → static configuration that rarely changes (capacity, model, manufacturer, installation date)
- **`elevator_status`** → dynamic data that changes constantly (current floor, current status)

**Why separate?**
An elevator's status changes thousands of times a day as it moves between floors. Storing this in the `elevators` table would mean constantly updating the same row, making it impossible to track status history and causing performance issues.

`elevator_status` stores each status update as a new row - giving a full timeline of where every elevator was and when.

---

### 4. Requests vs Rides - Two Different Concepts

These are often confused but serve completely different purposes:

- **`requests`** → a button press event on a floor. Records which floor made the request, which direction (UP/DOWN), when it was made and whether an elevator has been assigned yet. The `elevator_id` is **nullable** because at the moment of pressing the button no elevator is assigned yet.

- **`rides`** → the actual passenger trip. Created when the elevator arrives and the passenger travels from source floor to destination floor. Records start time, end time and destination.

This separation means you can track unserved requests, measure response times and analyse ride patterns independently.

---

### 5. Maintenance - Full History Preserved

Maintenance records are stored in a separate table with one row per maintenance incident. This means:

- Full maintenance history is preserved forever
- You can see how many times an elevator broke down
- You can track which technician handled each incident
- You can measure how long each maintenance took

If maintenance data was stored inside the `elevators` table it would be overwritten every time - losing all history.

---

## Entity Summary

| Entity | Purpose |
|---|---|
| `buildings` | Buildings connected to the LiftGrid platform |
| `floors` | Individual floors within each building |
| `elevators` | Static elevator configuration per building |
| `elevator_floors` | Junction table - which elevators serve which floors |
| `elevator_status` | Dynamic status and position tracking |
| `requests` | Floor button press events with assignment status |
| `rides` | Actual passenger trips with start and end times |
| `maintenance` | Full maintenance history per elevator |

---

## Relationship Summary

| Relationship | Type | Reason |
|---|---|---|
| buildings → floors | One-to-Many | One building has many floors |
| buildings → elevators | One-to-Many | One building has many elevators |
| elevators → elevator_floors | One-to-Many | One elevator serves many floors |
| floors → elevator_floors | One-to-Many | One floor served by many elevators |
| elevators → elevator_status | One-to-Many | Many status updates per elevator |
| floors → requests | One-to-Many | Many requests from one floor over time |
| elevators → requests | One-to-Many | One elevator assigned to many requests |
| requests → rides | One-to-One | One request triggers one ride |
| elevators → rides | One-to-Many | One elevator completes many rides |
| floors → rides | One-to-Many | One floor is destination for many rides |
| elevators → maintenance | One-to-Many | One elevator has many maintenance records |

---

## Files

- `code_erd.md` - Eraser.io ERD source code
- `SEC_ERD.png` - Exported diagram image from eraser.io