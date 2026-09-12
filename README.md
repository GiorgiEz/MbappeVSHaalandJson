# Mbappé vs Haaland — Data

Static JSON files powering the [Mbappé vs Haaland](https://mbappevshaaland.pages.dev/) frontend. Each file holds pre-computed statistics for both players, generated from match-log data sourced from FBref, Transfermarkt, and Fotmob.

## Table of Contents

- [Player Keying](#player-keying)
- [Common Types](#common-types)
  - [DetailedStatsType](#detailedstatstype)
  - [OverallCompetitionTiersType](#overallcompetitiontierstype)
- [All-Time Stats](#all-time-stats)
- [Club Stats](#club-stats)
- [Country Stats](#country-stats)
- [Honours](#honours)

## Player Keying

Every file in this repo (except `honours.json`, see [Honours](#honours)) is a flat object keyed by each player's full name, with an identical shape under each key:

```json
{
  "Kylian Mbappe": { /* per-player data — shape varies by file, see below */ },
  "Erling Haaland": { /* same shape as above */ }
}
```

## Common Types

These building blocks recur across every file below.

### GeneralStatsType

The core stat line — apps, output, and a nested breakdown of secondary stats.

```ts
interface GeneralStatsType {
    apps: number;
    goals: number;
    assists: number;
    minutes: number;
    minutes_per_goal: number | null;          // null when goals = 0
    minutes_per_goal_contribution: number | null; // null when goals + assists = 0
    details: DetailedStatsType;
}
```

### DetailedStatsType

Secondary stats, grouped by theme. Every ratio/percentage field is `null` when its denominator is 0 (e.g. no shots taken, no penalties attempted).

```ts
interface CountTotalType {
    count: number;
    total: number;
}

interface DetailedStatsType {
    scoring: {
        goals_per_game: number;
        hat_tricks: number;                 // games with 3+ goals
    };
    appearances: {
        games_started: CountTotalType;
        captain: CountTotalType;
        starting_percentage: number;
        captain_percentage: number;
    };
    penalties: {
        scored: number;
        attempted: number;
        conversion_percentage: number | null;
        won: number;                        // penalties won for the team
    };
    shooting: {
        shots: number;
        shots_on_target: number;
        shots_on_target_percentage: number | null;
    };
    discipline: {
        yellow_cards: number;
        red_cards: number;
    };
    general: {
        fouls_committed: number;
        fouls_drawn: number;
        offsides: number;
        crosses: number;
    };
    defending: {
        tackles_won: number;
        interceptions: number;
    };
}
```

### OverallCompetitionTiersType

The recurring "totals, then a breakdown by tier and competition" shape used by most grouped stats (by age, season, year, etc.).

```ts
interface OverallCompetitionTiersType {
    overall: GeneralStatsType;
    competition_tiers: {
        [competition_tier: string]: {   // e.g. "Domestic League", "Continental Club Cup"
            overall: GeneralStatsType;
            competitions: {
                [competition: string]: GeneralStatsType; // e.g. "Ligue 1", "UEFA Champions League"
            };
        };
    };
}
```

## All-Time Stats

Career-wide statistics across every club and international appearance.

**`career.json`** — total career, and a split by club vs. country.

```ts
{
  [category: "career" | "club" | "country"]: GeneralStatsType
}
```

**`age.json`** — stats grouped by player age, in full years.

```ts
{
  [age: string]: OverallCompetitionTiersType   // e.g. "19", "20", "21"
}
```

**`competitions.json`** — stats split by `club` vs. `country`, each with its own tier breakdown.

```ts
{
  [team_type: "club" | "country"]: OverallCompetitionTiersType
}
```

**`favourite_opponents.json`** — appearances and output against each opponent, split by club vs. country, ranked by goals then appearances.

```ts
{
  club: { opponent: string; apps: number; goals: number; assists: number }[];
  country: { opponent: string; apps: number; goals: number; assists: number }[];
}
```

**`finals.json`** — performance in cup and tournament finals. Every player included, even if their apps are 0.

```ts
{
  Y: OverallCompetitionTiersType   // "Y" is a fixed, single key — not a variable one
}
```

**`seasons.json`** — season-by-season breakdown (e.g. `"23/24"`), each with its own tier breakdown.

```ts
{
  [season: string]: OverallCompetitionTiersType
}
```

## Club Stats

Statistics scoped to club football only.

**`clubs.json`** — stats for each club played for.

```ts
{
  [club_name: string]: OverallCompetitionTiersType
}
```

**`seasons.json`** — club stats by season (club football only, distinct from the all-time `seasons.json` above).

```ts
{
  [season: string]: OverallCompetitionTiersType
}
```

**`competitions.json`** — ⚠️ **not confirmed.** The frontend types don't define a distinct shape for this file. Given the pattern used everywhere else, it's most likely the tier-breakdown shape alone (already scoped to club, no further wrapping needed):

```ts
{
  [competition_tier: string]: {
    overall: GeneralStatsType;
    competitions: { [competition: string]: GeneralStatsType };
  }
}
```

## Country Stats

Statistics scoped to international football only.

**`years.json`** — stats grouped by calendar year.

```ts
{
  [year: string]: OverallCompetitionTiersType
}
```

**`competitions.json`** — ⚠️ **not confirmed**, same caveat as club stats above — likely the same bare tier-breakdown shape, scoped to country football.

## Honours

**`honours.json`** is shaped differently from every other file — it's nested one level deeper per player, and its `overall` field takes one of two shapes depending on the section.

```ts
{
  [player_name: string]: {
    team_trophies: HonourType;
    individual_awards: HonourType;
  }
}

interface HonourType {
    overall: OverallType;
    breakdown: {
        title: string;
        count: number;
        entries: { [key: string]: string | number | boolean }[]; // varies by trophy — season, year, club, team, detail, etc.
    }[];
}

// team_trophies.overall: split into club vs. international
interface ClubInternationalOverallType {
    total: number;
    club: { count: number; breakdown: { category: string; count: number }[] };
    international: { count: number; breakdown: { category: string; count: number }[] };
}

// individual_awards.overall: a single flat breakdown (no club/international split)
interface FlatOverallType {
    total: number;
    breakdown: { category: string; count: number }[];
}

type OverallType = ClubInternationalOverallType | FlatOverallType;
```