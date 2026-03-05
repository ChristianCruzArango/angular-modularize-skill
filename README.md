# Angular Modularize Skill

An agent skill for Claude Code that modularizes and restructures Angular projects following official best practices from [angular.dev](https://angular.dev).

## What it does

This skill teaches AI agents how to properly modularize Angular applications by:

- Analyzing existing project structure and Angular version
- Proposing the right modularization strategy (standalone vs NgModules)
- Applying the Core/Shared/Features pattern
- Setting up lazy loading for feature modules
- Following official Angular style guide conventions
- Enforcing clean architecture and SOLID principles
- Including unit tests for every change

## Supported Angular Versions

| Version | Strategy |
|---------|----------|
| v17+ | Standalone components + route-based lazy loading |
| v14-v16 | NgModules with standalone migration path |
| < v14 | Classic NgModule pattern |

## Install

```bash
npx skills add hallc-dev/angular-modularize-skill
```

## Usage

After installation, the skill is automatically invoked when you ask Claude Code to:

- "Modularize this Angular project"
- "Organize the project structure"
- "Split into feature modules"
- "Apply Angular best practices to the structure"
- "Create a feature module for users"
- "Restructure the project"

You can also invoke it directly:

```
/angular-modularize
```

## Key Patterns Applied

### Project Structure

```
src/app/
├── core/          # Singleton services, guards, interceptors
├── shared/        # Reusable components, pipes, directives
└── features/      # Lazy-loaded feature modules
    ├── dashboard/
    ├── users/
    └── settings/
```

### Smart vs Presentational Components

- **Smart (container)**: manage state, call services
- **Presentational (dumb)**: receive inputs, emit outputs

### Lazy Loading

Route-based lazy loading for each feature to optimize bundle size.

## References

- [Angular Style Guide](https://angular.dev/style-guide)
- [Angular Standalone Components](https://angular.dev/guide/components)
- [Angular Lazy Loading](https://angular.dev/guide/ngmodules/lazy-loading-ngmodules)

## License

MIT
