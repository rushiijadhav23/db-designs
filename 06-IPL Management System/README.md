# IPL Management System - ER Diagram Documentation

## Business Context

We needed to design a db-design for IPL Management system.
The IPL Management System is a database design for managing all aspects of the Indian Premier League - teams, players, owners, seasons, matches, venues, sponsors and broadcasters. The system tracks player transfers across seasons, match results, career statistics and commercial relationships.

---

## Core Flow

```
Owners → Teams → Players (via auction each season)
           ↓
        Seasons → Matches → Venues
           ↓
        Sponsors (many-to-many with teams)
           ↓
        Broadcasters (one per season)
```

---

## Core Design Decisions

### 1. Seasons - The Central Entity

Every IPL season ties everything together. A season has:
- A title sponsor (stored as string - simple and practical)
- A broadcaster (FK - since broadcasters have name and platform info)
- A winner (FK to teams - null until season ends)

Matches, player transfers and sponsorships all link back to a specific season.

---

### 2. Teams and Owners - One-to-One

Each IPL team is owned by exactly one owner. The `owner_id fk` lives in `teams` since an owner is the "one" side and a team is uniquely tied to one owner. Owner stores company name since most IPL franchises are owned by corporations (e.g. Reliance Industries owns Mumbai Indians).

---

### 3. Player Team History - Handling Transfers

Players switch teams at the IPL auction every season. Storing just `team_id` in the `players` table would only capture the current team and lose all transfer history.

Instead a separate `player_team_history` table records:
- Which player played for which team in which season
- Their auction price that season
- Whether they were captain

This gives a complete transfer history for every player across all seasons.

---

### 4. Player Records - Career Totals

`player_records` stores overall career statistics per player - total runs, wickets, catches, fifties and hundreds. This is a one-to-one relationship with players since there is only one career record per player. Stats are updated after every match.

---

### 5. Matches - Two Teams, One Venue, One Season

Every match involves exactly two teams. Since a match has `team1_id` and `team2_id` both pointing to the `teams` table, two separate FK columns are used. The `winner_id` is nullable - null until the match is completed.

Match type tracks whether it's a league game, qualifier, eliminator or final.

---

### 6. Sponsors - Many-to-Many via Junction Table

One team can have multiple sponsors (title, jersey, equipment). One sponsor can back multiple teams. This many-to-many relationship is handled via a `team_sponsors` junction table.

The junction table also stores:
- Which season the sponsorship is for (sponsorships change each season)
- The type of sponsorship (TITLE, JERSEY, EQUIPMENT, ASSOCIATE)

---

### 7. Broadcasters - One Per Season

Broadcasting rights for an entire IPL season belong to one broadcaster. The `broadcaster_id fk` lives in `seasons` - one season has one broadcaster. Broadcaster stores name and platform type (TV, OTT or BOTH).

---

### 8. Venues - Standalone Entity

Venues are not tied to any specific team. Wankhede Stadium is MI's home ground but other teams also play there. Venues are standalone entities linked to matches via `venue_id fk` in `matches`.

---

## Entity Summary

| Entity | Purpose |
|---|---|
| `owners` | Franchise owners with company info |
| `teams` | IPL franchise teams |
| `seasons` | Each IPL season with winner and broadcaster |
| `players` | Player master data |
| `player_team_history` | Player-team mapping per season - handles transfers |
| `player_records` | Overall career statistics per player |
| `venues` | Standalone match venues |
| `matches` | Every IPL match with teams, venue and result |
| `sponsors` | Sponsor master data |
| `team_sponsors` | Many-to-many - team sponsorships per season |
| `broadcasters` | Broadcasting rights holders |

---

## Relationship Summary

| Relationship | Type | Reason |
|---|---|---|
| owners → teams | One-to-Many | One owner owns one team |
| teams → seasons (winner) | One-to-Many | One team can win multiple seasons |
| broadcasters → seasons | One-to-Many | One broadcaster covers multiple seasons |
| players → player_team_history | One-to-Many | Player has many team entries across seasons |
| teams → player_team_history | One-to-Many | Team has many players each season |
| seasons → player_team_history | One-to-Many | Season has many player-team mappings |
| players → player_records | One-to-One | One career record per player |
| seasons → matches | One-to-Many | One season has many matches |
| venues → matches | One-to-Many | One venue hosts many matches |
| teams → matches (team1/team2) | One-to-Many | One team plays many matches |
| teams → team_sponsors | One-to-Many | One team has many sponsors |
| sponsors → team_sponsors | One-to-Many | One sponsor backs many teams |
| seasons → team_sponsors | One-to-Many | Sponsorships tracked per season |

---

## Files

- `code_erd.md` - Eraser.io ERD source code
- `IPLMS_ERD.png` - Exported diagram image from eraser.io