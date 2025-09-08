---
id: REQ-002
type: functional
title: Sprint Planning & Velocity Tracking
priority: HOOG
status: todo
owner: business-analyst
created: 2025-09-08
updated: 2025-09-08
epics:
  - EPIC-001
applies_to: backlog-system@v1.0
---

# REQ-002: Sprint Planning & Velocity Tracking

## Requirement Statement
Het systeem moet sprint planning ondersteunen met velocity tracking en burndown visualisatie voor effectieve iteratieve ontwikkeling.

## Business Need
Als solo developer wil ik mijn werk in sprints kunnen plannen om een consistent ontwikkelritme te behouden en realistische deadlines te kunnen stellen.

## Acceptance Criteria

### AC-1: Sprint Creation
- **Gegeven** een gevulde product backlog
- **Wanneer** ik een nieuwe sprint start
- **Dan** kan ik een sprint naam en duur (1-4 weken) definiëren
- **En** kan ik items uit de backlog naar de sprint verplaatsen
- **En** wordt de totale story points automatisch berekend

### AC-2: Velocity Tracking
- **Gegeven** voltooide sprints
- **Wanneer** ik mijn velocity wil bekijken
- **Dan** zie ik het gemiddelde aantal story points per sprint
- **En** zie ik de trend over de laatste 3 sprints
- **En** krijg ik een voorspelling voor toekomstige capaciteit

### AC-3: Sprint Progress
- **Gegeven** een actieve sprint
- **Wanneer** ik de voortgang wil monitoren
- **Dan** zie ik hoeveel story points zijn voltooid
- **En** zie ik de remaining work
- **En** krijg ik een waarschuwing als de sprint at-risk is

## Domain Rules
- Sprints hebben een vaste tijdsduur (timeboxed)
- Eenmaal gestart wordt sprint scope niet meer aangepast
- Velocity wordt berekend over voltooide story points
- Minimum 3 sprints nodig voor betrouwbare velocity

## Technical Constraints
- Sprint data in YAML/Markdown format
- Automatische berekeningen via scripts
- Visualisatie via Markdown tabellen
- Git tags voor sprint milestones

## Dependencies
- REQ-001: Backlog Item Management
- Story point estimates op alle items
- Consistent sprint duratie