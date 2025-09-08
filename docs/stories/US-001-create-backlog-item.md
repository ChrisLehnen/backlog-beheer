---
id: US-001
epic: EPIC-001
title: Create New Backlog Item with Auto-ID Generation
status: todo
priority: KRITIEK
type: feature
story_points: 3
sprint: backlog
owner: developer
created: 2025-09-08
updated: 2025-09-08
assigned_to: solo-dev
dependencies: []
requirements:
  - REQ-001
---

# US-001: Create New Backlog Item with Auto-ID Generation

## User Story
**Als** solo programmeur  
**wil ik** snel nieuwe backlog items kunnen aanmaken met automatische ID generatie  
**zodat** ik ideeën direct kan vastleggen zonder administratieve overhead

## Problem Statement
### Current Situation
- Handmatig ID's bedenken kost tijd en leidt tot duplicates
- Geen consistente structuur voor verschillende item types
- Verlies van ideeën door gebrek aan snelle capture

### Desired Outcome
- Instant item creation met < 30 seconden setup
- Automatische unieke ID generatie (EPIC-XXX, US-XXX, TASK-XXX)
- Consistent YAML frontmatter structuur

## Acceptance Criteria

### AC-1: Command Line Creation
**Gegeven** een command line interface  
**Wanneer** ik `create-item --type story --title "My Story"` uitvoer  
**Dan** wordt een nieuw bestand aangemaakt met:
- Automatisch gegenereerd ID (US-XXX)
- Volledig ingevulde YAML frontmatter
- Template body sectie

### AC-2: ID Generation Logic
**Gegeven** bestaande items in de directory  
**Wanneer** een nieuw item wordt aangemaakt  
**Dan** wordt het volgende beschikbare ID nummer gebruikt  
**En** wordt dit ID uniek gegarandeerd

### AC-3: Template Selection
**Gegeven** verschillende item types (epic, story, task)  
**Wanneer** ik een type selecteer  
**Dan** wordt het juiste template gebruikt  
**En** zijn alle verplichte velden aanwezig

## Technical Notes

### Implementation Details
```bash
#!/bin/bash
# create-item.sh
TYPE=$1
TITLE=$2
# Find next available ID
# Generate file from template
# Update INDEX.md
```

### Affected Components
- `/scripts/create-item.sh` - Main creation script
- `/templates/` - Template directory
- `/docs/stories/INDEX.md` - Auto-update required

## Test Scenarios

### Test Case 1: Basic Creation
1. Run create command
2. Verify file exists
3. Check ID uniqueness
4. Validate YAML structure

### Test Case 2: Duplicate Prevention
1. Create item with ID US-001
2. Try to create another US-001
3. Should auto-increment to US-002

## Definition of Done
- [ ] Creation script implemented
- [ ] Templates for all item types
- [ ] ID generation tested
- [ ] INDEX auto-update working
- [ ] Documentation updated
- [ ] Error handling for edge cases