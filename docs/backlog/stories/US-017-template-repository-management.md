---
id: US-017
titel: Template Repository Structure and Management
epic: EPIC-006
requirement: REQ-014
status: backlog
prioriteit: KRITIEK
story_points: 3
sprint: 
aangemaakt: 08-09-2025
bijgewerkt: 08-09-2025
assigned_to: backend_developer
dependencies: [US-016]
---

# US-017: Template Repository Structure and Management

## User Story
**Als** developer  
**wil ik** een gestructureerde template repository met versioning  
**zodat** ik consistent templates kan gebruiken en updaten across projecten

## Context
Templates zijn het hart van de bootstrapper. Ze moeten goed georganiseerd, versioned, en makkelijk te updaten zijn. Dit systeem moet zowel built-in templates als custom templates ondersteunen.

## Acceptance Criteria

### Template Organization
- [ ] Hiërarchische structuur: base → type → framework → domain
- [ ] Elk template heeft metadata.yml met versie en dependencies
- [ ] Template inheritance ondersteund (extends: base)
- [ ] Variables via {{placeholders}} in alle file types
- [ ] Conditional sections voor optionele features

### Version Management
- [ ] Semantic versioning voor templates (major.minor.patch)
- [ ] Compatibility matrix tussen CLI en template versies
- [ ] Automatic migration scripts tussen versies
- [ ] Rollback mogelijk naar vorige template versie
- [ ] Change log per template maintained

### Template Registry
- [ ] Local cache van templates in ~/.backlog/templates/
- [ ] Remote repository sync mechanisme
- [ ] Template discovery via `backlog templates list`
- [ ] Custom template repositories toevoegen mogelijk
- [ ] Offline mode met cached templates

### Template Validation
- [ ] Schema validation voor template structure
- [ ] Variable validation (required vs optional)
- [ ] Dependency checking tussen templates
- [ ] Lint rules voor template quality
- [ ] Test suite voor template rendering

## Technical Design

### Template Structure
```
templates/
├── metadata.yml                # Global template metadata
├── base/                       # Base template all others extend
│   ├── metadata.yml
│   ├── structure.yml
│   └── files/
│       ├── .gitignore.tpl
│       ├── README.md.tpl
│       └── docs/
│           └── backlog/
│               ├── requirements/
│               ├── epics/
│               └── stories/
├── types/
│   ├── web-app/
│   │   ├── metadata.yml
│   │   ├── extends: base
│   │   └── files/
│   ├── api/
│   │   ├── metadata.yml
│   │   ├── extends: base
│   │   └── files/
│   └── library/
│       ├── metadata.yml
│       ├── extends: base
│       └── files/
├── frameworks/
│   ├── react/
│   │   ├── metadata.yml
│   │   ├── requires: type=web-app
│   │   └── files/
│   └── express/
│       ├── metadata.yml
│       ├── requires: type=api
│       └── files/
└── domains/
    └── justice/
        ├── metadata.yml
        ├── compatible: all
        └── files/
```

### Metadata Schema
```yaml
# metadata.yml
version: 1.2.0
name: Web Application Template
description: Complete setup for web applications
author: Platform Team
created: 2025-09-08
updated: 2025-09-08

extends: base
requires:
  - cli_version: ">=1.0.0"
  - node_version: ">=16.0.0"

variables:
  project_name:
    type: string
    required: true
    default: "${FOLDER_NAME}"
    description: Project name
  framework:
    type: enum
    options: [react, vue, angular]
    default: react
  include_tests:
    type: boolean
    default: true

files:
  - source: package.json.tpl
    destination: package.json
    condition: "{{if eq .framework 'react'}}"
  - source: "**/*.md"
    destination: docs/
    
hooks:
  pre_init:
    - validate_environment
  post_init:
    - npm_install
    - git_init
```

### Template Engine Implementation
```typescript
// src/core/template-engine.ts
import * as Handlebars from 'handlebars';
import * as yaml from 'js-yaml';
import * as fs from 'fs-extra';
import * as path from 'path';

export class TemplateEngine {
  private templates: Map<string, Template>;
  private handlebars: typeof Handlebars;
  
  constructor(private templateDir: string) {
    this.templates = new Map();
    this.handlebars = Handlebars.create();
    this.registerHelpers();
  }
  
  private registerHelpers(): void {
    // Conditional helper
    this.handlebars.registerHelper('if_eq', (a, b, options) => {
      return a === b ? options.fn(this) : options.inverse(this);
    });
    
    // Date helper
    this.handlebars.registerHelper('date', () => {
      return new Date().toISOString().split('T')[0];
    });
    
    // Case helpers
    this.handlebars.registerHelper('uppercase', (str) => str.toUpperCase());
    this.handlebars.registerHelper('lowercase', (str) => str.toLowerCase());
    this.handlebars.registerHelper('kebabcase', (str) => {
      return str.replace(/([a-z])([A-Z])/g, '$1-$2').toLowerCase();
    });
  }
  
  async loadTemplate(name: string): Promise<Template> {
    if (this.templates.has(name)) {
      return this.templates.get(name)!;
    }
    
    const templatePath = path.join(this.templateDir, name);
    const metadataPath = path.join(templatePath, 'metadata.yml');
    
    if (!await fs.pathExists(metadataPath)) {
      throw new Error(`Template ${name} not found`);
    }
    
    const metadata = yaml.load(await fs.readFile(metadataPath, 'utf8'));
    const template = new Template(name, templatePath, metadata);
    
    // Load parent template if extends
    if (metadata.extends) {
      template.parent = await this.loadTemplate(metadata.extends);
    }
    
    this.templates.set(name, template);
    return template;
  }
  
  async renderTemplate(
    templateName: string, 
    variables: Record<string, any>,
    outputDir: string
  ): Promise<void> {
    const template = await this.loadTemplate(templateName);
    const context = this.buildContext(template, variables);
    
    // Render all files
    for (const file of template.getFiles()) {
      if (this.evaluateCondition(file.condition, context)) {
        await this.renderFile(file, context, outputDir);
      }
    }
    
    // Run post-init hooks
    await this.runHooks(template.metadata.hooks?.post_init, context);
  }
  
  private async renderFile(
    file: TemplateFile,
    context: any,
    outputDir: string
  ): Promise<void> {
    const source = await fs.readFile(file.sourcePath, 'utf8');
    const rendered = this.handlebars.compile(source)(context);
    
    const destPath = path.join(outputDir, file.destination);
    await fs.ensureDir(path.dirname(destPath));
    await fs.writeFile(destPath, rendered);
  }
}

class Template {
  constructor(
    public name: string,
    public path: string,
    public metadata: any,
    public parent?: Template
  ) {}
  
  getFiles(): TemplateFile[] {
    const files = this.metadata.files || [];
    if (this.parent) {
      return [...this.parent.getFiles(), ...files];
    }
    return files;
  }
}
```

### Template Update System
```typescript
// src/core/template-updater.ts
export class TemplateUpdater {
  async checkForUpdates(): Promise<UpdateInfo[]> {
    const updates = [];
    const installed = await this.getInstalledTemplates();
    
    for (const template of installed) {
      const latest = await this.getLatestVersion(template.name);
      if (semver.gt(latest.version, template.version)) {
        updates.push({
          name: template.name,
          current: template.version,
          latest: latest.version,
          changelog: await this.getChangelog(template.name, template.version, latest.version)
        });
      }
    }
    
    return updates;
  }
  
  async updateTemplate(name: string, version?: string): Promise<void> {
    const targetVersion = version || 'latest';
    
    // Download new template
    const templateData = await this.downloadTemplate(name, targetVersion);
    
    // Backup current template
    await this.backupTemplate(name);
    
    try {
      // Install new template
      await this.installTemplate(templateData);
      
      // Run migration if needed
      await this.runMigration(name, templateData.version);
      
    } catch (error) {
      // Rollback on failure
      await this.restoreBackup(name);
      throw error;
    }
  }
}
```

## Test Scenarios

### Template Loading Tests
```javascript
describe('Template Engine', () => {
  test('loads base template', async () => {
    const template = await engine.loadTemplate('base');
    expect(template.name).toBe('base');
    expect(template.metadata.version).toBeDefined();
  });
  
  test('handles template inheritance', async () => {
    const template = await engine.loadTemplate('types/web-app');
    expect(template.parent?.name).toBe('base');
    const files = template.getFiles();
    expect(files).toContainEqual(expect.objectContaining({
      destination: 'README.md'
    }));
  });
  
  test('validates required variables', async () => {
    const template = await engine.loadTemplate('types/api');
    expect(() => engine.renderTemplate('types/api', {}))
      .toThrow(/Required variable: project_name/);
  });
});
```

### Version Management Tests
```javascript
describe('Template Updates', () => {
  test('detects available updates', async () => {
    const updates = await updater.checkForUpdates();
    expect(updates).toHaveLength(2);
    expect(updates[0]).toHaveProperty('changelog');
  });
  
  test('updates template with rollback on failure', async () => {
    mockDownload.mockRejectedValue(new Error('Network error'));
    await expect(updater.updateTemplate('base'))
      .rejects.toThrow('Network error');
    // Verify rollback happened
    const template = await engine.loadTemplate('base');
    expect(template.metadata.version).toBe('1.0.0'); // Original version
  });
});
```

## Tasks

### Development Tasks
- [ ] Design template directory structure - 2 uur
- [ ] Implement metadata schema and validation - 2 uur
- [ ] Build template inheritance system - 3 uur
- [ ] Create template rendering engine - 3 uur
- [ ] Add version management - 2 uur
- [ ] Implement update mechanism - 2 uur

### Testing Tasks
- [ ] Unit tests voor template engine - 2 uur
- [ ] Test template inheritance - 1 uur
- [ ] Test version updates - 1 uur
- [ ] Integration tests - 1 uur

## Definition of Done
- [ ] Template structure fully defined
- [ ] Metadata schema documented
- [ ] Template engine renders correctly
- [ ] Version management working
- [ ] Update mechanism tested
- [ ] All templates validated
- [ ] Documentation complete

## Notes
- Consider using Liquid or Nunjucks als alternative template engines
- Template marketplace voor community templates in v2.0
- Support voor binary files (images, etc.)
- Template composition (combine multiple templates)