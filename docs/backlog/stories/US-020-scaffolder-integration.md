---
id: US-020
titel: Integration with Existing Project Scaffolders
epic: EPIC-006
requirement: REQ-014
status: backlog
prioriteit: GEMIDDELD
story_points: 5
sprint: 
aangemaakt: 08-09-2025
bijgewerkt: 08-09-2025
assigned_to: backend_developer
dependencies: [US-016, US-017, US-018, US-019]
---

# US-020: Integration with Existing Project Scaffolders

## User Story
**Als** developer  
**wil ik** backlog bootstrapper integreren met bestaande project scaffolders  
**zodat** ik in één flow zowel mijn project als backlog structuur kan opzetten

## Context
Developers gebruiken vaak scaffolding tools zoals create-react-app, Vue CLI, Angular CLI, express-generator, etc. De backlog bootstrapper moet naadloos integreren met deze tools zodat het een natuurlijke extensie wordt van het bestaande development workflow.

## Acceptance Criteria

### Scaffolder Detection
- [ ] Detecteert populaire scaffolders (CRA, Vue CLI, Angular CLI, etc.)
- [ ] Herkent custom scaffolders via configuratie
- [ ] Identificeert project type uit scaffolder output
- [ ] Leest scaffolder configuratie voor context
- [ ] Respecteert scaffolder directory structuur

### Integration Methods
- [ ] Post-scaffolding hook automatisch uitvoeren
- [ ] NPM postinstall script integratie
- [ ] Scaffolder plugin/extension systeem
- [ ] Wrapper commands (backlog-create-react-app)
- [ ] IDE integration voor combined workflow

### Compatibility
- [ ] Werkt met alle major JavaScript frameworks
- [ ] Python project scaffolders (cookiecutter, etc.)
- [ ] Rust/Cargo project templates
- [ ] Go module initialization
- [ ] .NET project templates

### Conflict Resolution
- [ ] Merges configuratie files intelligent
- [ ] Respecteert existing .gitignore entries
- [ ] Behoudt scaffolder-generated structure
- [ ] Adds to package.json zonder overwrite
- [ ] Preserves existing git history

## Technical Design

### Scaffolder Adapters
```typescript
// src/adapters/scaffolder-adapter.ts
export interface ScaffolderAdapter {
  name: string;
  detect(): Promise<boolean>;
  getProjectInfo(): Promise<ProjectInfo>;
  integrate(backlogConfig: BacklogConfig): Promise<void>;
  postProcess(): Promise<void>;
}

// src/adapters/create-react-app.ts
export class CreateReactAppAdapter implements ScaffolderAdapter {
  name = 'create-react-app';
  
  async detect(): Promise<boolean> {
    // Check for CRA signatures
    const packageJson = await this.readPackageJson();
    return !!(
      packageJson?.dependencies?.react &&
      packageJson?.scripts?.start?.includes('react-scripts')
    );
  }
  
  async getProjectInfo(): Promise<ProjectInfo> {
    const packageJson = await this.readPackageJson();
    
    return {
      name: packageJson.name,
      type: 'web-app',
      framework: 'react',
      version: packageJson.dependencies.react,
      features: this.detectFeatures(packageJson),
      structure: {
        source: 'src',
        public: 'public',
        tests: 'src/__tests__'
      }
    };
  }
  
  async integrate(backlogConfig: BacklogConfig): Promise<void> {
    // 1. Merge package.json scripts
    await this.addPackageScripts({
      'backlog:validate': 'backlog validate',
      'backlog:update': 'backlog update',
      'precommit': 'npm run backlog:validate'
    });
    
    // 2. Add to .gitignore
    await this.updateGitignore([
      '# Backlog',
      '.backlog-cache/',
      '*.backlog.backup'
    ]);
    
    // 3. Create CRA-specific templates
    await this.createTemplates({
      'stories/frontend/component-story.md',
      'stories/frontend/page-story.md',
      'stories/testing/unit-test-story.md',
      'epics/ui-components.md'
    });
    
    // 4. Add VS Code settings
    await this.updateVSCodeSettings({
      'files.associations': {
        '**/docs/backlog/**/*.md': 'markdown'
      },
      'markdown.extension.toc.levels': '2..6'
    });
  }
  
  async postProcess(): Promise<void> {
    // Generate initial stories based on src structure
    const components = await this.scanComponents();
    for (const component of components) {
      await this.generateComponentStory(component);
    }
    
    // Create README section
    await this.addReadmeSection(`
## 📋 Backlog Management

This project uses structured backlog management.

- View backlog: \`npm run backlog:dashboard\`
- Validate structure: \`npm run backlog:validate\`
- Update templates: \`npm run backlog:update\`

See [docs/backlog/INDEX.md](docs/backlog/INDEX.md) for details.
    `);
  }
  
  private async scanComponents(): Promise<Component[]> {
    const srcDir = path.join(process.cwd(), 'src');
    const files = await glob('**/*.{jsx,tsx}', { cwd: srcDir });
    
    return files
      .filter(f => /^[A-Z]/.test(path.basename(f)))
      .map(f => ({
        name: path.basename(f, path.extname(f)),
        path: f,
        type: this.detectComponentType(f)
      }));
  }
}
```

### Wrapper Commands
```typescript
// src/wrappers/backlog-create-react-app.ts
#!/usr/bin/env node

import { spawn } from 'child_process';
import { CreateReactAppAdapter } from '../adapters/create-react-app';
import { BacklogBootstrapper } from '../core/bootstrapper';

async function main() {
  const args = process.argv.slice(2);
  const projectName = args[0];
  
  if (!projectName) {
    console.error('Please specify project name');
    process.exit(1);
  }
  
  console.log('🚀 Creating React app with backlog management...\n');
  
  // 1. Run create-react-app
  console.log('Step 1/3: Creating React app...');
  await runCommand('npx', ['create-react-app', ...args]);
  
  // 2. Change to project directory
  process.chdir(projectName);
  
  // 3. Initialize backlog
  console.log('\nStep 2/3: Initializing backlog structure...');
  const bootstrapper = new BacklogBootstrapper();
  await bootstrapper.init({
    type: 'web-app',
    framework: 'react',
    adapter: new CreateReactAppAdapter(),
    silent: false
  });
  
  // 4. Run post-processing
  console.log('\nStep 3/3: Finalizing setup...');
  await bootstrapper.postProcess();
  
  console.log(`
✅ Success! Created ${projectName} with backlog management.

Next steps:
  cd ${projectName}
  npm start           # Start development server
  npm run backlog     # View backlog dashboard
  
Happy coding! 🎉
  `);
}

function runCommand(command: string, args: string[]): Promise<void> {
  return new Promise((resolve, reject) => {
    const child = spawn(command, args, {
      stdio: 'inherit',
      shell: true
    });
    
    child.on('close', (code) => {
      if (code !== 0) {
        reject(new Error(`Command failed with code ${code}`));
      } else {
        resolve();
      }
    });
  });
}

main().catch(console.error);
```

### Plugin System
```typescript
// src/plugins/scaffolder-plugin.ts
export abstract class ScaffolderPlugin {
  abstract readonly scaffolderId: string;
  abstract readonly version: string;
  
  abstract async preScaffold(options: any): Promise<void>;
  abstract async postScaffold(projectInfo: ProjectInfo): Promise<void>;
  abstract async integrate(backlogConfig: BacklogConfig): Promise<void>;
}

// src/plugins/vue-cli-plugin.ts
export class VueCliPlugin extends ScaffolderPlugin {
  scaffolderId = 'vue-cli';
  version = '5.0.0';
  
  async preScaffold(options: any): Promise<void> {
    // Add backlog preset to Vue CLI
    options.preset = {
      ...options.preset,
      plugins: {
        ...options.preset?.plugins,
        '@backlog/vue-cli-plugin-backlog': {}
      }
    };
  }
  
  async postScaffold(projectInfo: ProjectInfo): Promise<void> {
    // Generate Vue-specific stories
    await this.generateStories([
      'Setup Vuex store',
      'Configure Vue Router',
      'Implement component library',
      'Add i18n support'
    ]);
    
    // Add Vue-specific acceptance criteria templates
    await this.addAcceptanceCriteria({
      component: [
        'Props validated with PropTypes',
        'Emits documented',
        'Slots documented',
        'Scoped styles applied'
      ],
      view: [
        'Route guards implemented',
        'Meta tags configured',
        'Loading states handled',
        'Error boundaries in place'
      ]
    });
  }
  
  async integrate(backlogConfig: BacklogConfig): Promise<void> {
    // Add to vue.config.js
    await this.updateVueConfig({
      chainWebpack: (config: any) => {
        config.plugin('backlog').use(BacklogWebpackPlugin);
      }
    });
  }
}
```

### Package Manager Integration
```typescript
// src/integrations/package-managers.ts
export class PackageManagerIntegration {
  async detectPackageManager(): Promise<'npm' | 'yarn' | 'pnpm'> {
    if (await fs.pathExists('yarn.lock')) return 'yarn';
    if (await fs.pathExists('pnpm-lock.yaml')) return 'pnpm';
    return 'npm';
  }
  
  async addScripts(scripts: Record<string, string>): Promise<void> {
    const packageJson = await this.readPackageJson();
    
    packageJson.scripts = {
      ...packageJson.scripts,
      ...scripts
    };
    
    await this.writePackageJson(packageJson);
  }
  
  async addDevDependency(name: string, version: string): Promise<void> {
    const pm = await this.detectPackageManager();
    
    const commands = {
      npm: ['install', '--save-dev', `${name}@${version}`],
      yarn: ['add', '--dev', `${name}@${version}`],
      pnpm: ['add', '--save-dev', `${name}@${version}`]
    };
    
    await this.runCommand(pm, commands[pm]);
  }
  
  async setupPostInstall(): Promise<void> {
    await this.addScripts({
      'postinstall': 'backlog init --check'
    });
  }
}
```

### IDE Integration
```typescript
// src/integrations/ide-integration.ts
export class IDEIntegration {
  async setupVSCode(): Promise<void> {
    const settings = {
      'backlog.enabled': true,
      'backlog.autoValidate': true,
      'backlog.showStatusBar': true,
      'files.associations': {
        '**/backlog/**/*.md': 'backlog-markdown'
      },
      'recommendations': [
        'backlog.vscode-backlog-manager'
      ]
    };
    
    await this.updateWorkspaceSettings(settings);
    await this.createTasksJson();
    await this.createLaunchJson();
  }
  
  private async createTasksJson(): Promise<void> {
    const tasks = {
      version: '2.0.0',
      tasks: [
        {
          label: 'Backlog: Validate',
          type: 'shell',
          command: 'npm run backlog:validate',
          group: 'test',
          presentation: {
            reveal: 'silent'
          }
        },
        {
          label: 'Backlog: Dashboard',
          type: 'shell',
          command: 'npm run backlog:dashboard',
          group: 'none',
          presentation: {
            reveal: 'always',
            panel: 'new'
          }
        }
      ]
    };
    
    await this.writeJson('.vscode/tasks.json', tasks);
  }
}
```

## Test Scenarios

### Scaffolder Detection Tests
```javascript
describe('Scaffolder Detection', () => {
  test('detects Create React App', async () => {
    mockFs({
      'package.json': JSON.stringify({
        dependencies: { react: '^18.0.0' },
        scripts: { start: 'react-scripts start' }
      })
    });
    
    const adapter = new CreateReactAppAdapter();
    expect(await adapter.detect()).toBe(true);
  });
  
  test('detects Vue CLI project', async () => {
    mockFs({
      'package.json': JSON.stringify({
        dependencies: { vue: '^3.0.0' },
        devDependencies: { '@vue/cli-service': '^5.0.0' }
      })
    });
    
    const adapter = new VueCliAdapter();
    expect(await adapter.detect()).toBe(true);
  });
});
```

### Integration Tests
```javascript
describe('Scaffolder Integration', () => {
  test('integrates with existing CRA project', async () => {
    // Setup mock CRA project
    await setupMockCRAProject();
    
    const bootstrapper = new BacklogBootstrapper();
    await bootstrapper.init({
      adapter: new CreateReactAppAdapter()
    });
    
    // Verify integration
    expect(fs.existsSync('docs/backlog')).toBe(true);
    
    const packageJson = JSON.parse(
      fs.readFileSync('package.json', 'utf8')
    );
    expect(packageJson.scripts['backlog:validate']).toBeDefined();
    
    const gitignore = fs.readFileSync('.gitignore', 'utf8');
    expect(gitignore).toContain('.backlog-cache/');
  });
  
  test('wrapper command creates project with backlog', async () => {
    const result = await exec('backlog-create-react-app test-app');
    
    expect(result.exitCode).toBe(0);
    expect(fs.existsSync('test-app/src')).toBe(true);
    expect(fs.existsSync('test-app/docs/backlog')).toBe(true);
  });
});
```

## Tasks

### Development Tasks
- [ ] Create scaffolder adapter interface - 2 uur
- [ ] Implement CRA adapter - 3 uur
- [ ] Implement Vue CLI adapter - 3 uur
- [ ] Implement Angular CLI adapter - 3 uur
- [ ] Create wrapper commands - 2 uur
- [ ] Build plugin system - 3 uur
- [ ] Package manager integration - 2 uur

### Testing Tasks
- [ ] Unit tests voor adapters - 2 uur
- [ ] Integration tests per scaffolder - 3 uur
- [ ] E2E tests voor wrappers - 2 uur
- [ ] Cross-platform testing - 1 uur

## Definition of Done
- [ ] All major scaffolders supported
- [ ] Detection logic accurate
- [ ] Integration non-destructive
- [ ] Wrapper commands functional
- [ ] Plugin system operational
- [ ] IDE integration working
- [ ] Documentation complete
- [ ] Published to registries

## Notes
- Consider scaffolder marketplace integration
- Auto-detect from npx commands
- Support for monorepo scaffolders
- Integration with Nx, Lerna, Rush
- Consider GitHub template repositories