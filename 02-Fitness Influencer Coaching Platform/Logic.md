# Fitness Influencer Coaching Platform - ER Diagram Documentation

## Business Context

A fitness influencer has grown their online coaching business beyond Instagram DMs and video calls. This database is designed to support a structured platform where one or more trainers can onboard clients, sell fitness plans, schedule sessions, track progress and maintain regular check-ins.

This is **not a gym management system**. It is an **online coaching ecosystem** where trainers manage clients remotely and provide structured fitness support.

---

## Core Design Decisions

### 1. Users, Trainers and Clients - Table Inheritance Pattern

Instead of creating two completely separate tables for trainers and clients (which would repeat common fields like name, email, phone), we use a **shared `users` table** for all common attributes and extend it with role-specific profile tables.

```
users         → stores common info (name, email, phone, auth)
client        → stores client-specific info (fitness goal, health conditions, height)
trainer       → stores trainer-specific info (specialization, experience, bio)
```

A user can be **both a trainer and a client** at the same time - for example, a fitness coach who is also being coached by someone else. This is handled by two boolean flags in the `users` table:

```
is_trainer boolean
is_client boolean
```

Each user has at most **one** trainer profile and **one** client profile - making both relationships one-to-one.

---

### 2. Plans vs Subscriptions - Template vs Purchase

A common mistake would be to mix the plan definition with the purchase record. We separate them cleanly:

- **`plans`** → a template created by a trainer. Defines the name, duration, price and what is included (consultations, live sessions, diet plan). One plan can be purchased by many clients.

- **`subscriptions`** → the actual purchase record. Tracks which client bought which plan, when it started, when it ends and its current status (active, completed, cancelled, paused).

This separation means if 50 clients buy the "3 Month Transformation" plan, there is only **one row in `plans`** but **50 rows in `subscriptions`**.

---

### 3. Sessions vs Check-ins - Two Very Different Things

These are often confused but serve completely different purposes:

- **`sessions`** → a live or consultation meeting between a trainer and client. Scheduled at a specific time, has a duration, and a type (LIVE or CONSULTATION). The trainer conducts it.

- **`checkins`** → a regular progress update **submitted by the client**. Not a meeting - just a structured report (weight, measurements, steps) submitted weekly or as required.

Both are linked to a `subscription` because they only happen when a client is actively subscribed to a plan.

---

### 4. Progress Tracking - Separated from Users

Progress data (weight, waist, steps) is **not stored in the `users` or `client` table** because it changes over time. Every check-in submission creates a new row in the `progress` table, giving a full history of the client's journey.

```
checkins → progress (one-to-one per submission)
```

Note: `height_cms` is stored in the `client` table since height does not change over time and does not need to be tracked historically.

---

### 5. Trainer Notes - Flexible Linking

Trainer notes are designed to be flexible. A note can be written:
- In response to a **check-in** (e.g. feedback on this week's progress)
- After a **session** (e.g. notes from a consultation)
- Or both

Both `session_id` and `checkin_id` are nullable foreign keys, meaning a note does not have to be tied to either - it can be a standalone observation about a client.

Every note always records:
- Which **trainer** wrote it
- Which **client** it is about

---

### 6. Workout and Diet Plans - Actual Content Storage

The `plans` table only defines what is **included** in a plan using boolean flags. The actual content is stored separately:

- **`workout_plans`** → the actual workout routine assigned to a subscription (title, description, schedule)
- **`diet_plans`** → the actual diet guidance assigned to a subscription (title, description, calorie target, notes)

Both are linked to a specific `subscription` (not just a plan template) because a trainer may customise the workout or diet for each individual client even if they are on the same plan.

---

### 7. Payments - Linked to Subscriptions

Payments are linked to `subscriptions` rather than directly to clients or plans. This allows tracking **multiple payment attempts** for the same subscription (e.g. a failed UPI attempt followed by a successful card payment).

Each payment records:
- Amount, method and status
- A `transaction_id` for reference
- Timestamp of when payment was made

---

## Entity Summary

| Entity | Purpose |
|---|---|
| `users` | Common identity for all platform users |
| `client` | Client-specific profile extending users |
| `trainer` | Trainer-specific profile extending users |
| `plans` | Plan templates created by trainers |
| `subscriptions` | Client purchases of a plan |
| `payments` | Payment attempts per subscription |
| `sessions` | Live or consultation meetings |
| `checkins` | Weekly progress submissions by clients |
| `progress` | Measurements recorded per check-in |
| `trainer_notes` | Trainer feedback on sessions or check-ins |
| `workout_plans` | Workout routines assigned per subscription |
| `diet_plans` | Diet guidance assigned per subscription |

---

## Relationship Summary

| Relationship | Type | Reason |
|---|---|---|
| users → client | One-to-One | One user has at most one client profile |
| users → trainer | One-to-One | One user has at most one trainer profile |
| trainer → plans | One-to-Many | A trainer can create multiple plans |
| plans → subscriptions | One-to-Many | One plan can be bought by many clients |
| client → subscriptions | One-to-Many | A client can subscribe to multiple plans over time |
| subscriptions → payments | One-to-Many | Multiple payment attempts per subscription |
| subscriptions → sessions | One-to-Many | Multiple sessions within a subscription period |
| subscriptions → checkins | One-to-Many | Multiple check-ins during a subscription |
| checkins → progress | One-to-One | One progress record per check-in submission |
| trainer → trainer_notes | One-to-Many | A trainer can write many notes |
| client → trainer_notes | One-to-Many | Many notes can exist for one client |
| subscriptions → workout_plans | One-to-Many | Multiple workout routines per subscription |
| subscriptions → diet_plans | One-to-Many | Multiple diet plans per subscription |

---

## Files

- `fitness_erd.md` - Eraser.io ERD source code
- `FICP_ERD.png` - Exported diagram image *(export from Eraser.io and add here)*