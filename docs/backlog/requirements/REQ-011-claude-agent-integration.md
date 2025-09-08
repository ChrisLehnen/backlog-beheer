---
id: REQ-011
titel: Claude Code Agent Backlog Integration
type: functional
prioriteit: HOOG
status: backlog
aangemaakt: 08-09-2025
bijgewerkt: 08-09-2025
epic: EPIC-005
stories: []
bron: Claude Code agent architecture analysis
---

# REQ-011: Claude Code Agent Backlog Integration

## Beschrijving
Uitbreiding van Claude Code's ingebouwde agents (zoals doc-standards-guardian, business-analyst-justice, quality-assurance-tester) met backlog-aware functionaliteiten zodat ze automatisch kunnen werken met het backlog beheer systeem.

## Rationale
De huidige Claude Code agents zijn krachtig maar niet bewust van backlog structuren. Door ze backlog-aware te maken kunnen ze:
- Automatisch requirements genereren tijdens development
- User stories updaten na code completie
- Test coverage koppelen aan acceptance criteria
- Documentation synchroniseren met backlog items
- Compliance tracking automatiseren

## Acceptatiecriteria

### Agent Verbeteringen

#### 1. doc-standards-guardian
**Current**: Valideert en update project documentatie
**Enhanced**: 
- [ ] Detecteert backlog item referenties in code (// REQ-001, # US-023)
- [ ] Synchroniseert README met backlog status
- [ ] Genereert CHANGELOG entries gekoppeld aan user stories
- [ ] Valideert dat completed stories gedocumenteerd zijn
- [ ] Update automatisch traceability matrix bij doc changes
- [ ] Controleert of Definition of Done criteria in docs zijn vastgelegd

#### 2. business-analyst-justice
**Current**: Vertaalt business requirements naar development artifacts
**Enhanced**:
- [ ] Genereert automatisch requirements in backlog formaat
- [ ] Cre\u00ebert epics met proper YAML frontmatter
- [ ] Splitst requirements op in user stories met story points
- [ ] Voegt compliance metadata toe (ASTRA, NORA, GEMMA)
- [ ] Linkt requirements aan bestaande epics
- [ ] Genereert acceptance criteria in BDD formaat

#### 3. quality-assurance-tester
**Current**: Cre\u00ebert en maintaint tests
**Enhanced**:
- [ ] Koppelt test files aan user stories via metadata
- [ ] Rapporteert test coverage per requirement
- [ ] Genereert test reports per sprint
- [ ] Valideert dat acceptance criteria testbaar zijn
- [ ] Update story status based op test results
- [ ] Cre\u00ebert bug items bij test failures

#### 4. workflow-router
**Current**: Routeert taken naar juiste workflow
**Enhanced**:
- [ ] Identificeert werk als backlog item (requirement/epic/story/bug)
- [ ] Selecteert workflow based op item type en status
- [ ] Update item status tijdens workflow execution
- [ ] Escaleert blocked items naar epic owner
- [ ] Triggert automations bij status changes

#### 5. developer-implementer
**Current**: Implementeert architectuur designs
**Enhanced**:
- [ ] Leest acceptance criteria uit user story
- [ ] Voegt story ID toe aan commit messages
- [ ] Update story status naar "in_progress" bij start
- [ ] Markeert story als "review" bij PR creation
- [ ] Documenteert implementation decisions in story
- [ ] Berekent actual hours voor story

#### 6. code-reviewer-comprehensive
**Current**: Voert code reviews uit
**Enhanced**:
- [ ] Controleert of code voldoet aan story acceptance criteria
- [ ] Valideert Definition of Done checklist
- [ ] Linkt review comments aan story/requirement
- [ ] Blokkeert merge als story niet compleet is
- [ ] Genereert quality metrics per epic

### Integration Patterns

#### Metadata in Code
```python
# @backlog: US-014
# @requirement: REQ-007
# @epic: EPIC-004
class BacklogDashboard:
    """
    Implementation for story US-014: HTML Dashboard
    Fulfills requirement REQ-007
    """
    pass
```

#### Automatic Story Updates
```python
# Agent detecteert:
git commit -m "feat: implement dashboard search - US-014"

# Agent update automatisch:
# - US-014 status -> "review"
# - US-014 actual_hours += git time tracking
# - EPIC-004 completion_percentage recalculated
```

#### Test-Story Linking
```python
# test_dashboard.py
"""
@story: US-014
@acceptance_criteria: 1,2,3
"""
def test_dashboard_search():
    # Test linked to acceptance criterium 1
    pass
```

### Agent Commands

#### Nieuwe Claude Code Commands
```bash
# Backlog aware commands
/backlog status          # Toon current sprint items
/backlog create story    # Cre\u00eber nieuwe user story
/backlog update US-014   # Update story status/metadata
/backlog link REQ-007    # Link current werk aan requirement
/backlog report sprint   # Genereer sprint report
```

### Agent Prompts Updates

#### Example: doc-standards-guardian
```markdown
ADDITIONAL CAPABILITIES:

When reviewing documentation:
1. Check for backlog item references (REQ-*, EPIC-*, US-*)
2. Validate these references exist in docs/requirements/, docs/epics/, docs/stories/
3. Ensure completed stories are reflected in CHANGELOG
4. Update traceability matrix when detecting new relationships
5. Add missing backlog metadata to documentation
```

## Implementation Strategy

### Phase 1: Detection & Parsing
- Agents kunnen backlog items detecteren in code/docs
- Agents kunnen YAML frontmatter lezen en interpreteren
- Agents begrijpen backlog relationships

### Phase 2: Validation & Reporting  
- Agents valideren backlog consistentie
- Agents genereren backlog-aware reports
- Agents checken compliance met DoD

### Phase 3: Automation & Updates
- Agents updaten backlog items automatisch
- Agents cre\u00ebren nieuwe items waar nodig
- Agents synchroniseren tussen code en backlog

### Phase 4: Intelligence & Suggestions
- Agents suggereren story splits
- Agents detecteren scope creep
- Agents voorspellen velocity impact

## Technical Requirements

### Agent Configuration
```yaml
# .claude-agents.yml
backlog:
  enabled: true
  base_path: ./docs
  item_types:
    requirements: docs/requirements/
    epics: docs/epics/
    stories: docs/stories/
    bugs: docs/bugs/
  
  auto_update:
    status_on_commit: true
    link_tests: true
    update_hours: true
    
  validations:
    require_story_reference: true
    check_acceptance_criteria: true
    enforce_dod: true
```

### Agent Capabilities Matrix

| Agent | Read Backlog | Update Items | Create Items | Validate | Report |
|-------|-------------|--------------|--------------|----------|--------|
| doc-standards-guardian | ✓ | ✓ | - | ✓ | ✓ |
| business-analyst-justice | ✓ | ✓ | ✓ | ✓ | ✓ |
| quality-assurance-tester | ✓ | ✓ | ✓ | ✓ | ✓ |
| workflow-router | ✓ | ✓ | - | ✓ | - |
| developer-implementer | ✓ | ✓ | - | - | - |
| code-reviewer | ✓ | ✓ | - | ✓ | ✓ |

## Dependencies
- REQ-007: HTML Dashboard (voor visualisatie)
- REQ-008: JSON Data Generation (voor agent data access)
- REQ-009: Enhanced Metadata (voor rich agent operations)
- REQ-010: Validation Suite (voor agent validation logic)

## Success Metrics
- 90% van code commits hebben story reference
- 100% van completed stories hebben updated documentation
- Test coverage automatisch getracked per requirement
- 50% reductie in manual backlog updates
- Zero orphaned code zonder backlog reference

## Risks & Mitigations
- **Risk**: Agents maken incorrect updates
  **Mitigation**: Dry-run mode, audit log, rollback capability
- **Risk**: Performance impact door backlog checks
  **Mitigation**: Caching, async operations, optional enablement
- **Risk**: Conflicting updates tussen agents
  **Mitigation**: Locking mechanism, transaction log