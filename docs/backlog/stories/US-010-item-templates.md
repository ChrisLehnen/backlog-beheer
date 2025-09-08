---
id: US-010
epic: EPIC-003
title: Create and Manage Item Templates
status: todo
priority: HOOG
type: feature
story_points: 3
sprint: backlog
owner: developer
created: 2025-09-08
updated: 2025-09-08
assigned_to: solo-dev
dependencies: []
requirements:
  - REQ-004
---

# US-010: Create and Manage Item Templates

## User Story
**Als** solo programmeur  
**wil ik** herbruikbare templates voor verschillende item types  
**zodat** ik consistent en snel nieuwe items kan aanmaken

## Problem Statement
### Current Situation
- Copy-paste van oude items
- Inconsistente structuur
- Missing required fields

### Desired Outcome
- Standaard templates per type
- Custom project templates
- Variable substitution

## Acceptance Criteria

### AC-1: Default Templates
**Gegeven** standaard item types  
**Wanneer** ik een nieuw item aanmaak  
**Dan** zijn templates beschikbaar voor:
- Epic (with business value sections)
- User Story (with acceptance criteria)
- Task (simple format)
- Bug (with reproduction steps)

### AC-2: Template Variables
**Gegeven** een template met variables  
**Wanneer** item wordt aangemaakt  
**Dan** worden variables vervangen:
- {{ID}} → generated ID
- {{DATE}} → current date
- {{SPRINT}} → active sprint
- {{USER}} → current user

### AC-3: Custom Templates
**Gegeven** project-specifieke needs  
**Wanneer** ik custom template maak  
**Dan** wordt deze opgeslagen in templates/custom/  
**En** verschijnt in template selector  
**En** kan worden gedeeld tussen projecten

## Technical Notes

### Template Structure
```yaml
# templates/user-story.template.md
---
id: {{ID}}
epic: {{EPIC}}
title: {{TITLE}}
status: todo
priority: {{PRIORITY}}
type: feature
story_points: {{POINTS}}
sprint: backlog
owner: developer
created: {{DATE}}
updated: {{DATE}}
assigned_to: {{USER}}
dependencies: []
requirements: []
---

# {{ID}}: {{TITLE}}

## User Story
**Als** ...
**wil ik** ...
**zodat** ...
```

### Affected Components
- `/templates/` directory structure
- `/scripts/apply-template.sh`
- Template variable engine

## Test Scenarios

### Test Case 1: Template Application
1. Select user story template
2. Provide required variables
3. Generate new item
4. Verify all substitutions

### Test Case 2: Custom Template
1. Create custom template
2. Add to library
3. Use in new item
4. Verify availability

## Definition of Done
- [ ] Default templates created
- [ ] Variable substitution engine
- [ ] Template selector
- [ ] Custom template support
- [ ] Template validation
- [ ] Documentation