---
id: US-015
titel: Generate JSON Data from Markdown Files
epic: EPIC-004
requirement: REQ-008
status: backlog
prioriteit: HOOG
story_points: 5
sprint: Sprint-5
aangemaakt: 08-09-2025
bijgewerkt: 08-09-2025
assigned_to: backend_developer
dependencies: []
---

# US-015: Generate JSON Data from Markdown Files

## User Story
**Als** dashboard gebruiker  
**wil ik** automatisch gegenereerde JSON data uit markdown files  
**zodat** het dashboard altijd actuele informatie toont zonder handmatige updates

## Context
Het dashboard heeft een gecentraliseerd data.json bestand nodig dat automatisch wordt gegenereerd uit de markdown files met YAML frontmatter. Dit pattern is succesvol toegepast in het definitie-app systeem.

## Acceptance Criteria

### Data Extraction
- [ ] Script scant alle .md files in docs/requirements/, docs/epics/, docs/stories/
- [ ] YAML frontmatter wordt correct geparsed uit elk bestand
- [ ] Body content eerste 200 karakters worden geëxtraheerd als summary
- [ ] Broken links worden gedetecteerd en gemarkeerd
- [ ] Statistics worden automatisch berekend

### JSON Output Structure
- [ ] Timestamp van generatie wordt toegevoegd
- [ ] Hierarchische structuur: epics → requirements → stories
- [ ] Bidirectionele relaties worden opgebouwd
- [ ] Alle metadata velden worden behouden
- [ ] Output is minified voor productie, pretty-printed voor development

### Error Handling
- [ ] Missing required fields worden gerapporteerd
- [ ] Duplicate IDs worden gedetecteerd
- [ ] Invalid YAML wordt graceful afgehandeld
- [ ] Warnings worden gelogd maar stoppen generatie niet
- [ ] Validation report wordt gegenereerd

### Performance
- [ ] Generatie < 3 seconden voor 500+ files
- [ ] Incrementele updates mogelijk (alleen changed files)
- [ ] Parallel file processing waar mogelijk
- [ ] Memory efficient voor grote datasets

## Technical Design

### Python Implementation
```python
#!/usr/bin/env python3
"""
generate_dashboard_data.py
Genereert data.json uit markdown backlog files
"""

import json
import yaml
from pathlib import Path
from datetime import datetime
from typing import Dict, List, Any
import logging
from concurrent.futures import ThreadPoolExecutor

class BacklogDataGenerator:
    def __init__(self, base_path: Path):
        self.base_path = base_path
        self.data = {
            "generated": datetime.now().isoformat(),
            "statistics": {},
            "requirements": [],
            "epics": [],
            "stories": []
        }
        self.errors = []
        self.warnings = []
        
    def scan_files(self) -> List[Path]:
        """Scan voor alle markdown files"""
        patterns = [
            "docs/requirements/*.md",
            "docs/epics/*.md", 
            "docs/stories/*.md"
        ]
        files = []
        for pattern in patterns:
            files.extend(self.base_path.glob(pattern))
        return files
    
    def parse_markdown_file(self, file_path: Path) -> Dict[str, Any]:
        """Parse een markdown file met YAML frontmatter"""
        try:
            content = file_path.read_text(encoding='utf-8')
            
            # Extract YAML frontmatter
            if content.startswith('---'):
                parts = content.split('---', 2)
                if len(parts) >= 3:
                    yaml_content = parts[1]
                    body_content = parts[2].strip()
                    
                    # Parse YAML
                    metadata = yaml.safe_load(yaml_content)
                    
                    # Add computed fields
                    metadata['path'] = str(file_path.relative_to(self.base_path))
                    metadata['summary'] = body_content[:200] + '...' if len(body_content) > 200 else body_content
                    
                    return metadata
                    
        except Exception as e:
            self.errors.append({
                'file': str(file_path),
                'error': str(e)
            })
            
        return None
    
    def validate_item(self, item: Dict[str, Any]) -> bool:
        """Valideer een backlog item"""
        required_fields = ['id', 'titel', 'status']
        
        for field in required_fields:
            if field not in item:
                self.warnings.append({
                    'item': item.get('id', 'unknown'),
                    'warning': f'Missing required field: {field}'
                })
                return False
                
        return True
    
    def build_relationships(self):
        """Bouw bidirectionele relaties"""
        # Create lookup maps
        items_by_id = {}
        for item_type in ['requirements', 'epics', 'stories']:
            for item in self.data[item_type]:
                items_by_id[item['id']] = item
        
        # Verify relationships
        for item_type in ['requirements', 'epics', 'stories']:
            for item in self.data[item_type]:
                # Check epic references
                if 'epic' in item:
                    if item['epic'] not in items_by_id:
                        item['epic'] = f"<!-- BROKEN: {item['epic']} -->"
                        
                # Check story references
                if 'stories' in item:
                    verified_stories = []
                    for story_id in item['stories']:
                        if story_id in items_by_id:
                            verified_stories.append(story_id)
                        else:
                            self.warnings.append({
                                'item': item['id'],
                                'warning': f'Broken link to story: {story_id}'
                            })
                    item['stories'] = verified_stories
    
    def calculate_statistics(self):
        """Bereken statistics"""
        self.data['statistics'] = {
            'total_requirements': len(self.data['requirements']),
            'total_epics': len(self.data['epics']),
            'total_stories': len(self.data['stories']),
            'total_story_points': sum(
                s.get('story_points', 0) for s in self.data['stories']
            ),
            'status_distribution': self._calculate_status_distribution(),
            'priority_distribution': self._calculate_priority_distribution()
        }
    
    def generate(self, output_path: Path, pretty: bool = False):
        """Genereer het data.json bestand"""
        # Scan files
        files = self.scan_files()
        logging.info(f"Found {len(files)} markdown files")
        
        # Parse files in parallel
        with ThreadPoolExecutor(max_workers=10) as executor:
            items = list(executor.map(self.parse_markdown_file, files))
        
        # Categorize items
        for item in items:
            if item and self.validate_item(item):
                if item['id'].startswith('REQ-'):
                    self.data['requirements'].append(item)
                elif item['id'].startswith('EPIC-'):
                    self.data['epics'].append(item)
                elif item['id'].startswith('US-'):
                    self.data['stories'].append(item)
        
        # Build relationships
        self.build_relationships()
        
        # Calculate statistics
        self.calculate_statistics()
        
        # Write output
        output_path.parent.mkdir(parents=True, exist_ok=True)
        with output_path.open('w', encoding='utf-8') as f:
            if pretty:
                json.dump(self.data, f, indent=2, ensure_ascii=False)
            else:
                json.dump(self.data, f, ensure_ascii=False, separators=(',', ':'))
        
        # Generate report
        self._generate_report()
        
        logging.info(f"Generated {output_path}")
        return len(self.errors) == 0

if __name__ == '__main__':
    generator = BacklogDataGenerator(Path.cwd())
    success = generator.generate(
        Path('dashboard/data/data.json'),
        pretty=True
    )
    exit(0 if success else 1)
```

### Watch Mode
```python
# watch_mode.py
import time
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

class BacklogWatcher(FileSystemEventHandler):
    def on_modified(self, event):
        if event.src_path.endswith('.md'):
            print(f"Detected change in {event.src_path}")
            # Regenerate data.json
            generator.generate()
```

## Tasks

### Development Tasks
- [ ] Setup Python project structure - 1 uur
- [ ] Implement file scanner - 1 uur
- [ ] Create YAML parser - 2 uur
- [ ] Build relationship mapper - 2 uur
- [ ] Implement validation logic - 2 uur
- [ ] Create statistics calculator - 1 uur
- [ ] Add error handling - 1 uur
- [ ] Implement watch mode - 2 uur

### Testing Tasks
- [ ] Unit tests voor parsing - 2 uur
- [ ] Test relationship building - 1 uur
- [ ] Test error scenarios - 1 uur
- [ ] Performance testing - 1 uur
- [ ] Integration testing - 1 uur

## Definition of Done
- [ ] Script successfully generates data.json
- [ ] All validation rules implemented
- [ ] Error handling tested
- [ ] Performance targets met
- [ ] Documentation written
- [ ] CI/CD integration configured
- [ ] Code reviewed and approved

## Notes
- Consider caching voor incremental updates
- Mogelijk uitbreiden met GraphQL endpoint later
- Watch mode is optioneel voor eerste versie