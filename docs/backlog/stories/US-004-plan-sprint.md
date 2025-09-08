---
id: US-004
epic: EPIC-001
title: Plan Sprint with Capacity Planning
status: todo
priority: HOOG
type: feature
story_points: 5
sprint: backlog
owner: developer
created: 2025-09-08
updated: 2025-09-08
assigned_to: solo-dev
dependencies:
  - US-001
  - US-002
requirements:
  - REQ-002
---

# US-004: Plan Sprint with Capacity Planning

## User Story
**Als** solo programmeur  
**wil ik** sprints kunnen plannen met capacity planning  
**zodat** ik realistische doelen stel en consistent lever

## Problem Statement
### Current Situation
- Over-commitment in sprints
- No velocity tracking
- Unclear sprint goals

### Desired Outcome
- Data-driven sprint planning
- Velocity-based capacity
- Clear sprint commitment

## Acceptance Criteria

### AC-1: Sprint Creation
**Gegeven** een product backlog  
**Wanneer** ik een nieuwe sprint start  
**Dan** kan ik:
- Sprint naam en duur (1-4 weken) definiëren
- Target velocity instellen
- Items selecteren tot capacity

### AC-2: Capacity Calculation
**Gegeven** historische velocity data  
**Wanneer** ik een sprint plan  
**Dan** wordt suggested capacity berekend:
- Gemiddelde van laatste 3 sprints
- Waarschuwing bij over-commitment
- Buffer voor onvoorziene werk (20%)

### AC-3: Sprint Commitment
**Gegeven** geselecteerde items voor sprint  
**Wanneer** ik de sprint commit  
**Dan** worden items gemarkeerd met sprint ID  
**En** wordt sprint document gegenereerd  
**En** start sprint automatisch op start datum

## Technical Notes

### Sprint Document Structure
```yaml
---
id: SPRINT-001
name: "Sprint 1: Foundation"
start_date: 2025-09-08
end_date: 2025-09-22
capacity: 21
committed: 20
status: active
items:
  - US-001
  - US-002
  - US-003
---
```

### Affected Components
- `/docs/sprints/` directory
- `/scripts/plan-sprint.sh`
- Velocity calculation logic

## Test Scenarios

### Test Case 1: First Sprint
1. Plan sprint zonder history
2. Use default capacity (20 points)
3. Select items
4. Generate sprint doc

### Test Case 2: Velocity-Based Planning
1. Complete 3 sprints
2. Calculate average velocity
3. Plan new sprint
4. Verify capacity suggestion

## Definition of Done
- [ ] Sprint planning script
- [ ] Velocity calculation
- [ ] Sprint document template
- [ ] Capacity warnings
- [ ] Sprint start/end automation
- [ ] Sprint burndown setup