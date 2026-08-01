# Mbappe VS Haaland - Stats
### GeneralStatsType: {apps, goals, assists, minutes, minutes_per_goal, minutes_per_goal_contribution}
### CompetitionType: {[competition]: GeneralStatsType}
### CompetitionTiersType: {[competition_tier]: {overall: GeneralStatsType, competitions: CompetitionType}}
### OverallCompetitionTiersType: {overall: GeneralStatsType, competition_tiers: CompetitionTiersType}

## All-Time Stats Types
### AgeType: {[age]: OverallCompetitionTiersType}
### CareerType: {career: GeneralStatsType, club: GeneralStatsType, country: GeneralStatsType}
### CompetitionsType: {[team_type]: OverallCompetitionTiersType}
### FavouriteOpponentsType: {[team_type]: {opponent, apps, goals, assists}[]}
### FinalsType: {Y: OverallCompetitionTiersType}
### SeasonsType: {[season]: OverallCompetitionTiersType}

## Club Stats Type
### ClubsType: {[team]: OverallCompetitionTiersType}
### ClubSeasonsType: {[season]: OverallCompetitionTiersType}

## Country Stats Type
### YearsType: {[year]: OverallCompetitionTiersType}

## Honours Type
### OverallBreakdown: {category, count}
### TeamType: {count, breakdown: OverallBreakdown[]}
### OverallType: {total, {club | international}: TeamType}

### EntryType: {[key: string]: string | number | boolean}
### BreakdownType: {title, count, entries: EntryType[]}

### HonourType: {overall: OverallType, breakdown: BreakdownType[]}
