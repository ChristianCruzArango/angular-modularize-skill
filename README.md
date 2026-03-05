# Angular Modularize

An agent skill that modularizes existing Angular components and projects following official [angular.dev](https://angular.dev) best practices, SOLID principles, and the Smart/Presentational pattern.

## What It Does

Teaches AI agents to properly analyze and modularize Angular applications:

- **Analyze** existing components to identify SRP violations and modularization candidates
- **Decide** when to split and when NOT to — not everything should be modularized
- **Decompose** monolithic components into Smart (container) + Presentational (dumb) components
- **Extract** services, child components, and shared modules following SOLID principles
- **Organize** projects with the Core/Shared/Features pattern
- **Lazy load** with route-based loading and `@defer` blocks
- **Modernize** — replace deprecated patterns with `input()`, `output()`, `signal()`, `computed()`, `linkedSignal()`, `httpResource()`, `inject()`, `@if`/`@for` control flow

## Supported Angular Versions

| Version | Strategy |
|---------|----------|
| v19+ | Standalone by default + signals + `httpResource()` + `@defer` + `linkedSignal()` |
| v17-v18 | Standalone components + route-based lazy loading + `@defer` |
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
- Component class > 200 lines or template > 100 lines
- More than 5 injected dependencies
- Multiple unrelated responsibilities (violates SRP)
- Repeated HTML blocks across components

### When NOT to Modularize
- Template < 50 lines with single responsibility
- Simple wrapper with < 3 inputs
- Splitting would create prop drilling (> 5 inputs)
- App has < 10 components total

### Smart vs Presentational Pattern
- **Smart (Container)**: uses `inject()`, fetches data with `httpResource()`, manages state with `signal()`
- **Presentational (Dumb)**: receives data via `input()`, emits events via `output()`, renders UI only

### Modern Angular APIs Used
- `input()` / `input.required()` / `model()` instead of `@Input()`
- `output()` instead of `@Output()`
- `signal()` / `computed()` / `linkedSignal()` for reactive state
- `httpResource()` for GET requests, `HttpClient` for mutations
- `inject()` instead of constructor injection
- `@if` / `@for` / `@switch` instead of `*ngIf` / `*ngFor`
- `@defer` for lazy loading heavy components
- `DestroyRef.onDestroy()` instead of `ngOnDestroy`

## References

- [Angular Style Guide](https://angular.dev/style-guide)
- [Angular Components Guide](https://angular.dev/guide/components)
- [Angular Signals Guide](https://angular.dev/guide/signals)
- [Angular DI Guide](https://angular.dev/guide/di)
- [Angular Inputs API](https://angular.dev/guide/components/inputs)
- [Angular Outputs API](https://angular.dev/guide/components/outputs)
- [Angular Control Flow](https://angular.dev/guide/templates/control-flow)
- [Angular @defer](https://angular.dev/guide/templates/defer)
- [Angular resource()](https://angular.dev/guide/signals/resource)
- [Angular linkedSignal()](https://angular.dev/guide/signals/linked-signal)
- [Smart vs Presentational Components](https://blog.angular-university.io/angular-2-smart-components-vs-presentation-components-whats-the-difference-when-to-use-each-and-why/)

## License

MIT
