---
id: REQ-001
type: functional
title: Backlog Item Management
priority: KRITIEK
status: todo
owner: business-analyst
created: 2025-09-08
updated: 2025-09-08
epics:
  - EPIC-001
applies_to: backlog-system@v1.0
---

# REQ-001: Backlog Item Management

## Requirement Statement
Het systeem moet de mogelijkheid bieden om backlog items (epics, user stories, taken) te creëren, bewerken, prioriteren en archiveren volgens Agile/Scrum principes.

## Business Need
Als solo programmeur heb ik een gestructureerd systeem nodig om al mijn projectwerk te organiseren, prioriteren en tracken zonder de overhead van complexe enterprise tools.

## Acceptance Criteria

### AC-1: Item Creation
- **Gegeven** een nieuwe requirement of idee
- **Wanneer** ik een nieuw backlog item aanmaak
- **Dan** kan ik kiezen tussen epic, user story of taak
- **En** wordt automatisch een uniek ID gegenereerd (EPIC-XXX, US-XXX, TASK-XXX)
- **En** wordt de creatie datum automatisch vastgelegd

### AC-2: Item Prioritization
- **Gegeven** meerdere backlog items
- **Wanneer** ik de prioriteit wil aanpassen
- **Dan** kan ik MoSCoW prioritering toepassen (Must, Should, Could, Won't)
- **En** kan ik story points toekennen (Fibonacci: 1,2,3,5,8,13)
- **En** worden items automatisch gesorteerd op prioriteit

### AC-3: Item Tracking
- **Gegeven** een backlog item
- **Wanneer** ik de voortgang wil tracken
- **Dan** kan ik de status aanpassen (todo, in_progress, done, blocked)
- **En** wordt de laatste update datum automatisch bijgewerkt
- **En** kan ik notities toevoegen voor context

## Domain Rules
- Items volgen de Agile/Scrum methodologie
- Elk item heeft precies één eigenaar
- Story points gebruiken Fibonacci reeks
- Prioritering volgens MoSCoW framework

## Technical Constraints
- Markdown bestanden met YAML frontmatter
- Git voor versiebeheer
- Geen externe dependencies
- Lokaal file-based systeem

## Dependencies
- Markdown editor ondersteuning
- Git repository setup
- YAML parser voor frontmatter