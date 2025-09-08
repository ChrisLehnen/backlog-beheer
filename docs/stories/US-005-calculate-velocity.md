---
id: US-005
epic: EPIC-001
title: Calculate and Track Team Velocity
status: todo
priority: HOOG
type: feature
story_points: 3
sprint: backlog
owner: developer
created: 2025-09-08
updated: 2025-09-08
assigned_to: solo-dev
dependencies:
  - US-004
requirements:
  - REQ-002
---

# US-005: Calculate and Track Team Velocity

## User Story
**Als** solo programmeur  
**wil ik** mijn velocity automatisch berekend zien  
**zodat** ik betere sprint planning kan doen en deadlines kan voorspellen

## Problem Statement
### Current Situation
- Geen inzicht in werkelijke productiviteit
- Estimates worden niet gevalideerd
- Planning is gebaseerd op gevoel

### Desired Outcome
- Automatische velocity tracking
- Trending en forecasting
- Data-driven estimates

## Acceptance Criteria

### AC-1: Velocity Calculation
**Gegeven** een voltooide sprint  
**Wanneer** de sprint wordt afgesloten  
**Dan** wordt velocity berekend als:
- Som van story points van DONE items
- Exclusief carried-over items
- Inclusief bugs/tech debt met points

### AC-2: Velocity Trending
**Gegeven** meerdere voltooide sprints  
**Wanneer** ik velocity trends bekijk  
**Dan** zie ik:
- Rolling average (3 sprints)
- Min/max range
- Trend indicator (↑↓→)

### AC-3: Forecasting
**Gegeven** velocity history  
**Wanneer** ik een release plan  
**Dan** krijg ik forecast:
- Sprints needed voor remaining backlog
- Confidence intervals (optimistic/realistic/pessimistic)
- Expected completion date

## Technical Notes

### Velocity Metrics
```yaml
velocity_report:
  sprint_001: 18
  sprint_002: 22
  sprint_003: 20
  average: 20
  trend: stable
  forecast:
    remaining_points: 150
    sprints_needed: 7-8
    completion_date: 2025-11-15
```

### Affected Components
- `/scripts/calculate-velocity.sh`
- `/docs/metrics/velocity.md`
- Dashboard velocity widget

## Test Scenarios

### Test Case 1: Single Sprint
1. Complete sprint with 20 points
2. Run velocity calculation
3. Verify correct calculation
4. Check report generation

### Test Case 2: Trend Analysis
1. Complete 5 sprints various velocities
2. Calculate rolling average
3. Verify trend detection
4. Test edge cases (0 velocity)

## Definition of Done
- [ ] Velocity calculation script
- [ ] Historical data storage
- [ ] Trend analysis logic
- [ ] Forecasting algorithm
- [ ] Velocity report template
- [ ] Dashboard integration