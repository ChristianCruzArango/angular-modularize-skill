# Angular Modularize

An agent skill that modularizes existing Angular components and projects following official best practices from [angular.dev](https://angular.dev), SOLID principles, and the Smart/Presentational pattern.

## What It Does

This skill teaches AI agents to properly analyze and modularize Angular applications by:

- **Analyzing** existing components to identify SRP violations and modularization candidates
- **Deciding** when to split and when NOT to — not everything should be modularized
- **Decomposing** monolithic components into Smart (container) + Presentational (dumb) components
- **Extracting** services, child components, and shared modules following SOLID principles
- **Organizing** projects with the Core/Shared/Features pattern
- **Setting up** lazy loading for optimal performance
- **Migrating** NgModules to Standalone when appropriate

## Supported Angular Versions

| Version | Strategy |
|---------|----------|
| v17+ | Standalone components + route-based lazy loading + `@defer` |
| v14-v16 | NgModules with standalone migration path |
| < v14 | Classic NgModule pattern |

## Install

```bash
npx skills add ChristianCruzArango/angular-modularize-skill
```

## Usage

After installation, the skill is automatically invoked when you ask to:

- "Modularize this component"
- "Split this large component"
- "Extract child components"
- "Organize the project structure"
- "Apply Angular best practices"
- "Refactor into smart and presentational components"

Or invoke directly: `/angular-modularize`

## Key Concepts

### When to Modularize
- Component class > 200 lines
- Template > 100 lines
- More than 5 injected dependencies
- Multiple unrelated responsibilities in one component
- Repeated HTML blocks across components

### When NOT to Modularize
- Template < 50 lines with single responsibility
- Component used only once with no children
- Splitting would create prop drilling (> 5 inputs)
- App has < 10 components total

### Smart vs Presentational Pattern
- **Smart (Container)**: injects services, fetches data, manages state
- **Presentational (Dumb)**: receives data via `input()`, emits events via `output()`, renders UI only

## References

- [Angular Style Guide](https://angular.dev/style-guide)
- [Angular Standalone Components](https://angular.dev/guide/components)
- [Smart vs Presentational Components](https://blog.angular-university.io/angular-2-smart-components-vs-presentation-components-whats-the-difference-when-to-use-each-and-why/)
- [Container & Presentational Components](https://www.telerik.com/blogs/clean-code-using-container-presentational-components-angular)
- [Refactoring into Smart and Dumb Components](https://modernangular.com/articles/refactoring-into-smart-and-dumb-components)
- [Angular Component Best Practices](https://angular.love/best-practices-of-working-with-angular-components/)

## License

MIT
