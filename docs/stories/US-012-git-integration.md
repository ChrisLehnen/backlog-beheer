---
id: US-012
epic: EPIC-003
title: Git Version Control Integration
status: todo
priority: HOOG
type: feature
story_points: 4
sprint: backlog
owner: developer
created: 2025-09-08
updated: 2025-09-08
assigned_to: solo-dev
dependencies: []
requirements:
  - REQ-006
---

# US-012: Git Version Control Integration

## User Story
**Als** solo programmeur  
**wil ik** volledige Git integratie voor version control  
**zodat** ik altijd history heb en kan terugkeren naar eerdere versies

## Problem Statement
### Current Situation
- Manual commits required
- No commit message standards
- History difficult to navigate

### Desired Outcome
- Automated meaningful commits
- Structured commit messages
- Easy history browsing

## Acceptance Criteria

### AC-1: Auto-Commit
**Gegeven** wijzigingen aan backlog items  
**Wanneer** ik een item save  
**Dan** wordt automatisch gecommit met:
- Descriptive commit message
- Item ID in message
- Change type (add/update/delete)

### AC-2: Commit Message Format
**Gegeven** een automatische commit  
**Wanneer** message wordt gegenereerd  
**Dan** volgt het format:
```
[TYPE] ID: Description

- Status: old → new
- Priority: old → new
- Other changes...
```

### AC-3: History Browsing
**Gegeven** item met history  
**Wanneer** ik history opvraag  
**Dan** zie ik:
- All changes chronologically
- Who made changes (if multi-user)
- Ability to view/restore old versions

## Technical Notes

### Commit Message Examples
```bash
[UPDATE] US-001: Status changed to done
- Status: in_progress → done
- Updated: 2025-09-08

[ADD] US-013: New story for authentication
- Epic: EPIC-006
- Priority: KRITIEK

[DELETE] TASK-005: Removed duplicate task
```

### Affected Components
- Git hooks (pre-commit, post-commit)
- `/scripts/auto-commit.sh`
- `/scripts/view-history.sh`

## Test Scenarios

### Test Case 1: Auto-Commit Flow
1. Modify item status
2. Save file
3. Verify commit created
4. Check message format

### Test Case 2: History View
1. Make 5 changes to item
2. View history
3. Restore version 3
4. Verify restoration correct

## Definition of Done
- [ ] Auto-commit script
- [ ] Commit message generator
- [ ] Git hooks setup
- [ ] History viewer script
- [ ] Restore functionality
- [ ] Merge conflict handler