# Comic-Con Parking System - ERD code

```
title Comic-Con Parking System

vehicles [icon: car, color: blue] {
  id serial pk

  category_id fk

  owner string
  ownerContact string
  vehicleNumber string

  createdAt timestamp
  updatedAt timestamp
}

vehicle_categories [icon: bus, color: blue] {
  id serial pk

  name string
  fees_per_hour decimal
  description string

  createdAt timestamp
  updatedAt timestamp
}

// Zone -> Levels -> Spots
parking_zones [icon: parking, color: orange] {
  id serial pk

  zoneName string
  zoneLocation string
  zoneDescription string

  createdAt timestamp
  updatedAt timestamp
}

parking_levels [icon: building, color: orange] {
  id serial pk 

  zone_id fk

  levelName string
  levelCapacity int

  createdAt timestamp
  updatedAt timestamp
}

spots_categories [icon: clipboard-check, color: green] {
  id serial pk 

  categoryName string

  createdAt timestamp
  updatedAt timestamp
}

parking_spots [icon: map-pin, color: ]{
  id serial pk

  level_id fk
  spot_category_id fk

  spotNumber string
  status enum('AVAILABLE', 'OCCUPIED')

  createdAt timestamp
  updatedAt timestamp
}

parking_sessions [icon: timer, color: purple] {
  id serial pk

  vehicle_id fk
  parkingSpot_id fk 

  arrivalTime timestamp
  exitTime timestamp
  status enum('ACTIVE', 'COMPLETED')
  total_fee decimal

  createdAt timestamp
  updatedAt timestamp
}

parking_tickets [icon: ticket, color: yellow] {
  id serial pk

  parkingSession_id fk
  
  ticketNumber string
  qrCode_url string

  createdAt timestamp
  updatedAt timestamp
}

payments [icon: cash, color: red] {
  id serial pk

  parkingSession_id fk

  amountPaid decimal
  payment_method enum('CASH', 'CARD', 'UPI')
  payment_status enum('SUCCESS', 'PENDING', 'FAILED')
  paid_at timestamp

  createdAt timestamp
  updatedAt timestamp
}


// A single vehicle category can have multiple vehicles in it
vehicle_categories.id < vehicles.category_id

// There can be multiple levels in a particular zone
parking_zones.id < parking_levels.zone_id

// There can be multiple spots in a particular level
parking_levels.id < parking_spots.level_id

// One stop category can have many spots
spots_categories.id < parking_spots.spot_category_id

// A vehicle can be parked multiple time so can have many sessions
vehicles.id < parking_sessions.vehicle_id

// A single session can have a single ticket
parking_sessions.id - parking_tickets.parkingSession_id

// One spot can be used by many sessions overtime
parking_spots.id < parking_sessions.parkingSpot_id

// A payment can be done on bases of session
parking_sessions.id < payments.parkingSession_id
```