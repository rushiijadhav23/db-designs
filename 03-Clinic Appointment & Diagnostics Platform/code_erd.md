# Clinic Appointment and Diagnostics Platform - ERD code
```
title Clinic Appointment and Diagnostics Platform
users [icon: users, color: blue] {
  id serial pk
  fname string
  lname string
  dob date
  email string not null unique
  phone string
  password string
  auth enum('local', 'google') not null
  role enum('DOCTOR', 'PATIENT', 'ADMIN')

  createdAt timestamp
  updatedAt timestamp
}

doctors [icon: stethoscope, color: blue] {
  id serial pk
  user_id fk

  qualification string
  yearsofexp int
  specialization string

  createdAt timestamp
  updatedAt timestamp
}

patients [icon: user, color: blue] {
  id serial pk
  user_id fk

  bloodGroup string
  emergencyContact string
  allergies string

  createdAt timestamp
  updatedAt timestamp
}

appointments [icon: calendar-check, color: orange] {
  id serial pk

  patient_id fk
  doctor_id fk

  scheduled_at timestamp
  appointment_reason string
  status enum('ABSENT', 'CANCELLED', 'SCHEDULED', 'COMPLETED')

  createdAt timestamp
  updatedAt timestamp
}

consultations [icon: table, color: purple] {
  id serial pk

  appointment_id fk

  height_cms decimal
  weight_kgs decimal
  bloodPressure string
  symptoms string
  diagnosis string
  fees decimal

  createdAt timestamp
  updatedAt timestamp
}

prescriptions [icon: checklist, color: purple] {
  id serial pk

  consultation_id fk
  medicine_name string
  dosage string
  frequency string
  duration_days int 

  createdAt timestamp
  updatedAt timestamp
}

tests_catalog [icon: test-tube, color: green] {
  id serial pk

  name string not null
  description text
  price decimal

  createdAt timestamp
  updatedAt timestamp
}

//Prescribed tests
prescribed_tests [icon: clipboard, color: green] {
  id serial pk

  consultation_id fk
  test_id fk

  status enum('PENDING', 'COMPLETED','CANCELLED')

  createdAt timestamp
  updatedAt timestamp
}

// Reports — results generated after a test is completed
reports [icon: file-text, color: red] {
  id serial pk

  prescribed_test_id fk

  findings text
  generated_by string
  generated_at timestamp

  createdAt timestamp
  updatedAt timestamp
}

payments [icon: paypal, color: yellow] {
  id serial pk

  consultation_id fk null
  prescribed_test_id fk null

  payment_type enum('CONSULTATION', 'TEST')
  amount decimal
  status enum('PENDING', 'SUCCESS', 'FAILED')
  payment_method enum('CASH', 'UPI', 'CARD')
  paid_at timestamp

  createdAt timestamp
  updatedAt timestamp
}

// A user can be a patient or a doctor
users.id - doctors.user_id
users.id - patients.user_id

// A patient can book many appointments
patients.id < appointments.patient_id

// A doctor can handle multiple appointments
doctors.id < appointments.doctor_id

// An single appointment can be converted to a consultation
appointments.id - consultations.appointment_id

// For a single consultation there can be many prescriptions
consultations.id < prescriptions.consultation_id

// One consultation can have many prescribed tests
consultations.id < prescribed_tests.consultation_id

// Many prescribed test references a tests from catalog
tests_catalog.id < prescribed_tests.test_id

// Each prescribed test can have one report
prescribed_tests.id - reports.prescribed_test_id

// A single consultation can have many payment retries
consultations.id < payments.consultation_id

// A prescribed_test can have many payment retries
prescribed_tests.id < payments.prescribed_test_id

```