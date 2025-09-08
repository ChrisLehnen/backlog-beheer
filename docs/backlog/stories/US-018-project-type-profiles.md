---
id: US-018
titel: Project Type Profiles Implementation
epic: EPIC-006
requirement: REQ-014
status: backlog
prioriteit: HOOG
story_points: 3
sprint: 
aangemaakt: 08-09-2025
bijgewerkt: 08-09-2025
assigned_to: backend_developer
dependencies: [US-016, US-017]
---

# US-018: Project Type Profiles Implementation

## User Story
**Als** developer  
**wil ik** voorgedefinieerde project type profiles kiezen  
**zodat** ik automatisch de juiste backlog structuur krijg voor mijn specifieke project type

## Context
Verschillende project types (web app, API, library, CLI tool) hebben verschillende backlog behoeften. Profiles bundelen de juiste templates, configuraties, en workflows voor elk type project.

## Acceptance Criteria

### Profile Types
- [ ] Web Application profile met frontend-specifieke stories
- [ ] API/Backend profile met service-oriented templates
- [ ] Library profile voor package development
- [ ] CLI Tool profile voor command-line applicaties
- [ ] Microservice profile voor distributed systems
- [ ] Full-Stack profile combineert frontend + backend

### Profile Configuration
- [ ] Elk profile heeft predefined story templates
- [ ] Acceptance criteria aangepast per project type
- [ ] Relevante epic templates per profile
- [ ] Domain-specific requirements included
- [ ] Workflow states aangepast aan project type

### Profile Selection
- [ ] Interactive selector toont beschrijving per profile
- [ ] Preview van structure voor geselecteerd profile
- [ ] Combinatie van profiles mogelijk (multi-profile)
- [ ] Custom profile creation ondersteund
- [ ] Profile recommendations based on detected files

### Profile Customization
- [ ] Override default settings per profile
- [ ] Add/remove components uit profile
- [ ] Save custom profiles voor hergebruik
- [ ] Share profiles via export/import
- [ ] Profile inheritance voor variations

## Technical Design

### Profile Structure
```yaml
# profiles/web-app.yml
id: web-app
name: Web Application
description: Full-featured web application with UI components
version: 1.0.0
category: frontend

includes:
  templates:
    - base
    - types/web-app
    - features/authentication
    - features/dashboard
    - testing/frontend
  
  epics:
    - user-interface
    - user-experience
    - accessibility
    - performance
    - security
  
  stories:
    - responsive-design
    - cross-browser-support
    - state-management
    - routing
    - api-integration

configuration:
  workflow_states:
    - backlog
    - design
    - in_progress
    - review
    - testing
    - deployed
  
  priorities:
    - P0: Security & Auth
    - P1: Core Features
    - P2: UX Polish
    - P3: Nice to Have
  
  acceptance_criteria_templates:
    ui_component:
      - Responsive on mobile/tablet/desktop
      - Accessible (WCAG 2.1 AA)
      - Cross-browser compatible
      - Loading states implemented
      - Error states handled
    
    api_integration:
      - Error handling for network failures
      - Loading indicators during requests
      - Retry logic for failures
      - Caching where appropriate

framework_support:
  react:
    additional_templates:
      - frameworks/react
    dev_dependencies:
      - "@testing-library/react"
      - "react-router-dom"
    scripts:
      start: "react-scripts start"
      build: "react-scripts build"
      test: "react-scripts test"
  
  vue:
    additional_templates:
      - frameworks/vue
    dev_dependencies:
      - "@vue/test-utils"
      - "vue-router"
  
  angular:
    additional_templates:
      - frameworks/angular
    dev_dependencies:
      - "@angular/router"
      - "karma"

integrations:
  ci_cd:
    - github-actions/web-app-pipeline.yml
    - netlify.toml
    - vercel.json
  
  monitoring:
    - sentry-config.js
    - analytics-setup.js
  
  testing:
    - jest.config.js
    - cypress.json
    - .storybook/

required_tools:
  - node: ">=16.0.0"
  - npm: ">=7.0.0"

optional_tools:
  - docker: "for containerization"
  - terraform: "for infrastructure"
```

### Profile Manager Implementation
```typescript
// src/core/profile-manager.ts
import * as yaml from 'js-yaml';
import * as fs from 'fs-extra';
import { Profile, ProfileConfig } from '../types';

export class ProfileManager {
  private profiles: Map<string, Profile> = new Map();
  private customProfiles: Map<string, Profile> = new Map();
  
  constructor(
    private profilesDir: string,
    private customProfilesDir: string
  ) {}
  
  async loadProfiles(): Promise<void> {
    // Load built-in profiles
    const builtInProfiles = await fs.readdir(this.profilesDir);
    for (const file of builtInProfiles) {
      if (file.endsWith('.yml')) {
        const profile = await this.loadProfile(
          path.join(this.profilesDir, file)
        );
        this.profiles.set(profile.id, profile);
      }
    }
    
    // Load custom profiles
    if (await fs.pathExists(this.customProfilesDir)) {
      const customFiles = await fs.readdir(this.customProfilesDir);
      for (const file of customFiles) {
        if (file.endsWith('.yml')) {
          const profile = await this.loadProfile(
            path.join(this.customProfilesDir, file)
          );
          this.customProfiles.set(profile.id, profile);
        }
      }
    }
  }
  
  async getProfile(id: string): Promise<Profile> {
    // Check custom profiles first (override built-in)
    if (this.customProfiles.has(id)) {
      return this.customProfiles.get(id)!;
    }
    
    if (this.profiles.has(id)) {
      return this.profiles.get(id)!;
    }
    
    throw new Error(`Profile ${id} not found`);
  }
  
  async detectProfile(projectDir: string): Promise<string> {
    // Detect based on existing files
    const files = await fs.readdir(projectDir);
    
    // Check for framework indicators
    if (files.includes('package.json')) {
      const pkg = await fs.readJson(
        path.join(projectDir, 'package.json')
      );
      
      if (pkg.dependencies?.react) return 'web-app';
      if (pkg.dependencies?.express) return 'api';
      if (pkg.bin) return 'cli-tool';
      if (pkg.main && !pkg.bin) return 'library';
    }
    
    if (files.includes('Cargo.toml')) return 'rust-app';
    if (files.includes('requirements.txt')) return 'python-app';
    if (files.includes('go.mod')) return 'go-app';
    
    // Default fallback
    return 'web-app';
  }
  
  async applyProfile(
    profileId: string,
    projectDir: string,
    options: ProfileConfig
  ): Promise<void> {
    const profile = await this.getProfile(profileId);
    
    // Apply templates
    for (const templateId of profile.includes.templates) {
      await this.templateEngine.render(
        templateId,
        projectDir,
        options.variables
      );
    }
    
    // Create epic templates
    for (const epicTemplate of profile.includes.epics) {
      await this.createEpicFromTemplate(
        epicTemplate,
        projectDir
      );
    }
    
    // Create story templates
    for (const storyTemplate of profile.includes.stories) {
      await this.createStoryFromTemplate(
        storyTemplate,
        projectDir,
        profile.configuration.acceptance_criteria_templates
      );
    }
    
    // Apply framework-specific additions
    if (options.framework && profile.framework_support[options.framework]) {
      await this.applyFrameworkAdditions(
        profile.framework_support[options.framework],
        projectDir
      );
    }
    
    // Setup integrations
    await this.setupIntegrations(
      profile.integrations,
      projectDir,
      options
    );
  }
  
  async createCustomProfile(
    name: string,
    baseProfile: string,
    modifications: Partial<Profile>
  ): Promise<void> {
    const base = await this.getProfile(baseProfile);
    
    const customProfile = {
      ...base,
      id: name,
      name: `Custom: ${name}`,
      ...modifications,
      custom: true,
      base_profile: baseProfile
    };
    
    const filePath = path.join(
      this.customProfilesDir,
      `${name}.yml`
    );
    
    await fs.ensureDir(this.customProfilesDir);
    await fs.writeFile(
      filePath,
      yaml.dump(customProfile)
    );
    
    this.customProfiles.set(name, customProfile);
  }
}
```

### Profile Selector UI
```typescript
// src/ui/profile-selector.ts
import * as inquirer from 'inquirer';
import chalk from 'chalk';

export class ProfileSelector {
  async selectProfile(profiles: Profile[]): Promise<string> {
    console.log(chalk.bold('\n📦 Select Project Type:\n'));
    
    const choices = profiles.map(p => ({
      name: `${p.name} - ${chalk.gray(p.description)}`,
      value: p.id,
      short: p.name
    }));
    
    choices.push({
      name: chalk.cyan('Custom Profile'),
      value: 'custom',
      short: 'Custom'
    });
    
    const { profile } = await inquirer.prompt([
      {
        type: 'list',
        name: 'profile',
        message: 'Choose a project profile:',
        choices,
        pageSize: 10
      }
    ]);
    
    if (profile === 'custom') {
      return await this.createCustomProfile();
    }
    
    return profile;
  }
  
  async previewProfile(profile: Profile): Promise<boolean> {
    console.log(chalk.bold(`\n📋 Profile: ${profile.name}\n`));
    console.log('This will create:');
    
    console.log(chalk.green('  ✓'), 'Epics:');
    profile.includes.epics.forEach(e => 
      console.log('    -', e)
    );
    
    console.log(chalk.green('  ✓'), 'Story Templates:');
    profile.includes.stories.forEach(s => 
      console.log('    -', s)
    );
    
    console.log(chalk.green('  ✓'), 'Workflow States:');
    profile.configuration.workflow_states.forEach(w => 
      console.log('    -', w)
    );
    
    const { confirm } = await inquirer.prompt([
      {
        type: 'confirm',
        name: 'confirm',
        message: 'Continue with this profile?',
        default: true
      }
    ]);
    
    return confirm;
  }
}
```

## Test Scenarios

### Profile Loading Tests
```javascript
describe('ProfileManager', () => {
  test('loads all built-in profiles', async () => {
    await manager.loadProfiles();
    const webProfile = await manager.getProfile('web-app');
    expect(webProfile.id).toBe('web-app');
    expect(webProfile.includes.templates).toContain('types/web-app');
  });
  
  test('detects profile from package.json', async () => {
    mockFs({
      'package.json': JSON.stringify({
        dependencies: { react: '^18.0.0' }
      })
    });
    
    const detected = await manager.detectProfile('.');
    expect(detected).toBe('web-app');
  });
  
  test('custom profile overrides built-in', async () => {
    await manager.createCustomProfile('web-app', 'web-app', {
      name: 'Modified Web App'
    });
    
    const profile = await manager.getProfile('web-app');
    expect(profile.name).toBe('Modified Web App');
    expect(profile.custom).toBe(true);
  });
});
```

### Profile Application Tests
```javascript
describe('Profile Application', () => {
  test('applies web-app profile correctly', async () => {
    await manager.applyProfile('web-app', './test-project', {
      framework: 'react',
      variables: { project_name: 'test' }
    });
    
    expect(fs.existsSync('./test-project/docs/backlog')).toBe(true);
    expect(fs.existsSync('./test-project/package.json')).toBe(true);
    
    const pkg = JSON.parse(
      fs.readFileSync('./test-project/package.json', 'utf8')
    );
    expect(pkg.dependencies.react).toBeDefined();
  });
});
```

## Tasks

### Development Tasks
- [ ] Create profile schema definition - 2 uur
- [ ] Implement profile manager - 3 uur
- [ ] Build profile selector UI - 2 uur
- [ ] Create built-in profiles - 3 uur
- [ ] Add profile detection logic - 2 uur
- [ ] Implement custom profiles - 2 uur

### Testing Tasks
- [ ] Unit tests voor profile manager - 2 uur
- [ ] Test profile application - 1 uur
- [ ] Test profile detection - 1 uur
- [ ] Integration tests - 1 uur

## Definition of Done
- [ ] All profile types implemented
- [ ] Profile selection UI working
- [ ] Profile detection accurate
- [ ] Custom profiles supported
- [ ] Framework integrations tested
- [ ] Documentation complete
- [ ] Examples for each profile

## Notes
- Consider AI-powered profile recommendations
- Profile marketplace in future versions
- Support for mono-repo profiles
- Industry-specific profiles (finance, healthcare, etc.)