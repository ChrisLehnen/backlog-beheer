---
id: REQ-003
type: functional
title: Dashboard & Reporting
priority: HOOG
status: todo
owner: business-analyst
created: 2025-09-08
updated: 2025-09-08
epics:
  - EPIC-002
applies_to: backlog-system@v1.0
---

# REQ-003: Dashboard & Reporting

## Requirement Statement
Het systeem moet een centraal dashboard bieden met real-time status overzicht en voortgangsrapportages voor alle actieve projecten.

## Business Need
Als solo programmeur wil ik in één oogopslag de status van al mijn projecten kunnen zien om gefocust te blijven en prioriteiten te kunnen bijstellen.

## Acceptance Criteria

### AC-1: Project Overview Dashboard
- **Gegeven** meerdere actieve projecten
- **Wanneer** ik het dashboard open
- **Dan** zie ik per project: status, voortgang percentage, open issues
- **En** zie ik de top 5 prioriteit items across alle projecten
- **En** wordt data automatisch gegenereerd uit backlog items

### AC-2: Sprint Dashboard
- **Gegeven** een actieve sprint
- **Wanneer** ik de sprint status bekijk
- **Dan** zie ik: burndown chart, velocity, remaining days
- **En** zie ik items per status (todo, in progress, done)
- **En** zie ik blockers en risks highlighted

### AC-3: Progress Reports
- **Gegeven** voltooide werk
- **Wanneer** ik een voortgangsrapport genereer
- **Dan** krijg ik weekly/monthly progress summary
- **En** zie ik completed vs planned story points
- **En** krijg ik trend analysis en forecasting

## Domain Rules
- Dashboard update frequentie: real-time bij file changes
- Metrics gebaseerd op story points en status
- Visualisatie via Markdown tabellen en ASCII charts
- Focus op actionable insights

## Technical Constraints
- Pure Markdown output voor compatibility
- Automatische generatie via scripts
- No JavaScript required voor viewing
- Git hooks voor auto-update

## Dependencies
- REQ-001: Backlog Item Management
- REQ-002: Sprint Planning
- Markdown viewer met tabel support