---
id: US-006
epic: EPIC-002
title: Generate Project Overview Dashboard
status: todo
priority: KRITIEK
type: feature
story_points: 5
sprint: backlog
owner: developer
created: 2025-09-08
updated: 2025-09-08
assigned_to: solo-dev
dependencies:
  - US-003
requirements:
  - REQ-003
---

# US-006: Generate Project Overview Dashboard

## User Story
**Als** solo programmeur  
**wil ik** een automatisch gegenereerd dashboard met project overzicht  
**zodat** ik in één oogopslag de status van al mijn projecten zie

## Problem Statement
### Current Situation
- Informatie verspreid over meerdere bestanden
- Geen quick overview mogelijk
- Manual updates required

### Desired Outcome
- Single dashboard view
- Auto-generated from source files
- Real-time status updates

## Acceptance Criteria

### AC-1: Dashboard Generation
**Gegeven** backlog items in markdown files  
**Wanneer** dashboard generatie wordt getriggerd  
**Dan** wordt een dashboard.md gegenereerd met:
- Project statistics
- Status distribution
- Priority matrix
- Recent updates

### AC-2: Multi-Project Support
**Gegeven** meerdere project directories  
**Wanneer** het dashboard genereert  
**Dan** toont het:
- Per project summary
- Cross-project priorities
- Combined metrics

### AC-3: Auto-Update
**Gegeven** wijzigingen in backlog items  
**Wanneer** een bestand wordt opgeslagen  
**Dan** wordt dashboard automatisch bijgewerkt  
**En** binnen 5 seconden zichtbaar

## Technical Notes

### Dashboard Layout
```markdown
# Backlog Dashboard
Generated: 2025-09-08 14:30

## Summary
- Total Items: 45
- In Progress: 5
- Completed: 20
- Blocked: 2

## Priority Matrix
| Priority | Todo | In Progress | Done |
|----------|------|-------------|------|
| KRITIEK  | 3    | 2           | 5    |
| HOOG     | 8    | 3           | 10   |

## Recent Updates
- US-001: Status changed to done
- US-005: New item created
```

### Affected Components
- `/scripts/generate-dashboard.sh`
- `/docs/DASHBOARD.md`
- Git hooks for auto-generation

## Test Scenarios

### Test Case 1: Initial Generation
1. Create 10 backlog items
2. Run dashboard generation
3. Verify all sections present
4. Check calculations correct

### Test Case 2: Auto-Update
1. Modify item status
2. Save file
3. Check dashboard updates
4. Verify < 5 second update

## Definition of Done
- [ ] Dashboard generation script
- [ ] Markdown template
- [ ] Statistics calculation
- [ ] Priority matrix logic
- [ ] Git hook for auto-update
- [ ] Multi-project support