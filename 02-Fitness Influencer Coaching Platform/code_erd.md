```
title Fitness Influencer Coaching Platform
users [icon: user, color: blue] {
  id serial pk
  fname string
  lname string
  email string not null unique
  phone string
  password string
  auth enum('local', 'google') not null

  createdAt timestamp
  updatedAt timestamp
}

client [icon: client, color: blue] {
  id serial pk
  user_id fk unique

  height_cms decimal
  fitness_goal string
  health_conditions string
}

trainer [icon: weight, color: blue] {
  id serial pk
  user_id fk unique

  specialization string
  yearsofexp int
  bio string
}



plans [icon: paperclip, color: orange] {
  id serial pk
  trainer_id fk

  name string
  duration_days int
  price decimal
  plan_type enum('FULL', 'CONSULTATION')

  is_consultation_included boolean
  is_livesessions_included boolean
  is_dietplan_included boolean

  createdAt timestamp
  updatedAt timestamp
}

subscriptions [icon: clipboard-check, color: green] {
  id serial pk

  client_id fk
  plan_id fk

  planstartedAt timestamp
  planendedAt timestamp

  status enum('active', 'completed', 'cancelled', 'paused')

  createdAt timestamp
  updatedAt timestamp
}

payments [icon: payment, color: purple] {
  id serial pk 

  subscription_id fk 

  amount decimal
  payment_method enum('UPI', 'CARD')
  status enum('PENDING', 'SUCCESS', 'FAILED')

  transaction_id string
  paid_at timestamp

  createdAt timestamp
  updatedAt timestamp
}

sessions [icon: google-meet, color: red] {
  id serial pk

  subscription_id fk
  trainer_id fk

  scheduled_at timestamp
  duration_mins int
  type enum('CONSULTATION', 'LIVE')

  status enum('SCHEDULED', 'COMPLETED', 'CANCELLED')

  createdAt timestamp
  updatedAt timestamp
}

checkins [icon: sum, color: red] {
  id serial pk

  client_id fk
  subscription_id fk

  submitted_at timestamp

  createdAt timestamp
  updatedAt timestamp
}

progress [icon: trending-up, color: yellow] {
  id serial pk

  checkin_id fk

  weight_kgs decimal
  waist_inchs decimal
  steps int
  
  createdAt timestamp
  updatedAt timestamp
}

trainer_notes [icon: notebook, color: blue] {
  id serial pk

  session_id fk null
  checkin_id fk null
  trainer_id fk
  client_id fk

  note text
  created_at timestamp
}

workout_plans [icon: activity, color: green] {
  id serial pk

  subscription_id fk
  trainer_id fk

  title string
  description text
  schedule text

  createdAt timestamp
  updatedAt timestamp
}

diet_plans [icon: coffee, color: green] {
  id serial pk

  subscription_id fk
  trainer_id fk

  title string
  description text
  calories_targer int
  notes text

  createdAt timestamp
  updatedAt timestamp
} 

// A client is just the role of a user
users.id - client.user_id
// A trainer is also a user 
users.id - trainer.user_id

// A trainer can create multiple plans
trainer.id < plans.trainer_id
// A client can have multiple subscriptions
client.id < subscriptions.client_id

// A plan can have many subscriptions
plans.id < subscriptions.plan_id
// A subscription can have many payment tries
subscriptions.id < payments.subscription_id

// There can be multiple sessions when subscribed
subscriptions.id < sessions.subscription_id
// A trainer can take multiple sessions
trainer.id < sessions.trainer_id

// there can be multiple checkins when subscribed
subscriptions.id < checkins.subscription_id
// Multiple checkins can be done by a single client
client.id < checkins.client_id

// For each checkin there is a progress data maintained 
checkins.id - progress.checkin_id

// A trainer can write multiple notes
trainer.id < trainer_notes.trainer_id
// A client can access his multiple notes
client.id < trainer_notes.client_id
// Multiple notes can exist for a session
sessions.id < trainer_notes.session_id
// Multiple notes exits for a checkins
checkins.id < trainer_notes.checkin_id

// A subscription can have many workout plans
subscriptions.id < workout_plans.subscription_id
// A trainer has access to multiple workout plans
trainer.id < workout_plans.trainer_id
// A subscription can have many diet plans
subscriptions.id < diet_plans.subscription_id
// A trainer can modify multiple diet plans
trainer.id < diet_plans.trainer_id
```