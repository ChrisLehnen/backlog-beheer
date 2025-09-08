---
id: REQ-008
titel: JSON Data Generatie voor Dashboard
type: functional
prioriteit: HOOG
status: backlog
aangemaakt: 08-09-2025
bijgewerkt: 08-09-2025
epic: EPIC-002
stories: []
bron: Analyse definitie-app data.json
---

# REQ-008: JSON Data Generatie voor Dashboard

## Beschrijving
Het systeem moet automatisch een data.json bestand genereren uit alle markdown backlog items, waarbij YAML frontmatter wordt geëxtraheerd en getransformeerd naar een gestructureerd JSON formaat voor dashboard consumptie.

## Rationale
Een centraal data.json bestand maakt het mogelijk om:
- Snel alle backlog data te laden zonder individuele file reads
- Consistente data structuur voor visualisatie
- Caching en performance optimalisatie
- Eenvoudige integratie met dashboard en andere tools

## Acceptatiecriteria

### Data Extractie
- [ ] Scan alle markdown files in requirements/, epics/, stories/ directories
- [ ] Parse YAML frontmatter uit elk bestand
- [ ] Extraheer body content eerste paragraaf als summary
- [ ] Valideer verplichte velden (id, titel, type, status)
- [ ] Handle missing of incorrecte metadata gracefully

### JSON Structuur
```json
{
  "generated": "2025-09-08T10:00:00Z",
  "statistics": {
    "total_requirements": 6,
    "total_epics": 3,
    "total_stories": 13,
    "total_points": 41
  },
  "epics": [
    {
      "id": "EPIC-001",
      "title": "Core Backlog Functionality",
      "status": "in_progress",
      "requirements": ["REQ-001", "REQ-002"],
      "stories": ["US-001", "US-002", "US-003"],
      "story_points": 21,
      "completion": 35,
      "path": "docs/epics/EPIC-001.md"
    }
  ],
  "requirements": [
    {
      "id": "REQ-001",
      "title": "Backlog Management",
      "type": "functional",
      "priority": "HOOG",
      "status": "in_progress",
      "epics": ["EPIC-001"],
      "stories": ["US-001", "US-002"],
      "path": "docs/requirements/REQ-001.md",
      "tags": ["core", "management"],
      "summary": "Eerste 200 karakters van beschrijving..."
    }
  ],
  "stories": [
    {
      "id": "US-001",
      "title": "Create Backlog Item",
      "epic": "EPIC-001",
      "requirement": "REQ-001",
      "status": "done",
      "story_points": 5,
      "sprint": "Sprint-1",
      "assigned_to": "developer",
      "path": "docs/stories/US-001.md"
    }
  ]
}
```

### Relatie Mapping
- [ ] Bidirectionele relaties tussen epics ↔ requirements
- [ ] Epic → stories mapping
- [ ] Requirement → stories mapping
- [ ] Dependency tracking tussen items
- [ ] Detectie van broken links

### Performance
- [ ] Generatie tijd < 5 seconden voor 500+ items
- [ ] Incrementele updates bij file wijzigingen
- [ ] File size optimalisatie (minified JSON optie)
- [ ] Parallelle file processing waar mogelijk

## Implementatie Notities

### Generator Script
```python
# scripts/generate_dashboard_data.py
- Gebruik pathlib voor cross-platform paths
- YAML safe_load voor security
- JSON schrijven met indent voor leesbaarheid
- Logging van warnings en errors
- Watch mode voor auto-regeneratie
```

### Validatie
- Schema validatie met JSON Schema
- Controle op duplicate IDs
- Controle op circulaire dependencies
- Missing file detectie

## Dependencies
- Python 3.8+ met PyYAML
- REQ-007: HTML Dashboard (consumer)

## Referenties
- Definitie-app data.json structuur
- JSON Schema specificatie