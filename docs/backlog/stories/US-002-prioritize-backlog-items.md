---
id: US-002
epic: EPIC-001
title: Prioritize Backlog Items with MoSCoW Method
status: todo
priority: HOOG
type: feature
story_points: 2
sprint: backlog
owner: developer
created: 2025-09-08
updated: 2025-09-08
assigned_to: solo-dev
dependencies:
  - US-001
requirements:
  - REQ-001
---

# US-002: Prioritize Backlog Items with MoSCoW Method

## User Story
**Als** solo programmeur  
**wil ik** backlog items kunnen prioriteren volgens MoSCoW  
**zodat** ik altijd weet wat het belangrijkste is om aan te werken

## Problem Statement
### Current Situation
- Alle taken lijken even belangrijk
- Geen systematische prioritering
- Time wasted op low-value items

### Desired Outcome
- Duidelijke prioriteit rankings
- MoSCoW categorisatie (Must, Should, Could, Won't)
- Automatische sortering op prioriteit

## Acceptance Criteria

### AC-1: Priority Assignment
**Gegeven** een backlog item  
**Wanneer** ik de prioriteit wil instellen  
**Dan** kan ik kiezen uit: KRITIEK, HOOG, GEMIDDELD, LAAG  
**En** wordt MoSCoW mapping toegepast

### AC-2: Automatic Sorting
**Gegeven** meerdere items met verschillende prioriteiten  
**Wanneer** ik de backlog view genereer  
**Dan** worden items gesorteerd op prioriteit (hoogste eerst)  
**En** binnen prioriteit op creation date

### AC-3: Priority Updates
**Gegeven** een bestaand item  
**Wanneer** ik de prioriteit aanpas  
**Dan** wordt de update datum bijgewerkt  
**En** wordt de sortering aangepast

## Technical Notes

### Priority Mapping
```yaml
priority_mapping:
  KRITIEK: Must Have
  HOOG: Should Have
  GEMIDDELD: Could Have
  LAAG: Won't Have (this sprint)
```

### Affected Components
- YAML frontmatter: `priority` field
- `/scripts/sort-backlog.sh`
- Dashboard generation scripts

## Test Scenarios

### Test Case 1: Priority Setting
1. Create item zonder prioriteit
2. Set priority to HOOG
3. Verify in YAML
4. Check dashboard sorting

### Test Case 2: Bulk Prioritization
1. Create 10 items various priorities
2. Run sort script
3. Verify correct order
4. Test edge cases (same priority)

## Definition of Done
- [ ] Priority field in all templates
- [ ] MoSCoW mapping documented
- [ ] Sort script implemented
- [ ] Dashboard reflects priorities
- [ ] Bulk update capability
- [ ] Priority distribution report