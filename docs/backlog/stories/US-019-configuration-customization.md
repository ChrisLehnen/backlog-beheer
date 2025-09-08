---
id: US-019
titel: Configuration and Customization Options
epic: EPIC-006
requirement: REQ-014
status: backlog
prioriteit: HOOG
story_points: 5
sprint: 
aangemaakt: 08-09-2025
bijgewerkt: 08-09-2025
assigned_to: backend_developer
dependencies: [US-016, US-017, US-018]
---

# US-019: Configuration and Customization Options

## User Story
**Als** developer  
**wil ik** uitgebreide configuratie en customization opties  
**zodat** ik de backlog bootstrapper volledig kan aanpassen aan mijn project needs

## Context
Elk project is uniek. De bootstrapper moet flexibel genoeg zijn om aangepast te worden via configuratie files, environment variables, command-line flags, en interactive prompts. Dit omvat alles van directory structures tot naming conventions.

## Acceptance Criteria

### Configuration Sources (Priority Order)
- [ ] Command-line flags (hoogste prioriteit)
- [ ] Environment variables (BACKLOG_*)
- [ ] Project config file (.backlog-init.yml)
- [ ] User config file (~/.backlog/config.yml)
- [ ] System defaults (laagste prioriteit)

### Configuration Options
- [ ] Directory structure customization
- [ ] Naming conventions (kebab-case, PascalCase, etc.)
- [ ] File formats (Markdown, AsciiDoc, RST)
- [ ] Language preferences (EN, NL, etc.)
- [ ] Workflow state definitions
- [ ] Priority levels en labels
- [ ] Template variable defaults

### Advanced Customization
- [ ] Custom validation rules
- [ ] Hook scripts (pre/post init)
- [ ] Plugin system voor extensions
- [ ] Theme support voor generated docs
- [ ] Custom ID generation patterns
- [ ] Metadata field definitions

### Configuration Management
- [ ] Config validation met schema
- [ ] Config migration tussen versies
- [ ] Export/import configuratie
- [ ] Team shared configurations
- [ ] Configuration inheritance

## Technical Design

### Configuration Schema
```yaml
# .backlog-init.yml
version: 2.0
extends: team-defaults  # Optional inheritance

project:
  name: ${PROJECT_NAME}
  type: web-app
  framework: react
  language: nl
  
structure:
  base_path: docs/backlog
  directories:
    requirements: requirements
    epics: epics
    stories: user-stories
    bugs: bugs
    spikes: research-spikes
  
  naming:
    requirements: "REQ-{number:04d}"  # REQ-0001
    epics: "EPIC-{number:03d}"       # EPIC-001
    stories: "US-{year}-{number:03d}" # US-2025-001
    bugs: "BUG-{number:05d}"         # BUG-00001
    
formats:
  markdown:
    frontmatter: yaml  # or toml, json
    line_width: 80
    heading_style: atx  # or setext
  
  dates:
    format: "YYYY-MM-DD"  # or DD-MM-YYYY
    timezone: "Europe/Amsterdam"

workflow:
  states:
    - id: backlog
      label: Backlog
      color: gray
    - id: refined
      label: Refined
      color: blue
    - id: in_progress
      label: In Progress
      color: yellow
    - id: review
      label: In Review
      color: orange
    - id: testing
      label: Testing
      color: purple
    - id: done
      label: Done
      color: green
    - id: cancelled
      label: Cancelled
      color: red
      
  transitions:
    backlog: [refined, cancelled]
    refined: [in_progress, backlog]
    in_progress: [review, blocked, backlog]
    review: [testing, in_progress]
    testing: [done, in_progress]
    
priorities:
  levels:
    - id: critical
      label: Kritiek
      weight: 100
      sla_days: 1
    - id: high
      label: Hoog
      weight: 70
      sla_days: 3
    - id: medium
      label: Gemiddeld
      weight: 40
      sla_days: 7
    - id: low
      label: Laag
      weight: 10
      sla_days: 30

metadata:
  required_fields:
    story:
      - id
      - title
      - status
      - epic
    epic:
      - id
      - title
      - status
      - owner
      
  custom_fields:
    - name: compliance_level
      type: enum
      options: [none, basic, strict]
      default: basic
    - name: customer
      type: string
      required: false
    - name: deadline
      type: date
      required: false

templates:
  variables:
    author: ${GIT_AUTHOR_NAME}
    email: ${GIT_AUTHOR_EMAIL}
    organization: "Justitie Nederland"
    default_assignee: "team"
    
  selection:
    auto_detect: true
    prefer_local: true
    fallback: base

integrations:
  git:
    enabled: true
    auto_commit: false
    branch_pattern: "feature/{story-id}-{title-slug}"
    commit_template: "[{story-id}] {message}"
    
  claude:
    enabled: true
    config_file: .claude-backlog.yml
    agents:
      - business-analyst-justice
      - doc-standards-guardian
      
  ci_cd:
    github_actions: true
    gitlab_ci: false
    jenkins: false
    
  ide:
    vscode:
      workspace_settings: true
      recommended_extensions:
        - "yzhang.markdown-all-in-one"
        - "davidanson.vscode-markdownlint"
        
hooks:
  pre_init:
    - script: ./scripts/validate-environment.sh
      required: false
    - script: ./scripts/backup-existing.sh
      required: true
      
  post_init:
    - script: npm install
      condition: "file_exists('package.json')"
    - script: git add -A && git commit -m "Initialize backlog structure"
      condition: "git_initialized"
      
  pre_update:
    - script: ./scripts/check-uncommitted.sh
      
validation:
  rules:
    - rule: unique_ids
      severity: error
    - rule: valid_references
      severity: error
    - rule: required_fields
      severity: error
    - rule: workflow_transitions
      severity: warning
    - rule: naming_conventions
      severity: warning
      
  ignore:
    - "*.draft.md"
    - "archive/**"
    
plugins:
  enabled: true
  directory: ~/.backlog/plugins
  auto_load: true
  whitelist:
    - jira-sync
    - slack-notifier
    - ai-estimator
```

### Configuration Manager Implementation
```typescript
// src/core/config-manager.ts
import * as yaml from 'js-yaml';
import * as fs from 'fs-extra';
import * as path from 'path';
import { merge } from 'lodash';
import * as Joi from 'joi';

export class ConfigManager {
  private config: BacklogConfig;
  private schema: Joi.Schema;
  
  constructor() {
    this.schema = this.createValidationSchema();
  }
  
  async loadConfiguration(options: LoadOptions): Promise<BacklogConfig> {
    // 1. Load system defaults
    const defaults = await this.loadDefaults();
    
    // 2. Load user config (~/.backlog/config.yml)
    const userConfig = await this.loadUserConfig();
    
    // 3. Load project config (.backlog-init.yml)
    const projectConfig = await this.loadProjectConfig(options.projectDir);
    
    // 4. Apply environment variables
    const envConfig = this.loadEnvironmentVariables();
    
    // 5. Apply command-line flags
    const cliConfig = options.cliFlags || {};
    
    // 6. Handle inheritance (extends)
    let baseConfig = defaults;
    if (projectConfig?.extends) {
      const extended = await this.loadExtendedConfig(projectConfig.extends);
      baseConfig = merge(baseConfig, extended);
    }
    
    // 7. Merge in priority order
    this.config = merge(
      baseConfig,
      userConfig,
      projectConfig,
      envConfig,
      cliConfig
    );
    
    // 8. Validate final configuration
    await this.validateConfiguration();
    
    // 9. Resolve variables
    this.config = await this.resolveVariables(this.config);
    
    return this.config;
  }
  
  private createValidationSchema(): Joi.Schema {
    return Joi.object({
      version: Joi.string().required(),
      project: Joi.object({
        name: Joi.string().required(),
        type: Joi.string().valid('web-app', 'api', 'library', 'cli').required(),
        framework: Joi.string().optional(),
        language: Joi.string().default('en')
      }).required(),
      
      structure: Joi.object({
        base_path: Joi.string().default('docs/backlog'),
        directories: Joi.object().pattern(
          Joi.string(),
          Joi.string()
        ),
        naming: Joi.object().pattern(
          Joi.string(),
          Joi.string().regex(/\{[^}]+\}/)
        )
      }),
      
      workflow: Joi.object({
        states: Joi.array().items(
          Joi.object({
            id: Joi.string().required(),
            label: Joi.string().required(),
            color: Joi.string()
          })
        ).min(2),
        transitions: Joi.object().pattern(
          Joi.string(),
          Joi.array().items(Joi.string())
        )
      }),
      
      priorities: Joi.object({
        levels: Joi.array().items(
          Joi.object({
            id: Joi.string().required(),
            label: Joi.string().required(),
            weight: Joi.number().min(0).max(100),
            sla_days: Joi.number().min(0)
          })
        )
      })
    });
  }
  
  private async validateConfiguration(): Promise<void> {
    const { error, value } = this.schema.validate(this.config, {
      abortEarly: false,
      allowUnknown: true
    });
    
    if (error) {
      const errors = error.details.map(d => `  - ${d.message}`).join('\n');
      throw new Error(`Configuration validation failed:\n${errors}`);
    }
  }
  
  private loadEnvironmentVariables(): Partial<BacklogConfig> {
    const config: any = {};
    
    // Map environment variables to config paths
    const envMappings = {
      'BACKLOG_TYPE': 'project.type',
      'BACKLOG_FRAMEWORK': 'project.framework',
      'BACKLOG_LANGUAGE': 'project.language',
      'BACKLOG_BASE_PATH': 'structure.base_path',
      'BACKLOG_GIT_ENABLED': 'integrations.git.enabled',
      'BACKLOG_CLAUDE_ENABLED': 'integrations.claude.enabled'
    };
    
    for (const [envVar, configPath] of Object.entries(envMappings)) {
      if (process.env[envVar]) {
        this.setNestedProperty(config, configPath, process.env[envVar]);
      }
    }
    
    return config;
  }
  
  private async resolveVariables(config: any): Promise<any> {
    const variables = {
      PROJECT_NAME: path.basename(process.cwd()),
      GIT_AUTHOR_NAME: await this.getGitConfig('user.name'),
      GIT_AUTHOR_EMAIL: await this.getGitConfig('user.email'),
      TODAY: new Date().toISOString().split('T')[0],
      YEAR: new Date().getFullYear(),
      ...config.templates?.variables
    };
    
    // Recursively replace variables
    return this.replaceVariables(config, variables);
  }
  
  private replaceVariables(obj: any, variables: Record<string, any>): any {
    if (typeof obj === 'string') {
      return obj.replace(/\$\{([^}]+)\}/g, (match, key) => {
        return variables[key] || match;
      });
    }
    
    if (Array.isArray(obj)) {
      return obj.map(item => this.replaceVariables(item, variables));
    }
    
    if (typeof obj === 'object' && obj !== null) {
      const result: any = {};
      for (const [key, value] of Object.entries(obj)) {
        result[key] = this.replaceVariables(value, variables);
      }
      return result;
    }
    
    return obj;
  }
  
  async saveConfiguration(
    config: BacklogConfig,
    target: 'project' | 'user'
  ): Promise<void> {
    const configPath = target === 'project' 
      ? path.join(process.cwd(), '.backlog-init.yml')
      : path.join(os.homedir(), '.backlog', 'config.yml');
    
    await fs.ensureDir(path.dirname(configPath));
    await fs.writeFile(
      configPath,
      yaml.dump(config, { 
        lineWidth: 120,
        noRefs: true 
      })
    );
  }
  
  async exportConfiguration(outputPath: string): Promise<void> {
    const exportData = {
      config: this.config,
      version: this.config.version,
      exported_at: new Date().toISOString(),
      exported_by: await this.getGitConfig('user.email')
    };
    
    await fs.writeFile(
      outputPath,
      yaml.dump(exportData, { lineWidth: 120 })
    );
  }
  
  async importConfiguration(importPath: string): Promise<void> {
    const importData = yaml.load(
      await fs.readFile(importPath, 'utf8')
    ) as any;
    
    // Validate imported config
    await this.validateConfiguration(importData.config);
    
    // Check version compatibility
    if (!this.isVersionCompatible(importData.version)) {
      throw new Error(
        `Incompatible config version: ${importData.version}`
      );
    }
    
    this.config = importData.config;
  }
}
```

### Configuration UI
```typescript
// src/ui/config-wizard.ts
import * as inquirer from 'inquirer';
import chalk from 'chalk';

export class ConfigurationWizard {
  async runWizard(): Promise<BacklogConfig> {
    console.log(chalk.bold('\n⚙️  Configuration Wizard\n'));
    
    const answers = await inquirer.prompt([
      {
        type: 'input',
        name: 'project.name',
        message: 'Project name:',
        default: path.basename(process.cwd())
      },
      {
        type: 'list',
        name: 'project.type',
        message: 'Project type:',
        choices: [
          { name: 'Web Application', value: 'web-app' },
          { name: 'API/Backend', value: 'api' },
          { name: 'Library/Package', value: 'library' },
          { name: 'CLI Tool', value: 'cli' }
        ]
      },
      {
        type: 'input',
        name: 'structure.base_path',
        message: 'Backlog directory path:',
        default: 'docs/backlog'
      },
      {
        type: 'checkbox',
        name: 'integrations',
        message: 'Enable integrations:',
        choices: [
          { name: 'Git', value: 'git', checked: true },
          { name: 'Claude Agents', value: 'claude', checked: true },
          { name: 'GitHub Actions', value: 'github', checked: false },
          { name: 'VS Code', value: 'vscode', checked: true }
        ]
      },
      {
        type: 'confirm',
        name: 'advanced',
        message: 'Configure advanced options?',
        default: false
      }
    ]);
    
    if (answers.advanced) {
      const advanced = await this.configureAdvanced();
      Object.assign(answers, advanced);
    }
    
    return this.answersToConfig(answers);
  }
  
  private async configureAdvanced(): Promise<any> {
    return inquirer.prompt([
      {
        type: 'input',
        name: 'naming.stories',
        message: 'Story ID pattern:',
        default: 'US-{number:03d}',
        validate: (input) => {
          return input.includes('{number') || 'Must include {number} placeholder';
        }
      },
      {
        type: 'editor',
        name: 'workflow.states',
        message: 'Define workflow states (YAML format):'
      },
      {
        type: 'checkbox',
        name: 'hooks',
        message: 'Enable hooks:',
        choices: [
          'pre_init',
          'post_init',
          'pre_update',
          'post_update'
        ]
      }
    ]);
  }
}
```

## Test Scenarios

### Configuration Loading Tests
```javascript
describe('ConfigManager', () => {
  test('loads and merges configurations correctly', async () => {
    // Setup test configs
    mockFs({
      '.backlog-init.yml': yaml.dump({
        project: { name: 'test-project' }
      }),
      '~/.backlog/config.yml': yaml.dump({
        structure: { base_path: 'custom/path' }
      })
    });
    
    process.env.BACKLOG_LANGUAGE = 'nl';
    
    const config = await manager.loadConfiguration({
      projectDir: '.',
      cliFlags: { 'project.type': 'api' }
    });
    
    expect(config.project.name).toBe('test-project');
    expect(config.project.type).toBe('api'); // CLI override
    expect(config.project.language).toBe('nl'); // Env var
    expect(config.structure.base_path).toBe('custom/path'); // User config
  });
  
  test('validates configuration schema', async () => {
    const invalidConfig = {
      version: '1.0',
      // Missing required project field
    };
    
    await expect(manager.validateConfiguration(invalidConfig))
      .rejects.toThrow(/project.*required/);
  });
  
  test('resolves variables correctly', async () => {
    const config = {
      project: {
        name: '${PROJECT_NAME}',
        author: '${GIT_AUTHOR_NAME}'
      }
    };
    
    const resolved = await manager.resolveVariables(config);
    expect(resolved.project.name).toBe('my-project');
    expect(resolved.project.author).toBe('John Doe');
  });
});
```

## Tasks

### Development Tasks
- [ ] Design configuration schema - 3 uur
- [ ] Implement config manager - 4 uur
- [ ] Build configuration wizard UI - 3 uur
- [ ] Create validation system - 2 uur
- [ ] Add variable resolution - 2 uur
- [ ] Implement hooks system - 3 uur
- [ ] Build plugin architecture - 3 uur

### Testing Tasks
- [ ] Unit tests voor config loading - 2 uur
- [ ] Test configuration merging - 1 uur
- [ ] Test validation rules - 1 uur
- [ ] Integration tests - 2 uur

## Definition of Done
- [ ] Configuration system fully functional
- [ ] All config sources working
- [ ] Validation comprehensive
- [ ] Variables resolved correctly
- [ ] Hooks system operational
- [ ] Plugin system ready
- [ ] Documentation complete
- [ ] Migration tools available

## Notes
- Consider JSON Schema als alternative voor Joi
- TOML support voor frontmatter
- Configuration snippets/presets
- Live configuration reload tijdens development