# Clinic Appointment and Diagnostics Platform - ER Diagram Documentation

## Business Context

A modern clinic wants to digitize its operations. The platform manages doctors, patients, appointments, consultations, diagnostic tests, reports and payments. Patients can visit doctors, book appointments, undergo prescribed tests and receive reports - all tracked in a structured and scalable database.

This is **not a hospital-level system**. It is focused on a clinic that handles appointments, consultations, diagnostics and reporting in a clean and practical way.

---

## Core Flow

```
Users (Doctor / Patient)
        ↓
   Appointments        ← patient books a slot with a doctor
        ↓
  Consultations        ← actual interaction happens (if patient shows up)
        ↓
Prescriptions + Prescribed Tests    ← doctor prescribes medicines and/or tests
        ↓
      Reports          ← test results generated after lab work
        ↓
     Payments          ← separate payments for consultation and tests
```

---

## Core Design Decisions

### 1. Users, Doctors and Patients - Table Inheritance Pattern

Instead of creating completely separate tables for doctors and patients (which would repeat common fields like name, email, phone), we use a **shared `users` table** for all common attributes and extend it with role-specific profile tables.

```
users     → stores common info (name, dob, email, phone, auth, role)
doctors   → stores doctor-specific info (qualification, specialization, experience)
patients  → stores patient-specific info (bloodGroup, allergies, emergencyContact)
```

A `role enum('DOCTOR', 'PATIENT', 'ADMIN')` in the `users` table identifies what type of user they are. Each user has at most **one** doctor profile and **one** patient profile - making both relationships one-to-one.

**Why `dob` instead of `age`?**
Age becomes incorrect every year if stored as a number. Date of birth is stored instead so age can always be calculated accurately at any point in time.

---

### 2. Appointments vs Consultations - Scheduled vs Actual

This is the most important distinction in the entire ERD:

- **`appointments`** → a scheduled time slot. The patient books it, but may or may not show up. Tracks who booked, with which doctor, when, and the current status.
- **`consultations`** → the actual interaction between doctor and patient. Only created if the patient shows up and the doctor sees them.

**Appointment statuses:**
- `SCHEDULED` → booked, waiting for the day
- `COMPLETED` → patient showed up, doctor saw them
- `CANCELLED` → clinic or patient cancelled
- `ABSENT` → patient never showed up

**One appointment leads to at most one consultation.** This is enforced by making `appointment_id` unique in the `consultations` table (one-to-one relationship).

> If you query `consultations` for a given `appointment_id` and no row exists - the appointment did not result in a consultation. No extra boolean column needed.

---

### 3. Consultations - Capturing the Visit

During a consultation the doctor records the patient's current physical stats and clinical findings. These are stored **per consultation** (not in the `patients` table) because they change every visit:

- `height_cms`, `weight_kgs`, `bloodPressure` → current physical measurements
- `symptoms` → what the patient is experiencing this visit
- `diagnosis` → what the doctor concludes
- `fees` → consultation charge

This gives a full historical record of every visit without overwriting previous data.

---

### 4. Prescriptions - One Row Per Medicine

Instead of storing all prescribed medicines as a single text string inside `consultations`, a separate `prescriptions` table is used with **one row per medicine**. This makes it queryable, structured and easy to update.

Each prescription row stores:
- Which consultation it belongs to
- Medicine name, dosage, frequency and duration

---

### 5. Test Catalog vs Prescribed Tests vs Reports - Three Separate Concepts

A common mistake is mixing all test-related data into one table. We separate them into three distinct concepts:

- **`tests_catalog`** → the clinic's menu of available tests (Blood Test, X-Ray, MRI). Defined once, referenced many times. Avoids typing "Blood Test" repeatedly.

- **`prescribed_tests`** → a junction between a consultation and the test catalog. One row per test prescribed during a consultation. Tracks the status of each test (PENDING, COMPLETED, CANCELLED).

- **`reports`** → the actual result generated after a test is completed. Contains findings, who generated it and when. Linked back to the specific prescribed test.

```
tests_catalog ← prescribed_tests → consultations
                      ↓
                   reports
```

---

### 6. Payments - One Table for Both Types

Payments are kept in a **single table** with a `payment_type` enum to distinguish between consultation and test payments. This avoids duplicating the same structure across two separate tables.

```
payment_type = 'CONSULTATION' → consultation_id has value, prescribed_test_id is null
payment_type = 'TEST'         → prescribed_test_id has value, consultation_id is null
```

Both FKs are **nullable** since a payment only belongs to one type at a time. Multiple payment attempts are supported per consultation or test - useful for tracking failed UPI attempts followed by successful payments.

---

## Entity Summary

| Entity | Purpose |
|---|---|
| `users` | Common identity for all platform users |
| `doctors` | Doctor-specific profile extending users |
| `patients` | Patient-specific medical profile extending users |
| `appointments` | Scheduled time slots between patient and doctor |
| `consultations` | Actual doctor-patient interaction per appointment |
| `prescriptions` | Medicines prescribed during a consultation |
| `tests_catalog` | Master list of available diagnostic tests |
| `prescribed_tests` | Tests prescribed during a specific consultation |
| `reports` | Results generated after a prescribed test is completed |
| `payments` | Payment records for consultations and tests |

---

## Relationship Summary

| Relationship | Type | Reason |
|---|---|---|
| users → doctors | One-to-One | One user has at most one doctor profile |
| users → patients | One-to-One | One user has at most one patient profile |
| patients → appointments | One-to-Many | A patient can book many appointments over time |
| doctors → appointments | One-to-Many | A doctor can attend many appointments |
| appointments → consultations | One-to-One | One appointment leads to at most one consultation |
| consultations → prescriptions | One-to-Many | Multiple medicines can be prescribed per consultation |
| consultations → prescribed_tests | One-to-Many | Multiple tests can be prescribed per consultation |
| tests_catalog → prescribed_tests | One-to-Many | One test type can be prescribed many times |
| prescribed_tests → reports | One-to-One | One report generated per prescribed test |
| consultations → payments | One-to-Many | Multiple payment attempts per consultation |
| prescribed_tests → payments | One-to-Many | Multiple payment attempts per test |

---

## Files

- `code_erd.md` - Eraser.io ERD source code
- `CADP_ERD.png` - Exported diagram image from eraser.io