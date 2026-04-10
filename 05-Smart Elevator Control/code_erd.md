# Smart Elevator Control - ERD code

```
entity-relationship-diagram
title Smart Elevator Control
colorMode pastel
styleMode plain

// ─────────────────────────────────────────────
// BUILDINGS & FLOORS
// ─────────────────────────────────────────────

buildings [icon: building, color: blue] {
  id serial pk

  buildingName string

  createdAt timestamp
  updatedAt timestamp
}

floors [icon: layers, color: blue] {
  id serial pk

  building_id fk

  floorNumber int
  floorName string

  createdAt timestamp
  updatedAt timestamp
}

// ─────────────────────────────────────────────
// ELEVATORS
// ─────────────────────────────────────────────

// Static elevator configuration - no dynamic data here
elevators [icon: box, color: orange] {
  id serial pk

  building_id fk

  maxCapacity int
  model string
  manufacturer string
  installationDate date

  createdAt timestamp
  updatedAt timestamp
}

// Junction table - many elevators serve many floors
elevator_floors [icon: layout, color: orange] {
  id serial pk

  floor_id fk
  elevator_id fk
  is_active boolean

  createdAt timestamp
  updatedAt timestamp
}

// Dynamic status tracked separately from static elevator config
elevator_status [icon: adjust, color: orange] {
  id serial pk

  elevator_id fk

  currentFloor int
  elevatorStatus enum('IDLE', 'MOVING', 'MAINTENANCE')
  updatedStatusTime timestamp

  createdAt timestamp
  updatedAt timestamp
}

// ─────────────────────────────────────────────
// REQUESTS & RIDES
// ─────────────────────────────────────────────

// Button press event - elevator assigned after
requests [icon: receipt, color: green] {
  id serial pk

  floor_id fk
  elevator_id fk null

  direction enum('UP', 'DOWN')
  reqTime timestamp
  assignedStatus enum('ASSIGNED', 'NOTASSIGNED', 'DECLINED')

  createdAt timestamp
  updatedAt timestamp
}

// Actual passenger trip from source floor to destination floor
rides [icon: travis-ci, color: green] {
  id serial pk

  request_id fk
  elevator_id fk
  destination_floor_id fk

  rideStartTime timestamp
  rideEndTime timestamp

  createdAt timestamp
  updatedAt timestamp
}

// ─────────────────────────────────────────────
// MAINTENANCE
// ─────────────────────────────────────────────

// Full maintenance history per elevator - never overwritten
maintenance [icon: azure-maintenance-configuration, color: red] {
  id serial pk

  elevator_id fk

  maintenanceReason string
  maintenancePerson string
  maintenanceStartTime timestamp
  maintenanceEndTime timestamp
  maintenanceStatus enum('RESOLVED', 'PENDING', 'PARTS_NEEDED')

  createdAt timestamp
  updatedAt timestamp
}

// ─────────────────────────────────────────────
// RELATIONSHIPS
// ─────────────────────────────────────────────

// A building can have many floors
buildings.id < floors.building_id

// A building can have many elevators
buildings.id < elevators.building_id

// An elevator can serve many floors (junction table)
elevators.id < elevator_floors.elevator_id

// One floor can be served by many elevators
floors.id < elevator_floors.floor_id

// An elevator updates status many times
elevators.id < elevator_status.elevator_id

// A floor can generate many requests
floors.id < requests.floor_id

// An elevator can be assigned to many requests
elevators.id < requests.elevator_id

// A request triggers a single ride
requests.id - rides.request_id

// A single elevator can handle many rides over time
elevators.id < rides.elevator_id

// A destination floor can appear in many rides
floors.id < rides.destination_floor_id

// One elevator can have many maintenance records
elevators.id < maintenance.elevator_id
```