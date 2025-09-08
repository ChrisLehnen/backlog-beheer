---
id: US-011
epic: EPIC-003
title: Automatic INDEX File Updates
status: todo
priority: KRITIEK
type: feature
story_points: 3
sprint: backlog
owner: developer
created: 2025-09-08
updated: 2025-09-08
assigned_to: solo-dev
dependencies:
  - US-001
requirements:
  - REQ-004
---

# US-011: Automatic INDEX File Updates

## User Story
**Als** solo programmeur  
**wil ik** dat INDEX bestanden automatisch worden bijgewerkt  
**zodat** ik nooit manual updates hoef te doen en altijd accurate overzichten heb

## Problem Statement
### Current Situation
- Manual INDEX maintenance
- Out-of-sync indexes
- Missing items in overview

### Desired Outcome
- Fully automated INDEX updates
- Always current overview
- Zero manual intervention

## Acceptance Criteria

### AC-1: Auto-Generation
**Gegeven** items in een directory  
**Wanneer** een item wordt toegevoegd/gewijzigd/verwijderd  
**Dan** wordt INDEX.md automatisch bijgewerkt:
- Nieuwe items toegevoegd
- Verwijderde items removed
- Metadata synchronized

### AC-2: INDEX Format
**Gegeven** gegenereerde INDEX  
**Wanneer** ik het bestand open  
**Dan** zie ik:
- Tabel met key fields (ID, Title, Status, Priority)
- Gesorteerd op priority dan ID
- Links naar individuele files
- Statistics summary

### AC-3: Multi-Level INDEX
**Gegeven** nested directory structure  
**Wanneer** INDEX wordt gegenereerd  
**Dan** heeft elke directory eigen INDEX  
**En** root INDEX aggregeert subdirectories  
**En** cross-links zijn correct

## Technical Notes

### INDEX Format Example
```markdown
# User Stories INDEX
Generated: 2025-09-08 15:00

## Summary
- Total Stories: 25
- In Progress: 3
- Completed: 10
- Blocked: 1

## Stories

| ID | Title | Epic | Status | Priority | Points |
|----|-------|------|--------|----------|--------|
| [US-001](US-001.md) | Create Backlog Item | EPIC-001 | done | KRITIEK | 3 |
| [US-002](US-002.md) | Prioritize Items | EPIC-001 | in_progress | HOOG | 2 |
```

### Affected Components
- `/scripts/generate-index.sh`
- Git hooks (post-commit)
- Watch scripts for live update

## Test Scenarios

### Test Case 1: Add Item
1. Create new story file
2. Verify INDEX updates
3. Check correct placement
4. Validate links work

### Test Case 2: Bulk Changes
1. Modify 10 items
2. Run INDEX generation
3. Verify all updates reflected
4. Check performance < 2 seconds

## Definition of Done
- [ ] INDEX generation script
- [ ] Git hook integration
- [ ] File watcher setup
- [ ] Multi-level support
- [ ] Statistics calculation
- [ ] Error recovery