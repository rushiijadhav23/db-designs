# IPL Management System - ERD code

```
title IPL Management System

// Each IPL season - stores title sponsor, broadcaster and winner
seasons [icon: calendar, color: blue] {
  id serial pk 

  seasonWinner_id fk null
  broadcaster_id fk 

  seasonName string
  titleSponsor string 
  startDate date
  endDate date

  createdAt timestamp
  updatedAt timestamp
}

// IPL Franchise teams
teams [icon: teams, color: blue] {
  id serial pk

  owner_id fk

  teamName string
  city string
  homeGround string
  logo_url string

  createdAt timestamp
  updatedAt timestamp
}

// Owners of IPL franchises
owners [icon: user-check, color: blue] {
  id serial pk 

  ownerName string
  companyName string
  
  createdAt timestamp
  updatedAt timestamp
}

// all players list
players [icon: users, color: yellow] {
  id serial pk

  name string
  role enum('BATSMAN', 'BOWLER', 'ALLROUNDER', WICKETKEEPER')
  nationality string
  dob date

  createdAt timestamp
  updatedAt timestamp
}

// Player team history
player_team_history [icon: refresh-cw, color: yellow] {
  id serial pk

  player_id fk
  team_id fk
  season_id fk

  auctionPrice decimal
  isCaptain boolean

  createdAt timestamp
  updatedAt timestamp
}

// Overall career stats per player
player_records [icon: trending-up, color: yellow] {
  id serial pk 

  player_id fk

  runsScored int
  wicketsTaken int
  totalCatches int
  fifties int
  hundreds int
  matchesPlayed int

  createdAt timestamp
  updatedAt timestamp
}

// match grounds 
venues [icon: map-pin, color: green] {
  id serial pk 

  name string
  city string
  capacity int

  createdAt timestamp
  updatedAt timestamp
}

// Every match has 2 teams, 1 venue, and under which season this match is played
matches [icon: activity, color: orange] {
  id serial pk

  season_id fk
  venue_id fk
  team1_id fk
  team2_id fk
  winner_id fk null

  matchDate date
  matchType enum('LEAGUE', 'QUALIFIER', 'ELIMINATOR', 'FINAL')
  status enum('SCHEDULE', 'COMPLETED', 'ABANDONED')

  createdAt timestamp
  updatedAt timestamp
}

// all sponsor list
sponsors [icon: solar-panel, color: purple] {
  id serial pk 

  sponsorName string
  logo_url string

  createdAt timestamp
  updatedAt timestamp
}

// one team can have many sponsors and also one sponsor can sponsor many teams
team_sponsors [icon: link, color: purple] {
  id serial pk 

  team_id fk
  sponsor_id fk
  season_id fk

  sponsorType enum('TITLE', 'JERSEY', 'KIT')

  createdAt timestamp
  updatedAt timestamp
}

broadcasters [icon: video, color: red] {
  id serial pk 

  broadcasterName string
  platform enum('TV', 'OTT', 'BOTH')

  createdAt timestamp
  updatedAt timestamp
}

// One owner owns one team
owners.id - teams.owner_id

// Season has one winner team and one broadcaster like wise many seasons can also have one winner/broadcaster
teams.id < seasons.seasonWinner_id
broadcasters.id < seasons.broadcaster_id

// A team can have many players across seasons
teams.id < player_team_history.team_id
// A player can have many past history of teams
players.id < player_team_history.player_id
// A season can have many players
seasons.id < player_team_history.season_id

// Each player has one career record
players.id - player_records.player_id

// A season can have many matches
seasons.id < matches.season_id

// A venue can host many matches
venues.id < matches.venue_id

// Two teams play a match
teams.id < matches.team1_id
teams.id < matches.team2_id

// Winner of the match is also a team
teams.id < matches.winner_id

// Sponsors and teams have many to many relationship via the junction table
// A team can have many sponsors
teams.id < team_sponsors.team_id

// sponsors can sponsor many teams
sponsors.id < team_sponsors.sponsor_id

// A season can have many sponsors 
seasons.id < team_sponsors.sponsor_id
```