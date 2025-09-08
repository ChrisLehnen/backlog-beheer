---
id: REQ-009
titel: Enhanced Metadata voor Backlog Items
type: functional
prioriteit: HOOG
status: backlog
aangemaakt: 08-09-2025
bijgewerkt: 08-09-2025
epic: EPIC-003
stories: []
bron: Analyse definitie-app YAML frontmatter
---

# REQ-009: Enhanced Metadata voor Backlog Items

## Beschrijving
Uitbreiding van de huidige metadata structuur met additionele velden voor betere tracking, compliance, en project management capabilities gebaseerd op patterns uit enterprise backlog systemen.

## Rationale
De huidige basis metadata is onvoldoende voor:
- Sprint planning en velocity tracking
- Compliance en audit requirements
- Multi-stakeholder collaboratie
- Dependency management
- Release planning

## Acceptatiecriteria

### Nieuwe Metadata Velden

#### Planning & Tracking
- [ ] `story_points`: Numerieke waarde voor effort inschatting
- [ ] `sprint`: Sprint identifier (Sprint-1, Sprint-2, etc.)
- [ ] `target_release`: Versie nummer voor release planning
- [ ] `completion_percentage`: 0-100 voor progress tracking
- [ ] `actual_hours`: Werkelijke tijd besteed
- [ ] `estimated_hours`: Geschatte tijd

#### Ownership & Collaboration
- [ ] `owner`: Primaire verantwoordelijke
- [ ] `assigned_to`: Huidige assignee
- [ ] `reviewer`: Code/document reviewer
- [ ] `stakeholders`: Array van belanghebbenden
- [ ] `team`: Team assignment

#### Dependencies & Relations
- [ ] `dependencies`: Array van blocking items
- [ ] `blocks`: Array van items die dit item blockt
- [ ] `related_to`: Array van gerelateerde items
- [ ] `parent`: Parent item voor hierarchie
- [ ] `children`: Child items array

#### Compliance & Quality
- [ ] `compliance`: Object met standaard mappings
  - `astra`: Array van ASTRA control IDs
  - `nora`: Array van NORA principe IDs  
  - `wcag`: WCAG compliance level
  - `custom`: Project-specifieke standaarden
- [ ] `test_coverage`: Percentage test dekking
- [ ] `review_status`: draft/in_review/approved/rejected
- [ ] `quality_score`: Automatische kwaliteit score

#### Lifecycle & Versioning
- [ ] `version`: Item versie nummer
- [ ] `last_verified`: Laatste verificatie datum
- [ ] `expires_on`: Vervaldatum voor tijdelijke items
- [ ] `archived`: Boolean voor archief status
- [ ] `deprecated_by`: ID van vervangend item

### Template Structuur
```yaml
---
# Core (verplicht)
id: REQ-001
titel: Requirement titel
type: functional|nonfunctional|domain|technical
prioriteit: KRITIEK|HOOG|GEMIDDELD|LAAG
status: backlog|in_progress|review|done|blocked

# Planning (optioneel)
story_points: 8
sprint: Sprint-3
target_release: v2.1.0
completion_percentage: 75
estimated_hours: 16
actual_hours: 12

# Ownership (optioneel)
owner: product_owner_naam
assigned_to: developer_naam
reviewer: senior_dev_naam
stakeholders: [stakeholder1, stakeholder2]
team: team-alpha

# Relations (optioneel)
epic: EPIC-001
stories: [US-001, US-002]
dependencies: [REQ-002, REQ-003]
blocks: [REQ-004]
related_to: [REQ-005]
parent: REQ-000
children: [REQ-010, REQ-011]

# Compliance (optioneel)
compliance:
  astra: [ASTRA-SEC-001, ASTRA-QUA-002]
  nora: [NORA-BP-15]
  wcag: AA
  custom: [PROJ-STD-001]

# Quality (optioneel)
test_coverage: 85
review_status: approved
quality_score: 8.5

# Lifecycle (optioneel)
aangemaakt: 08-09-2025
bijgewerkt: 08-09-2025
last_verified: 08-09-2025
version: 1.2.0
expires_on: 31-12-2025
archived: false
deprecated_by: null

# Sources (optioneel)
sources:
  - description: Originele requirement
    path: docs/original/req.md
    line: 42
---
```

### Validatie Rules
- [ ] ID format validatie (REQ-NNN, EPIC-NNN, US-NNN)
- [ ] Datum format validatie (DD-MM-YYYY)
- [ ] Enum validatie voor status, type, prioriteit
- [ ] Dependency cycle detectie
- [ ] Required field aanwezigheid

## Implementatie Notities

### Backwards Compatibility
- Nieuwe velden zijn optioneel
- Default waarden voor missing fields
- Migration script voor bestaande items
- Graceful degradation in visualisaties

### Tooling Updates
- Template generator met nieuwe velden
- Validator voor enhanced metadata
- Dashboard update voor nieuwe velden
- Reporting voor nieuwe metrics

## Dependencies
- REQ-010: Metadata Validatie
- REQ-011: Template System Updates

## Referenties
- Definitie-app metadata structuur
- ASTRA/NORA compliance frameworks
- Agile project management best practices